# Ansible Deep Dive

A comprehensive guide to advanced Ansible concepts, playbooks, roles, and automation patterns.

## What is Ansible?
- Agentless configuration management and automation platform
- Uses SSH for communication (no agents required)
- Uses YAML for playbooks (human-readable)
- Idempotent by design - safe to run multiple times
- Push-based model (Ansible pushes configs to targets)

## Core Concepts

### Inventory Management
```ini
# Static inventory file (hosts.ini)
[webservers]
web1.example.com ansible_host=10.0.1.10
web2.example.com ansible_host=10.0.1.11

[databases]
db1.example.com ansible_host=10.0.2.10
db2.example.com ansible_host=10.0.2.11

[production:children]
webservers
databases

[production:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/path/to/key.pem
```

```yaml
# Dynamic inventory (AWS EC2)
plugin: aws.ec2.aws_ec2
regions:
  - us-east-1
  - us-west-2
keyed_groups:
  - key: tags
    prefix: tag
  - key: instance_type
    prefix: type
compose:
  ansible_host: public_ip_address
```

### Advanced Playbook Structure
```yaml
---
- name: Deploy web application
  hosts: webservers
  become: yes
  vars:
    app_name: myapp
    app_version: "{{ version | default('latest') }}"
  
  pre_tasks:
    - name: Update package cache
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"

  roles:
    - { role: common, tags: ['common'] }
    - { role: nginx, tags: ['nginx'] }
    - { role: application, tags: ['app'] }

  post_tasks:
    - name: Verify application is running
      uri:
        url: "http://{{ ansible_default_ipv4.address }}/health"
        method: GET
        status_code: 200
      register: health_check
      retries: 5
      delay: 10

  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
        enabled: yes
```

## Advanced Features

### Ansible Vault for Secrets
```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt vars/passwords.yml

# Decrypt for viewing
ansible-vault view secrets.yml

# Run playbook with vault
ansible-playbook -i inventory site.yml --ask-vault-pass
```

```yaml
# Using vaulted variables
$ANSIBLE_VAULT;1.1;AES256
66386439653...

# In playbook
- name: Create database user
  mysql_user:
    name: "{{ db_user }}"
    password: "{{ vault_db_password }}"
    priv: "*.*:ALL"
```

### Complex Variable Handling
```yaml
# Group variables (group_vars/webservers.yml)
---
nginx_config:
  worker_processes: 4
  worker_connections: 1024
  keepalive_timeout: 65

ssl_config:
  certificate_path: /etc/ssl/certs/server.crt
  private_key_path: /etc/ssl/private/server.key

# Host variables (host_vars/web1.example.com.yml)  
---
server_id: 1
max_connections: 1000

# Variable precedence in playbook
- name: Configure nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  vars:
    nginx_worker_processes: "{{ nginx_config.worker_processes }}"
  notify: restart nginx
```

### Custom Modules
```python
#!/usr/bin/python
# library/my_custom_module.py

from ansible.module_utils.basic import AnsibleModule

def main():
    module = AnsibleModule(
        argument_spec=dict(
            name=dict(type='str', required=True),
            state=dict(type='str', default='present', choices=['present', 'absent'])
        ),
        supports_check_mode=True
    )
    
    name = module.params['name']
    state = module.params['state']
    
    if module.check_mode:
        module.exit_json(changed=True, msg="Check mode - would create %s" % name)
    
    # Custom logic here
    result = {'changed': True, 'name': name, 'state': state}
    module.exit_json(**result)

if __name__ == '__main__':
    main()
```

