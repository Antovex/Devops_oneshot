# Advanced Ansible Patterns

A comprehensive guide to advanced Ansible concepts, enterprise patterns, custom modules, and testing frameworks.

## What is Advanced Ansible?
- Enterprise-scale automation with Ansible Tower/AWX
- Custom module development and plugin architecture
- Advanced inventory management and dynamic inventories
- Testing and validation with Molecule framework
- Role development and Galaxy distribution

## Core Advanced Concepts

### Role Development Best Practices
```yaml
# roles/webapp/meta/main.yml
---
galaxy_info:
  author: DevOps Team
  description: Production-ready web application deployment
  company: Example Corp
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
  galaxy_tags:
    - web
    - nginx
    - deployment

dependencies:
  - role: common
    vars:
      security_hardening: true
  - role: nginx
    vars:
      nginx_worker_processes: "{{ ansible_processor_cores }}"
```

### Advanced Role Structure
```
roles/webapp/
├── defaults/
│   └── main.yml              # Default variables (lowest precedence)
├── vars/
│   └── main.yml              # Role variables (higher precedence)
├── tasks/
│   ├── main.yml              # Main task entry point
│   ├── install.yml           # Installation tasks
│   ├── configure.yml         # Configuration tasks
│   └── security.yml          # Security hardening
├── handlers/
│   └── main.yml              # Event handlers
├── templates/
│   ├── nginx.conf.j2         # Jinja2 templates
│   └── systemd.service.j2
├── files/
│   ├── ssl-certs/            # Static files
│   └── scripts/
├── tests/
│   ├── inventory             # Test inventory
│   └── test.yml              # Role testing
├── molecule/
│   └── default/              # Molecule test scenarios
└── meta/
    └── main.yml              # Role metadata and dependencies
```

### Custom Module Development
```python
#!/usr/bin/python
# library/custom_service_manager.py

from ansible.module_utils.basic import AnsibleModule
import subprocess
import json

ANSIBLE_METADATA = {
    'metadata_version': '1.1',
    'status': ['preview'],
    'supported_by': 'community'
}

DOCUMENTATION = '''
---
module: custom_service_manager
short_description: Manage custom application services
description:
    - This module manages custom application services with health checks
    - Supports start, stop, restart, and status operations
    - Includes built-in health checking and monitoring
author:
    - DevOps Team (@example-corp)
options:
    name:
        description:
            - Name of the service to manage
        required: true
        type: str
    state:
        description:
            - Desired state of the service
        required: false
        default: started
        choices: ['started', 'stopped', 'restarted']
        type: str
    health_check_url:
        description:
            - URL to perform health check after service operation
        required: false
        type: str
    timeout:
        description:
            - Timeout for health check in seconds
        required: false
        default: 30
        type: int
'''

EXAMPLES = '''
# Start a service with health check
- name: Start webapp service
  custom_service_manager:
    name: webapp
    state: started
    health_check_url: http://localhost:8080/health
    timeout: 60

# Stop a service
- name: Stop webapp service
  custom_service_manager:
    name: webapp
    state: stopped
'''

RETURN = '''
changed:
    description: Whether the service state was changed
    type: bool
    returned: always
service_status:
    description: Current status of the service
    type: str
    returned: always
health_check_result:
    description: Result of health check if performed
    type: dict
    returned: when health check is performed
'''

def run_command(module, cmd):
    """Execute command and return result"""
    try:
        result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
        return result.returncode, result.stdout, result.stderr
    except Exception as e:
        module.fail_json(msg=f"Command execution failed: {str(e)}")

def check_service_status(module, service_name):
    """Check if service is running"""
    cmd = f"systemctl is-active {service_name}"
    rc, stdout, stderr = run_command(module, cmd)
    return stdout.strip() == 'active'

def manage_service(module, service_name, desired_state):
    """Manage service state"""
    current_active = check_service_status(module, service_name)
    changed = False
    
    if desired_state == 'started' and not current_active:
        cmd = f"systemctl start {service_name}"
        rc, stdout, stderr = run_command(module, cmd)
        if rc != 0:
            module.fail_json(msg=f"Failed to start service: {stderr}")
        changed = True
        
    elif desired_state == 'stopped' and current_active:
        cmd = f"systemctl stop {service_name}"
        rc, stdout, stderr = run_command(module, cmd)
        if rc != 0:
            module.fail_json(msg=f"Failed to stop service: {stderr}")
        changed = True
        
    elif desired_state == 'restarted':
        cmd = f"systemctl restart {service_name}"
        rc, stdout, stderr = run_command(module, cmd)
        if rc != 0:
            module.fail_json(msg=f"Failed to restart service: {stderr}")
        changed = True
    
    return changed

def perform_health_check(module, url, timeout):
    """Perform HTTP health check"""
    import urllib.request
    import urllib.error
    import time
    
    start_time = time.time()
    while time.time() - start_time < timeout:
        try:
            response = urllib.request.urlopen(url, timeout=5)
            if response.getcode() == 200:
                return True, {"status": "healthy", "response_code": 200}
        except urllib.error.URLError:
            time.sleep(2)
    
    return False, {"status": "unhealthy", "timeout": timeout}

def main():
    module_args = dict(
        name=dict(type='str', required=True),
        state=dict(type='str', required=False, default='started',
                  choices=['started', 'stopped', 'restarted']),
        health_check_url=dict(type='str', required=False),
        timeout=dict(type='int', required=False, default=30)
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    service_name = module.params['name']
    desired_state = module.params['state']
    health_check_url = module.params['health_check_url']
    timeout = module.params['timeout']

    # Check mode
    if module.check_mode:
        current_active = check_service_status(module, service_name)
        would_change = (
            (desired_state == 'started' and not current_active) or
            (desired_state == 'stopped' and current_active) or
            (desired_state == 'restarted')
        )
        module.exit_json(changed=would_change, service_status='check_mode')

    # Manage service
    changed = manage_service(module, service_name, desired_state)
    current_status = 'active' if check_service_status(module, service_name) else 'inactive'
    
    result = {
        'changed': changed,
        'service_status': current_status
    }

    # Perform health check if URL provided and service should be running
    if health_check_url and desired_state in ['started', 'restarted']:
        health_ok, health_result = perform_health_check(module, health_check_url, timeout)
        result['health_check_result'] = health_result
        
        if not health_ok:
            module.fail_json(msg="Service started but failed health check", **result)

    module.exit_json(**result)

if __name__ == '__main__':
    main()
```

