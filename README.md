# ansible-openshift_origin

Author: Adam Miller

#NOTE - THIS IS A WORK IN PROGRESS, THINK "ALPHA" QUALITY.

# About

This playbook helps install [OpenShift Origin](https://openshift.redhat.com/community/open-source) Platform As A Service.

Currently this playbook only supports installing on a single machine or 
(single-node install) which that the OpenShift Broker and OpenShift Node 
services are running on the same machine. This can be done on bare metal, in a 
virtual machine, or in a cloud instance as OpenShift only depends upon the 
Operating System (with SELinux, cgroups, and PAM magic under the hood).

# Requirements

* Fedora 18 - at least @standard if using a kickstart installation.
* Ansible >= 0.9

# Installation

The module can be obtained from the
[github repository](https://github.com/maxamillion/ansible-openshift_origin).

1. Download the [Zip file from github](https://github.com/maxamillion/ansible-openshift_origin/archive/master.zip)

(See "Configuration" and "Using" sections below for details)
2. For broker installs run 'ansible-playbook broker.yaml' ('brokers' is
a host group)
3. For node installs run 'ansible-playbook node.yaml' ('nodes' is a host 
group)


Reminder: This is currently only supporting a single-node install so 'brokers' 
and 'nodes' should both contain the same, single, ip address or hostname in the
inventory file.

# Configuration

## Ansible
You will either need to make entries in your /etc/ansible/hosts file for a 
[brokers] and a [nodes] section, or optionally create a hosts file and use the 
ansible -i option to point to your custom inventory file.

NOTE: This is currently only for single node deployments

Example:

    [brokers]
    192.168.1.100

    [nodes]
    192.168.1.100


# Using 

Once the configuration step is complete you can then run these configs with or
without a non-default inventory file.

Example with default inventory file (/etc/ansible/hosts):
    
    ansible-playbook broker.yml

    ansible-playbook node.yml

Example with non-default inventory file (/tmp/myhosts):
    
    ansible-playbook broker.yml -i /tmp/myhosts

    ansible-playbook node.yml -i /tmp/myhosts

# Web Console

Once the installation is complete, navigate to your machine (we'll assume here 
that the DNS pointer is broker.example.com) to https://broker.example.com/console
and the default username/password is admin/admin .... you SHOULD CHANGE THIS 
IMMEDIATELY if you plan to do anything of any amount of seriousness with your 
deployment.

# rhc - OpenShift command line util
If you are going to use the rhc client you will need to create a config,
there is a config example called express.conf.example in the docs dir of this
repo

You can also use the rhc wizard by pointing it to your broker node.

The password for the command line utility will be the same as the web console.

Example:

    ## NOTE: Need the -k because we're using a self signed cert in our example.
    rhc -k setup --server=192.168.1.100


# Notes

Just some fun little "gotchyas" that are being worked through, but in the mean
time warrant some amount of mention. This section will be updated as necessary.

0. This is a pet project that I've been fortunate enough to carve off a little 
time to work on during my $dayjob because my job, boss, and company are awesome.
As with any pet project, it might get stale but I will do my best to keep it 
up to date as well as expand on it as time goes on to handle more sophisticated
configrations and deployments.

1. Sometimes ActiveMQ doesn't start, no idea why but it just doesn't. SystemD 
and Ansible return that it's started but it hasn't, I only notice this on first
run deployment. If you run notice any odd issues with the broker, check 
'systemctl status activemq.service' and verify it's running.

2. Strange DNS(dnsruby) error that is intermittent. It's known to be
happening but the root cause is still being traked down. If you get this error, 
try the operation you were attempting again and it should succeed, if not and 
you're getting an decent amount of information in 
/var/log/openshift/broker/production.log please feel free to open an issue on 
github or contact me, info below. Error will look similar to the following:


    Cartridge dnsruby can't connect to 192.168.59.162:53 from 0.0.0.0:61670, 
    use_tcp=false, exception = Errno::EACCES, Permission denied - bind(2)


UPDATE: 2013-04-01
    This issue was previously thought to be closely related to a dbus event but
it appears to pop up in other scenarios where that dbus event is not present in
the logs. The root cause is still unknown, but it looks as though it is related
to using dhcp on the all-in-one broker/node. Still investigating as I find time.

UPDATE2: 2013-04-01
    Looks like it's related to running on a machine that recieved it's ip 
address via NetworkManager. Not full confirmed but I've set a static ip using 
the "classic" network scripts in /etc/sysconfig/network-scripts/ and have not 
run into this problem at all (yet). Originally thought it might be dhcp, but the
error occurred with NetworkManager assigning a static ip.


# Contact Info

If you'd like, just open an issue against this on github and I'll get to is asap.

If you'd like to try for more immediate feedback, feel free to ping me on irc. I
only frequent freenode and am in more channels than is healthy, but below are 
the ones I spend the most time in and/or paying attention to:
- IRC Nick: maxamillion
- IRC Channels: #openshift-dev #openshift #ansible #fedora #fedora-devel #rhel 
- IRC Network: irc.freenode.net
