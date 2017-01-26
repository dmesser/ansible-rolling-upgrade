# Infrastructure Orchestration Example

This is a Continuous Integraton / Continuous Deployment example leveraging the concept of Infrastructure-as-Data provided by Ansible.

A sufficiently complex multi-tier application is orchestrated across multiple systems, networks and load balancers stood up in an OpenStack environment. The deployment process is covered in Ansible Plays and Roles end-to-end. Updates to the application are enabled by a rolling re-deployment of the application.

Requirements

  - OpenStack environment with Glance, Cinder, Nova, Neutron and LBaaS v2
  - Two OpenStack Tenants with Keys, Images and Security Groups, 4 floating IPs
  - OpenStack Images running RHEL 7.x or CentOS 7.x
  - Ansible 2.2.1.0
  - python-shade
  - python-openstackclient
  - (optionally Red Hat Satellite Server)

OpenStack credentials with appropriate access to the tenants needs to be provided either via os-cloud-config or credentials stored in the keystonerc_* files. It's is advised to encrypt the latter with ansible-vault.

SSH private keys need to be present in the openstack/ directory and named according to the tenant.

If no Red Hat Satellite Server is present call the playbooks with --skip-tags subscription. In this case you are responsible to provide access to packages

### How to run

To initially deploy the app:

```sh
$ ansible-playbook -e "target=staging" site.yml
```

with an existing tenant named demo-staging and a private key openstack/staging.pem.

To initiate the rolling upgrade:

```sh
$ ansible-playbook -e "target=staging" rolling_upgrade.yml
```

To cleanup a deployment and remove all components:

```sh
$ ansible-playbook -e "target=staging" destroy.yml
```

To achieve the "Continuous" part of CI/CD triggering these playbooks could for instance be called upon code commits. GitHub Webhooks or Jenkins SCM Polling Jobs are popular choices. Integration in this regard is outside of the scope of this example.
