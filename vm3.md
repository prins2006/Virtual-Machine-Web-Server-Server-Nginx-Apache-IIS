# Virtualization

## Table of Contents

1. Introduction
2. What is Virtualization?
3. Why Virtualization?
4. How Virtualization Works
5. Components of Virtualization
6. Types of Virtualization
7. Virtualization Architecture
8. Hypervisor
9. Type 1 vs Type 2 Hypervisor
10. Advantages
11. Disadvantages
12. Real-World Examples
13. Virtualization vs Virtual Machine
14. Virtualization in Cloud Computing
15. Best Practices
16. Summary

---

# 1. Introduction

**Virtualization** is the technology of creating a virtual (software-based) version of a physical resource instead of using the physical resource directly.

These resources can include:

- Servers
- Operating Systems
- Storage
- Networks
- Applications
- Desktops

Virtualization allows one physical computer to run multiple independent virtual environments.

---

# 2. What is Virtualization?

Virtualization is the process of dividing the resources of a physical machine into multiple isolated virtual environments.

Instead of dedicating one physical server to one application, virtualization enables many Virtual Machines (VMs) to share the same hardware efficiently.

### Example

Without Virtualization

```
Application A → Server 1

Application B → Server 2

Application C → Server 3
```

With Virtualization

```
One Physical Server

├── VM 1 → Application A
├── VM 2 → Application B
└── VM 3 → Application C
```

---

# 3. Why Virtualization?

Companies adopt virtualization because it offers many business and technical benefits.

### Benefits

- Better hardware utilization
- Lower infrastructure costs
- Reduced power consumption
- Faster server deployment
- Easy backup and recovery
- High availability
- Disaster recovery
- Easy scalability
- Application isolation
- Foundation of cloud computing

---

# 4. How Virtualization Works

Virtualization is enabled by a **Hypervisor**.

The hypervisor sits between the physical hardware and the virtual machines.

```
+--------------------------------------+
| Applications                         |
+--------------------------------------+
| Guest Operating Systems              |
+--------------------------------------+
| Virtual Machines                     |
+--------------------------------------+
| Hypervisor                           |
+--------------------------------------+
| Physical Hardware                    |
+--------------------------------------+
```

The hypervisor shares CPU, RAM, storage, and network resources among all virtual machines.

---

# 5. Components of Virtualization

## 1. Physical Hardware

The real machine that provides resources.

Examples:

- CPU
- RAM
- Hard Disk
- SSD
- Network Card

---

## 2. Hypervisor

Software that creates and manages virtual machines.

Responsibilities:

- Allocate CPU
- Allocate RAM
- Manage storage
- Create virtual hardware
- Isolate VMs

---

## 3. Virtual Machines (VMs)

Independent software computers.

Each VM has:

- Operating System
- CPU
- RAM
- Storage
- Network

---

## 4. Guest Operating System

The operating system installed inside the VM.

Examples:

- Ubuntu
- Windows Server
- Rocky Linux
- Debian

---

## 5. Virtual Hardware

Each VM receives virtual hardware such as:

- Virtual CPU
- Virtual RAM
- Virtual Disk
- Virtual Network Interface
- Virtual BIOS

---

# 6. Types of Virtualization

## 6.1 Server Virtualization

Creates multiple virtual servers from one physical server.

Example

```
One Server

↓

Web VM

Database VM

Mail VM
```

Most common virtualization type.

---

## 6.2 Desktop Virtualization

Runs desktop operating systems inside virtual machines.

Example

```
Windows Desktop

↓

Virtual Desktop

↓

Users connect remotely
```

Used in organizations and schools.

---

## 6.3 Storage Virtualization

Combines multiple storage devices into one logical storage pool.

Example

```
Disk 1

Disk 2

Disk 3

↓

Storage Pool
```

---

## 6.4 Network Virtualization

Creates virtual networks independent of physical hardware.

Examples

- VLAN
- Virtual Switch
- SDN (Software Defined Networking)

---

## 6.5 Application Virtualization

Applications run without being installed directly on the host operating system.

Example

Microsoft App-V

---

## 6.6 Operating System Virtualization

