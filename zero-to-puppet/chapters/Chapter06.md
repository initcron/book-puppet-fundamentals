# Templates

## Converting tomcat.conf into a template 

We always start with the original file (e.g. `https://gist.github.com/initcron/01f8554fba3305a1bceee9df5ff0aa24`)

Create file structure to store the templates

  * Create a director **modules/manifests/templates**
  * Create a ERB file for tomcat.conf at  **modules/manifests/templates/tomcat.conf.erb**
  * Copy over the contents of the original tomcat.conf. You could use the link above as a sample content 
  
What could go wrong ?

  * You may have named **templates** directory as **template** Note the singular vs plural. It should be plural, with **s**.
  * There is also a possibility that you have a typo in the name. 
  * You may have created templates directory at a wrong path. It MUST be at `...modules/tomcat/templates`
  
  
Now lets start converting the file into a template. How do we do that? 
  * Keep the property names intact.  e.g. JAVA_HOME or TOMCAT_USER
  * Start replacing the values, with template variables. This is where ERB tags are handy. e.g. 
  
Original content such as this

```
TOMCAT_USER="tomcat"                                                                                                            
SHUTDOWN_WAIT="30"                                                             
```

becomes

```
TOMCAT_USER="<%= @user >"                                                                                                            
SHUTDOWN_WAIT="<%= @shutdown_wait >"
```

  * The template variables which start with @var need to be defined as well. Do that in **tomcat::params**. We already have user defined, lets define the shutdown_wait. What value do we define here?  Well params.pp contains the sane default. So whatever came with the package/configs, and was present in **tomcat.conf** by default is what we use.
e.g. 

```
  $shutdown_wait = 30

```

**TIP** : Its always useful to define the params first in params.pp before replacing the values in the template with ERB tags. That way, you would not have to go figure what were the default values, as once replaces, those values are gone. Its also easir to just copy over the params back as template variables.


## Exercise

Update the following properties in **tomcat.conf** in the template, and define the defaults in `params.pp`

* TOMCAT_CFG_LOADED
* JAVA_HOME
* xms
* xmx
* maxpermsize
* CATALINA_BASE
* JASPER_HOME
* CATALINA_TMPDIR
* SECURITY_MANAGER
* SHUTDOWN_VERBOSE
* CATALINA_PID

## References :

* tomcat.conf.erb   : https://gist.github.com/initcron/71e939a22e018dbc27dd4ae7deb3a55e
* tomcat::params    : https://gist.github.com/initcron/b08de87a1e835852ac00772252a71a97
  

## Calling templates from config class 

Creating templates and params does not have an effect on the system unless you start generating the configs using templates. Currently we are using a file resource to generate this file. Lets convert that to using a template. 

In **tomcat::config** class, update property of the file resource from

```
source    => 'puppet:///modules/tomcat/tomcat.conf',

```

to

```
content  => template('tomcat/tomcat.conf.erb'),

```

Here is the code after making that change for your reference  https://gist.github.com/initcron/87d99703590825716bed515f7e8bf64f
