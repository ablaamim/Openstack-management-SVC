## Deployment with Kolla-Ansible:

</p>
<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/e/e6/OpenStack%C2%AE_Logo_2016.svg" width="400">
</p>

---

### Benefits Overview

Kolla-Ansible is the vanilla, production-ready method for deploying OpenStack. It combines Ansible automation with Docker containerization to provide a robust and reliable private cloud.

Choosing Kolla-Ansible over a manual OpenStack install is like choosing a modern assembly line over building something by hand.

With a manual install, you must carefully put every single piece in place yourself on every server. This is very slow and it is easy to make a mistake. If one thing is wrong, the whole system can break. Upgrading is even harder and riskier.

Kolla-Ansible automates all of this. It gives you a proven recipe to build your cloud quickly and the same way every time. It puts each service in its own container, so they don't interfere with each other. If you need to upgrade or fix something, you can usually just replace one container without stopping the whole cloud. This makes the system much more reliable and easy to take care of.

Supported Matrix [link](https://docs.openstack.org/kolla/latest/support_matrix.html)

## Requirements:

A successful Kolla-Ansible deployment requires meeting several key prerequisites. First, all target nodes must have a supported operating system, such as CentOS 7/8, Rocky Linux 8/9, or Ubuntu 20.04/22.04, with Python 3 installed. Second, each server requires a minimum of two network interfaces: one for management and one for provider/data traffic. Third, the deployment node (which can be one of the controllers) must have Ansible and Docker installed, and password-less SSH access configured to all other nodes in the cluster. Finally Meeting these requirements ensures a stable foundation for the automated installation of the OpenStack environment.

### Key Advantages:

> Production-Ready & Vanilla: Provides an official, upstream OpenStack deployment without vendor lock-in. It uses unmodified, vanilla service configurations optimized for production stability and high availability out-of-the-box.

> Reliability: Designed for production, with built-in high availability.

> Consistency: Containerization eliminates dependency conflicts and ensures uniform service behavior across all nodes.

> Safe Operations: Enables atomic, single-service upgrades and easy rollbacks, minimizing downtime.

> Simplified Management: A single configuration file (globals.yml) manages the entire cloud deployment.

> Security: Services run in isolation as non-root users within containers.

> Scalability: Modular architecture allows for easy expansion by adding new nodes or services.

> Community Support: An official OpenStack project with active development and wide adoption.

In short, Kolla-Ansible provides a robust, automated foundation for deploying and maintaining a complex OpenStack cloud.

---

### Deployment Overview

> Kolla-Ansible deploys a highly available (HA) OpenStack cloud by distributing all critical control plane services across a cluster of three or more controller nodes. As the illustration shows, a Virtual IP (VIP) fronts the cluster, acting as a single endpoint for all API traffic, which is load-balanced across the active nodes.

Keeping Your Cloud Reliable

The system is designed to have no single point of failure. It works as a team.

If one server stops working, the others immediately take over. Everything keeps running smoothly without any interruption for users.

This makes the cloud highly resilient and always available.

---

</p>
<p align="center">
<img src="https://github.com/ablaamim/Openstack-management-SVC/blob/main/images/openstack_HA.png" width="800">
</p>

---

### Architecture Explanation: 

This Kolla-Ansible architecture deploys a full, production-grade OpenStack cloud by logically grouping all specified services into a highly available (HA) control plane and a scalable data plane.

The Control Plane Cluster hosts all API endpoints (Keystone, Nova, Neutron, Ironic, Designate, etc.), the web dashboard (Horizon), and the central brain for services like orchestration (Heat), metering (Ceilometer), and rating (CloudKitty). High availability for stateful services is provided by a MariaDB Galera cluster and a RabbitMQ cluster, ensuring no single point of failure.

The Data Plane is where the workload executes. Compute Nodes run the Nova hypervisor and host VM instances. The Storage Cluster (Ceph and Swift) provides unified storage for images (Glance), volumes (Cinder), shares (Manila), and objects (Swift), ensuring data resilience. Network services like routing and load balancing (Octavia) can be distributed or centralized. The Bare Metal service (Ironic manages physical servers separately, provisioning them via PXE boot for use as either bare metal instances or hypervisors.

This design ensures scalability (by adding more nodes to each plane), resilience (through HA pairs and clusters), and logical separation of management and data traffic, fulfilling the requirements for a modern private cloud.

</p>
<p align="center">
<img src="https://github.com/ablaamim/Openstack-management-SVC/blob/main/images/openstack-svc.png" width="800">
</p>

### OpenStack Deployment Timing Plan:

| Phase | Activity | Duration | Start Week | End Week | Deliverable |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Planning** | Requirements gathering and design | 2 weeks | Month 1, Week 1 | Month 1, Week 2 | Technical specification document |
| **2. Setup** | Hardware preparation and network configuration | 2 weeks | Month 1, Week 3 | Month 1, Week 4 | Ready infrastructure |
| **3. Core Deployment** | Deploy fundamental services (Keystone, Nova, Neutron, Glance, Horizon) | 2 weeks | Month 2, Week 1 | Month 2, Week 2 | Basic cloud operational |
| **4. Storage Setup** | Deploy and configure Cinder, Swift | 2 weeks | Month 2, Week 3 | Month 2, Week 4 | Storage services ready |
| **5. Advanced Services** | Deploy remaining services (Octavia, Heat, Barbican, etc.) | 3 weeks | Month 3, Week 1 | Month 3, Week 3 | Full feature set available |
| **6. Testing & Handover** | User acceptance testing and documentation | 2 weeks | Month 3, Week 4 | Month 3, Week 4 | Production-ready cloud |

### Proof of concept :

---

#### DigitalOCean Deployement :

> I deployed minimal Openstack accross 4 Virtual machines on DigitalOcean plateform

##### OpenStack Deployment Diagram

# OpenStack Deployment Diagram

```bash
+------------------------------------------------------------------+
|                  DigitalOcean Project                            |
|                                                                  |
|  +-------------+      +----------------+      +----------------+ |
|  | Controller  |------| Block-Storage  |      |    Compute1    | |
|  | 178.128.4.42|      |206.189.208.220 |      | 146.190.123.49 | |
|  +-------------+      +----------------+      +----------------+ |
|      |     |                                      |     |        |
|      |     |--------------------------------------|     |        |
|      |                                                  |        |
|      |     +----------------+                           |        |
|      |-----|    Compute2    |---------------------------|        |
|            | 143.198.103.29 |                                    |
|            +----------------+                                    |
+-------------------------------------------------------------------
```


Legend / Flow:

* The Controller node manages the entire OpenStack environment (API services, scheduler, etc.).

* The Compute nodes (Compute1, Compute2) host and run the virtual machine instances.

* The Block-Storage node provides persistent storage volumes that can be attached to instances running on the compute nodes.

* All nodes communicate with each other over the network (represented by the connecting lines).

</p>
<p align="center">
<img src="https://github.com/ablaamim/Openstack-management-SVC/blob/main/images/digitalocean.png" width="1000">
</p>


#### Runing containers on each node :

##### Compute and controller nodes running sercvices :

> This OpenStack deployment was automated using Kolla Ansible, which containerizes all OpenStack services for improved isolation, manageability, and consistency. Each service—including the core compute (Nova), networking (Neutron), and identity (Keystone) components—runs within its own dedicated Docker container across the controller and compute nodes. This container-based approach, orchestrated by Ansible, streamlines the initial installation, simplifies future upgrades, and ensures a highly reproducible environment across all nodes in the cloud infrastructure.

### compute Node

</p>
<p align="center">
<img src="https://github.com/ablaamim/Openstack-management-SVC/blob/main/images/compute-containers.png" width="1000">
</p>

### Controller Node 

</p>
<p align="center">
<img src="https://github.com/ablaamim/Openstack-management-SVC/blob/main/images/controller services.png" width="1000">
</p>

#### core OpenStack services deployed :

* Keystone (identity): The identity service provides authentication and authorization for all other OpenStack services.

* Nova (compute): The compute service manages and provisions virtual machine instances on the compute nodes.

* Neutron (network): The networking service provides Software-Defined Networking (SDN) capabilities, managing networks and IP addresses for instances.

* Glance (image): The image service stores and manages disk images and templates used to create virtual machine instances.

* Placement: The placement service tracks resource inventory and assists in selecting optimal compute nodes for new instances.

* Heat (orchestration): The orchestration service allows for the automated provisioning of cloud infrastructure using templated definitions.

* Heat-cfn (cloudformation): Provides an API compatible with AWS CloudFormation, enabling the use of AWS templates within OpenStack.

</p>
<p align="center">
<img src="https://github.com/ablaamim/Openstack-management-SVC/blob/main/images/various data.png" width="1000">
</p>

#### Network Topology :

</p>
<p align="center">
<img src="https://github.com/ablaamim/Openstack-management-SVC/blob/main/images/network-topology.png" width="1000">
</p>

#### Router :

</p>
<p align="center">
<img src="https://github.com/ablaamim/Openstack-management-SVC/blob/main/images/router.png" width="1000">
</p>

#### Instance test :

</p>
<p align="center">
<img src="https://github.com/ablaamim/Openstack-management-SVC/blob/main/images/instance.png" width="1000">
</p>