---
title: "What is a Netlab"
cascade:
  type: docs
---

## Introduction

📡 In a world where computer networks play an increasingly significant role in our daily lives, understanding the principles and logic that drive them becomes more and more essential.

Virtual Network Labs (commonly referred to as "NetLab" or "Virtual Network Lab") are an ideal approach for teaching these concepts, allowing us to simulate complex network environments and experiment without risks.

I would like to share with you how my "NetLabs" work, which will be regularly linked to documentary articles (in the category: **documentation**). They will allow us to practice, observe, or understand the operation of concepts explained theoretically beforehand.

In the context of NetLabs, they will primarily be deployed via the ContainerLab tool. For more complex architectures, we will use GNS3.

## What is ContainerLab? 🛠️

ContainerLab is a powerful open-source tool that enables the creation of complete virtual network labs. By using it, one can simulate a multitude of complex network architectures, with equipment such as routers, switches, servers, and other network devices.

This platform offers great flexibility in designing exercises, allowing the exploration of various topics such as learning network protocols, security, or configuring equipment. Users can thus focus on analysis and problem-solving without worrying about the underlying technical details.

The installation of ContainerLab will not be presented here, but all information is available on the official website [here](https://containerlab.dev/install/).

## What is GNS3? 💻

GNS3, or Graphical Network Simulator-3, is an open-source software primarily used for the **simulation** and **emulation** of computer networks. It allows network engineers, students, and professionals to design, test, and troubleshoot complex networks in a virtual environment before deploying them in the real world. GNS3 is particularly appreciated for its ability to integrate various network hardware and software, such as Cisco routers and switches, as well as virtual machines to create realistic network topologies.

As previously mentioned, the installation of GNS3 will not be discussed here. For more information, the documentation is available [here](https://docs.gns3.com/docs/).

## GNS3 vs ContainerLab ⚔️

GNS3 and ContainerLab are two powerful tools for network simulation and emulation, but they differ in their approach, features, and primary use cases. Here is a quick comparison between the two:

### GNS3

**Advantages:**

1. **Intuitive Graphical Interface:** GNS3 offers a user-friendly graphical interface that allows users to drag and drop components to create network topologies.
2. **Multivendor Support:** It supports a wide range of network hardware and software, including Cisco routers and switches, as well as virtual machines.
3. **Flexibility:** GNS3 can be used on Windows, macOS, and Linux, and it integrates well with other tools like Wireshark for traffic analysis.
4. **Active Community:** A large community of users and developers provides extensive support and a wealth of online resources.

**Disadvantages:**

1. **System Resources:** GNS3 can be resource-intensive, especially when emulating complex devices or large topologies.
2. **Configuration Complexity:** The initial setup can be complex, particularly for new users.

### ContainerLab

**Advantages:**

1. **Lightweight and Performance:** ContainerLab uses containers to emulate network devices, making it more lightweight and performant than solutions based on virtual machines.
2. **Automation and DevOps:** It integrates well with DevOps and automation tools like Ansible, facilitating automated deployment and network management.
3. **Simplified Configuration:** Topologies are defined via YAML files, making the configuration simpler and scriptable.
4. **Support for Modern Technologies:** It supports modern technologies like Docker and Kubernetes, offering greater flexibility for cloud-native environments.

**Disadvantages:**

1. **Less Multivendor Support:** Although ContainerLab supports several types of network containers, it may not have the same level of multivendor support as GNS3.
2. **Learning Curve:** For those unfamiliar with containerization concepts, the learning curve can be steeper.

## Conclusion 📊

**GNS3** is ideal for those looking for an intuitive graphical interface and broad support for network devices, particularly useful for students and traditional network engineers. **ContainerLab**, on the other hand, is more suited to modern environments and DevOps practices, offering a lightweight and scriptable solution for network simulation.

The choice between GNS3 and ContainerLab thus primarily depends on the specific needs of the user in terms of flexibility, performance, and integration with other tools and technologies.
