---
title: "VXLAN for Beginners"
date: 2024-08-02T20:00:00+02:00
draft: false
categories: "Documentations"
tags:
    - Network
    - VXLAN
    - Architecture
UseHugoToc: true
TocOpen: false
showToc: true
cover: 
  image: "/img/cover_vxlan_dummies.webp"
  caption: "Building the Future of Networking: Exploring VLANs and VXLANs Across Scalable Digital Landscapes"
---

## Understanding VLAN and VXLAN: Simplified for Non-Techies

In today’s fast-paced tech world, understanding networking concepts can feel intimidating, especially if you're not an expert.  
Let's break down two important networking concepts: **VLAN** and **VXLAN**, using simple analogies and clear explanations.  
We’ll also discuss their limitations, real-world use cases, and some technical details for the curious ones.

Let’s dive in! 🚀

---

## What is a VLAN? 🏢

A **VLAN (Virtual Local Area Network)** is like organizing a large office building with multiple departments: Marketing, Sales, HR, and IT. To maintain order, each department gets its own floor. This way, Marketing stays on its floor, Sales on theirs, and so on.

A **VLAN** works similarly for computer networks. It divides a large physical network into smaller, isolated networks. Each VLAN is like a separate floor for a department, allowing devices within the same VLAN to communicate easily while keeping traffic isolated from other VLANs.

### Key Points About VLANs ✅

- **Separation:** Keeps different groups (like departments) apart.  
- **Efficiency:** Reduces unnecessary traffic and potential network issues.  
- **Security:** Enhances security by isolating groups.

### VLAN Limitations ⚠️

- **ID Limit:** Historically, a VLAN is identified by 12 bits, allowing up to 4094 VLANs (from 1 to 4094). For large enterprises or data centers, this might be insufficient.  
- **Local Isolation:** VLANs are designed for local use (a single site or connected switches). Expanding this concept across multiple sites requires advanced solutions.

---

## What is VXLAN? 🌆

**VXLAN (Virtual Extensible LAN)** takes things further. Imagine your business grows and spreads across multiple buildings in the city. You still want departments to feel like they’re on their own floors, even if they’re in different locations. To achieve this, you create a virtual system that connects all the floors across buildings, so Marketing on the 3rd floor of one building remains virtually connected to Marketing on the 3rd floor of another.

**VXLAN** does this for networks. It extends VLANs across multiple physical locations using a technique called **tunneling**. This way, devices in the same VLAN can communicate as if they’re on the same local network, even when geographically separated.

### Key Points About VXLAN ⭐

- **Scalability:** Extends networks to different locations and exceeds the 4094 VLAN limit.  
- **Flexibility:** Enables larger and more dynamic network designs.  
- **Connectivity:** Ensures smooth communication across dispersed networks.

---

## Technical Dive into VXLAN 🔍

**VXLAN** was developed to overcome the limitations of traditional VLANs (scalability, geographic scope). It uses a VXLAN Network Identifier (**VNI**) of 24 bits to identify up to **16 million** logical segments, far surpassing the 4094 VLAN limit.

In virtualized environments, MAC address tables in data centers can become very large, while physical switches have limited capacity. VXLAN addresses this challenge by using **MAC-in-UDP encapsulation**, transporting Ethernet frames (Layer 2) over an IP network (Layer 3).

### How It Works 🤔

The goal of **VXLAN** is to **extend Layer 2** over a Layer 3 (IP) network. It’s like “tricking” Layer 3 into thinking the user or virtual machine is still on the same local network (Layer 2).

> **Simply put:** Ethernet frames (Layer 2) are encapsulated inside a UDP packet (Layer 4), which is then transported over IP (Layer 3).  

---

