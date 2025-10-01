# Chef Infrastructure Automation

A comprehensive guide to Chef configuration management, cookbook development, and enterprise Chef patterns.

## What is Chef?
- Infrastructure automation platform using Ruby DSL
- Agent-based configuration management with Chef Server
- Declarative resource management with imperative flexibility
- Strong testing framework with Test Kitchen and InSpec
- Enterprise-focused with compliance and reporting features

## Core Chef Concepts

### Chef Architecture
```
Chef Workstation ←→ Chef Server ←→ Chef Nodes
     ↓                   ↓              ↓
  Cookbooks         Cookbook Store    Chef Client
  Test Kitchen       Node Objects     Ohai (facts)
  Knife CLI         Search Index      Resources
```

### Resources and Recipes
```ruby
# recipes/default.rb
# Install and configure nginx web server

package 'nginx' do
  action :install
end

service 'nginx' do
  action [:enable, :start]
  supports restart: true, reload: true
end

template '/etc/nginx/nginx.conf' do
  source 'nginx.conf.erb'
  owner 'root'
  group 'root'
  mode '0644'
  variables(
    worker_processes: node['nginx']['worker_processes'],
    worker_connections: node['nginx']['worker_connections']
  )
  notifies :reload, 'service[nginx]', :immediately
end

directory '/var/www/html' do
  owner 'www-data'
  group 'www-data'
  mode '0755'
  recursive true
end

file '/var/www/html/index.html' do
  content lazy { 
    "<h1>Welcome to #{node['fqdn']}</h1>
     <p>Configured by Chef on #{Time.now}</p>"
  }
  owner 'www-data'
  group 'www-data'
  mode '0644'
end
```

### Attributes and Data Bags
```ruby
# attributes/default.rb
default['nginx']['worker_processes'] = 'auto'
default['nginx']['worker_connections'] = 1024
default['nginx']['keepalive_timeout'] = 65
default['nginx']['server_names_hash_bucket_size'] = 64

# Environment-specific attributes
case node.chef_environment
when 'production'
  default['nginx']['worker_processes'] = node['cpu']['total']
  default['nginx']['worker_connections'] = 2048
when 'development'
  default['nginx']['worker_processes'] = 1
  default['nginx']['worker_connections'] = 512
end

# Platform-specific attributes  
case node['platform_family']
when 'debian'
  default['nginx']['package_name'] = 'nginx'
  default['nginx']['service_name'] = 'nginx'
  default['nginx']['config_dir'] = '/etc/nginx'
when 'rhel'
  default['nginx']['package_name'] = 'nginx'
  default['nginx']['service_name'] = 'nginx'
  default['nginx']['config_dir'] = '/etc/nginx'
end
```

### Data Bags for Secrets
```ruby
# Create encrypted data bag
# knife data bag create secrets database --secret-file ~/.chef/encrypted_data_bag_secret

# Data bag content (data_bags/secrets/database.json)
{
  "id": "database",
  "mysql": {
    "root_password": "SuperSecretPassword123!",
    "app_user": "webapp",
    "app_password": "WebAppPassword456!"
  },
  "redis": {
    "password": "RedisPassword789!"
  }
}

# Using encrypted data bag in recipe
db_secrets = data_bag_item('secrets', 'database')

mysql_service 'default' do
  port '3306'
  version '8.0'
  initial_root_password db_secrets['mysql']['root_password']
  action [:create, :start]
end

mysql_database 'webapp' do
  connection(
    host: '127.0.0.1',
    username: 'root',
    password: db_secrets['mysql']['root_password']
  )
  action :create
end

mysql_database_user db_secrets['mysql']['app_user'] do
  connection(
    host: '127.0.0.1',
    username: 'root',
    password: db_secrets['mysql']['root_password']
  )
  password db_secrets['mysql']['app_password']
  privileges ['SELECT', 'INSERT', 'UPDATE', 'DELETE']
  database_name 'webapp'
  action [:create, :grant]
end
```

## Advanced Cookbook Development

