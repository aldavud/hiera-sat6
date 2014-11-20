# Using Hiera with Satellite 6

## Installation and configuration

Hiera does not need to be installed manually. Since Puppet 3.0 Hiera is shipped as a part of Puppet and thus automatically available on a fresh Satellite 6 install.

When using Hiera via Puppet the default config file location for hiera is $confdir/hiera.yaml on the Puppet capsule server. By default this translates to /etc/puppet/hiera.yaml, if you want to use a different file edit the [hiera_config](https://docs.puppetlabs.com/references/latest/configuration.html#hieraconfig) in /etc/puppet/puppet.conf.

Please note that in a scenario with multiple Puppet capsules, the Hiera configuration and hierarchy needs to be replicated across all Puppet capsules.

Here is an example Hiera config for use with Satellite 6:
```
---
:backends:
- yaml
:hierarchy:
- "hostname/%{::fqdn}"
- "environment/%{::kt_org}/%{::kt_env}"
- "hostgroup/%{::kt_org}/%{::hostgroup}"
- common
:yaml:
:datadir: /var/lib/hiera
```

This translates to the following file hierarchy being considered

1. /var/lib/hiera/hostname/<fqdn>.yaml
2. /var/lib/hiera/environment/<Satellite 6 Organization>/<Satellite 6 Lifecycle Environment>.yaml
3. /var/lib/hiera/hostgroup/<Satellite 6 Organization>/<Satellite 6 Host Group>.yaml (this automatically tranlates nested hostgroups to filesystem paths, so the hostgroup Prod/JBoss gets translated to /var/lib/hiera/hostgroup/Example_Org/Prod/JBoss.yaml, given the current organization is Example_Org)
4. /var/lib/hiera/common.yaml

Regarding the _"hostgroup/%{::kt_org}/%{::hostgroup}"_ hierarchy above it is important to note that hostgroups along the path will not be considered. In the example above this would mean that the file _/var/lib/hiera/hostgroup/Example_Org/Prod.yaml_ would not be considered for hosts in the _Prod/JBoss_ hostgroup. It would be only considered by hosts which are directly in the _Prod_ hostgroup.

Remember to restart the Puppet master process after configuring hiera! As Passenger is used this can be done as follows:
```
service httpd restart # for RHEL6

systemctl restart httpd.service # for RHEL7
```

