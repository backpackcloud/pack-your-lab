# OpenShift Packed Lab

This is the playbook I'm using to deploy Red Hat OpenShift Container Platform on top of my Intel Nuc
Skull Canyon. It requires the Red Hat OpenStack Platform already installed.

The playbook will:

- create OpenStack instances for all nodes
- create and attach a volume for the docker storage on each instance
- attach a volume for the docker registry (you need to create it before)
- apply the pre reqs
- generate the Ansible inventory
- run the OpenShift installation playbook

In the end, you'll have a nice and crispy OpenShift waiting for you.

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
credentials and the pool id which gives you access to the Red Hat OpenShift Container Platform.

You might want to query the available subscriptions using `subscription-manager list --available`.

### OpenStack Parameters

This is the main structure of the section

```yaml
    osp:
      auth:
        auth_url: http://openstack.example.com:5000/v3
        username: admin
        password: openstack
        user_domain_name: Default
        project_name: admin
```

The authentication parameters will be added as a cloud provider in OpenShift installation.

### OpenShift Parameters

This is the main structure of the section

```yaml
      deploy_metrics: true
      deploy_logging: true
      subdomain: cloud.example.com
      console: openshift.example.com
      registry:
        volume:
          id: e7fe29c5-a4bd-4640-b647-8a6555853e23
          size: 50Gi
      cluster_admin_user: admin
```

- `deploy_metrics`: sets the installer to deploy the metrics component
- `deploy_logging`: sets the installer to deploy the logging component
- `subdomain`: configures the subdomain for the router
- `console`: configures the console hostname
- `registry`: configures the already created storage for docker registry
- `cluster_admin_user`: sets a cluster admin user (it's useful for a lab environment)

### Host vars

In order to create the instances and configure them, a set of host vars is needed:

- `bastion`: sets the hosts as the bastion host (the host that will run the OCP installer playbook)
- `kind`: sets the kind of the node (master, compute, infra or master-infra)
- `internal_ip`: sets the internal ip (OpenStack network)
- `external_ip`: sets the external ip (physical network)
- `docker_storage`: sets the parameters for creating and attaching the docker storage volume
