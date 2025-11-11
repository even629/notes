---
typora-copy-images-to: ./${filename}.assets
---

# ARM64虚拟化

## 芯片手册

- **Arm Architecture Reference Manual Armv8, for Armv8-A architecture profile, v8.6**
  - 关于虚拟化的内容：D1.5
  - 关于异常的内容：
    - D1.1 Exception levels
    - D1.2 Exception terminology
    - D1.10 Exception entry
    - D1.11 Exception return
    - D1.12 Synchronous exception types, routing and priorities
    - D1.13 Asynchronous exception types, routing, masking and priorities
    - D13.2.37 ESR_EL2, Exception Syndrome Register(EL2)
    - D13.2.138 VBAR_EL2, Vector Base Address Register(EL2)
    - D13.2.47 HCR_EL2, Hypervisor Configuration Register
    - FAR_EL2 Fault Address Register(EL2)
    - D13.2.53 HPFAR_EL2, Hypervisor IPA Fault Address Register
  - 关于内存管理的内容
    - D5.2.3 Controlling address translation stages
    - D5.2.6 Overview of the VMSAv8-64 address translation stages
    - D5.2.7 The VMSAv8-64 translation table format
    - D5.3 VMSAv8-64 translation table format descriptors
    - D13.2.61 ID_AA64MMFR0_EL1, AArch64 Memory Model Feature Register 0
    - D13.2.146 VTTBR_EL2, Virtualization Translation Table Base Register
    - D13.2.145 VTCR_EL2, Virtualization Translation Control Register
- **Armv8-A virtualization**
- **ARM Generic Interrupt Controller Architecture version 2.0, Architecture Specification**





## 虚拟化介绍

### 虚拟化技术

虚拟化技术作为一种资源管理技术，能对计算机的各种物理资源（比如CPU、内存、I/O设备等）进行抽象组合并分配给多个虚拟机

- **平台虚拟化**（**platform virtualization**）：针对计算机和操作系统的虚拟化，例如KVM等。
- **资源虚拟化**（**resource virtualization**）：针对特定系统资源的虚拟化，包括内存、存储器、网络资源等，例如容器技术。
- **应用程序虚拟化**（**application virtualization**）：包括仿真、模拟、解释技术等，如Java 虚拟机。



### 虚拟化应用场景

- 桌面虚拟化
- 服务器虚拟化
- 嵌入式虚拟化



### 虚拟化技术的重要理论基础

1974年：论文"Formal Requirements for Virtualizable Third Generation Architectures"

实现虚拟化的3个必要的要素：

- 资源控制（resource control）。VMM必须能够管理所有的系统资源。
- 等价性（equivalence）。客户机的运行行为与在裸机上一致。
- 效率性（efficiency）。客户机运行的程序不受VMM的干涉。



x86架构在实现虚拟化的过程中遇到了一些挑战，特别是不能满足上述第二个条件。计算机架构里包含两种指令。

- 敏感指令：操作某些特权资源的指令，比如访问、修改虚拟机模式或机器状态的指令。
- 特权指令：具有特殊权限的指令。这类指令只用于操作系统或其他系统软件，一般不
  直接提供给用户使用。

### 硬件辅助虚拟化技术

- 2005年，Intel开始在CPU中引入硬件虚拟化技术，这项技术称为VT（virtualization technology）。
- VT的基本思想是创建可以运行虚拟机的容器。在使能了VT的CPU里有两种操作模式——根（VMX root）模式和非根（VMX non-root）模式。这两种操作模式都支持Ring 0～Ring 3这4个特权级别，因此虚拟机管理程序和虚拟机都可以自由选择它们期望的运行级别。

![Intel根模式与非跟模式](virtualization.assets/image-20251029200156809.png)

![Intel Rring0-Ring3](virtualization.assets/image-20251029200233375.png)

### 全虚拟化与半虚拟化

![全虚拟化与半虚拟化](virtualization.assets/image-20251029200316869.png)

目前主要用硬件辅助虚拟化技术，这两项技术都不怎么用了

### 虚拟机的分类