### Dynamic Inventory Scripts
```python
#!/usr/bin/env python3
# inventory/aws_dynamic_inventory.py

import boto3
import json
import argparse

class EC2Inventory:
    def __init__(self):
        self.inventory = {}
        self.read_cli_args()
        
        if self.args.list:
            self.inventory = self.get_inventory()
        elif self.args.host:
            self.inventory = self.get_host_info(self.args.host)
            
        print(json.dumps(self.inventory, indent=2))

    def read_cli_args(self):
        parser = argparse.ArgumentParser()
        parser.add_argument('--list', action='store_true')
        parser.add_argument('--host', action='store')
        self.args = parser.parse_args()

    def get_inventory(self):
        """Generate dynamic inventory from AWS EC2"""
        ec2 = boto3.client('ec2')
        inventory = {'_meta': {'hostvars': {}}}
        
        # Get all running instances
        response = ec2.describe_instances(
            Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
        )
        
        for reservation in response['Reservations']:
            for instance in reservation['Instances']:
                # Extract instance information
                instance_id = instance['InstanceId']
                public_ip = instance.get('PublicIpAddress', '')
                private_ip = instance.get('PrivateIpAddress', '')
                instance_type = instance['InstanceType']
                
                # Get tags
                tags = {tag['Key']: tag['Value'] for tag in instance.get('Tags', [])}
                name = tags.get('Name', instance_id)
                environment = tags.get('Environment', 'unknown')
                role = tags.get('Role', 'unknown')
                
                # Add to inventory groups
                groups = [
                    f"tag_Environment_{environment}",
                    f"tag_Role_{role}",
                    f"type_{instance_type}",
                    "ec2"
                ]
                
                for group in groups:
                    if group not in inventory:
                        inventory[group] = {'hosts': []}
                    inventory[group]['hosts'].append(name)
                
                # Add host variables
                inventory['_meta']['hostvars'][name] = {
                    'ansible_host': public_ip or private_ip,
                    'ec2_instance_id': instance_id,
                    'ec2_instance_type': instance_type,
                    'ec2_public_ip': public_ip,
                    'ec2_private_ip': private_ip,
                    'ec2_tags': tags,
                    'ansible_user': 'ubuntu'  # Default for Ubuntu AMIs
                }
        
        return inventory

    def get_host_info(self, hostname):
        """Get specific host information"""
        inventory = self.get_inventory()
        return inventory['_meta']['hostvars'].get(hostname, {})

if __name__ == '__main__':
    EC2Inventory()
```

### Advanced Playbook Patterns

