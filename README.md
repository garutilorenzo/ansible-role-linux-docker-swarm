[![GitHub issues](https://img.shields.io/github/issues/garutilorenzo/ansible-role-linux-docker-swarm)](https://github.com/garutilorenzo/ansible-role-linux-docker-swarm/issues)
![GitHub](https://img.shields.io/github/license/garutilorenzo/ansible-role-linux-docker-swarm)
[![GitHub forks](https://img.shields.io/github/forks/garutilorenzo/ansible-role-linux-docker-swarm)](https://github.com/garutilorenzo/ansible-role-linux-docker-swarm/network)
[![GitHub stars](https://img.shields.io/github/stars/garutilorenzo/ansible-role-linux-docker-swarm)](https://github.com/garutilorenzo/ansible-role-linux-docker-swarm/stargazers)

# Install and configure docker swarm cluster on Linux

Ansible role used to install and configure docker swarm cluster on Linux boxes

## Requirements

Install ansible, ipaddr and netaddr:

```
pip install -r requirements.txt
```

Download the role form GitHub:

```
ansible-galaxy install git+https://github.com/garutilorenzo/ansible-role-linux-docker-swarm.git
```

Your inventory file must have the following groups:

* swarm_managers
* swarm_workers

## Role Variables

This role accept this variables:

| Var   | Required |  Default | Desc |
| ------- | ------- | ----------- |  ----------- |
| `docker_subnet`       | `no`       | ``       | If the server have multiple network interfaces, specify the subnet where the docker daemon is listening. This will determinate the Docker advertise_addr  |

## Notes

The task from the role must be executed sequentially in the follwoing order:

1. gather_network_facts (get network interfaces facts)
2. create the swarm cluster or get the join tokens if the cluster already exist (task must be run only on one node, first one for example)
3. all the other tasks

**NOTE** join_manager_node task needs to be executed on all managers node except the first one

See [Example playbook](#example-playbook) for one example playbook

## Example playbook

Here one example playbook:

```yaml
- hosts: 
    - swarm_managers
    - swarm_workers
  become: yes
  remote_user: vagrant
  tasks:
    - ansible.builtin.import_role:
        name: ansible-role-linux-docker-swarm
        tasks_from: gather_network_facts

- hosts: swarm_managers[0]
  become: yes
  remote_user: vagrant
  tasks:
    - ansible.builtin.import_role:
        name: ansible-role-linux-docker-swarm
        tasks_from: init_swarm

- hosts: swarm_managers[1:]
  become: yes
  remote_user: vagrant
  tasks:
    - ansible.builtin.import_role:
        name: ansible-role-linux-docker-swarm
        tasks_from: join_manager_node

- hosts: swarm_workers
  become: yes
  remote_user: vagrant
  tasks:
    - ansible.builtin.import_role:
        name: ansible-role-linux-docker-swarm
        tasks_from: join_worker_node
```