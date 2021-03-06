# Sample code for basic puppet tasks

_Note : Remember, we define a_ **state** _in puppet. We do not tell Puppet, how to get to that state._

For eg.

We would say "Hey Puppet, ensure that package 'apache' is installed." This is a **state**.

Now Puppet translates it to relevant commands based on the system it has to run on. So on a RHEL-based system, Puppet would run **something like** a ```yum list installed | grep apache``` and if it is not installed then do something like ```yum install apache```.

Imp thing to note is, Puppet doesn't exactly know what "apache" is. All it knows is the desired state for a node i.e. in this case, "the node must have a package called apache". 

Puppet's job is to intelligently translate that **state definition** to a system level command and give you the result of the command execution. So, if the actual package name for Apache is "apache2", **you** will have to tell that to Puppet.

## [File resource example](https://docs.puppet.com/puppet/4.7/types/file.html)

### Create a file
  
  ```
  file { '/tmp/testfile' :
  ensure => file,  
  }
  
  # Checks if file '/tmp/testfile' exists else creates one.
  # Values "ensure => present" does the same. "directory" will create a directory. "absent" will delete.
  ``` 

  - Examples with more options
  
  ```
  file { 'random_data_file' :  
    path   => '/tmp/random_filename_with_very_long_name',
    ensure => file,
    owner  => 'root',
    group  => 'root',
    mode   => '0750',
    content => "Hello world",
  }
  
  # The 'random_data_file' is called the title. "path", "ensure" etc are called attributes.
  # 'path' attribute can be used instead of writing the long name in title.
  ```
  
  ```
  file { '/tmp/symlink_to_hostsfile':
    ensure => 'link',  
    target => '/etc/hosts',
  }
  
  # creates a symlink to /etc/hosts.
  ```

  ```
  file { '/tmp/special_msg_file.txt':
    ensure => file , 
    source => 'puppet://modules/<your module name>/hello_world.txt', 
    }
    
  # your module will have a file named "hello_world.txt" under 
      # /etc/puppetlabs/code/environments/production/modules/<your module name>/files/. 
  # This code will be in 
      # /etc/puppetlabs/code/environments/production/modules/<your module name>/manifests/<filename>.pp
  # The above code will ensure "/tmp/special_msg_file.txt" exists with default file permissions 
      # and content same as "hello_world.txt".
  ```



## [Package resource example](https://docs.puppet.com/puppet/4.7/types/package.html)

### Install a package
  
  ```
  package { 'emacs' :
  ensure => installed, 
  }
  
  # Checks if the package 'emacs' is installed, else installs it.
  # Values "latest" will install ensure the latest version is installed. 
  # "absent" will remove the package. "purged" will remove pkg and config files
  ```
  - Installing multiple packages with more options
  
  ```
  $pkg_list = [ 'bash', 'curl' ]
  
  package { $pkg_list :
  ensure => installed,
  install_options => ['--enablerepo', 'my_repo'], 
  }

  # sends an array of package list in the title.
  # you can send special options to be added to the command while running.
  ```

## [Service resource example](https://docs.puppet.com/puppet/4.7/types/service.html)

### Start a service

  ```
  service { 'httpd' :
  ensure => running,  
  enable => true,     
  }
  
  # "ensure => stopped" will stop the process.
  # Optional attribute "enable => true" is to enable service startup during boot. Put "false" to NOT start at boot.
  ```

  - Restart a service everytime a particular file changes
  
 
  ```
  service { 'apache service' :
  name => $::osfamily?       
        {
         'Debian' => 'apache2',
         'RedHat' => 'httpd',
         default => 'apache',
         }
  ensure => running,
  enable => true,
  subscribe => File['apache_config_file'], 
  } 
 
  # The "name attribute has a condition to set service name based on type of OS. "osfamily" is fact. 
  # you can do a "facter osfmaily" on the node to see what its value is for each node.     
  
  # This service "subscribes" to a File resource called "apache_config_file". 
  # Meaning the Service will restart everytime the file resource "apache_config_file" changes.
  
  # You will be defining this "apache_config_file" file resource somwhere in the manifest. 
  # Typically that file resource will set the right configuration of apache.
  ```



## [User resource example](https://docs.puppet.com/puppet/4.7/types/user.html)/[Group resource example](https://docs.puppet.com/puppet/4.7/types/group.html)

### Create a user
  
  ```
  user { 'foo':
  ensure     => 'present',  
  }
  
  # Value "absent" deletes the user.
  ```
  
  - Example with more options
  
  ```
  user { 'System admin':
  name    => 'michealj', 
  ensure  => present, 
  home    => '/data/michealj', 
  gid     => '1003', 
  groups  => ['sysadmins','worldsavers'],
  shell   => '/bin/zsh', # default shell.
  comment => 'The rock star user to manage our systems.', 
  }
  
  # You can use "name" to mention the actual system username instead of the title. So, title can be more relevant name.
  # "gid" sets its primary group.
  ```

### Create a group


```
group { 'research':
    ensure => 'present',
    gid    => 3001,
  }
  
```



## [A Puppet Class example](https://docs.puppet.com/puppet/4.7/lang_classes.html)

A class in Puppet is just a collection of resources ( file, package, service etc. ) There are two steps to using a class in Puppet : 
  - Defining a class and 
  - Declaring it ( i.e. instantiating it ).

### Defining a class

```
class example_class {

# resource definitions here


}
```

### Declaring a class

You can call the class in various ways. You would do that in code as follows :
```
include example_class
```
OR you can declare it like a resource -

```
class { 'example_class' :
}
```

_Note : When you attach a class to a Node Group in the PE console, you are actually instantiating the class for each of the nodes in the group._

### Important note about puppet classes

In puppet, classes are singleton. Meaning they can be declared/instantiated only once in a catalog. 


---
_inspired by www.puppetcookbook.com_