#### Error Handling and Recovery
```yaml
---
- name: Robust deployment with error handling
  hosts: webservers
  serial: 1
  max_fail_percentage: 0
  
  tasks:
    - name: Application deployment block
      block:
        - name: Stop application
          systemd:
            name: webapp
            state: stopped
          
        - name: Backup current version
          archive:
            path: /opt/webapp
            dest: "/backup/webapp-{{ ansible_date_time.epoch }}.tar.gz"
          register: backup_result
          
        - name: Deploy new version
          unarchive:
            src: "{{ artifact_url }}"
            dest: /opt/webapp
            remote_src: yes
            owner: webapp
            group: webapp
          register: deploy_result
          
        - name: Start application
          systemd:
            name: webapp
            state: started
          register: start_result
          
        - name: Health check
          uri:
            url: "http://{{ ansible_default_ipv4.address }}:8080/health"
            method: GET
            status_code: 200
          register: health_result
          retries: 10
          delay: 5
          
      rescue:
        - name: Log deployment failure
          debug:
            msg: "Deployment failed, initiating rollback"
            
        - name: Stop failed application
          systemd:
            name: webapp
            state: stopped
          ignore_errors: yes
          
        - name: Restore from backup
          unarchive:
            src: "{{ backup_result.dest }}"
            dest: /opt/
            remote_src: yes
          when: backup_result is defined
          
        - name: Start restored application
          systemd:
            name: webapp
            state: started
            
        - name: Verify rollback
          uri:
            url: "http://{{ ansible_default_ipv4.address }}:8080/health"
            method: GET
            status_code: 200
          retries: 5
          delay: 3
          
        - name: Fail deployment
          fail:
            msg: "Deployment failed and rollback completed"
            
      always:
        - name: Send notification
          uri:
            url: "{{ slack_webhook_url }}"
            method: POST
            body_format: json
            body:
              text: "Deployment {{ 'succeeded' if health_result is succeeded else 'failed' }} on {{ inventory_hostname }}"
          delegate_to: localhost
```

#### Advanced Variable Handling
```yaml
---
- name: Complex variable demonstration
  hosts: all
  vars:
    # Complex data structures
    applications:
      frontend:
        port: 3000
        replicas: 3
        health_check: "/health"
        env_vars:
          NODE_ENV: production
          API_URL: "{{ backend_url }}"
      backend:
        port: 8080
        replicas: 2
        health_check: "/api/health"
        env_vars:
          DB_HOST: "{{ database_host }}"
          REDIS_URL: "{{ redis_url }}"
    
    # Environment-specific overrides
    environment_config:
      production:
        log_level: "INFO"
        debug: false
        monitoring_enabled: true
      staging:
        log_level: "DEBUG"
        debug: true
        monitoring_enabled: false
        
  tasks:
    - name: Set environment-specific variables
      set_fact:
        config: "{{ environment_config[environment] }}"
        
    - name: Deploy applications
      include_tasks: deploy_app.yml
      vars:
        app_name: "{{ item.key }}"
        app_config: "{{ item.value }}"
      loop: "{{ applications | dict2items }}"
      loop_control:
        loop_var: item
        
    - name: Generate configuration files
      template:
        src: "{{ app_name }}.conf.j2"
        dest: "/etc/{{ app_name }}/{{ app_name }}.conf"
      vars:
        merged_config: "{{ app_config | combine(config) }}"
      loop: "{{ applications | dict2items }}"
      loop_control:
        loop_var: item
      vars:
        app_name: "{{ item.key }}"
        app_config: "{{ item.value }}"
```

## Testing with Molecule

### Molecule Configuration
```yaml
# molecule/default/molecule.yml
---
dependency:
  name: galaxy
  options:
    requirements-file: requirements.yml
driver:
  name: docker
platforms:
  - name: ubuntu-20
    image: ubuntu:20.04
    pre_build_image: true
    privileged: true
    command: /lib/systemd/systemd
    tmpfs:
      - /run
      - /tmp
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
  - name: centos-8
    image: centos:8
    pre_build_image: true
    privileged: true
    command: /usr/lib/systemd/systemd
    tmpfs:
      - /run
      - /tmp
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
provisioner:
  name: ansible
  config_options:
    defaults:
      callback_whitelist: profile_tasks,timer
      stdout_callback: yaml
  inventory:
    host_vars:
      ubuntu-20:
        ansible_python_interpreter: /usr/bin/python3
      centos-8:
        ansible_python_interpreter: /usr/bin/python3
verifier:
  name: ansible
lint: |
  set -e
  yamllint .
  ansible-lint .
```

