Set up Docker Swarm with Ansible
===

## Set up new Linux Machines

If you want a worker node running on a separate machine, you need to create at least two linux machines, one manager and one worker. Docker advises using static IP addresses for manager nodes to prevent the swarm from becoming unstable on manager reboot.

You must create and copy over SSH keys for your controller machine to the target machine(s) before you can run the playbook.

````
ssh-keygen -t rsa
# hit enter until the promt returns. Setting a password is optional, see below for how to handle this.
````
When the keygen is complete, transfer them to the target server(s) with the following command:

````
# ssh-copy-id user@hostname_or_ip
ssh-copy-id root@your-hostname
````
When the copy is complete, try connecting via SSH:

````
# ssh user@hostname_or_ip
ssh root@your-hostname
````
If the keys are in place you should not be promted to input login information.
To avoid getting the message about trusting the Linux machine you want to SSH into or run a playbooks against, run `export ANSIBLE_HOST_KEY_CHECKING=False`.


## Update `.inventory` file

When that is done, edit the `.inventory` file for the environment and add two host-groups for linuxDockerSwarmManager and linuxDockerSwarmWorker. If you have multiple swarms in the same inventory, you need to distinguish them using the variable swarm_name to indicate which workers and managers belong together. If you want the manager and worker on a single machine, place it only in the host group linuxDockerSwarmManager. By default, all managers are also workers:

```yaml
[linuxDockerSwarmManager]
manager01.domain.local swarm_name=test
manager02.domain.local swarm_name=test
manager03.domain.local swarm_name=qa

[linuxDockerSwarmWorker]
worker01.domain.local swarm_name=test
worker02.domain.local swarm_name=test
worker03.domain.local swarm_name=qa
```

## Run the playbook

Run the playbook:

```
ansible-playbook LinuxDockerSwarm.yaml -i inventory/environment.inventory
```
You can use `--list-hosts` to do a dry-run and see which hosts will be affected.
This playbook will set up docker and docker swarm with the manager and worker nodes you have indicated in your inventory file. To verify that the setup works as intended, log in to your server and check the status of the docker nodes:

```
ssh root@your-manager-server
docker node ls
```


## Known limitations

* This playbook currently only works with Linux
* This playbook does not detect that a machine has been removed from the linuxDockerSwarmManager or linuxDockerSwarmWorker host groups. If you want to remove a node from the swarm, you must do so manually.
* Docker advises using static IP addresses for manager nodes to prevent the swarm from becoming unstable on manager reboot
* Note that Docker/Linux/templates/daemon.json.j2 contains default log rotation settings for the docker daemon. Modify this file according to your requirements.

