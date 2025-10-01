# Puppet Configuration Management

A comprehensive guide to Puppet configuration management, module development, and enterprise Puppet patterns.

## What is Puppet?
- Declarative configuration management using Puppet DSL
- Agent-based architecture with Puppet Master/Server
- Strong emphasis on compliance and reporting
- Mature ecosystem with Puppet Forge modules
- Enterprise features with Puppet Enterprise (PE)

## Core Puppet Concepts

### Puppet Architecture
```
Puppet Master/Server ←→ Puppet Agents
       ↓                     ↓
   Catalog Compilation    Fact Collection (Facter)
   Module Storage         Resource Application
   Certificate Authority  Report Generation
```

### Resources and Manifests
```puppet
# manifests/site.pp
# Install and configure nginx web server

package { 'nginx':
  ensure => installed,
}

service { 'nginx':
  ensure     => running,
  enable     => true,
  hasrestart => true,
  hasstatus  => true,
  require    => Package['nginx'],
}

file { '/etc/nginx/nginx.conf':
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  content => template('nginx/nginx.conf.erb'),
  notify  => Service['nginx'],
  require => Package['nginx'],
}

file { '/var/www/html':
  ensure => directory,
  owner  => 'www-data',
  group  => 'www-data',
  mode   => '0755',
}

file { '/var/www/html/index.html':
  ensure  => file,
  owner   => 'www-data',
  group   => 'www-data',
  mode    => '0644',
  content => "Welcome to ${::fqdn}\nConfigured by Puppet on ${::timestamp}",
  require => File['/var/www/html'],
}
```

### Classes and Modules
```puppet
# modules/webapp/manifests/init.pp
class webapp (
  String $version = '2.1.0',
  Integer $port = 8080,
  String $user = 'webapp',
  Hash $environment_variables = {},
  Boolean $manage_user = true,
  String $config_template = 'webapp/app.conf.erb',
) {

  if $manage_user {
    user { $user:
      ensure     => present,
      system     => true,
      shell      => '/bin/false',
      home       => "/opt/${user}",
      managehome => true,
    }
  }

  file { "/opt/${user}":
    ensure  => directory,
    owner   => $user,
    group   => $user,
    mode    => '0755',
    require => User[$user],
  }

  archive { "/tmp/webapp-${version}.tar.gz":
    ensure       => present,
    source       => "https://releases.example.com/webapp-${version}.tar.gz",
    extract      => true,
    extract_path => "/opt/${user}",
    creates      => "/opt/${user}/bin/webapp",
    user         => $user,
    group        => $user,
    require      => File["/opt/${user}"],
  }

  file { "/opt/${user}/config/app.conf":
    ensure  => file,
    owner   => $user,
    group   => $user,
    mode    => '0600',
    content => template($config_template),
    notify  => Service['webapp'],
    require => Archive["/tmp/webapp-${version}.tar.gz"],
  }

  systemd::unit_file { 'webapp.service':
    content => template('webapp/webapp.service.erb'),
    notify  => Service['webapp'],
  }

  service { 'webapp':
    ensure     => running,
    enable     => true,
    hasrestart => true,
    hasstatus  => true,
    require    => [
      Systemd::Unit_file['webapp.service'],
      File["/opt/${user}/config/app.conf"],
    ],
  }
}
```

### Parameterized Classes
```puppet
# modules/webapp/manifests/params.pp
class webapp::params {
  case $::operatingsystem {
    'Ubuntu', 'Debian': {
      $package_provider = 'apt'
      $service_provider = 'systemd'
    }
    'CentOS', 'RedHat': {
      $package_provider = 'yum'
      $service_provider = 'systemd'
    }
    default: {
      fail("Unsupported operating system: ${::operatingsystem}")
    }
  }

  case $::webapp_environment {
    'production': {
      $worker_processes = $::processorcount
      $log_level = 'info'
      $debug_mode = false
    }
    'staging': {
      $worker_processes = 2
      $log_level = 'debug'
      $debug_mode = true
    }
    'development': {
      $worker_processes = 1
      $log_level = 'debug'
      $debug_mode = true
    }
    default: {
      $worker_processes = 1
      $log_level = 'info'
      $debug_mode = false
    }
  }
}
```

