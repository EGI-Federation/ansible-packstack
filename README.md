# ansible-packstack

Deploying a packstack-based OpenStack test instance using Ansible.

Based on:
* https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/2/html/Getting_Started_Guide/part-Deploying_OS_using_PackStack.html
* https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/
* https://www.tecmint.com/openstack-installation-guide-rhel-centos/
* https://linuxacademy.com/howtoguides/posts/show/topic/12453-deploying-openstack-rdo-allinone-vm-for-multidomain-support

## Using

 * Prerequisites: a user having password-less sudo access (here the local user)

```sh
ansible-playbook packstack.yaml -i inventory.ini -u $(whoami)
```

## Testing

It can be easy to test using a docke wrapper.

* Copy `/root/keystonerc_admin` or `/root/keystonerc_demo` configuratio to a site-specific file in `~/.stack` directory

```sh
$ cat ~/.stack/my_site
export OS_USERNAME=admin
export OS_PASSWORD=XXXXXXXXXX
export OS_AUTH_URL=http://XXX.XXX.XXX.XXX:5000/v3
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