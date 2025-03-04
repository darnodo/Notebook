---
title: "My First Lab"
date: 2025-02-14T12:00:00+02:00
weight: 1
cascade:
  type: docs
---

## Introduction 📚

In this article, we’ll explore how to install our very first Containerlab netlab using **DevPod**. We'll focus on leveraging a cloud provider—specifically **AWS**—to host our project. Why **Cloud**? Because network labs can consume a huge amount of resources, and we need the ability to deploy, stop, and destroy them quickly for both performance and financial efficiency. 💡💰

We'll achieve this using a combination of:

- **DevPod**
- **DevContainer**
- **Containerlab**

Additionally, we will use a small topology, which you can find on my [GitHub repository](https://github.com/darnodo/VXLAN-EVPN). Our main objective is to deploy this lab on AWS using DevPod. Let's dive in and get started! 🚀😊

## Prerequisites 🔧

Before we get started, there are a couple of important steps to complete:

1. **AWS Environment Authorization**:  
   Ensure that DevPod is authorized to access your AWS environment. For a detailed guide on configuring DevPod with AWS, please refer to my post on this [topic](/Notebook/documentation/devpod). 🔑

2. **Containerlab Topology**:  
   We need a topology file that Containerlab can understand. In our case, we're building a simple VXLAN topology. 🗺️

## Containerlab Topology 🔄

Our lab will simulate a VXLAN topology with:

- **1 Spine switch**
- **2 Leaf switches**
- **2 Host nodes**

The following diagram illustrates the VXLAN topology:

![VXLAN Topology](VXLAN.svg#center)

Below is the Containerlab topology file (`lab_vxlan.yml`) used for this setup:

```yaml
name: vxlan-evpn-irb
topology: 
  nodes:
    spine1:
      kind: ceos
      image: ceos:4.32.0.1F
      mgmt-ipv4: 172.20.20.101
    leaf1:
      kind: ceos
      image: ceos:4.32.0.1F
      mgmt-ipv4: 172.20.20.11
    leaf2:
      kind: ceos
      image: ceos:4.32.0.1F
      mgmt-ipv4: 172.20.20.12
    host1:
      kind: linux
      image: alpine:latest
      binds:
        - hosts/h1_interfaces:/etc/network/interfaces
      mgmt-ipv4: 172.20.20.21
    host2:
      kind: linux
      image: alpine:latest
      binds:
        - hosts/h2_interfaces:/etc/network/interfaces
      mgmt-ipv4: 172.20.20.22
  links:
    - endpoints: ["spine1:eth1", "leaf1:eth1"]
    - endpoints: ["spine1:eth2", "leaf2:eth1"]
    - endpoints: ["leaf1:eth2", "host1:eth1"]
    - endpoints: ["leaf2:eth2", "host2:eth1"]
```

### Breaking Down the Topology 🧐

1. **Name and Structure**:
   - `name: vxlan-evpn-irb` – This is the lab's name.
   - The topology is divided into **nodes** (devices) and **links** (connections between devices).

2. **Nodes**:
   - **Spine Layer**:
     - `spine1`: A containerized Arista cEOS switch using image version `4.32.0.1F`.
     - **Management IP**: `172.20.20.101`
   - **Leaf Layer**:
     - `leaf1` and `leaf2`: Arista cEOS switches with the same image version.
     - **Management IPs**: `172.20.20.11` and `172.20.20.12`
   - **Host Layer**:
     - `host1` and `host2`: Linux containers running Alpine Linux.
     - They include custom network interface configurations mounted from the host.
     - **Management IPs**: `172.20.20.21` and `172.20.20.22`

3. **Links**:
   - **Spine to Leaf**:
     - `spine1:eth1` ↔ `leaf1:eth1`
     - `spine1:eth2` ↔ `leaf2:eth1`
   - **Leaf to Host**:
     - `leaf1:eth2` ↔ `host1:eth1`
     - `leaf2:eth2` ↔ `host2:eth1`

This topology represents a typical spine-leaf architecture, common in data center networks to enable both Layer 2 and Layer 3 connectivity with VXLAN EVPN configurations. 🔗💻

## Deploy the Lab 🛠️

We will deploy the lab using **DevPod** in two ways:

### 1. Using the Repository 📥

1. **Validate AWS Provider Configuration**:  
   Ensure that your AWS provider is correctly configured. More details can be found [here](/Notebook/documentation/devpod). ✅

2. **Create a Workspace**:  
   - Navigate to the **Workspace** tab and click on **Create Workspace**.
   - Enter the **Workspace source**: use the [GitHub repository](https://github.com/darnodo/VXLAN-EVPN).
   - Select **AWS** as the provider.
   - Choose your default IDE.
   - Finally, click on **Create Workspace**.

   ![DevPod Configuration](devpod_configuration.png#center)

### 2. Using a Local Folder 🗂️

If you prefer to use your local repository:

- The only difference is in the **Workspace source**.
- Simply point it to your local repository.

   ![DevPod Configuration - Local](devpod_configuration_local.png#center)

## Starting the Lab 🎬

> [!WARNING] cEOS Images
> The lab uses **cEOS image v4.32.0.1F**.  
> To download this image, visit the [Arista download webpage](https://www.arista.com/en/support/software-download). ⚠️

1. **Import the cEOS Image**:  
   Save the cEOS image in your `network_images` folder by dragging and dropping it into VSCode.  
   Import the image using the following command:

   ```bash
   docker import network_images/cEOS64-lab-4.32.0.1F.tar.xz ceos:4.32.0.1F
   ```

2. **Deploy the Lab**:  
   Deploy the lab using Containerlab:

   ```bash
   sudo containerlab deploy -t lab_vxlan.yml
   ```

   Follow the CLI prompts to configure your devices. For detailed configuration steps, refer to [this guide](https://github.com/darnodo/VXLAN-EVPN/tree/main/documentation/eos_configuration). 🔧🖥️

3. **Visualize the Architecture**:  
   Validate the deployed topology using Containerlab’s graph view:

   ```bash
   containerlab graph -t lab_vxlan.yml
   ```

   Ports (e.g., port 50080 as mentioned in the `devcontainer.json`) are forwarded. Access the graph view via [localhost](http://localhost:50080).

   ![Graph View](Graph_view.png#center)

## Using EdgeShark 🦈

EdgeShark is a web UI tool that helps capture packets from your lab environment. It tunnels captures from the lab to Wireshark running locally. 📡🔍

For more details, check out [EdgeShark’s Getting Started Guide](https://edgeshark.siemens.io/#/getting-started?id=optional-capture-plugin).

### EdgeShark Setup in the DevContainer 🐳

In the **DevContainer** configuration, the following `postCreateCommand` has been added:

```bash
sudo mkdir -p /opt/edgeshark && sudo curl -sL https://github.com/siemens/edgeshark/raw/main/deployments/wget/docker-compose.yaml -o /opt/edgeshark/docker-compose.yaml
```

This command downloads a Docker Compose file to facilitate the use of EdgeShark. 🚀

### Running EdgeShark ⚡

To start EdgeShark, execute:

```bash
cd /opt/edgeshark
DOCKER_DEFAULT_PLATFORM= docker compose up -d
```

Access EdgeShark via [localhost:5001](http://localhost:5001).

- **EdgeShark View**:  
  ![Edgeshark View](edgeshark.png#center)

- **Wireshark Integration**:  
  Using the Wireshark icon in EdgeShark, you can launch Wireshark locally.  
  ![Edgeshark Interface](edgeshark_interface.png#center)  
  ![Edgeshark and Wireshark](edge_wireshark.png#center)

## Conclusion 🎉

In this article, we walked through the steps to deploy a VXLAN EVPN lab using Containerlab, DevPod, and AWS. We covered the following key points:

- **Setting up prerequisites** for AWS and Containerlab. 🔑
- **Creating a detailed topology** file for a spine-leaf network. 🗺️
- **Deploying the lab** using both repository and local methods. 📥🗂️
- **Starting the lab** with Docker and Containerlab. 🚀🐳
- **Using EdgeShark** to capture packets and integrate with Wireshark for detailed analysis. 🦈🔍

By following these steps, you can easily deploy and manage a scalable network lab environment in the cloud. Happy networking and enjoy your lab adventures! 😄🎊