### Hiera Data Management
```yaml
# hieradata/common.yaml
---
webapp::version: '2.1.0'
webapp::port: 8080
webapp::user: 'webapp'
webapp::manage_user: true

nginx::worker_processes: 'auto'
nginx::worker_connections: 1024
nginx::keepalive_timeout: 65

mysql::root_password: 'changeme'
mysql::databases:
  webapp:
    ensure: 'present'
    charset: 'utf8'

# hieradata/environment/production.yaml
---
webapp::version: '2.0.5'
webapp::environment_variables:
  NODE_ENV: 'production'
  LOG_LEVEL: 'info'
  
nginx::worker_processes: "%{facts.processors.count}"
nginx::worker_connections: 2048

mysql::root_password: "%{lookup('mysql_root_password')}"

# hieradata/nodes/web1.example.com.yaml
---
webapp::port: 8080
webapp::environment_variables:
  SERVER_ID: 'web1'
  DATACENTER: 'us-east-1'
```

### Custom Facts
```ruby
# lib/facter/webapp_version.rb
Facter.add(:webapp_version) do
  confine :kernel => 'Linux'
  setcode do
    if File.exist?('/opt/webapp/VERSION')
      File.read('/opt/webapp/VERSION').strip
    else
      'unknown'
    end
  end
end

# lib/facter/webapp_status.rb
Facter.add(:webapp_status) do
  confine :kernel => 'Linux'
  setcode do
    begin
      output = Facter::Core::Execution.execute('systemctl is-active webapp', timeout: 10)
      output.strip == 'active' ? 'running' : 'stopped'
    rescue
      'unknown'
    end
  end
end

# lib/facter/database_connections.rb
Facter.add(:database_connections) do
  confine :kernel => 'Linux'
  setcode do
    begin
      if File.exist?('/usr/bin/mysql')
        output = Facter::Core::Execution.execute(
          "mysql -u root -e 'SHOW PROCESSLIST;' | wc -l", 
          timeout: 10
        )
        output.to_i - 1  # Subtract header line
      else
        0
      end
    rescue
      'unknown'
    end
  end
end
```

## Advanced Puppet Patterns

### Defined Types
```puppet
# modules/webapp/manifests/vhost.pp
define webapp::vhost (
  String $server_name = $title,
  Integer $port = 80,
  Optional[String] $ssl_cert = undef,
  Optional[String] $ssl_key = undef,
  String $document_root = "/var/www/${title}",
  Array[String] $server_aliases = [],
  Hash $custom_config = {},
) {

  $vhost_name = $title
  
  file { $document_root:
    ensure => directory,
    owner  => 'www-data',
    group  => 'www-data',
    mode   => '0755',
  }

  if $ssl_cert and $ssl_key {
    $ssl_enabled = true
    $listen_port = 443
    $redirect_port = 80
  } else {
    $ssl_enabled = false
    $listen_port = $port
    $redirect_port = undef
  }

  file { "/etc/nginx/sites-available/${vhost_name}":
    ensure  => file,
    owner   => 'root',
    group   => 'root',
    mode    => '0644',
    content => template('webapp/vhost.conf.erb'),
    notify  => Service['nginx'],
    require => Package['nginx'],
  }

  file { "/etc/nginx/sites-enabled/${vhost_name}":
    ensure  => link,
    target  => "/etc/nginx/sites-available/${vhost_name}",
    notify  => Service['nginx'],
    require => File["/etc/nginx/sites-available/${vhost_name}"],
  }

  if $ssl_enabled {
    file { "/etc/ssl/certs/${vhost_name}.crt":
      ensure  => file,
      owner   => 'root',
      group   => 'root',
      mode    => '0644',
      source  => $ssl_cert,
      notify  => Service['nginx'],
    }

    file { "/etc/ssl/private/${vhost_name}.key":
      ensure  => file,
      owner   => 'root',
      group   => 'root',
      mode    => '0600',
      source  => $ssl_key,
      notify  => Service['nginx'],
    }
  }
}

# Usage in manifest
webapp::vhost { 'example.com':
  port           => 80,
  document_root  => '/var/www/example',
  server_aliases => ['www.example.com'],
}

webapp::vhost { 'secure.example.com':
  port        => 443,
  ssl_cert    => 'puppet:///modules/webapp/ssl/secure.example.com.crt',
  ssl_key     => 'puppet:///modules/webapp/ssl/secure.example.com.key',
  custom_config => {
    'client_max_body_size' => '50M',
    'proxy_read_timeout'   => '300',
  },
}
```

