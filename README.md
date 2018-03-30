# ansible-packstack

Deploying a packstack-based OpenStack test instance using Ansible.

Based on:

  * Packstack setup
    * https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/2/html/Getting_Started_Guide/part-Deploying_OS_using_PackStack.html
    * https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/
    * https://www.tecmint.com/openstack-installation-guide-rhel-centos/
    * https://linuxacademy.com/howtoguides/posts/show/topic/12453-deploying-openstack-rdo-allinone-vm-for-multidomain-support
    * https://docs.openstack.org/security-guide/secure-communication.html
  * Federated Cloud (Keystone-VOMS, OOI) setup
    * https://wiki.egi.eu/wiki/Federated_Cloud_Ocata_guide
    * https://keystone-voms.readthedocs.io/en/stable-newton/configuration.html
    * http://ooi.readthedocs.io/en/stable/index.html
    * https://wiki.infn.it/progetti/cloud-areapd/egi_federated_cloud/newton-centos7_testbed

## Using

 * Prerequisites
  * a VM with CentOS 7
  * an ssh access with a user having password-less sudo (here the local user)
  * a public IP registered with a FQDN
  * a certificate with its key and CA files

```sh
# Install usual CLI tools
ansible-playbook weapons.yaml -i inventory.ini -u $(whoami)
# Install and Packstack
ansible-playbook packstack.yaml -i inventory.ini -u $(whoami)
# Enable Keystone VOMS support
ansible-playbook keystone_voms.yaml -i inventory.ini -u $(whoami)
# Install OOI for OCCI endpoint
ansible-playbook ooi.yaml -i inventory.ini -u $(whoami)
```

## Testing

It can be easy to test using a docke wrapper.

* Copy `/root/keystonerc_admin` or `/root/keystonerc_demo` configuratio to a site-specific file in `~/.stack` directory

```sh
$ cat ~/.stack/my_site
export OS_USERNAME=admin
export OS_PASSWORD=XXXXXXXXXX
export OS_AUTH_URL=https://XXX.XXX.XXX.XXX:5000/v3
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_IDENTITY_API_VERSION=3
```

```sh
docker run -it --rm -v ~/.stack:/root/.stack gbraad/openstack-client:centos stack my_site openstack image list
```

## Troubleshooting

In case of problems with installing/updating python-urllib3:

```sh
(...)
Error unpacking rpm package python2-urllib3-1.16-1.el7.noarch
error: unpacking of archive failed on file /usr/lib/python2.7/site-packages/urllib3/packages/ssl_match_hostname: cpio: rename
(...)

% ls /usr/lib/python2.7/site-packages/urllib3/packages/
backports    ordered_dict.py  six.pyc  ssl_match_hostname           ssl_match_hostname;5aba127b  ssl_match_hostname;5aba12e7
__init__.py  six.py           six.pyo  ssl_match_hostname;5aba09c7  ssl_match_hostname;5aba12a0
```

```sh
sudo yum remove -y python-urllib3
sudo rm -rf /usr/lib/python2.7/site-packages/urllib3/packages
sudo yum install -y python-urllib3
```
