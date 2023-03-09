# check_nagios_host_discovery
nagios check to notify of IP addresses / hostnames that are not yet defined as nagios hosts

# USAGE

Create a section similar to the following in the services.cfg file on the nagios server:
```
   # run a ping scan for new hosts that have not yet been added to nagios monitoring
   define service{
           use                             generic-service
           host_name                       mynagioshost.example.com
           service_description             nagios host discovery
           check_command                   check_nagios_host_discovery
           notification_period             8x5                 ; only alert Mon-Fri 8am-5pm
           check_interval                  720                 ; only check once every 720 minutes
           }
```

Create a section similar to the following in the commands.cfg file on the nagios server:
```
     # ---------------------------------------------------------------------------
     # 'check_nagios_host_discovery' command definition
     define command{
             command_name    check_nagios_host_discovery
             command_line    $USER1$/check_nagios_host_discovery
             }
```


# OUTPUT

You will see output similar to the following:
```
nagios host discovery WARN - the following IP addresses were detected, but do not have corresponding nagios host definitions. Please add these hosts to nagios monitoring: 10.99.99.99 MyNewHost.example.com, 10.55.55.55 IPaddr_not_in_DNS,  
```

```
nagios host discovery OK - all detected IP addresses are already defined as nagios hosts
```