### Custom Resource Types
```puppet
# lib/puppet/type/webapp_deployment.rb
Puppet::Type.newtype(:webapp_deployment) do
  @doc = "Manage webapp deployments with health checks"

  ensurable

  newparam(:name, :namevar => true) do
    desc "The name of the deployment"
  end

  newproperty(:version) do
    desc "The version to deploy"
    
    def insync?(is)
      is == should
    end
  end

  newparam(:artifact_url) do
    desc "URL to download the artifact"
  end

  newparam(:health_check_url) do
    desc "URL for health checking after deployment"
  end

  newparam(:timeout) do
    desc "Timeout for health check"
    defaultto 60
  end

  newparam(:rollback_on_failure) do
    desc "Whether to rollback on deployment failure"
    defaultto true
  end

  validate do
    fail("artifact_url is required") unless self[:artifact_url]
    fail("version is required") unless self[:version]
  end
end

# lib/puppet/provider/webapp_deployment/default.rb
Puppet::Type.type(:webapp_deployment).provide(:default) do
  desc "Default provider for webapp deployments"

  def exists?
    current_version == resource[:version]
  end

  def create
    deploy_version(resource[:version])
  end

  def destroy
    # Implement rollback logic
    previous_version = get_previous_version
    if previous_version
      deploy_version(previous_version)
    end
  end

  def version
    current_version
  end

  def version=(value)
    deploy_version(value)
  end

  private

  def current_version
    if File.exist?('/opt/webapp/VERSION')
      File.read('/opt/webapp/VERSION').strip
    else
      :absent
    end
  end

  def deploy_version(version)
    Puppet.info("Deploying webapp version #{version}")
    
    # Stop service
    system('systemctl stop webapp')
    
    # Backup current version
    if File.exist?('/opt/webapp')
      backup_path = "/backup/webapp-#{Time.now.to_i}"
      system("mv /opt/webapp #{backup_path}")
    end
    
    # Download and extract new version
    artifact_url = resource[:artifact_url].gsub('VERSION', version)
    system("wget -O /tmp/webapp-#{version}.tar.gz #{artifact_url}")
    system("mkdir -p /opt/webapp")
    system("tar -xzf /tmp/webapp-#{version}.tar.gz -C /opt/webapp")
    
    # Write version file
    File.write('/opt/webapp/VERSION', version)
    
    # Start service
    system('systemctl start webapp')
    
    # Health check
    if resource[:health_check_url]
      health_check_passed = perform_health_check
      
      if !health_check_passed && resource[:rollback_on_failure]
        Puppet.warning("Health check failed, rolling back")
        rollback
        fail("Deployment failed health check and was rolled back")
      end
    end
  end

  def perform_health_check
    require 'net/http'
    timeout = resource[:timeout]
    url = URI(resource[:health_check_url])
    
    start_time = Time.now
    while Time.now - start_time < timeout
      begin
        response = Net::HTTP.get_response(url)
        return true if response.code == '200'
      rescue
        # Continue trying
      end
      sleep 5
    end
    
    false
  end

  def rollback
    # Implement rollback logic
    backup_dirs = Dir.glob('/backup/webapp-*').sort.reverse
    if backup_dirs.any?
      latest_backup = backup_dirs.first
      system('systemctl stop webapp')
      system('rm -rf /opt/webapp')
      system("mv #{latest_backup} /opt/webapp")
      system('systemctl start webapp')
    end
  end
end

# Usage in manifest
webapp_deployment { 'production':
  ensure            => present,
  version           => '2.1.0',
  artifact_url      => 'https://releases.example.com/webapp-VERSION.tar.gz',
  health_check_url  => 'http://localhost:8080/health',
  timeout           => 120,
  rollback_on_failure => true,
}
```

## Testing with rspec-puppet

### Module Testing Structure
```
modules/webapp/
├── spec/
│   ├── spec_helper.rb
│   ├── classes/
│   │   └── webapp_spec.rb
│   ├── defines/
│   │   └── vhost_spec.rb
│   └── fixtures/
│       ├── manifests/
│       └── modules/
└── Rakefile
```

