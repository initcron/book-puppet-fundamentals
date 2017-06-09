# Modules

## Update code dir to use your current workspace 

On puppet master

```
puppet
cp -r /etc/puppetlabs/code /workspace/code    
```

Update config in the server's config  file **/etc/puppetlabs/puppetserver/conf.d/puppetserver.conf** and update 
 
```
master-code-dir: /etc/puppetlabs/code
```

to 

```
master-code-dir: /workspace/code
```
(assuming you are using /workspace as your working/developement directory on the puppet master to store the code at )

Restart Puppet Server 

```
shell
service puppetserver restart  
```

This may take time. 

## Generating Modules 

Change into the directory for production env 

```
sh
cd /workspace/code/environments/production/modules
```

Using puppet module command, generate the scaffolding for **java** and **tomcat** modules

```
bash

puppet module generate --skip-interview user-java

puppet module generate --skip-interview  gshah-tomcat
```

## Writing classes 


file: `modules/java/manifests/init.pp`

```
class java {

  package { [ 'epel-release', 'java-1.7.0-openjdk'] : 
    ensure => installed,
  }

}
```

## Node Definitions - Applying the code

To apply the default class from java module, create a node definition 

create file: `environments/production/manifests/app.pp`

add the node definition 

```
node 'node1' {

   include java

}
```


To apply, login to **node1** and run puppet agent 

```
ssh devops@node1

sudo su

puppet agent -t 

```

### Writing the class to install tomcat 

file: `modules/tomcat/manifests/install.pp`

```
class tomcat::install {
    
    include java
    
    package { [ 'tomcat', 'tomcat-webapps' ]:
      ensure   => installed, 
      require  => Package['epel-release']
    }

}


```

Add `tomcat::install` to node definition for node1 which should now look like 

```
node 'node1' {

   include java
   include tomcat::install

}
```

Apply on node1 by running puppet agent 


```
[on node1, as root]

puppet agent -t 

```

##  Nano Project 

Now that you have learnt how to write a manifest and apply, and have gone through the class naming conventions, you have been tasked to create  the following classe with the specifications as give below, 

  * class :  tomcat::service 
      resource : service 
      name : tomcat 
      ensure : running 
      enable : true 
      
  * service resource should have a dependency on install class

  * apply it to node1 by updating the node definition.  Validate by visiting the IP address of the server and port 8081. 
  
  
Note : Tomcat may take up to 10mins to come up for the first time. This is discussed in details on this page https://wiki.apache.org/tomcat/HowTo/FasterStartUp at the Entropy Source section. We are going to apply that fix in the subsequenct sections. 
 
 
Also bootstrap node2 ( configure it to talk to the master), create a node definition identical to node1, and apply. Validate that the tomcat service is started on port 8082. 



## Simplify Run List 

file: `modules/tomcat/manifests/init.pp`

```
class tomcat {

  include tomcat::install
  include tomcat::service 

}

```

And `production/manifests/app.pp`

```

node 'node1' {

  include tomcat
  
}


node 'node2' {

   include tomcat

}

```
**Question** : Why are we not including java anymore? 


## Managing Files

  * Create file 
    * create directory to hold files : modules/tomcat/files
    * create a file modules/tomcat/files/tomcat.conf and add the content from https://gist.github.com/initcron/01f8554fba3305a1bceee9df5ff0aa24
  
  * Create a class tomcat::config to copy this file to destinition 


tomcat::config class 

```
class tomcat::config {

  file { '/etc/tomcat/tomcat.conf':
    source    => 'puppet:///modules/tomcat/tomcat.conf',
    owner    => 'tomcat', 
    group    => 'tomcat', 
    mode     => '0644',
    notify   => Service['tomcat'] 
  }

}

```


Call it from tomcat class (init.pp)

```
class tomcat {

  include tomcat::install
  include tomcat::config
  include tomcat::service 
}
```


Apply on Node1 and Node2