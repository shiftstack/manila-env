=================
manila-env on MOC
=================

This playbook creates a minimal devstack environment on MOC with Manila, Keystone and the CephNFS backend.

The playbook has been tested thus far on Fedora 31.

::

  [centos@worker0 ~]$ manila service-list
  +----+------------------+-----------------------------+---------------+---------+-------+------------+
  | Id | Binary           | Host                        | Zone          | Status  | State | Updated_at |
  +----+------------------+-----------------------------+---------------+---------+-------+------------+
  | 1  | manila-share     | worker0.rdocloud@cephfsnfs1 | manila-zone-0 | enabled | down  | None       |
  | 2  | manila-scheduler | worker0.rdocloud            | nova          | enabled | down  | None       |
  | 3  | manila-data      | worker0.rdocloud            | nova          | enabled | down  | None       |
  +----+------------------+-----------------------------+---------------+---------+-------+------------+

Prerequisites
=============

We assume that you have credentials to use MOC and that you have set them up in ${HOME}/.config/openstack/clouds.yaml along the following lines. You can copy the clouds.yaml.sample into ~/.config/openstack/clouds.yaml and make necessary changes.

::

  $ cat ~/.config/openstack/clouds.yaml
  clouds:
    moc:
        auth:
            auth_url: https://kaizen.massopen.cloud:13000/v3
            project_name: <your-project-name>
            user_domain_name: Default
            project_domain_name: Default
            username: <your-user-name>
            password: <your-password>
        region: RegionOne

The ansible playbook needs the shade library. In Fedora 31 run ``sudo dnf -y install python3-shade``.

Before you run the playbook edit copy keys.yml.sample to keys.yml and edit the latter with the paths to your public and private ssh keys:

::

  $ cat keys.yml.sample
  ---
  # Set the name of the key known to the OpenStack cloud.
  key_name: devstack_keypair
  # Set the name of the local private key file associated
  # with the named key.
  private_key_file: "/path/to/your/private/key"
  public_key_file: "/path/to/your/public/key"

Note that the playbook starts four m1.large Nova instances in MOC. Check your remaining quota before running.  You can scale down the number of worker nodes by editing roles/local/tasks/main.yml.

Playbooks
=========

Deploy
------

Now you can run the playbook::

$ ansible-playbook site.yml

The playbook will set up an appropriate security group, keypair, and private network, and a router to connect the private network to the MOC external network. It boots up a Nova VM that comprises the minimal devstack with CephNFS.

For convenience, the playbook installs a script `login-devstack.sh`, in the local directory that you can use to login to the VM.

On the VM, the playbook installs devstack using a ``local.conf`` that starts up only the minimal services -- keystone, mariadb, and rabbit -- needed to run manila with a CephFS with NFS back end.

If you want to run more OpenStack services or run with other back ends you can edit roles/devstack/files/local.conf before running the playbook.

Cleanup
-------

If you need to start over and you need to clean your environment, you can use the cleanup.yml playbook.

Just run::

$ ansible-playbook cleanup.yml

This playbook will remove the VM, the security group and the network resources that were allocated.
