---
title: "VXLAN for Beginners"
date: 2024-08-01T20:00:00+02:00
---

## Understanding VLAN and VXLAN: Simplified for Non-Technicians

In the fast-paced world of technology, understanding networking concepts can be intimidating, especially if you’re not an expert in the field.  
  
Today, we’ll break down two important network concepts: **VLAN** and **VXLAN**, using simple analogies and clear explanations.  
  
We’ll also cover their limitations, real-world use cases, and some technical points for the more curious readers.

Let’s go! 🚀

---

## What is a VLAN? 🏢

**VLAN (Virtual Local Area Network)** is like organizing a large office building with multiple departments: Marketing, Sales, HR, and IT. To keep things orderly, each department gets its own floor. This way, Marketing stays on its floor, Sales stays on theirs, and so on.

A **VLAN** works similarly for computer networks. It divides a large physical network into smaller, isolated networks. Each VLAN is like a separate floor for a department, allowing devices within the same VLAN to communicate easily while keeping traffic separate from other VLANs.

### Key points about VLAN ✅

- **Segregation:** Keeps different groups (like departments) separate.  
- **Efficiency:** Reduces unnecessary traffic and potential network issues.  
- **Security:** Limits access and enhances security by isolating groups.

### VLAN Limitations ⚠️

- **ID Limit:** Historically, a VLAN is identified by a 12-bit field, allowing up to 4094 VLANs (1 to 4094). For a large company or a datacenter, this might be insufficient.  
- **Local Isolation:** VLANs are primarily designed for local use (on the same site or a connected set of switches). Extending this concept to multiple sites requires more advanced solutions.

---

## What is VXLAN? 🌆

**VXLAN (Virtual Extensible LAN)** goes further. Imagine your company grows and expands to multiple office buildings across the city. You still want each department to feel as if it’s on its own floor, even though they’re now spread out across different locations. To do this, you create a virtual system that connects all floors across buildings so that Marketing on the 3rd floor of one building is still virtually connected to Marketing on the 3rd floor of another building.

**VXLAN** does this for networks. It extends VLANs across multiple physical locations using a technique called **tunneling**. This allows devices in the same VLAN to communicate as if they were on the same local network, even if they’re geographically distant.

### Key points about VXLAN ⭐

- **Scalability:** Extends networks to different locations and surpasses the 4094 VLAN limit.  
- **Flexibility:** Enables larger, more dynamic network designs.  
- **Connectivity:** Ensures seamless communication across dispersed networks.

---

## Technical Dive into VXLAN 🔍

**VXLAN** was developed to address the limitations of traditional VLANs (scalability, geographic reach). It uses a **VNI** (VXLAN Network Identifier) with 24 bits to identify up to **16 million** logical segments, far exceeding the 4094 VLAN limit.

Due to virtualization, MAC address tables in datacenters can grow very large, while physical switches have limited capacity. VXLAN addresses this challenge by using **MAC-in-UDP** encapsulation, allowing Ethernet frames (Layer 2) to be transported over an IP network (Layer 3).

### How does it work? 🤔

The goal of **VXLAN** is to **extend Layer 2** across a Layer 3 (IP) network. Essentially, it “tricks” Layer 3 into believing that the user or virtual machine is still on the same local (Layer 2) network.

> **In simple terms:** Ethernet frames (Layer 2) are encapsulated inside a UDP packet (Layer 4), which is then carried by IP (Layer 3).

---

