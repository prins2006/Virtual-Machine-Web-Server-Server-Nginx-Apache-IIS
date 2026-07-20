# Virtual Machines (VMs)

## Table of Contents

1. Introduction
2. What is a Virtual Machine?
3. Physical Machine vs Virtual Machine
4. How a Virtual Machine Works
5. Hypervisor
6. Types of Hypervisors
7. Components of a Virtual Machine
8. Virtual Machine Lifecycle
9. Virtual Machine States
10. Resource Allocation
11. Networking Modes
12. Snapshots
13. Cloning
14. VM Templates
15. Advantages
16. Disadvantages
17. Real-World Use Cases
18. Virtual Machine vs Container
19. Example: Ubuntu VM on VirtualBox
20. Virtual Machines in Cloud Computing
21. Best Practices
22. Summary

---

# 1. Introduction

A **Virtual Machine (VM)** is a software-based computer that behaves exactly like a physical computer. It has its own operating system, CPU, memory, storage, and network interface, but all of these are virtual and created by software.

Using virtualization, a single physical computer can run multiple operating systems at the same time.

Example:

- Host OS → Windows 11
- VM 1 → Ubuntu 24.04
- VM 2 → Windows Server 2022
- VM 3 → Kali Linux

All three operating systems can run simultaneously on one physical machine.

---

# 2. What is a Virtual Machine?

A Virtual Machine is an isolated environment that shares the hardware resources of the host computer.

Each VM has its own:

- Virtual CPU (vCPU)
- Virtual RAM
- Virtual Hard Disk
- Virtual Network Card
- BIOS/UEFI
- Operating System

Although these resources are virtual, the guest operating system treats them as real hardware.

---

# 3. Physical Machine vs Virtual Machine

| Physical Machine | Virtual Machine |
|------------------|-----------------|
| Real hardware | Virtual hardware |
| One operating system | Multiple operating systems |
| Cannot be copied easily | Can be cloned easily |
| Hardware failure affects system | Easy backup and recovery |
| Expensive | Cost-effective |

---

# 4. How a Virtual Machine Works

```
+----------------------------------------+
| Applications                           |
+----------------------------------------+
| Guest Operating System                 |
+----------------------------------------+
| Virtual Hardware                       |
| CPU | RAM | Disk | Network | BIOS      |
+----------------------------------------+
| Hypervisor                             |
+----------------------------------------+
| Physical Hardware                      |
| CPU | RAM | Storage | Network          |
+----------------------------------------+
```

The hypervisor creates virtual hardware.

The guest operating system believes it is running on a physical computer.

---

# 5. Hypervisor

A **Hypervisor** is software that creates, manages, and runs Virtual Machines.

Its responsibilities include:

- CPU allocation
- Memory allocation
- Disk management
- Network virtualization
- Device virtualization
- Resource isolation

Without a hypervisor, virtual machines cannot run.

---

# 6. Types of Hypervisors

## Type 1 (Bare Metal)

Runs directly on physical hardware.

```
Hardware
    │
Hypervisor
    │
VM1
VM2
VM3
```

Examples:

- VMware ESXi
- Microsoft Hyper-V Server
- Xen
- KVM

### Advantages

- High performance
- Better security
- Used in production
- Enterprise data centers

---

## Type 2 (Hosted)

Runs on top of an operating system.

```
Hardware
     │
Host Operating System
     │
VirtualBox
VMware Workstation
     │
Virtual Machines
```

Examples:

- Oracle VirtualBox
- VMware Workstation
- VMware Fusion
- Parallels Desktop

### Advantages

- Easy to install
- Best for learning
- Personal labs
- Software testing

---

# 7. Components of a Virtual Machine

## 7.1 Virtual CPU (vCPU)

The hypervisor assigns CPU cores to the VM.

Example

Physical CPU

```
8 Core CPU
```

Allocation

```
VM1 → 2 vCPU

VM2 → 4 vCPU

VM3 → 2 vCPU
```

---

## 7.2 Virtual RAM

Memory allocated to the VM.

Example

Host RAM

```
32 GB
```

Allocation

```
VM1 → 8 GB

VM2 → 8 GB

VM3 → 4 GB

Host → 12 GB
```

---

## 7.3 Virtual Hard Disk

Each VM stores data inside a virtual disk file.

Common formats:

| Hypervisor | Disk Format |
|------------|------------|
| VirtualBox | .vdi |
| VMware | .vmdk |
| Hyper-V | .vhdx |

---

## 7.4 Virtual Network Adapter

Every VM has its own virtual network interface.

Linux examples

```
eth0

ens33

enp0s3
```

---

## 7.5 Virtual BIOS

The VM also has BIOS/UEFI.

Functions:

- Hardware initialization
- Boot loader execution
- Operating system startup

---

# 8. Virtual Machine Lifecycle

```
Create VM
      │
Install Operating System
      │
Power On
      │
Use VM
      │
Suspend
      │
Resume
      │
Shutdown
      │
Delete
```

---

# 9. Virtual Machine States

## Running

VM is powered on.

---