### Custom Resources (LWRP/HWRP)
```ruby
# resources/webapp.rb
provides :webapp
unified_mode true

property :app_name, String, name_property: true
property :version, String, required: true
property :port, Integer, default: 8080
property :user, String, default: 'webapp'
property :config_template, String, default: 'webapp.conf.erb'
property :environment_variables, Hash, default: {}

action :install do
  # Create application user
  user new_resource.user do
    system true
    shell '/bin/false'
    home "/opt/#{new_resource.app_name}"
    action :create
  end

  # Create application directory
  directory "/opt/#{new_resource.app_name}" do
    owner new_resource.user
    group new_resource.user
    mode '0755'
    action :create
  end

  # Download and extract application
  remote_file "/tmp/#{new_resource.app_name}-#{new_resource.version}.tar.gz" do
    source "https://releases.example.com/#{new_resource.app_name}-#{new_resource.version}.tar.gz"
    checksum lazy { node['webapp']['checksums'][new_resource.version] }
    action :create
  end

  archive_file "/tmp/#{new_resource.app_name}-#{new_resource.version}.tar.gz" do
    destination "/opt/#{new_resource.app_name}"
    owner new_resource.user
    group new_resource.user
    action :extract
  end

  # Create configuration file
  template "/opt/#{new_resource.app_name}/config/app.conf" do
    source new_resource.config_template
    owner new_resource.user
    group new_resource.user
    mode '0600'
    variables(
      port: new_resource.port,
      environment: new_resource.environment_variables
    )
    notifies :restart, "systemd_unit[#{new_resource.app_name}.service]", :delayed
  end

  # Create systemd service
  systemd_unit "#{new_resource.app_name}.service" do
    content({
      Unit: {
        Description: "#{new_resource.app_name} application",
        After: 'network.target'
      },
      Service: {
        Type: 'simple',
        User: new_resource.user,
        WorkingDirectory: "/opt/#{new_resource.app_name}",
        ExecStart: "/opt/#{new_resource.app_name}/bin/start",
        Restart: 'always',
        Environment: new_resource.environment_variables.map { |k, v| "#{k}=#{v}" }
      },
      Install: {
        WantedBy: 'multi-user.target'
      }
    })
    action [:create, :enable, :start]
  end
end

action :remove do
  systemd_unit "#{new_resource.app_name}.service" do
    action [:stop, :disable, :delete]
  end

  directory "/opt/#{new_resource.app_name}" do
    recursive true
    action :delete
  end

  user new_resource.user do
    action :remove
  end
end
```

### Using Custom Resources
```ruby
# recipes/application.rb
webapp 'myapp' do
  version '2.1.0'
  port 8080
  user 'myapp'
  environment_variables(
    'DB_HOST' => node['myapp']['database']['host'],
    'DB_USER' => node['myapp']['database']['user'],
    'DB_PASS' => db_secrets['mysql']['app_password'],
    'REDIS_URL' => "redis://localhost:6379"
  )
  action :install
end
```

### Libraries and Helpers
```ruby
# libraries/helper.rb
module MyappCookbook
  module Helpers
    def database_connection_string
      secrets = data_bag_item('secrets', 'database')
      host = node['myapp']['database']['host']
      user = node['myapp']['database']['user']
      password = secrets['mysql']['app_password']
      database = node['myapp']['database']['name']
      
      "mysql://#{user}:#{password}@#{host}/#{database}"
    end

    def app_version_from_environment
      case node.chef_environment
      when 'production'
        node['myapp']['stable_version']
      when 'staging'
        node['myapp']['testing_version']
      else
        node['myapp']['development_version']
      end
    end

    def platform_specific_packages
      case node['platform_family']
      when 'debian'
        %w[build-essential libssl-dev]
      when 'rhel'
        %w[gcc openssl-devel]
      else
        []
      end
    end
  end
end

# Include in recipe context
Chef::Recipe.send(:include, MyappCookbook::Helpers)
Chef::Resource.send(:include, MyappCookbook::Helpers)
```

## Testing with Test Kitchen

### Kitchen Configuration
```yaml
# .kitchen.yml
---
driver:
  name: docker
  privileged: true

provisioner:
  name: chef_zero
  product_name: chef
  product_version: 17
  deprecations_as_errors: true

verifier:
  name: inspec

platforms:
  - name: ubuntu-20.04
    driver_config:
      image: ubuntu:20.04
      platform: ubuntu
      run_command: /lib/systemd/systemd
      provision_command:
        - apt-get update
        - apt-get install -y systemd
      tmpfs:
        - /run
        - /tmp
      volumes:
        - /sys/fs/cgroup:/sys/fs/cgroup:ro
        
  - name: centos-8
    driver_config:
      image: centos:8
      platform: centos
      run_command: /usr/lib/systemd/systemd
      provision_command:
        - yum install -y systemd
      tmpfs:
        - /run
        - /tmp
      volumes:
        - /sys/fs/cgroup:/sys/fs/cgroup:ro

suites:
  - name: default
    run_list:
      - recipe[myapp::default]
    attributes:
      myapp:
        version: "2.1.0"
        database:
          host: "localhost"
          user: "webapp"
          name: "webapp"
    verifier:
      inspec_tests:
        - test/integration/default

  - name: production
    run_list:
      - recipe[myapp::default]
    attributes:
      myapp:
        version: "2.0.5"
        worker_processes: 4
    verifier:
      inspec_tests:
        - test/integration/production
```