![OSI Layers](media_layers.png#center)

> ✏️ **The “Physical” Layers**  
>
> - The **Link Layer (Layer 2)** is typically managed by switches.  
> - The **Network Layer (Layer 3)** is typically managed by routers.  

By encapsulating Layer 2 inside Layer 3, you benefit from the advantages of IP routing (flexibility, scalability) while preserving the isolation and simplicity of Layer 2 for applications and virtual machines.

---

### VXLAN Explained by the Container Transport Analogy 🚚 🚂

#### 1. Trucks (lower layers)  

Imagine trucks on the road. Their job is to transport containers (your data) from Point A to Point B. These trucks represent the **Ethernet layer** (Layer 2), where each vehicle (frame) has a “license plate” (MAC address).

#### 2. The Train (VXLAN tunnel)  

When it’s time to travel longer distances or through different infrastructures, loading the trucks onto a train becomes more efficient. Here, **the train represents VXLAN**: it encapsulates the trucks (Ethernet frames) into a wagon (the tunnel). Each train is identified by a **VNI (VXLAN Network Identifier)**, much like a convoy number for each freight line.

#### 3. The Railway Tracks (IP network)  

The train runs on rails (the **IP network**, Layer 3). The railroad is already built and managed to find the best path; it ensures route convergence and can reroute traffic in case of issues (breakdowns, congestion, etc.). Similarly, the IP network automatically selects the optimal path to transport VXLAN packets.

---

### Key Takeaways

- **Overlay:** VXLAN is a transport system “on top of” Layer 3 (the rails). It interconnects multiple Layer 2 networks (the trucks) as if they were just one.  
- **Dual Addressing:**  
  - Trucks (Ethernet frames) are identified by **MAC addresses** (license plates).  
  - The train (VXLAN tunnel) uses **IP addresses** (routing plans) to travel on the rails.  
- **Isolation and Segmentation:** Just like multiple trains can run on the same railway line, you can have several VXLAN tunnels (each with its own VNI) over the same IP infrastructure.  
- **Elasticity and Reliability:** By relying on Layer 3, VXLAN takes advantage of all IP routing optimizations (route recalculations, fault tolerance, etc.).

![Container transport](transports.png#center)

---

## Real-World Use Cases 🏭

- **Multi-datacenter:** For connecting multiple geographically dispersed data centers while preserving the feel of a single Layer 2 network.  
- **Hybrid Cloud:** Extending a corporate network to a public or private cloud provider without reconfiguring the entire address plan.  
- **Virtual Machine Migration:** Enabling VM mobility between distant sites without losing Layer 2 connectivity.  
- **Massive Virtualization:** In highly dense environments (e.g., hundreds of thousands of virtual machines), the 24-bit VNI is indispensable.

---

## VXLAN Control: BGP EVPN and Other Protocols 🤝

In modern deployments, especially in data centers, VXLAN isn’t just configured statically. It’s often paired with a **control plane** via **BGP EVPN (Ethernet VPN)**.

- **BGP EVPN:** Exchanges MAC and IP table information between devices, facilitating automation and scalability.  
- **Other Technologies:** Historically, other overlay protocols (NVGRE, STT) existed, but VXLAN has become the de facto standard.

---

## Performance Considerations ⚙️

- **Encapsulation Overhead:** VXLAN adds an extra header (8 bytes + UDP/IP header). This can affect the **Maximum Transmission Unit (MTU)** size, often requiring **Jumbo MTU** (usually 9000 bytes) to avoid packet fragmentation.  
- **Resilience of the IP Network:** The tunnel’s reliability depends on the underlying IP network’s quality (routes, congestion, etc.).

---

## Example Configuration (for the curious) 💡

Below is a **simplified excerpt** of a VXLAN configuration on a Cisco NX-OS device (syntax can vary by vendor):

```plaintext
interface nve1
  no shutdown
  source-interface loopback1
  member vni 5001
    ingress-replication protocol static
    mcast-group 239.1.1.1
```

- **interface nve1:** Creates an “NVE” (Network Virtualization Endpoint) interface to handle VXLAN encapsulation.  
- **source-interface loopback1:** The IP address of loopback1 is used to establish tunnels.  
- **member vni 5001:** Associates a specific VXLAN Network Identifier (VNI) with the overlay network.

*Note:* In more complex environments, the control plane (e.g., BGP EVPN) is also configured.

---

## Summary 🎯

- **VLAN**  
  It’s like having separate floors for different departments in a building, keeping their activities isolated. 🏢  
  \- **Major limitation:** A maximum of 4094 VLANs and scope often limited to a single site.

- **VXLAN**  
  It’s like connecting those separate floors across multiple buildings while maintaining the illusion they’re in one building. 🌆  
  \- **Key advantages:** Huge addressing capability (16 million segments), L2 extension over L3, flexibility for virtualization and multi-site.

**VXLAN** addresses the need for large-scale isolation, overcomes the limitations of switch MAC address tables, and allows for flexible deployment of services. Paired with an efficient control plane (BGP EVPN), it greatly simplifies the management of modern overlay networks.

---

### Conclusion 🏁

In short, if you need **basic segmentation** for your local network, **VLAN** is more than enough. But as soon as you want to connect multiple sites, build a highly virtualized network, or exceed the traditional 4094 VLAN limit, **VXLAN** becomes essential.

Whether you’re a **network lab** enthusiast, a NetOps engineer, or simply curious about the underlying infrastructure, understanding these two concepts will help you better grasp the magic that happens when your data travels farther and farther while preserving the illusion of being “at home” on the same local network!

---

> **Want to learn more?**  
>
> - Check out **BGP EVPN** for VXLAN’s control plane.  
> - Look into **Jumbo MTU configuration** to optimize performance.  
> - Compare VXLAN with other protocols (NVGRE, GENEVE) to understand network design choices.