Multiple isolated environments share the same operating system kernel.

Examples

- Docker
- LXC
- Podman

This is also called **containerization**.

---

# 7. Virtualization Architecture

```
Applications
      │
Guest Operating Systems
      │
Virtual Machines
      │
Hypervisor
      │
Physical Hardware
```

---

# 8. Hypervisor

A Hypervisor is software that creates and manages Virtual Machines.

Main functions

- CPU scheduling
- Memory allocation
- Storage management
- Network management
- Resource isolation

Without a hypervisor, virtualization cannot work.

---

# 9. Type 1 vs Type 2 Hypervisor

## Type 1 (Bare Metal)

Runs directly on hardware.

```
Hardware

↓

Hypervisor

↓

Virtual Machines
```

Examples

- VMware ESXi
- Hyper-V
- Xen
- KVM

Advantages

- High performance
- Better security
- Used in production

---

## Type 2 (Hosted)

Runs on an operating system.

```
Hardware

↓

Operating System

↓

VirtualBox

↓

Virtual Machines
```

Examples

- Oracle VirtualBox
- VMware Workstation
- VMware Fusion

Advantages

- Easy installation
- Best for learning
- Home labs

---

# 10. Advantages

## Better Hardware Utilization

Run multiple VMs on one server.

---

## Cost Reduction

Fewer physical servers.

Lower electricity bills.

Lower cooling costs.

---

## Isolation

One VM failure does not affect other VMs.

---

## Fast Deployment

Clone an existing VM in minutes.

---

## Disaster Recovery

Restore VMs quickly from backups.

---

## Scalability

Create or remove VMs whenever needed.

---

## Easy Testing

Snapshots allow safe testing before making changes.

---

## Cloud Ready

Virtualization is the foundation of cloud computing.

---

# 11. Disadvantages

- Requires powerful hardware
- Uses significant RAM
- Slight performance overhead
- Large VM disk files
- Licensing costs for some enterprise hypervisors
- Management complexity in large environments

---

# 12. Real-World Examples

## Software Development

Developers test applications on multiple operating systems.

---

## DevOps

Run separate VMs for:

- Jenkins
- GitLab
- Docker
- Monitoring
- Kubernetes Lab

---

## Banking

Virtualize servers for better resource utilization and disaster recovery.

---

## Healthcare

Host hospital applications securely with isolated workloads.

---

## Education

Students practice Linux, Windows Server, and networking without needing multiple physical computers.

---

# 13. Virtualization vs Virtual Machine

| Virtualization | Virtual Machine |
|----------------|-----------------|
| Technology | Software computer |
| Creates virtual resources | Uses virtual resources |
| Requires a hypervisor | Runs on a hypervisor |
| One virtualization platform can create many VMs | A VM is one instance created by virtualization |

### Simple Example

```
Virtualization

↓

Creates

↓

VM 1

VM 2

VM 3
```

Virtualization is the **technology**, while a Virtual Machine is the **result** of that technology.

---

# 14. Virtualization in Cloud Computing

Cloud providers use virtualization to create virtual servers for customers.

Examples

- AWS EC2
- Microsoft Azure Virtual Machines
- Google Compute Engine
- Oracle Cloud Infrastructure

When you launch a cloud server, the provider creates a Virtual Machine on shared physical hardware.

---

# 15. Best Practices

- Allocate only the required CPU and RAM.
- Avoid overcommitting resources.
- Keep hypervisors updated.
- Use snapshots only for short-term rollback.
- Perform regular VM backups.
- Monitor CPU, memory, and storage usage.
- Use SSD storage for better VM performance.
- Secure management interfaces.
- Separate production and testing workloads.

---

# 16. Summary

Virtualization is the technology that allows multiple isolated virtual environments to run on a single physical computer. It improves hardware utilization, reduces costs, simplifies management, and provides flexibility for modern IT infrastructure.

Today, virtualization is widely used in:

- Cloud Computing
- DevOps
- Data Centers
- Enterprise IT
- Software Development
- Cybersecurity Labs
- Testing and Training

Without virtualization, modern cloud platforms, enterprise data centers, and Infrastructure as a Service (IaaS) would not be possible.