### InSpec Tests
```ruby
# test/integration/default/default_test.rb
describe package('nginx') do
  it { should be_installed }
end

describe service('nginx') do
  it { should be_enabled }
  it { should be_running }
end

describe port(80) do
  it { should be_listening }
  its('protocols') { should include 'tcp' }
end

describe file('/etc/nginx/nginx.conf') do
  it { should exist }
  it { should be_owned_by 'root' }
  it { should be_grouped_into 'root' }
  its('mode') { should cmp '0644' }
end

describe file('/var/www/html/index.html') do
  it { should exist }
  its('content') { should match(/Welcome to/) }
end

describe http('http://localhost') do
  its('status') { should eq 200 }
  its('body') { should match(/Welcome to/) }
end

# Custom application tests
describe file('/opt/myapp/config/app.conf') do
  it { should exist }
  its('content') { should match(/port.*8080/) }
end

describe service('myapp') do
  it { should be_enabled }
  it { should be_running }
end

describe http('http://localhost:8080/health') do
  its('status') { should eq 200 }
  its('body') { should match(/healthy/) }
end
```

### Testing Commands
```bash
# List test suites
kitchen list

# Create test instances
kitchen create

# Converge (run Chef)
kitchen converge

# Run tests
kitchen verify

# Full test cycle
kitchen test

# Test specific suite
kitchen test default-ubuntu-2004

# Debug mode
kitchen diagnose

# SSH into test instance
kitchen login default-ubuntu-2004
```

## Enterprise Chef Patterns

### Environments and Roles
```ruby
# environments/production.rb
name 'production'
description 'Production environment'
cookbook_versions(
  'myapp' => '= 2.1.0',
  'nginx' => '~> 8.0.0',
  'mysql' => '~> 9.0.0'
)
default_attributes(
  'myapp' => {
    'worker_processes' => 4,
    'log_level' => 'info',
    'monitoring_enabled' => true
  }
)

# roles/webserver.rb
name 'webserver'
description 'Web server role'
run_list(
  'recipe[nginx::default]',
  'recipe[myapp::default]'
)
default_attributes(
  'nginx' => {
    'worker_processes' => 'auto',
    'keepalive_timeout' => 65
  }
)
override_attributes(
  'myapp' => {
    'type' => 'webserver'
  }
)
```

### Policy Files (Policyfile.rb)
```ruby
# Policyfile.rb - Modern Chef approach
name 'myapp'
default_source :supermarket

run_list 'myapp::default'

cookbook 'myapp', path: '.'
cookbook 'nginx', '~> 8.0'
cookbook 'mysql', '~> 9.0'

default['myapp']['version'] = '2.1.0'
default['myapp']['environment'] = 'production'

named_run_list :webserver, 'nginx::default', 'myapp::webserver'
named_run_list :database, 'mysql::server', 'myapp::database'
```

### Multi-Node Orchestration
```ruby
# recipes/orchestrated_deployment.rb
# Deploy application across multiple nodes with coordination

# Get all web servers
web_servers = search(:node, "role:webserver AND chef_environment:#{node.chef_environment}")

# Rolling deployment logic
web_servers.each_with_index do |server, index|
  # Wait for previous server to be healthy
  if index > 0
    previous_server = web_servers[index - 1]
    
    ruby_block "wait_for_#{previous_server['fqdn']}" do
      block do
        require 'net/http'
        require 'timeout'
        
        Chef::Log.info("Waiting for #{previous_server['fqdn']} to be healthy")
        
        Timeout.timeout(300) do
          loop do
            begin
              response = Net::HTTP.get_response(
                URI("http://#{previous_server['ipaddress']}:8080/health")
              )
              break if response.code == '200'
            rescue
              Chef::Log.info("#{previous_server['fqdn']} not ready yet, waiting...")
            end
            sleep 10
          end
        end
      end
      only_if { server['fqdn'] == node['fqdn'] }
    end
  end

  # Deploy to current node
  webapp 'myapp' do
    version node['myapp']['version']
    action :install
    only_if { server['fqdn'] == node['fqdn'] }
  end
end
```

## Chef Server Administration