- **Hypervisor**：虚拟机管理程序，也叫做VMM（Virtual Machine Manager），Hypervisor位于计算机硬件和虚拟机之间，负责管理和分配计算机资源给各个虚拟机。

  - Type1：第一类虚拟机管理程序就像小型操作系统，目的就是管理所有的虚拟机，常见的虚拟化软件有Xen、ACRN等。

  ![Type1 Hypeervisor](virtualization.assets/image-20251029200540779.png)

  - Type2：第二类虚拟机管理程序依赖于Windows、Linux等操作系统来分配和管理调度资源，常见的虚拟化软件有VMware Player、KVM以及Virtual Box等。

![Type2 Hypervisor](virtualization.assets/image-20251029200605696.png)

### 内存虚拟化

- 软件模拟：影子页表（Shadow Page）

  - 效率低

![影子页表](virtualization.assets/image-20251029200930123.png)

- 硬件内存虚拟化技术—
    - Intel: 扩展页表（Extended Page Table，EPT）技术
    - ARM: stage1 & stage2页表
    
- 虚拟化用到的4种地址
    - GVA（Guest Virtual Address）：客户机虚拟地址。
    - GPA（Guest Physical Address）：客户机物理地址。
    - HVA（Host Virtual Address）：宿主机虚拟地址。
    - HPA（Host Physical Address）：宿主机物理地址



### IO虚拟化

- **软件模拟设备**。以磁盘为例，虚拟机管理程序可以在实际的磁盘上创建一个文件或一块区域来模拟虚拟磁盘，并把它传递给客户机。

- **设备透传**（Device Pass Through）。虚拟机管理程序把物理设备直接分配给特定的虚拟机。
- **SR-IOV**（Single Root I/O Virtualization）技术



## CPU虚拟化

### vCPU与VM概念

![vCPU与VM概念](virtualization.assets/image-20251029201450950.png)

- ARMv8/v9的CPU虚拟化技术构建在不同的异常等级上

![ARMv8/v9的CPU虚拟化技术构建在不同的异常等级上](virtualization.assets/image-20251029201607898.png)

### 创建多个vCPU

一个物理CPU可能创建多个vCPU，利用OS的多进程/多线程机制

![创建多个vCPU](virtualization.assets/image-20251029201846461.png)

一个物理CPU可以运行多个进程/线程，hypervisor可以分时复用地调度它们运行。当进程从EL2切入到EL1之后，vCPU就创建了。

### ARMv8/v9异常处理

- **非虚拟化场景**：异常的处理集中在EL1，比如异常向量表，异常处理，中断处理，系统调用等

![非虚拟化场景](virtualization.assets/image-20251029202133752.png)

- **虚拟化场景**：

  ![虚拟化场景](virtualization.assets/image-20251029202311091.png)

  - **对于hypervisor**：需要处理来自hypervisor本身的异常，来自VM的异常，来自VM的系统调用，来自VM的中断等等。
  - **对于VM**：VM本身也需要处理 来自VM的异常和系统调用等（GuestOS处理app的异常和系统调用）





### 进入虚拟机

![进入虚拟机](virtualization.assets/image-20251104171224606.png)

1. 设置HCR_EL2.RW，让cpu运行在aarch64中
2. 暂时关闭vCPU的MMU功能（可选项）
3. 设置SPSR_EL2，当eret指令之后，让CPU跳转到EL1
4. 设置ELR_EL2寄存器，执行eret之后跳转到vm_entry函数中执行执行eret指令，进行处理器模式转换，让CPU
   进入EL1
5. 设置SP_EL1栈，让vCPU指向一个新的栈空间





### 从VM退出到Hypervisor

- VM的Guest OS主动调用HVC
- VM发生了异常（EL1中处理不了，比如GPA缺页异常）
- 发生了硬件中断



### HVC系统调用

- SVC系统调用：运行在EL0的 app 请求 进入到EL1的OS。

- HVC系统调用：运行在EL1中的 VM，请求进入到EL2的hypervisor中。
- SMC系统调用：运行在hypervisor或者VM的软件，请求进入EL3的安全固件中



![hvc的指令格式](virtualization.assets/image-20251104171733804.png)