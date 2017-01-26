# Infrastructure Orchestration Example

This is a Continuous Integration / Continuous Deployment example leveraging the concept of Infrastructure-as-Data provided by Ansible.

A sufficiently complex multi-tier application is orchestrated across multiple systems, networks and load balancers stood up in an OpenStack environment. The deployment process is covered in Ansible Plays and Roles end-to-end. Updates to the application are enabled by a rolling re-deployment of the application.

### Requirements

  - OpenStack environment with Glance, Cinder, Nova, Neutron and LBaaS v2
  - Two OpenStack Tenants with Keys, Images and Security Groups, 4 floating IPs
  - OpenStack Images running RHEL 7.x or CentOS 7.x
  - Ansible 2.2.1.0
  - python-shade
  - python-openstackclient
  - (optionally Red Hat Satellite Server)


### Credential management

OpenStack credentials with appropriate access to the tenants need to be provided via [os-client-config]. While you can put multiple cloud definitions in one file it is encouraged to use separate files for separate tenants of the same OpenStack cloud since the OpenStack dynamic inventory script will otherwise merge all instances found into a large pool. This will lead to access problems when using the wrong SSH keys defined in the static inventory.

SSH private keys need to be present in the openstack/ directory and named according to the tenant.

### Package Management

If no Red Hat Satellite Server is present call the playbooks with --skip-tags subscription. In this case you are responsible to provide access to packages. The systems check out the application code from a public GitHub repository - therefore they need internet access.

## How to run

To initially deploy the app:

```sh
$ export OS_CLIENT_CONFIG_FILE=staging-cloud.yaml
$ ansible-playbook -e "target=staging" site.yml
```

with an existing tenant named demo-staging and a private key openstack/staging.pem.

To initiate the rolling upgrade with the running instances retrieved via the dynamic inventory:

```sh
$ export OS_CLIENT_CONFIG_FILE=staging-cloud.yaml
$ ansible-playbook -i inventory-staging -e "target=staging" rolling_upgrade.yml
```

To cleanup a deployment and remove all components:

```sh
$ export OS_CLIENT_CONFIG_FILE=staging-cloud.yaml
$ ansible-playbook -e "target=staging" destroy.yml
```

To achieve the "Continuous" part of CI/CD triggering these playbooks could for instance be called upon code commits. GitHub Webhooks or Jenkins SCM Polling Jobs are popular choices. Integration in this regard is outside of the scope of this example.


  [os-client-config]: <https://pypi.python.org/pypi/os-client-config>
