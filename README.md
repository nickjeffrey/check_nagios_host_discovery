# check_nagios_host_discovery
nagios check to notify of IP addresses / hostnames that are not yet defined as nagios hosts

# Usage

Create a section similar to the following in the services.cfg file on the nagios server:
```
   # run a ping scan for new hosts that have not yet been added to nagios monitoring
   define service{
           use                             generic-service
           host_name                       mynagioshost.example.com
           service_description             nagios host discovery
           check_command                   check_nagios_host_discovery
           notification_period             8x5                 ; only alert Mon-Fri 8am-5pm
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

If you have a large network, it may take more than 60 seconds to perform the ping scan, which can cause the nagios check to time out.
Run this script as an hourly cron job, and save the results to a temporary file that will be queried for cached results.
```
5 * * * * /usr/local/nagios/libexec/check_nagios_host_discovery 1>/dev/null 2>/dev/null  #nagios helper script    
```

# Output

You will see output similar to the following:
```
nagios host discovery WARN - the following IP addresses were detected, but do not have corresponding nagios host definitions. Please add these hosts to nagios monitoring: 10.99.99.99 MyNewHost.example.com, 10.55.55.55 IPaddr_not_in_DNS,  
```

```
nagios host discovery OK - all detected IP addresses are already defined as nagios hosts
```

# Troubleshooting

You must have working forward and reverse name resolution, either via the /etc/hosts file on the nagios server, or (preferably) in DNS.

HINT: if you have working forward lookups (A records) but are missing the reverse lookups (PTR records), Microsoft Windows hosts may need these checkboxes selected.
<img src=images/dns_ptr.png>
