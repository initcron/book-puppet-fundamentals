# Deploy Sysfoo to Application Servers

### Create a defined type to deploy a warfile


file: modules/tomcat/manifests/deploy.pp
```
define tomcat::deploy(
   $deploy_url,
   $checksum_value,
   $checksum    =  'md5',
   $deploy_path =  $::tomcat::deploy_path
) {

   file { "${deploy_path}/${name}.war" :
      source          => "${deploy_url}",
      owner           => $::tomcat::user,
      group           => $::tomcat::group,
      checksum_value  => "${checksum_value}",
      checksum        => "${checksum}",
      notify          => Exec['purge_context'],
   }

   exec { 'purge_context':
     command       => "rm -rf  ${deploy_path}/${name}",
     path          => '/usr/bin:/usr/sbin:/bin',
     refreshonly   => true,
     notify        => Service['tomcat'],
   }
}
```

### Define relevant params

file: modules/tomcat/manifests/params.pp

```
  $deploy_path  = '/var/lib/tomcat/webapps'
```


### Use the defined type

file: code/environments/production/manifests/app.pp

```
  tomcat::deploy { "sysfoo":
    deploy_url      => 'https://6-94848332-gh.circle-artifacts.com/0/tmp/circle-artifacts.3grfYBu/sysfoo.war',
    checksum_value  => 'e1611c2f62b5c01e7f620a19a73446ea',
    checksum        => 'md5'
  }
```
