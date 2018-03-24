# ansible-packstack

Deploying a packstack-based OpenStack test instance using Ansible.

Based on https://linuxacademy.com/howtoguides/posts/show/topic/12453-deploying-openstack-rdo-allinone-vm-for-multidomain-support

## Using

 * Prerequisites: a user having password-less sudo access (here the local user)

```sh
ansible-playbook packstack.yaml -i inventory.ini -u $(whoami)
```