### Molecule Test Scenarios
```yaml
# molecule/default/converge.yml
---
- name: Converge
  hosts: all
  become: true
  vars:
    webapp_environment: testing
    webapp_debug: true
  roles:
    - role: webapp

# molecule/default/verify.yml
---
- name: Verify
  hosts: all
  become: true
  tasks:
    - name: Check if webapp service is running
      systemd:
        name: webapp
      register: webapp_service
      
    - name: Verify webapp service is active
      assert:
        that:
          - webapp_service.status.ActiveState == "active"
        fail_msg: "Webapp service is not active"
        
    - name: Check if webapp is responding
      uri:
        url: "http://localhost:8080/health"
        method: GET
        status_code: 200
      register: health_check
      
    - name: Verify health check response
      assert:
        that:
          - health_check.status == 200
          - "'healthy' in health_check.json.status"
        fail_msg: "Webapp health check failed"
        
    - name: Check configuration files
      stat:
        path: "/etc/webapp/webapp.conf"
      register: config_file
      
    - name: Verify configuration file exists
      assert:
        that:
          - config_file.stat.exists
          - config_file.stat.mode == "0644"
        fail_msg: "Configuration file missing or incorrect permissions"
```

### Testing Commands
```bash
# Full test cycle
molecule test

# Test specific scenario
molecule test -s docker

# Development workflow
molecule create        # Create test instances
molecule converge      # Run playbook
molecule verify        # Run tests
molecule destroy       # Clean up

# Debug mode
molecule --debug test

# Test on multiple platforms
molecule test --all
```

## Ansible Tower/AWX Enterprise Features

### Job Templates
```yaml
# tower_job_template.yml
---
- name: Create job template
  tower_job_template:
    name: "Deploy Web Application"
    job_type: "run"
    inventory: "Production"
    project: "WebApp Project"
    playbook: "deploy.yml"
    credentials: 
      - "AWS SSH Key"
      - "Vault Password"
    extra_vars:
      environment: production
      version: "{{ version }}"
    survey_enabled: yes
    survey_spec:
      name: "Deployment Survey"
      description: "Deployment parameters"
      spec:
        - question_name: "Application Version"
          question_description: "Version to deploy"
          required: true
          type: "text"
          variable: "version"
          default: "latest"
```

### Workflow Templates
```yaml
---
- name: Multi-stage deployment workflow
  tower_workflow_job_template:
    name: "Complete Application Deployment"
    organization: "DevOps"
    schema:
      - identifier: "deploy-staging"
        unified_job_template:
          name: "Deploy to Staging"
          type: "job_template"
        success:
          - identifier: "run-tests"
      - identifier: "run-tests"
        unified_job_template:
          name: "Run Integration Tests"
          type: "job_template"
        success:
          - identifier: "deploy-production"
        failure:
          - identifier: "rollback-staging"
      - identifier: "deploy-production"
        unified_job_template:
          name: "Deploy to Production"
          type: "job_template"
        failure:
          - identifier: "rollback-production"
```

## Performance Optimization

### Parallel Execution
```yaml
# ansible.cfg optimizations
[defaults]
host_key_checking = False
forks = 20
timeout = 30
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_fact_cache
fact_caching_timeout = 86400

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
```

### Efficient Playbook Design
```yaml
---
- name: Optimized deployment
  hosts: webservers
  gather_facts: no  # Skip if facts not needed
  strategy: free    # Don't wait for all hosts
  
  tasks:
    - name: Gather minimal facts
      setup:
        gather_subset:
          - "!all"
          - "network"
      when: ansible_default_ipv4 is not defined
      
    - name: Parallel package installation
      package:
        name: "{{ packages }}"
        state: present
      vars:
        packages:
          - nginx
          - postgresql-client
          - redis-tools
      async: 300
      poll: 0
      register: package_install
      
    - name: Wait for package installation
      async_status:
        jid: "{{ package_install.ansible_job_id }}"
      register: job_result
      until: job_result.finished
      retries: 30
      delay: 10
```

## Best Practices Summary

### Code Organization
- Use roles for reusable components
- Implement proper variable precedence
- Version control everything including inventories
- Use tags for selective execution
- Implement proper error handling

### Security
- Use Ansible Vault for secrets
- Implement least privilege access
- Validate inputs and outputs
- Use secure communication channels
- Regular security audits

### Testing
- Unit test roles with Molecule
- Integration testing in staging
- Validate idempotency
- Performance testing for large deployments
- Documentation testing

## Resources
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Molecule Documentation](https://molecule.readthedocs.io/)
- [Ansible Tower Documentation](https://docs.ansible.com/ansible-tower/)
- [Galaxy Role Development](https://galaxy.ansible.com/docs/contributing/creating_role.html)