### Advanced Loops and Conditionals
```yaml
---
- name: Advanced looping examples
  hosts: all
  tasks:
    # Loop with complex data structures
    - name: Create users with different privileges
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups | join(',') }}"
        shell: "{{ item.shell | default('/bin/bash') }}"
      loop:
        - { name: 'alice', groups: ['sudo', 'developers'], shell: '/bin/zsh' }
        - { name: 'bob', groups: ['developers'] }
        - { name: 'charlie', groups: ['ops', 'sudo'] }

    # Conditional loops
    - name: Install packages based on OS
      package:
        name: "{{ item }}"
        state: present
      loop: "{{ packages[ansible_os_family] | default([]) }}"
      vars:
        packages:
          Debian: ['nginx', 'mysql-server', 'php-fpm']
          RedHat: ['nginx', 'mariadb-server', 'php-fpm']

    # Loop until condition is met
    - name: Wait for service to be ready
      uri:
        url: "http://localhost:8080/health"
      register: result
      until: result.status == 200
      retries: 30
      delay: 5
```

## Role Development

### Role Structure
```
roles/
└── webapp/
    ├── defaults/
    │   └── main.yml          # Default variables
    ├── files/
    │   └── app.conf          # Static files
    ├── handlers/
    │   └── main.yml          # Handlers
    ├── meta/
    │   └── main.yml          # Role metadata
    ├── tasks/
    │   ├── main.yml          # Main tasks
    │   ├── install.yml       # Installation tasks
    │   └── configure.yml     # Configuration tasks
    ├── templates/
    │   └── nginx.conf.j2     # Jinja2 templates
    └── vars/
        └── main.yml          # Role variables
```

### Role Example
```yaml
# roles/webapp/defaults/main.yml
---
webapp_port: 8080
webapp_user: webapp
webapp_dir: /opt/webapp

# roles/webapp/tasks/main.yml
---
- import_tasks: install.yml
  tags: ['install']

- import_tasks: configure.yml  
  tags: ['configure']

- name: Start webapp service
  systemd:
    name: webapp
    state: started
    enabled: yes
    daemon_reload: yes
  tags: ['service']

# roles/webapp/tasks/install.yml
---
- name: Create webapp user
  user:
    name: "{{ webapp_user }}"
    system: yes
    shell: /bin/false

- name: Create webapp directory
  file:
    path: "{{ webapp_dir }}"
    state: directory
    owner: "{{ webapp_user }}"
    group: "{{ webapp_user }}"

# roles/webapp/templates/webapp.service.j2
[Unit]
Description=Web Application
After=network.target

[Service]
Type=simple
User={{ webapp_user }}
WorkingDirectory={{ webapp_dir }}
ExecStart={{ webapp_dir }}/start.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

## Advanced Patterns

### Rolling Deployments
```yaml
---
- name: Rolling deployment
  hosts: webservers
  serial: 1  # Deploy one server at a time
  max_fail_percentage: 0
  
  pre_tasks:
    - name: Remove server from load balancer
      uri:
        url: "http://{{ load_balancer_host }}/api/servers/{{ inventory_hostname }}"
        method: DELETE
      delegate_to: localhost
      
  roles:
    - application
    
  post_tasks:
    - name: Health check
      uri:
        url: "http://{{ ansible_default_ipv4.address }}/health"
        method: GET
        status_code: 200
      retries: 5
      delay: 10
      
    - name: Add server back to load balancer
      uri:
        url: "http://{{ load_balancer_host }}/api/servers"
        method: POST
        body_format: json
        body:
          host: "{{ inventory_hostname }}"
      delegate_to: localhost
```

### Blue-Green Deployment
```yaml
---
- name: Blue-Green deployment
  hosts: localhost
  vars:
    current_env: "{{ (deployment_color == 'blue') | ternary('green', 'blue') }}"
    new_env: "{{ deployment_color }}"
    
  tasks:
    - name: Deploy to {{ new_env }} environment
      include: deploy_app.yml
      vars:
        target_env: "{{ new_env }}"
        
    - name: Run smoke tests on {{ new_env }}
      include: smoke_tests.yml
      vars:
        target_env: "{{ new_env }}"
        
    - name: Switch traffic to {{ new_env }}
      include: switch_traffic.yml
      vars:
        from_env: "{{ current_env }}"
        to_env: "{{ new_env }}"
        
    - name: Stop {{ current_env }} environment
      include: stop_env.yml
      vars:
        target_env: "{{ current_env }}"
