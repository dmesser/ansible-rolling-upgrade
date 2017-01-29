# Infrastructure Orchestration Example

This is a Continuous Integration / Continuous Deployment example leveraging the concept of Infrastructure-as-Data provided by Ansible.

A sufficiently complex multi-tier application (Rails + PostgreSQL) is orchestrated across multiple systems, networks and load balancers stood up in an OpenStack environment. The deployment process is covered in Ansible Plays and Roles end-to-end. Updates to the application are enabled by a rolling re-deployment of the application.

### Requirements

  - OpenStack environment with Glance, Cinder, Nova, Neutron and LBaaS v2
  - Two OpenStack Tenants with Keys, Images and Security Groups, 4 floating IPs
  - OpenStack Images running RHEL 7.x or CentOS 7.x
  - Ansible 2.2.1.0
  - python-shade
  - python-openstackclient
  - (optionally Red Hat Satellite Server)


### Credential management

OpenStack credentials with appropriate access to the tenants need to be provided via [os-client-config]. The name of the cloud is provided via the target variable and is expected to be the same as the tenant name in OpenStack. If it is not the same it can be override with the variable os_client_target. Please configure the os-client-config file to contain the following directives:

```sh
ansible:
  private: ''
  use_hostnames: True
```

SSH private keys need to be present in openstack/*tenant*.pem.

### Package Management

If no Red Hat Satellite Server is present call the playbooks with --skip-tags subscription. In this case you are responsible to provide access to packages. The systems check out the application code from a public GitHub repository - therefore they need internet access.

## How to run with ansible-playbook

> While you can put multiple cloud definitions in one clouds.yaml file it is encouraged to use separate files for separate tenants of the same OpenStack cloud since the OpenStack dynamic inventory script will otherwise merge all instances found into a large pool. This will lead to access problems when using the wrong SSH keys defined in the static inventory.

To initially deploy the app:

```sh
$ ansible-playbook -e "target=staging" site.yml
```

with an existing tenant named staging and a private key openstack/staging.pem.

To initiate the rolling upgrade with the running instances retrieved via the dynamic inventory:

```sh
$ ansible-playbook -i inventory-staging -e "target=staging" rolling_upgrade.yml
```

To cleanup a deployment and remove all components:

```sh
$ ansible-playbook -e "target=staging" destroy.yml
```

## How to run in Ansible Tower

> Currently there is an issue in Ansible Tower 3.0.3 and the os-client-config version shipped with it when using the built-in OpenStack inventory script.
> os-client-config is statically initialized with the *private* parameter set to true. This can be overridden with source variable definitions in the Tower Inventory. However insufficient string conversion in os-client-config leads to this parameter always be set to true unless it's initalized with '' in YAML.
> When the parameter is true it will cause python-shade to treat the OpenStack cloud as not capable of providing floating IPs.
> As such all IPs reported in the inventory will be OpenStack-internal tenant IPs and the floating IPs will be ignored.

1. Create a project with SCM type set to GitHub pointing to this GitHub project
2. Create Machine credentials, one for each of the tenants containing the SSH private key
 * enable privilege escalation using sudo for the user 'cloud-user'
3. Create OpenStack credentials, each containing the endpoint URLs, username, password and region for your tenants
4. Create OpenSack Dynamic Inventories for all tenants separately
 * set the following source variables exactly like this:
    - private: ''
    - use_hostnames: True
5. Create a Job Template for the site.yml, rolling_upgrade.yml and destroy.yml, for each tenant separately with the specific credentials, keys and Inventories for this tenant
  * configure the template with the additional parameter os_client_target set to "devstack"

The latter variable is needed because currently in Ansible Tower it's not possible to influence the name of the cloud provided by the temporary os-client-config file created during Job Execution - it will statically be set to 'devstack'.

## Todo

To achieve the "Continuous" part of CI/CD triggering these playbooks could for instance be called upon code commits. GitHub Webhooks or Jenkins SCM Polling Jobs are popular choices. Integration in this regard is outside of the scope of this example.


  [os-client-config]: <https://pypi.python.org/pypi/os-client-config>