## Powered Off

VM is completely shut down.

---

## Paused

CPU execution stops temporarily.

---

## Saved

Memory is written to disk.

VM resumes from the same point.

---

## Suspended

Similar to sleep mode.

---

# 10. Resource Allocation

Suppose your laptop has:

```
16 GB RAM

8 CPU Cores

500 GB SSD
```

Create two VMs

Ubuntu

```
RAM = 4 GB

CPU = 2

Disk = 50 GB
```

Windows Server

```
RAM = 8 GB

CPU = 4

Disk = 100 GB
```

Remaining resources stay with the host operating system.

---

# 11. Networking Modes

## NAT

```
Internet
    │
Host
    │
VM
```

Features

- Internet access available
- External devices cannot directly access the VM
- Best for beginners

---

## Bridged Adapter

```
Router
   │
Host
   │
VM
```

Features

- VM gets its own IP address
- Accessible from other devices
- Good for servers

---

## Host-only Adapter

```
Host <------> VM
```

Features

- No internet
- Communication only between host and VM

---

## Internal Network

```
VM1 <----> VM2 <----> VM3
```

Features

- Only VMs communicate
- Host cannot access

---

# 12. Snapshots

A snapshot saves the current state of a VM.

It stores:

- Disk
- Memory (optional)
- VM configuration

Example

```
Install Ubuntu

↓

Take Snapshot

↓

Install Docker

↓

Configuration breaks

↓

Restore Snapshot
```

The VM returns to its previous working state.

---

# 13. Cloning

Cloning creates a copy of an existing VM.

```
Ubuntu VM

↓

Clone

↓

Ubuntu Copy
```

Types

## Full Clone

Independent copy.

## Linked Clone

Depends on parent VM.

Uses less storage.

---

# 14. VM Templates

Templates are preconfigured VMs.

Example

```
Ubuntu

Docker

Git

VS Code

↓

Save as Template

↓

Deploy 100 identical VMs
```

Used in cloud environments.

---

# 15. Advantages

- Run multiple operating systems
- Better hardware utilization
- Isolation between applications
- Easy backup
- Easy cloning
- Fast deployment
- Cost-effective
- Safe testing environment
- Disaster recovery

---

# 16. Disadvantages

- Requires high RAM
- Uses CPU resources
- Storage consumption
- Slight performance overhead
- GPU virtualization can be limited

---

# 17. Real-World Use Cases

## Development

Developers test applications on different operating systems.

---

## DevOps

Separate VMs for:

- Jenkins
- GitLab
- Docker
- Kubernetes
- Monitoring

---

## Cyber Security

Safe environment for malware analysis and penetration testing.

---

## Education

Students practice:

- Linux
- Windows Server
- Networking
- Shell scripting

---

## Enterprise

Organizations run:

- Web servers
- Database servers
- Mail servers
- Active Directory
- Application servers

---

# 18. Virtual Machine vs Container

| Feature | Virtual Machine | Container |
|----------|-----------------|-----------|
| Operating System | Full OS | Shares host kernel |
| Startup Time | Minutes | Seconds |
| Size | GB | MB |
| Isolation | Strong | Moderate |
| Performance | Slight overhead | Near native |
| Best Use | Different operating systems | Microservices |

---

# 19. Example: Ubuntu VM on VirtualBox

Host Machine

```
Windows 11

16 GB RAM

Intel i7

512 GB SSD
```

Create VM

```
Name : Ubuntu 24.04

RAM : 4096 MB

CPU : 2

Disk : 50 GB

Network : NAT
```

Installation Steps

1. Install VirtualBox.
2. Download Ubuntu ISO.
3. Create a new VM.
4. Attach the ISO.
5. Install Ubuntu.
6. Install Guest Additions.
7. Take a snapshot.
8. Install Docker or Nginx for practice.

---

# 20. Virtual Machines in Cloud Computing

Cloud providers use virtualization to create Virtual Machines on demand.

Examples

- AWS EC2
- Microsoft Azure Virtual Machines
- Google Compute Engine
- Oracle Cloud Infrastructure

Benefits

- Pay only for what you use
- Easy scaling
- High availability
- Automatic backups
- Disaster recovery

---

# 21. Best Practices

- Allocate only the required RAM and CPU.
- Keep snapshots temporary.
- Use backups instead of relying only on snapshots.
- Install Guest Additions or VMware Tools.
- Keep the guest OS updated.
- Use bridged networking only when needed.
- Monitor CPU, RAM, and disk usage.
- Shut down unused VMs to save resources.
- Store VM disk files on SSDs for better performance.

---

# 22. Summary

A Virtual Machine is a software-based computer that runs on a physical computer using a hypervisor. It allows multiple operating systems to share the same hardware while remaining isolated from each other.

Virtual Machines are widely used in:

- DevOps
- Cloud Computing
- Software Development
- Testing
- Cyber Security
- Enterprise Data Centers
- Education

Learning Virtual Machines is one of the most important skills for Linux, DevOps, Cloud Computing, and System Administration because almost every modern cloud platform relies on virtualization technology.