### RSpec Tests
```ruby
# spec/spec_helper.rb
require 'puppetlabs_spec_helper/module_spec_helper'
require 'rspec-puppet-facts'

include RspecPuppetFacts

RSpec.configure do |c|
  c.hiera_config = 'spec/fixtures/hiera.yaml'
  c.default_facts = {
    :kernel                 => 'Linux',
    :osfamily              => 'Debian',
    :operatingsystem       => 'Ubuntu',
    :operatingsystemrelease => '20.04',
    :processorcount        => 2,
    :fqdn                  => 'test.example.com',
  }
end

# spec/classes/webapp_spec.rb
require 'spec_helper'

describe 'webapp' do
  let(:facts) { { :kernel => 'Linux', :osfamily => 'Debian' } }

  context 'with default parameters' do
    it { is_expected.to compile }
    
    it { is_expected.to contain_user('webapp') }
    
    it { is_expected.to contain_file('/opt/webapp').with(
      'ensure' => 'directory',
      'owner'  => 'webapp',
      'group'  => 'webapp',
      'mode'   => '0755'
    )}
    
    it { is_expected.to contain_service('webapp').with(
      'ensure' => 'running',
      'enable' => true
    )}
  end

  context 'with custom parameters' do
    let(:params) do
      {
        :version => '1.0.0',
        :port    => 9000,
        :user    => 'myapp',
      }
    end

    it { is_expected.to contain_user('myapp') }
    
    it { is_expected.to contain_file('/opt/myapp/config/app.conf').with_content(
      /port.*9000/
    )}
  end

  context 'on CentOS' do
    let(:facts) do
      {
        :kernel             => 'Linux',
        :osfamily          => 'RedHat',
        :operatingsystem   => 'CentOS',
      }
    end

    it { is_expected.to compile }
    it { is_expected.to contain_package('webapp') }
  end
end

# spec/defines/vhost_spec.rb
require 'spec_helper'

describe 'webapp::vhost' do
  let(:title) { 'example.com' }
  let(:facts) { { :kernel => 'Linux', :osfamily => 'Debian' } }

  context 'with minimal parameters' do
    it { is_expected.to compile }
    
    it { is_expected.to contain_file('/var/www/example.com') }
    
    it { is_expected.to contain_file('/etc/nginx/sites-available/example.com') }
    
    it { is_expected.to contain_file('/etc/nginx/sites-enabled/example.com').with(
      'ensure' => 'link',
      'target' => '/etc/nginx/sites-available/example.com'
    )}
  end

  context 'with SSL enabled' do
    let(:params) do
      {
        :ssl_cert => 'puppet:///modules/webapp/ssl/example.com.crt',
        :ssl_key  => 'puppet:///modules/webapp/ssl/example.com.key',
      }
    end

    it { is_expected.to contain_file('/etc/ssl/certs/example.com.crt') }
    it { is_expected.to contain_file('/etc/ssl/private/example.com.key').with(
      'mode' => '0600'
    )}
  end
end
```

### Running Tests
```bash
# Install dependencies
bundle install

# Run all tests
bundle exec rake spec

# Run specific test
bundle exec rspec spec/classes/webapp_spec.rb

# Run with coverage
bundle exec rake spec:coverage

# Lint puppet code
bundle exec rake lint

# Validate syntax
bundle exec rake syntax

# Run all checks
bundle exec rake test
```

## Puppet Enterprise Features

### Console and RBAC
```puppet
# Configure node groups in PE Console
node_group { 'Web Servers':
  ensure => present,
  environment => 'production',
  parent => 'All Nodes',
  rule => ['and', ['~', 'name', '^web\d+']],
  classes => {
    'webapp' => {
      'version' => '2.1.0',
      'port'    => 8080,
    },
    'nginx' => {},
  },
  variables => {
    'webapp_environment' => 'production',
  },
}

# RBAC role definition
rbac_role { 'Web Admins':
  ensure => present,
  permissions => [
    {
      'object_type' => 'nodes',
      'action'      => 'edit_data',
      'instance'    => '*',
    },
    {
      'object_type' => 'puppet_agent',
      'action'      => 'run',
      'instance'    => '*',
    },
  ],
  users => ['webadmin@example.com'],
}
```

### Orchestration with Puppet Orchestrator
```bash
# Run Puppet on specific nodes
puppet job run --nodes web1.example.com,web2.example.com

# Run on node group
puppet job run --node-group "Web Servers"

# Application orchestration
puppet app show
puppet job run --application MyApp

# Task execution
puppet task run package action=status name=nginx --nodes web1.example.com

# Plan execution
puppet plan run mymodule::deploy_app version=2.1.0 --nodes web1.example.com
```

