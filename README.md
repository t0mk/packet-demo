# Ansible for Packet training demo 02/06/20017

Video at https://www.youtube.com/watch?v=MF-qdMHLCow

Follows vaguely https://docs.ansible.com/ansible/guide_packet.html

## Pre

- Do we know what's Ansible?
- Overview of current status in shared Doc - merged and pending PRs.
- Maybe leave proposed features for the end?
- Do we want to do a demo? I prepared it anyway.


## Get Ansible

You can install ansible with pip or some other way, but I prefer just using the git repo.

```
$ git clone https://github.com/ansible/ansible ansible_devel
$ source ansible_devel/hacking/env-setup
```

The `source` command sets environment variables so that the given ansible tree is used by ansbile-* commands.

## Auth

Your API token must be in exported envvar PACKET_API_TOKEN.

## Device

Playbook to create 2 devices in [playbook_create.yml](playbook_create.yml). Go through it.

Run playbook

```
$ ansible-playbook playbook_create.yml
```

Show what's in my project with packet-cli.

Now the hosts are in packet. The dynamic inventory script lists them:

```
$ ansible_devel/contrib/inventory/packet_net.py --list --refresh-cache
```

The inventory also groups the devices by attributes, e.g. by operating systems. Let's install nginx on "all the ubuntu_16_06 device" (i.e. both of the created devices).

A simple playbook installing nginx on all the "ubuntu_16_06" devices is in [playbook_install_nginx.yml](playbook_install_nginx.yml).

We supply the dynamic inv with the `-i` flag of ansible-playbook.


```
# When the devices are already in => active <=

$ ansible-playbook -u root -i ansible_devel/contrib/inventory/packet_net.py playbook_install_nginx.yml

# if ssh doesn't work, try $ ssh-keygen -f ~/.ssh/known_hosts -R <ip>
```

## IP Address

This is not merged yet. I use modified Ansible, namely with these 2 PRs merged:
- https://github.com/ansible/ansible/pull/23127
- https://github.com/ansible/ansible/pull/23133

```
$ cp ansible_devel -r ansible_packet_ip_address
$ cd ansible_packet_ip_address
$ git fetch origin pull/23127/head:device_improvements
$ git merge device_improvements
# resolve simple comment conflict
$ git fetch origin pull/23133/head:packet_ip
$ git merge packet_ip
$ cd ..
$ source ansible_packet_ip_address/hacking/env-setup
```

Adding IP in [playbook_ip_assign.yml](playbook_ip_assign.yml) 

```
$ ansible-playbook playbook_ip_assign.yml
```

Releasing IP in [playbook_ip_release.yml](playbook_ip_release.yml) 

```
$ ansible-playbook playbook_ip_release.yml
```

## Notes

The current state of Packet resources in Ansible devel is kind of a mess.

Some thigns in this doc don't work:
https://docs.ansible.com/ansible/guide_packet.html

Some things in the device module doc don't work:
https://docs.ansible.com/ansible/packet_device_module.html

The inventory doesn't group by device names.

Most of the things will be fixed if the pending PRs were merged.

## Starvation of PRs in Ansible

Some of the pending PRs rot, and then get conflict, which makes them even more likely to rot. I don't know about that because I can't poll all of them regularly.

When someone comments, I get an email, and I can fix, but when a commit to devel creates conflict with my PRs, I don't get to know about it.


