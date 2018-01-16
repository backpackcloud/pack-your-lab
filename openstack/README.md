# OpenStack on a Nuc Shell Playbook

This is the playbook I'm using to deploy Red Hat OpenStack Platform on top of my Intel Nuc Skull
Canyon.

The playbook will:

1. Subscribe the host
2. Enable the required repositories
3. Install the required packages
4. Create and customize the packstack answer file
5. Invoke the packstack installer
6. Create a public flat network with a subnet
7. Create the resources mapped in the hosts file

## The hosts file

Check the `hosts.example.yml` for a full working example (almost like mine). The variable
sections are explained below.

### Subscription Parameters

```yaml
    subscription:
      user: some-user
      password: some-password
      pool_ids:
        - some-pool-id
```

This section configures the subscription, it's pretty straightforward, just provide your Red Hat
credentials and the pool id which gives you access to the Red Hat OpenStack Platform.

You might want to query the available subscriptions using `subscription-manager list --available`.

### OpenStack Parameters

This is the main structure of the section

```yaml
    osp:
      resource_path:
      packstack:
      keypairs:
      network:
      images:
      flavors:
      lab:
```

#### Upload Path

The `resource_path` takes a path on the remote host that will receive the resources (like keypairs,
images, packstack file, etc.). It will be created so just throw the desired path and let the
playbook create it for you.

#### Packstack Configuration

The `packstack` configuration is structured as this example:

```yaml
      packstack:
        CONFIG_KEY: config_value
```

The key-value pairs will be dded to the packstack answers file before the packstack installation.
This allow you to customize the installation. Following is an example:

```yaml
    osp:
      resource_path: /tmp/openstack
      packstack:
        CONFIG_DEFAULT_PASSWORD: openstack
        CONFIG_KEYSTONE_ADMIN_PW: openstack
        CONFIG_CINDER_VOLUMES_SIZE: 300G
        CONFIG_HEAT_INSTALL: y
```

#### Keypairs

The `keypairs` section allows you to configure your ssh keys for using with OpenStack instances.
Just add entries to the configuration in the format name=key_path, like the example:

```yaml
      keypairs:
        laptop: ~/.ssh/id_rsa.pub
```

The key name will be the keypair name in OpenStack. Remember that the paths should be local to the
host executing the playbook.

#### Network Configuration

The `network` section configures the public network in OpenStack. The playbook will create and
configure a public network based on the provided configuration. The structure is described below:

```yaml
      network:
        nic: "the network interface for public access"
        cidr: "the CIDR value of your network"
        dns: "the DNS server to use"
        dhcp:
          start: "the start range of the DHCP to configure"
          end: "the end range of the DHCP to configure"
```

The `nic` will be used to configure the public network using the `flat` provider. Below is an
example:

```yaml
      network:
        nic: eno1
        cidr: 192.168.0.0/24
        dns: 192.168.0.1
        dhcp:
          start: 192.168.0.20
          end: 192.168.0.100
```

#### Images Configuration

The `images` section allows you to specify images to be available for OpenStack users. The structure
is specified below:

```yaml
      images:
        name:
          local_path: path/to/the/image
          disk_format: (ami, ari, aki, vhd, vmdk, raw, qcow2, vhdx, vdi, iso or ploop)
          min_disk: size in GB
          min_ram: size in MB
```

The `image_name` will be used as the name in the OpenStack. Below is an example:

```yaml
      images:
        rhel:
          local_path: ~/workspace/pack-your-lab/images/rhel-server-7.4-x86_64-kvm.qcow2
          disk_format: qcow2
          min_disk: 10
          min_ram: 256
        rhel-atomic:
          local_path: ~/workspace/pack-your-lab/images/rhel-atomic-cloud-7.4.3-8.x86_64.qcow2
          disk_format: qcow2
          min_disk: 10
          min_ram: 128
```

This will create a `rhel` and a `rhel-atomic` image ready to be used. Remember that the paths should
be local to the host executing the playbook.

#### Flavors Configuration

The `flavours` section allows you to configure the flavors that will be available to use. The
structure is specified below:

```yaml
      flavors:
        name:
          ram: amount in MB
          disk: amount in GB
          ephemeral: amount in GB
          swap: amount in MB
          vcpus: number
```

Each entry will be added using the provided `name`. Below is an example:

```yaml
      flavors:
        m1.nano:
          ram: 128
          disk: 1
          ephemeral: 0
          swap: 0
          vcpus: 1
        m1.micro:
          ram: 256
          disk: 1
          ephemeral: 0
          swap: 0
          vcpus: 1
```

#### Lab Configuration

The lab will be created as a project having the `admin` user as its owner. The structure to
configure the project is specified below:

```yaml
      lab:
        floating_ip: number
        networks:
          name:
            cidr: subnet cidr
            dhcp:
              start: ip start range
              end: ip end range
```

The `floating_ip` defines the number of floating IPs that will be created for you. In the `network`
section, you can specify networks to create. Below is an example:

```yaml
      lab:
        floating_ip: 5
        networks:
          workshop:
            cidr: 10.0.0.0/24
            dhcp:
              start: 10.0.0.10
              end: 10.0.0.100
          test:
            cidr: 10.0.1.0/24
            dhcp:
              start: 10.0.1.10
              end: 10.0.1.100
```

This will create 5 floating IPs and 2 private networks (one named `workshop` and the other named
`test`) both added to a router that is connected to the external network.

## Installing OpenStack

To install OpenStack, just run the `install.yml` playbook. It should take a while, specially if
you have lots of images to create.

At the end you will have a nice and crispy OpenStack lab just waiting for you to launch your
instances.
