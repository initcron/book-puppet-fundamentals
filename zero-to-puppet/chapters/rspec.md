# Rspec

#### Set the Path

The following path need to be set in order to use rake, rspec etc.

```

export PATH=$PATH:/opt/puppetlabs/puppet/bin/

echo "export PATH=$PATH:/opt/puppetlabs/puppet/bin/" >> ~/.bashrc

source ~/.bashrc
```


### Install Required Tools


```
gem install --no-ri --no-rdoc  puppet-lint puppet-syntax puppetlabs_spec_helper
cd modules/java
rspec-puppet-init
gem install bundler
bundle install
```

**Validate**

```
rake help
rspec --help
```

### Smoke testing rspec and rake on Java module

  * Observe Rakefile in modules/java
  * Validate and run rspec tests

```
rake validate

rake spec

rake test

echo $?
```

metadata may need to be changed to

```
{
  "name": "gshah-java",
  "version": "0.1.0",
  "author": "gshah",
  "summary": "this module installs openjdk 7 on RedHat",
  "license": "Apache-2.0",
  "source": "",
  "project_page": null,
  "issues_url": null,
  "dependencies": [
    {"name":"puppetlabs-stdlib","version_requirement":"= 4.17.0"}
  ],
  "data_provider": null
}


```


## Writing first unit test spec for tomcat  


file: modules/tomcat/.fixtures.yml

```
fixtures:
  symlinks:
    tomcat: "#{source_dir}"
    java: "#{source_dir}/../java"

```

Smoke Test

```
rake validate
```

[fix issues if any until the above command is successful]



Create a simple unit test for class tomcat (init.pp)

file: modules/tomcat/specs/classes/init_spec.rb

```
require 'spec_helper'
describe 'tomcat' do
  context 'with default values for all parameters' do
    it { should contain_class('tomcat') }
  end

    it { should contain_class('tomcat::install') }
    it { should contain_class('tomcat::config') }
    it { should contain_class('tomcat::service') }


end
```

Run Tests
```
rake spec
```

### Creating specs for tomcat::config

file: specs/classes/config_spec.rb

```
require 'spec_helper'
describe 'tomcat::config' do

  it { should contain_class('tomcat::config') }
  it { is_expected.to compile }
  it { is_expected.to contain_file('/etc/tomcat/tomcat.conf').with({
        :mode    => '0644',
        :owner   => 'tomcat',
        :group   => 'tomcat',
      }).that_notifies('Service[tomcat]')



  }
end

```


#### Exercise

**Scanario 1**: Create a spec for service class which validates
  * if tomcat::service is defined
  * if it contains a service resource with
    * ensure value as running
    * enable set to true
    * requires on Class['tomcat::install']


**Scanario 2**: Create a spec for **base** class which validates
  * if ::base class is defined
  * if it contains a service resource for **ntp** with
    * ensure value as running
    * enable set to true
  * Given the osfamily
    * as RedHat, service class should expect **ntpd** is the service name
    * as Debian, should expect **ntp** as the service name


### Reading List

  * Unit Testing with Puppet
    https://puppet.com/blog/unit-testing-rspec-puppet-for-beginners

  * Next Generation of Puppet Module Testing
    https://puppet.com/blog/next-generation-of-puppet-module-testing

  * Rspec Matchers for Puppet
    http://rspec-puppet.com/matchers/

  * Puppet-rspec Tutorial
    http://rspec-puppet.com/tutorial/

  * Sample Unit Tests with Spec
    https://github.com/desc/puppet-reprepro/blob/master/spec/classes/init_spec.rb

  * http://terrarum.net/blog/puppet-testing-part-1.html

  * https://wikimatze.de/getting-started-with-rspec-puppet/
