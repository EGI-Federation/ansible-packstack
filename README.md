# ansible-packstack

Deploying a packstack-based OpenStack test instance using Ansible.

Based on https://linuxacademy.com/howtoguides/posts/show/topic/12453-deploying-openstack-rdo-allinone-vm-for-multidomain-support

## Using

 * Prerequisites: a user having password-less sudo access (here the local user)

```sh
ansible-playbook packstack.yaml -i inventory.ini -u $(whoami)
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
