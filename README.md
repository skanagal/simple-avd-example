# A simple ansible-avd example

## What is AVD?

- Basically an Ansible collection with a bunch of roles and modules that let you manage Arista devices both via eAPI and CVP.
- It also has design specific modules such as for a typical EVPN/VXLAN DC environment. We will be working on add more..
- More info can be found [here](https://github.com/aristanetworks/ansible-avd)

## What roles are we using for this example?

I am using two AVD roles:

1. [eos_cli_config_gen](https://github.com/aristanetworks/ansible-avd/tree/devel/ansible_collections/arista/avd/roles/eos_cli_config_gen).

Device specific variables are fed into jinja2 templates to generate configuration files. For this, the [template](https://docs.ansible.com/ansible/latest/modules/template_module.html) module is used in the background.

2. [eos_config_deploy_eapi](https://github.com/aristanetworks/ansible-avd/tree/devel/ansible_collections/arista/avd/roles/eos_config_deploy_eapi)

This role basically uses the [eos_config](https://docs.ansible.com/ansible/latest/modules/eos_config_module.html) to push configuration to the device. By default, this role replaces the configuration on the device with the configuration you just created. Be aware of this before you run your playbooks.

## What are we doing in this example?

- Building configuration for a device that is running BGP
- Pushing that configuration file to the device via eAPI.

## Building Blocks

### The Inventory File

- In this file you specify what hosts you want the playbook to run on. You can also define an hierarchical structure.
- For the example here, I have a group called AtlantaDC. Under this I have two more groups, namely, AtlantaSpines and AtlantaLeafs. The AtlantaSpines has one device while the AtlantaLeafs group has two.
- In this example, I am running my playbook only AtlantaSpines i.e. the *ghb265.sjc.aristanetworks.com* device.

```yaml
---
Tenant:
  children:
    AtlantaDC:
      children:
        AtlantaSpines:
          hosts:
            ghb265.sjc.aristanetworks.com:
              ansible_port: 443
        AtlantaLeafs:
          hosts:
            LEAF1.sjc.aristanetworks.com:
              ansible_port: 443
            LEAF2.sjc.aristanetworks.com:
              ansible_port: 443
```

### Device variables for configuration generation

Under the *intended>structured_configs* directory we have to specify the variables for the individual devices. These variables are used by our playbook to feed into jinja2 templates and generate CLI configuration. The generated config is stored in the *intended>configs* directory.

More info about this can be found [here](https://github.com/aristanetworks/ansible-avd/tree/devel/ansible_collections/arista/avd/roles/eos_cli_config_gen)

### The playbook

The playbook is very straightforward. I am specifying the group of hosts on which the playbook will run and calling out the two roles.

```yaml
- hosts: AtlantaSpines
  gather_facts: no
  tasks:

    - name: generate device intended config and documention
      import_role:
         name: arista.avd.eos_cli_config_gen

    - name: deploy configuration to device
      import_role:
         name: arista.avd.eos_config_deploy_eapi
```

### Execution of the playbook

```
bash-5.0# ansible-playbook -i inventory.yml simple-avd-example.yml

PLAY [AtlantaSpines] ******************************************************************************************************************************************************************************************************************************************************************************

TASK [eos_cli_config_gen : include device intended structure configuration variables] *************************************************************************************************************************************************************************************************************
ok: [ghb265.sjc.aristanetworks.com -> localhost]

TASK [eos_cli_config_gen : Generate eos intended configuration] ***********************************************************************************************************************************************************************************************************************************
ok: [ghb265.sjc.aristanetworks.com -> localhost]

TASK [eos_cli_config_gen : Generate device documentation] *****************************************************************************************************************************************************************************************************************************************
ok: [ghb265.sjc.aristanetworks.com -> localhost]

TASK [eos_config_deploy_eapi : replace configuration with intended configuration] *****************************************************************************************************************************************************************************************************************
ok: [ghb265.sjc.aristanetworks.com]

PLAY RECAP ****************************************************************************************************************************************************************************************************************************************************************************************
ghb265.sjc.aristanetworks.com : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

bash-5.0#
```
