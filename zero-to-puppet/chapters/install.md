# Installation

### With Vagrant

```
# Import Vagrant Tempalte
# Open iterm/ git bash  and browse to the dir where the box file is copied. Run the following

# From puppet directory
$ vagrant box add puppet-template devops-training-centos-v2.box

# Validate
$ vagrant box list

puppet-template          (virtualbox, 0)


# To Bring up the VMs with Vagrant and Virtualbox
# Open iterm or git bash from virtual/multi directory
# Open 3 different windows and cd into virtual/multi dir in each
# bring up the VMs as

cd virtual/multi

$ vagrant up master
$ vagrant up web
#$ vagrant up db

# Open three different windows, one for master, one for web and one for db servers, and login
$ vagrant ssh master
$ vagrant ssh web
#$ vagrant ssh db

#On Puppet Master
$ sudo su
$ yum install puppetserver puppet-agent -y
$ service puppetserver start

# Validate
$ service puppetserver status


#On Puppet Agents : all nodes
$ sudo su
$ yum install puppet-agent
$ service puppet status
$ service puppet start

# Validate
$ service puppet status

# On Master, check for certificate request
# Outstanding
$ puppet cert list
# All
$ puppet cert list -a

# Sign
$ puppet cert sign -a

# On agents run puppet
puppet agent -t
```