### Knife Commands
```bash
# Node management
knife node list
knife node show web1.example.com
knife node edit web1.example.com

# Cookbook management
knife cookbook upload myapp
knife cookbook list
knife cookbook show myapp 2.1.0

# Environment management
knife environment list
knife environment show production
knife environment from file environments/production.rb

# Data bag management
knife data bag create secrets
knife data bag from file secrets data_bags/secrets/database.json
knife data bag show secrets database

# Bootstrap new nodes
knife bootstrap web1.example.com \
  --ssh-user ubuntu \
  --sudo \
  --identity-file ~/.ssh/aws-key.pem \
  --node-name web1 \
  --run-list 'role[webserver]' \
  --environment production

# Search functionality
knife search node "platform:ubuntu"
knife search node "role:webserver AND chef_environment:production"
knife search role "run_list:*mysql*"
```

### Chef Server API Usage
```ruby
# libraries/chef_api.rb
require 'chef/rest'

class ChefServerAPI
  def initialize
    @rest = Chef::REST.new(Chef::Config[:chef_server_url])
  end

  def get_nodes_by_role(role_name)
    search_query = "role:#{role_name}"
    @rest.get("/search/node?q=#{URI.encode(search_query)}")['rows']
  end

  def update_node_attribute(node_name, attribute_path, value)
    node = @rest.get("/nodes/#{node_name}")
    keys = attribute_path.split('.')
    current = node['normal']
    
    keys[0..-2].each do |key|
      current[key] ||= {}
      current = current[key]
    end
    
    current[keys.last] = value
    @rest.put("/nodes/#{node_name}", node)
  end

  def get_environment_nodes(environment)
    search_query = "chef_environment:#{environment}"
    @rest.get("/search/node?q=#{URI.encode(search_query)}")['rows']
  end
end
```

## Performance and Scaling

### Cookbook Optimization
```ruby
# Use guards to prevent unnecessary work
package 'nginx' do
  action :install
  not_if 'dpkg -l | grep -q nginx'
end

# Efficient file operations
remote_file '/tmp/large-file.tar.gz' do
  source 'https://example.com/large-file.tar.gz'
  checksum 'sha256sum_here'
  action :create
  only_if { ::File.size('/tmp/large-file.tar.gz') rescue 0 == 0 }
end

# Use lazy evaluation for dynamic content
file '/etc/myapp/config.yml' do
  content lazy {
    config = node['myapp']['config'].to_hash
    config['database_url'] = database_connection_string
    YAML.dump(config)
  }
  action :create
end

# Batch operations
node['myapp']['packages'].each do |pkg|
  package pkg do
    action :nothing
  end
end

execute 'install-packages' do
  command "apt-get update && apt-get install -y #{node['myapp']['packages'].join(' ')}"
  only_if { node['platform_family'] == 'debian' }
  notifies :nothing, 'package[*]', :before
end
```

### Chef Client Configuration
```ruby
# client.rb optimizations
node_name 'web1.example.com'
chef_server_url 'https://chef.example.com/organizations/myorg'
validation_client_name 'myorg-validator'
validation_key '/etc/chef/myorg-validator.pem'

# Performance tuning
ssl_verify_mode :verify_peer
file_cache_path '/var/chef/cache'
cookbook_path ['/var/chef/cache/cookbooks']

# Reduce Chef runs frequency for stable environments
interval 3600  # Run every hour instead of every 30 minutes
splay 300      # Random delay up to 5 minutes

# Reduce network calls
ohai.disabled_plugins = [:Passwd]
automatic_attribute_whitelist = ['fqdn', 'hostname', 'ipaddress', 'platform']
```

## Best Practices

### Code Organization
```
cookbooks/
├── myapp/
│   ├── attributes/
│   ├── recipes/
│   ├── resources/
│   ├── libraries/
│   ├── templates/
│   ├── test/
│   └── metadata.rb
├── base/
├── nginx/
└── mysql/

environments/
├── production.rb
├── staging.rb
└── development.rb

roles/
├── webserver.rb
├── database.rb
└── loadbalancer.rb

data_bags/
├── secrets/
├── users/
└── certificates/
```

### Security Best Practices
- Use encrypted data bags for secrets
- Implement proper file permissions
- Regular security updates through Chef
- Audit trails with Chef reporting
- Least privilege principle

### Testing Strategy
- Unit tests for custom resources
- Integration tests with Test Kitchen
- Compliance tests with InSpec
- Performance testing for large deployments
- Continuous integration with cookbook updates

## Resources
- [Chef Documentation](https://docs.chef.io/)
- [Chef Supermarket](https://supermarket.chef.io/)
- [Test Kitchen Documentation](https://kitchen.ci/)
- [InSpec Documentation](https://docs.chef.io/inspec/)