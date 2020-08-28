# A simple ansible-avd example



## What are we doing here?

- Building configuration for a device that is running BGP
- Pushing that configuration file to the device via eAPI.

## Building Blocks

### The Inventory File

- In this file you specify what hosts you want the playbook to run on. You can also define an hierarchical structure.
- For the example here, I have a group called AtlantaDC. Under this I have two more groups, namely, AtlantaSpines and AtlantaLeafs. The AtlantaSpines has one device while the AtlantaLeafs group has two.
- In this example, I am running my playbook on AtlantaSpines i.e. the ghb265 device.


```
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
