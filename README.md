# check_nagios_host_discovery
nagios check to notify of IP addresses / hostnames that are not yet defined as nagios hosts

# Usage

Copy the ```check_nagios_host_discovery``` and ```check_nagios_host_discovery.cfg``` files to the ```/usr/local/nagios/libexec/``` folder on the nagios host.

Edit the contents of the ```check_nagios_host_discovery.cfg``` file using the embedded comments as a guide.
```
# config file for nagios check
# subnets parameter is required, excluded_hosts parameter is optional
# subnets= parameter expects a comma-separated list of subnets in CIDR format or single IP address or range of IP addresses as expected by nmap
# excluded_hosts parameter is a comma-separated list of IP addresses and/or hostnames
# EXAMPLES:
# subnets=10.1.1.0/24,192.168.2.1-10,172.16.12.99
# excluded_hosts=10.10.30.1,myhost01,myhost02.example.com

subnets=
excluded_hosts=
```

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
Run this script as an hourly cron job, and save the results to a temporary file that will be queried for cached results.  HINT: nmap requires root privileges to perform ICMP scans, so you will need to add the following entry to the root crontab.
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


Question: What if I have working forward name resolution (A record) but do not have working reserve name resolution (PTR record)?

Answer: If reverse name resolution (PTR record) is not working, AND your nagios host definitions use an IP address rather than a hostname, you will be fine.  However, if reverse name resolution (PTR record) is not working, AND your nagios host definitions use hostnames rather than IP addresses, this script will perform an IP address scan, find a particular IP address as being pingable, but will have no way to determine if that IP address is associated with a nagios host definition, so will generate an alert.  Please address the root cause by fixing your broken name resolution.
