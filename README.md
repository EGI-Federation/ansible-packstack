# ansible-packstack

Deploying a [packstack-based OpenStack](https://www.rdoproject.org/install/packstack/) test instance using Ansible.
Some tools specific to [EGI Federated Cloud](https://wiki.egi.eu/wiki/EGI_Federated_Cloud) will also be installed:
* [Keystone VOMS authentication](https://github.com/IFCA/keystone-voms)
* [OOI](https://github.com/openstack/ooi/) for OCCI support

OpenStack client configuration file will be created as
`~/.config/openstack/clouds.yaml`, pre-existing file will be backed-up.

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
  * an ssh access with a user having password-less sudo
  * a public IP registered with a FQDN
  * a certificate with its key and the CA

The ansible user is packstack by default, specify another one using `-u`.
```sh
# Configure Ansible remote host
cp inventory/inventory.ini.sample inventory/inventory.ini
vim inventory/inventory.ini
# Install usual CLI tools
ansible-playbook playbooks/weapons.yaml
# Install and run Packstack, configure HTTPS for Horizon and Keystone
ansible-playbook playbooks/packstack.yaml
# Once this is done it's recommended to reboot the server
# Create default OpenStack projects for FedCloud
ansible-playbook playbooks/projects.yaml
# Enable IGTF CA
ansible-playbook playbooks/igtf.yaml
# Enable Keystone VOMS support
ansible-playbook playbooks/keystone_voms.yaml
# Install OOI for OCCI endpoint
ansible-playbook playbooks/ooi.yaml
```

## Testing

OpenStack client can use `~/.config/openstack/clouds.yaml` that was created by
`packstack.yaml`:

```sh
openstack --os-cloud server_fqdn image list
```

It can be easy to test using a [docker wrapper](https://github.com/gbraad/dockerfile-openstack-client).

Copy `/root/keystonerc_admin` configuration to a site-specific file in `~/.stack`:

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
# Using configuration in ~/.config/openstack/clouds.yaml
docker run -it --rm -v ~/.config/openstack:/root/.config/openstack gbraad/openstack-client:centos openstack --os-cloud server_fqdn image list
# Using configuration in ~/.stack
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