![OSI Layers](media_layers.png#center)

> ✏️ **The "Hardware" Layers**  
>
> - **Layer 2 (Data Link):** Typically managed by switches.  
> - **Layer 3 (Network):** Typically managed by routers.  

By encapsulating Layer 2 within Layer 3, you benefit from IP routing’s flexibility and scalability while maintaining the isolation and simplicity of Layer 2 for applications and virtual machines.

---

### Analogy: Container Transport 🚚 🚂

- **Lower Layers (trucks):** Transport data (containers) from point A to point B.  
- **VXLAN (train):** Loads these trucks (Ethernet packets) onto a train (VXLAN tunnel) to travel longer distances across Layer 3.  
- **Railways (IP network):** Are already “converged” and determine the best route for the trains (VXLAN tunnels).

![Container transport analogy](transports.png#center)

---

## Real-World Use Cases 🏭

- **Multi-Data Center:** Connect geographically dispersed data centers while maintaining the feel of a single Layer 2 network.  
- **Hybrid Cloud:** Extend a corporate network to a public or private cloud without reconfiguring the entire IP plan.  
- **Virtual Machine Migration:** Enable VM mobility between distant sites without losing Layer 2 connectivity.  
- **Massive Virtualization:** In highly dense environments (e.g., hundreds of thousands of virtual machines), the 24-bit VNI is essential.

---

## VXLAN Control: BGP EVPN and Other Protocols 🤝

In modern deployments, especially in data centers, VXLAN isn’t configured manually. It’s often paired with a **control plane** using the **BGP EVPN (Ethernet VPN)** protocol.  

- **BGP EVPN:** Exchanges MAC and IP table information between devices, enabling automation and scalability.  
- **Other Technologies:** Historically, other overlay protocols (e.g., NVGRE, STT) existed, but VXLAN has become the de facto standard.

---

## Performance Considerations ⚙️

- **Encapsulation Overhead:** VXLAN adds extra headers (8 bytes + UDP/IP header). This can impact the **Maximum Transmission Unit (MTU)**, and it’s common to configure **Jumbo MTU** (usually 9000 bytes) to avoid packet fragmentation.  
- **IP Network Resilience:** The reliability of VXLAN tunnels depends on the quality of the underlying IP network (routes, congestion, etc.).

---

## Example Configuration (For the Curious) 💡

Here’s a **simplified example** of VXLAN configuration on Cisco NX-OS (syntax varies by vendor):

```plaintext
interface nve1
  no shutdown
  source-interface loopback1
  member vni 5001
    ingress-replication protocol static
    mcast-group 239.1.1.1
```

- **interface nve1:** Creates an “NVE” (Network Virtualization Endpoint) interface for VXLAN encapsulation.  
- **source-interface loopback1:** The IP address of the loopback1 interface is used to establish tunnels.  
- **member vni 5001:** Associates a VNI (VXLAN Network Identifier) with the overlay network.  

*Note:* In more complex environments, a control plane (e.g., BGP EVPN) is also configured.

---

## Summary 🎯

- **VLAN**  
  Like having separate floors for different departments in a building, keeping activities isolated. 🏢  
  - **Main Limitation:** 4094 VLANs maximum, often limited to a single site.

- **VXLAN**  
  Like connecting these separate floors across multiple buildings while keeping the illusion they’re in the same building. 🌆  
  - **Key Advantages:** Massive address capacity (16 million segments), Layer 2 over Layer 3 extension, flexibility for virtualization and multi-site connectivity.

VXLAN addresses the need for large-scale isolation, surpasses MAC address table limits on switches, and enables flexible service deployment. Combined with an efficient control plane (BGP EVPN), it simplifies the management of modern overlay networks.

---

### Conclusion 🏁

In short, if you need **basic segmentation** for your local network, a **VLAN** is sufficient. But when connecting multiple sites, creating a highly virtualized network, or exceeding the traditional VLAN limit, **VXLAN** becomes essential.

Whether you’re a **network lab enthusiast**, a NetOps engineer, or simply curious about the infrastructure magic that allows your data to travel seamlessly across distances, understanding these concepts will give you a better appreciation for the illusion of being "at home" on the same local network!

---

> **Want to Go Further?**  
>
> - Explore **BGP EVPN** for VXLAN control plane management.  
> - Learn about **Jumbo MTU configuration** to optimize performance.  
> - Compare VXLAN with other protocols (NVGRE, GENEVE) to understand design choices.