### Compliance and Reporting
```puppet
# Compliance profiles with InSpec integration
# profiles/linux_baseline/controls/ssh.rb
control 'ssh-config' do
  impact 1.0
  title 'SSH Configuration'
  desc 'SSH should be configured securely'
  
  describe file('/etc/ssh/sshd_config') do
    it { should exist }
    its('content') { should match /PasswordAuthentication no/ }
    its('content') { should match /PermitRootLogin no/ }
  end
  
  describe port(22) do
    it { should be_listening }
  end
end

# Apply compliance in manifests
class baseline::ssh {
  $ssh_config = {
    'PasswordAuthentication' => 'no',
    'PermitRootLogin'        => 'no',
    'MaxAuthTries'           => '3',
    'ClientAliveInterval'    => '300',
  }

  $ssh_config.each |$setting, $value| {
    file_line { "ssh_config_${setting}":
      path  => '/etc/ssh/sshd_config',
      line  => "${setting} ${value}",
      match => "^${setting}",
      notify => Service['ssh'],
    }
  }
}
```

## Performance and Scaling

### Catalog Compilation Optimization
```puppet
# site.pp optimization
# Use stages for ordering
stage { 'pre':
  before => Stage['main'],
}

stage { 'post':
  require => Stage['main'],
}

# Avoid virtual resources when possible
class nginx {
  if $facts['virtual'] != 'docker' {
    package { 'nginx':
      ensure => installed,
    }
  }
}

# Use resource defaults
Package {
  provider => 'apt',
}

File {
  owner => 'root',
  group => 'root',
  mode  => '0644',
}

# Optimize fact collection
# puppet.conf
[agent]
pluginsync = true
fact_terminus = facter
factsync = false

# Disable unnecessary facts
[main]
stringify_facts = false
```

### PuppetDB Optimization
```ini
# puppetdb.conf
[database]
classname = org.postgresql.Driver
subprotocol = postgresql
subname = //localhost:5432/puppetdb
username = puppetdb
password = puppetdb

# Performance tuning
[puppetdb]
node-ttl = 7d
node-purge-ttl = 14d
report-ttl = 14d

[jetty]
max-threads = 200
```

## Best Practices

### Module Structure
```
webapp/
├── manifests/
│   ├── init.pp              # Main class
│   ├── install.pp           # Installation logic
│   ├── config.pp            # Configuration
│   ├── service.pp           # Service management
│   └── params.pp            # Parameters
├── templates/
│   ├── app.conf.erb
│   └── systemd.service.erb
├── files/
│   └── scripts/
├── lib/
│   ├── facter/              # Custom facts
│   ├── puppet/type/         # Custom types
│   └── puppet/provider/     # Custom providers
├── spec/                    # Tests
├── examples/                # Usage examples
├── metadata.json            # Module metadata
└── README.md
```

### Code Quality
```puppet
# Use proper resource relationships
class webapp {
  package { 'webapp':
    ensure => installed,
  } ->
  
  file { '/etc/webapp/config.yaml':
    ensure  => file,
    content => template('webapp/config.yaml.erb'),
  } ~>
  
  service { 'webapp':
    ensure => running,
    enable => true,
  }
}

# Validate parameters
class webapp (
  String $version,
  Integer[1, 65535] $port = 8080,
  Enum['running', 'stopped'] $ensure = 'running',
) {
  # Class implementation
}

# Use proper variable scoping
class webapp::config {
  $config_dir = $webapp::config_dir
  $user = $webapp::user
  
  file { "${config_dir}/app.conf":
    ensure  => file,
    owner   => $user,
    content => template('webapp/app.conf.erb'),
  }
}
```

### Security Best Practices
- Use Hiera for sensitive data with eyaml
- Implement proper file permissions
- Regular security updates
- Audit and compliance monitoring
- Certificate management

## Resources
- [Puppet Documentation](https://puppet.com/docs/)
- [Puppet Forge](https://forge.puppet.com/)
- [rspec-puppet Documentation](http://rspec-puppet.com/)
- [Puppet Style Guide](https://puppet.com/docs/puppet/latest/style_guide.html)