```

### Multi-Environment Configuration
```yaml
# inventory/production
[webservers]
prod-web-01
prod-web-02

[webservers:vars]
environment=production
app_replicas=3
db_host=prod-db.internal

# inventory/staging  
[webservers]
staging-web-01

[webservers:vars]
environment=staging
app_replicas=1
db_host=staging-db.internal

# site.yml
---
- name: Deploy application
  hosts: webservers
  roles:
    - role: webapp
      vars:
        webapp_replicas: "{{ app_replicas }}"
        database_host: "{{ db_host }}"
        debug_mode: "{{ environment != 'production' }}"
```

## Testing and Validation

### Molecule Testing Framework
```yaml
# molecule/default/molecule.yml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: ubuntu
    image: ubuntu:20.04
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible

# molecule/default/converge.yml
---
- name: Converge
  hosts: all
  become: true
  roles:
    - role: webapp

# molecule/default/verify.yml  
---
- name: Verify
  hosts: all
  tasks:
    - name: Check if webapp is running
      systemd:
        name: webapp
      register: webapp_service
      
    - name: Verify webapp service is running
      assert:
        that:
          - webapp_service.status.ActiveState == "active"
```

### Testing Commands
```bash
# Install molecule
pip install molecule[docker]

# Initialize new role with molecule
molecule init role webapp --driver-name docker

# Test role
molecule test

# Test specific scenario
molecule test -s docker
```

### Ansible Lint
```yaml
# .ansible-lint
---
exclude_paths:
  - .cache/
  - .github/
  - molecule/
skip_list:
  - yaml[line-length]
  - name[casing]

# Custom rules
rules:
  line-length:
    max: 120
```

## Best Practices

### Project Structure
```
ansible-project/
├── inventories/
│   ├── production/
│   │   ├── hosts.ini
│   │   └── group_vars/
│   └── staging/
│       ├── hosts.ini
│       └── group_vars/
├── roles/
├── playbooks/
├── requirements.yml
├── ansible.cfg
├── site.yml
└── deploy.yml
```

### Error Handling
```yaml
---
- name: Robust task execution
  block:
    - name: Risky operation
      command: /usr/bin/risky-command
      register: result
      
    - name: Process results
      debug:
        msg: "Command succeeded: {{ result.stdout }}"
        
  rescue:
    - name: Handle failure
      debug:
        msg: "Command failed: {{ result.stderr | default('Unknown error') }}"
        
    - name: Fallback action
      command: /usr/bin/safe-fallback
      
  always:
    - name: Cleanup
      file:
        path: /tmp/tempfile
        state: absent

- name: Continue on some failures
  command: /usr/bin/might-fail
  register: result
  failed_when: result.rc not in [0, 2]  # Accept return codes 0 and 2
  changed_when: result.rc == 0
```

## Integration and Orchestration

### AWX/Tower Integration
```yaml
# Job template configuration
---
name: "Deploy Web Application"
description: "Deploy webapp to production"
job_type: "run"
inventory: "Production"
project: "WebApp Deployment"  
playbook: "site.yml"
credentials:
  - "AWS Credentials"
  - "SSH Key"
extra_vars:
  version: "{{ version }}"
  environment: "production"
```

### CI/CD Integration
```yaml
# .gitlab-ci.yml
stages:
  - test
  - deploy

test_playbook:
  stage: test
  script:
    - ansible-lint playbooks/
    - molecule test

deploy_production:
  stage: deploy
  script:
    - ansible-playbook -i inventories/production site.yml --vault-password-file vault_pass
  only:
    - main
  when: manual
```

## Exercises & Next Steps

- Exercise: Create a complete role for a web application with testing
- Next: Build a multi-tier application deployment with Ansible
- Advanced: Implement infrastructure provisioning with Ansible and cloud modules

## Resources
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Molecule Testing](https://molecule.readthedocs.io/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)