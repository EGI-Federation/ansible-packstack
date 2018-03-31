# ansible-packstack

Deploying a [packstack-based OpenStack](https://www.rdoproject.org/install/packstack/) test instance using Ansible.
Some tools specific to [EGI Federated Cloud](https://wiki.egi.eu/wiki/EGI_Federated_Cloud) will also be installed:
* [Keystone VOMS authentication](https://github.com/IFCA/keystone-voms)
* [OOI](https://github.com/openstack/ooi/) for OCCI support

Based on:

  * Packstack and OpenStack
    * https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/
    * https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/2/html/Getting_Started_Guide/part-Deploying_OS_using_PackStack.html
    * https://docs.openstack.org/security-guide/secure-communication.html
    * https://www.tecmint.com/openstack-installation-guide-rhel-centos/
    * https://linuxacademy.com/howtoguides/posts/show/topic/12453-deploying-openstack-rdo-allinone-vm-for-multidomain-support
  * Federated Cloud (Keystone-VOMS, OOI)
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
# Configure Ansible remote host
cp inventory.ini.sample inventory.ini
vim inventory.ini
# Install usual CLI tools
ansible-playbook weapons.yaml -i inventory.ini -u $(whoami)
# Install and run Packstack, configure HTTPS for Horizon and Keystone
ansible-playbook packstack.yaml -i inventory.ini -u $(whoami)
# Once this is done it's recommended to reboot the server
# Create default OpenStack projects
ansible-playbook projects.yaml -i inventory.ini -u $(whoami)
# Enable Keystone VOMS support
ansible-playbook keystone_voms.yaml -i inventory.ini -u $(whoami)
# Install OOI for OCCI endpoint
ansible-playbook ooi.yaml -i inventory.ini -u $(whoami)
```

## Testing

It can be easy to test using a [docker wrapper](https://github.com/gbraad/dockerfile-openstack-client).

Copy `/root/keystonerc_admin` or `/root/keystonerc_demo` configuration to a
site-specific file in `~/.stack` directory

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
# Listing available images
docker run -it --rm -v ~/.stack:/root/.stack gbraad/openstack-client:centos stack my_site openstack image list
# Retrieving a Keystone token
OS_TOKEN=$(docker run -it --rm -v ~/.stack:/root/.stack gbraad/openstack-client:centos stack my_site openstack token issue -f value -c id)
# Testing OOI/OCCI endpoint
curl -H "x-auth-token: $OS_TOKEN" 'http://XXX.XXX.XXX.XXX:8787/occi1.1/-/'
```

## Troubleshooting

When running the `packstack.yaml` playbook In case of problems during the
system update after the OpenStack repository configuration you may have errors
related to updating python-urllib3 and to files unknown to the RPM database:

```sh
(...)
Error unpacking rpm package python2-urllib3-1.16-1.el7.noarch
error: unpacking of archive failed on file /usr/lib/python2.7/site-packages/urllib3/packages/ssl_match_hostname: cpio: rename
(...)
% ls /usr/lib/python2.7/site-packages/urllib3/packages/
backports    ordered_dict.py  six.pyc  ssl_match_hostname           ssl_match_hostname;5aba127b  ssl_match_hostname;5aba12e7
__init__.py  six.py           six.pyo  ssl_match_hostname;5aba09c7  ssl_match_hostname;5aba12a0
```

In that case it's possible to clear the files and relaunch the update:

```sh
sudo yum remove -y python-urllib3
sudo rm -rf /usr/lib/python2.7/site-packages/urllib3/packages
sudo yum install -y python-urllib3
```
