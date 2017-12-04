# Parameters and Facts


### Playing with Scopes

app.pp

```


$color = 'blue'
$car   = 'maruti'

node 'node1' {

  include tomcat

}


node 'node2' {

   $color = 'green'
   include tomcat

}
```

Create tomcat::scope class

```
class tomcat::scope {

   notify{"print the scope":
     message => "

       FAVOURITE COLOR :  ${color}
       FAVOURITE CAR   :  ${car}

    "
  }

}
```

Add this class to init.pp

```
class tomcat {

  include tomcat::scope
  include tomcat::install
  include tomcat::config
  include tomcat::service
}
```

Apply on both node1 and node2 and see the difference?  


Question: Why does it print two different values?



### Inheritance and story of params.pp

create file: modules/tomcat/params.pp

```
class tomcat::params {

  $color = 'white'
  $car   = 'figo'

}


```


Update tomcat::scope definition to inherit tomcat::params. Lets also define $color in this child class.  

```
class tomcat::scope inherits tomcat::params{

   $color = 'yellow'


   notify{"print the scope":
     message => "

       FAVOURITE COLOR :  ${color}
       FAVOURITE CAR   :  ${car}

    "
  }

}
```


#### Parameterising Tomcat Configs

tomcat::params

```
class tomcat::params {

  $color = 'white'
  $car   = 'figo'
  $user  = 'tomcat'
  $group = 'tomcat'
  $config_path  = '/etc/tomcat/tomcat.conf'
  $packages  = [ 'tomcat', 'tomcat-webapps' ]
  $service_name = 'tomcat'
  $service_state = running

}

```

tomcat (init.pp)

```
class tomcat inherits tomcat::params{

  include tomcat::scope
  include tomcat::install
  include tomcat::config
  include tomcat::service
}
```

class tomcat::install

```
class tomcat::install inherits tomcat{

    include java

    package { $::tomcat::packages:
      ensure   => installed,
      require  => Package['epel-release']
    }

}

```

class tomcat::config

```
class tomcat::config inherits tomcat{

  file { $::tomcat::config_path:
    source    => 'puppet:///modules/tomcat/tomcat.conf',
    owner    => $::tomcat::user,
    group    => $::tomcat::group,
    mode     => '0644',
    notify   => Service['tomcat']
  }

}

```

class tomcat::service

```
class tomcat::service inherits tomcat{

   service { $::tomcat::service_name:
     ensure   => $::tomcat::service_state,
     enable   => true,
     require  => Class['tomcat::install'],
   }
}

```


### Creating base module

```
cd /workspace/code/environments/production/modules

puppet module generate --skip-interview user-base

mv /workspace/base.pp base/manifests/init.pp

```

Add class to modules/base/manifests/init.pp. Code for this class can be availed from
https://gist.github.com/initcron/dd302fe9fabcc9a1af071394bf8e8d6a


Add the base class to one of the nodes, and see if you could apply

e.g. for **node2** definition in app.pp
```

node 'node2' {

   include base

   $color = 'green'
   include tomcat

}
```

**Question** :  What happens when you apply this class on node2 by running puppet agent ??

Lets use params.pp for doing platform specific configurations,

```
class base::params {

  case $::os['family'] {
    'Debian': {
      $ntp_service = 'ntp'
    }
    'RedHat': {
      $ntp_service = 'ntpd'
    }
  }


}
```
And to use these params, we need to
  * inherit base::params in base class (init.pp)
  * parameterize the service name

Update init.pp with the above two changes. Here is the updated file for your reference https://gist.github.com/initcron/c8b6530dbae41c47c0d864d2a594844e


## Using Facts

Lets also start using facts inside modules. To do this, lets create a file resource which generates /etc/motd and prints information about the system

Update modules/base/manifests/init.pp and add the following resource.

```
    file { '/etc/motd':
      ensure   => file,
      owner    => 'root',
      content  => "

         This server is a property of XYZ Inc.

         SYSTEM INFO
         ============

         Hostname     : ${::fqdn}
         IP Address   : ${::ipaddress}
         Memory       : ${::memory['system']['total']}
         Cores        : ${::processors['count']}
         OS           : ${::os['distro']['description']}


      "

    }
```

Apply on all nodes and validate.
