﻿---
title: 'Exchange 2013 虚拟化: Exchange 2013 Help'
TOCTitle: Exchange 2013 虚拟化
ms:assetid: 36184b2f-4cd9-48f8-b100-867fe4c6b579
ms:mtpsurl: https://technet.microsoft.com/zh-cn/library/JJ619301(v=EXCHG.150)
ms:contentKeyID: 50490272
ms.date: 01/16/2018
mtps_version: v=EXCHG.150
ms.translationtype: HT
---

# Exchange 2013 虚拟化

 

_**适用于：** Exchange Server 2013_

_**上一次修改主题：** 2017-08-08_

可以在虚拟环境中部署 Microsoft Exchange Server 2013。本主题概述了支持在硬件虚拟化软件上部署 Exchange 2013 的方案。

**目录**

硬件虚拟化要求

主机的存储要求

Exchange 存储要求

Exchange 的内存要求和建议

Exchange 基于主机的故障转移群集和迁移

在此处的 Exchange 虚拟化讨论中将使用以下术语：

  - **冷启动**   在将系统从断电状态切换到重新启动操作系统时，此操作称为*冷启动*。在这种情况下，不会持续任何操作系统状态。

  - **保存的状态**   虚拟机关机时，虚拟机管理程序通常可以保存虚拟机的状态，这样一来，当计算机重新开机时，虚拟机会恢复到*保存的状态*，而不必经过冷启动过程。

  - **计划的迁移**   系统管理开始将虚拟机从一个虚拟机管理程序主机移到另一个主机时，该操作就称为*计划的迁移*。该操作可以是单次迁移；系统管理员也可以配置自动操作在既定的时间移动虚拟机。计划的迁移也可能是系统中发生的其他某个事件（非硬件或软件故障）的结果。关键点在于：Exchange 虚拟机是在正常运行，由于某个原因需要重定位。这种重定位可通过技术完成，如 Live Migration 或 vMotion。但是，如果 Exchange 虚拟机或虚拟机所在的虚拟机管理程序主机遇到了某种故障情况，则结果不具备计划的迁移的特征。

## 硬件虚拟化要求

仅当满足以下所有条件时，Microsoft 才支持 Exchange 2013 投入生产硬件虚拟化软件：

  - 硬件虚拟化软件正在运行下列软件之一：
    
      - 具有 Hyper-V 技术或 Microsoft Hyper-V Server 的任何 Windows Server 版本
    
      - 在 [Windows 服务器虚拟化验证计划](https://go.microsoft.com/fwlink/p/?linkid=125375)下经验证的任何第三方管理程序。
    
    > [!NOTE]  
    > 如果满足所有可支持性要求，则支持在服务架构 (IaaS) 提供程序上部署 Exchange 2013。如果提供程序要设置虚拟机，则这些要求包括确保要用于 Exchange 虚拟机的虚拟机监控程序完全受支持，并且 Exchange 使用的基础结构满足大小调整过程中我们确定的性能要求。如果用于 Exchange 数据库和数据库事务日志（包括传输数据库）的所有存储卷都针对 Azure 高级存储进行了配置，则支持在 Microsoft Azure 虚拟机上进行部署。


  - Exchange 来宾虚拟机具有以下条件：
    
      - 正在运行 Exchange 2013。
    
      - 部署在 Windows Server 2008 R2 SP1（或更高版本）、Windows Server 2012 或 Windows Server 2012 R2 上。

对于 Exchange 2013 的部署：

  - 一个虚拟机可支持所有 Exchange 2013 服务器角色。

  - 只要将虚拟机配置为在移动或脱机时不在磁盘上保存和还原状态，Exchange Server 虚拟机（包括属于数据库可用性组 (DAG) 的 Exchange 邮箱虚拟机）就可以与基于主机的故障转移群集和迁移技术结合。如果在目标节点上激活了虚拟机，则所有在虚拟机管理程序级别发生的故障转移活动必然导致冷启动。所有计划的迁移必然导致关机和冷启动，或者导致使用 Hyper-V Live 迁移之类技术的联机迁移。虚拟机的管理程序迁移由管理程序供应商提供支持；因此，必须确保管理程序供应商已经测试并支持 Exchange 虚拟机的迁移。Microsoft 支持这些虚拟机的 Hyper-V Live 迁移。

  - 物理主机上仅可部署管理软件（例如，防病毒软件、备份软件或虚拟机管理软件）。主机上不应安装其他基于服务器的应用程序（例如，Exchange、SQL Server、Active Directory 或 SAP）。主机应专门用来运行来宾虚拟机。

  - 有些管理程序具有为虚拟机拍摄快照的功能。虚拟机快照可捕获虚拟机运行时的状态。使用此功能，您可为虚拟机拍摄多张快照，然后通过将快照应用到虚拟机将虚拟机还原到先前任一状态。但是，应用程序无法识别虚拟机快照，使用这些快照可能会导致维护状态数据的服务器应用程序（例如 Exchange）出现意外结果。因此，不支持为 Exchange 来宾虚拟机拍摄虚拟机快照。

  - 使用许多硬件虚拟化产品，都可以指定应分配给每个来宾虚拟机的虚拟处理器数量。来宾虚拟机中的虚拟处理器共用物理系统中固定数量的物理处理器核心。Exchange 支持的虚拟处理器与物理处理器核心之比不得超过 2:1，但建议的比例为 1:1。例如，如果某个双处理器系统使用的是四核处理器，主机系统中总共有 8 个物理处理器核心。在使用此配置的系统中，分配给所有来宾虚拟机的虚拟处理器总数不得超过 16 个。

  - 计算主机所需的虚拟处理器总数时，还必须同时考虑 I/O 和操作系统的要求。在大多数情况下，对于托管 Exchange 虚拟机的系统而言，主机操作系统中所需虚拟处理器数量为 2。在计算物理内核与虚拟处理器的总比率时，此值应用作主机操作系统虚拟处理器数的基准。如果主机操作系统的性能监控指出，您的处理器利用率已超过两个处理器，则应相应减少分配给来宾虚拟机的虚拟处理器计数，并验证虚拟处理器总数与物理内核之比是否不超过 2:1。

  - Exchange 来宾计算机的操作系统使用的磁盘的大小必须至少等于 15 GB 外加分配给该来宾计算机的虚拟内存大小。此要求必须考虑操作系统和页面文件磁盘的要求。例如，如果来宾计算机分配的内存为 16 GB，则来宾操作系统所需的最低磁盘空间为 31 GB。
    
    此外，来宾虚拟机可能被阻止直接与主机中安装的光纤通道或 SCSI 主机总线适配器 (HBA) 通信。在这种情况下，您必须在主机的操作系统中配置适配器，并向来宾虚拟机显示逻辑单元号 (LUN) 作为虚拟磁盘或传递磁盘。

  - 支持将电子邮件从 Azure 计算资源发送到外部域的唯一途径是通过 SMTP 中继（也称为 SMTP 智能主机）。Azure 计算资源将电子邮件发送到 SMTP 中继，然后 SMTP 中继提供程序将电子邮件传送到外部域。Microsoft Exchange Online Protection 是 SMTP 中继的一个提供程序，但也有很多第三方提供程序。有关详细信息，请参阅 Microsoft Azure 支持团队博客文章 [Sending E-mail from Azure Compute Resource to External Domains](https://go.microsoft.com/fwlink/p/?linkid=799723)（将电子邮件从 Azure 计算资源发送到外部域）。

硬件虚拟化要求

## 主机的存储要求

每个主机的最低磁盘空间要求如下所示：

  - 某些硬件虚拟化应用程序中的主机可能需要存储空间来存储操作系统及其组件。例如，运行采用 Hyper-V 技术的 Windows Server 2008 R2，至少需要 10 GB 空间才能满足 Windows Server 2008 的要求。有关更多详细信息，请参阅 [Windows Server 2008 R2 系统要求](https://go.microsoft.com/fwlink/p/?linkid=125378)。此外，还需要额外的存储空间来支持操作系统的页面文件、管理软件和崩溃恢复（转储）文件。

  - 有些管理程序将维护主机中每个来宾虚拟机所特有的文件。例如，在 Hyper-V 环境中，系统将为每个来宾计算机创建并维护临时内存存储文件 (BIN 文件)。每个 BIN 文件的大小等于分配给该来宾计算机的内存量。此外，系统还会在主机中为每个来宾计算机创建并维护其他文件。

  - 如果你的主机运行 Windows Server 2012 Hyper-V 或 Hyper-V 2012，而且你要配置将在数据库可用性组中托管 Exchange 邮箱服务器的基于主机的故障转移群集，那么我们建议你遵照 Microsoft 知识库文章 2872325 [Hyper-V 中的来宾群集节点可能无法创建或加入](https://support.microsoft.com/kb/2872325)中描述的指南。

硬件虚拟化要求

## Exchange 存储要求

对连接到虚拟 Exchange 服务器的存储的要求如下：

  - 如果固定磁盘包含来宾的操作系统、任何正在使用的临时内存存储文件以及主机中托管的相关虚拟机文件，则必须在主机上为每个 Exchange 来宾计算机分配足够的存储空间。此外，对于每个 Exchange 来宾计算机，您还必须为邮件队列分配足够的存储空间，并为邮箱服务器中的数据库和日志文件分配足够的存储空间。

  - Exchange 来宾计算机用于存储 Exchange 数据（例如，邮箱数据库和传输队列）的存储空间可以为固定大小的虚拟存储空间（例如，Hyper-V 环境中的固定虚拟硬盘 VHD 或 VHDX）、动态虚拟存储（结合使用 VHDX 文件和 Hyper-V 时）、SCSI 通过存储或 Internet SCSI (iSCSI) 存储。共享存储是指在主机级别中配置的、专用于某一来宾计算机的存储。Exchange 来宾计算机用于存储 Exchange 数据的所有存储空间必须为块级存储空间，因为 Exchange 2013 不支持使用网络附加存储 (NAS) 卷，但在本主题后面所述的 SMB 3.0 情况下除外。此外，不支持通过虚拟机监控程序以块级存储形式向来宾提供的 NAS 存储。

  - 如果来宾计算机运行的是 Windows Server 2012 Hyper-V（或更高版本的 Hyper-V），固定或动态虚拟磁盘可以存储在由块级存储支持的 SMB 3.0 文件中。对于 SMB 3.0 文件共享而言，唯一支持的用途就是存储固定或动态虚拟磁盘。这类文件共享不能用于直接存储 Exchange 数据。使用 SMB 3.0 文件共享存储固定或动态虚拟磁盘时，用于支持文件共享的存储应进行高可用性配置，以确保 Exchange 服务尽可能可用。

  - Exchange 所使用的存储应在磁盘心轴中加以托管，且磁盘心轴应与托管来宾虚拟机操作系统的存储分开。

  - 支持将 iSCSI 存储配置为使用 Exchange 来宾虚拟机内部的 iSCSI Initiator。但是，如果虚拟机内的网络堆栈功能不完整（例如，并非所有虚拟网络堆栈都支持巨帧），则此配置的性能将有所降低。

硬件虚拟化要求

## Exchange 的内存要求和建议

某些虚拟机监控程序能够基于特定来宾计算机中内存的感知利用率与相同虚拟机监控程序管理的其他来宾计算机需求的比较情况，来过量订阅或动态调整对该来宾计算机可用的内存量。此技术适用于短时间内需要内存，随后可以向其他用户交出内存的工作负载。但是不适用于旨在持续使用内存的工作负载。与具有涉及在内存中缓存数据的性能优化的许多服务器应用程序一样，Exchange 在无法完全控制分配给运行其的物理计算机或虚拟机的内存时，容易形成糟糕的系统性能和不可接受的客户端体验。因此，不支持对 Exchange 使用动态内存功能。

硬件虚拟化要求

## Exchange 基于主机的故障转移群集和迁移

以下是对有关使用 Exchange 2013 DAG 的基于主机的故障转移群集和迁移技术的一些常见问题解答。

  - **Microsoft 支持第三方迁移技术吗？**
    
    Microsoft 不能为使用这些技术的第三方虚拟机管理程序产品与 Exchange 的集成做出支持声明，因为这些技术不属于服务器虚拟化验证计划（Server Virtualization Validation Program，SVVP）。SVVP 涵盖了 Microsoft 对第三方虚拟机管理程序支持的其他方面。您需要确保您的虚拟机管理程序供应商支持将其迁移和群集技术与 Exchange 结合。如果您的虚拟机管理程序供应商支持将其迁移技术与 Exchange 结合，则 Microsoft 也支持 Exchange 与其迁移技术结合。

  - **Microsoft 如何定义基于主机的故障转移群集？**
    
    基于主机的故障转移群集是指可以自动响应主机级别故障并在备用服务器上启动受影响虚拟机的任何技术。支持使用此技术的前提是：在发生故障的情况下，虚拟机通过冷启动在备用主机上启动。此技术可帮助确保虚拟机绝不会从磁盘上持续的保存状态启动，因为虚拟机将会比其余的 DAG 成员陈旧。

  - **Microsoft 的迁移支持意味着什么？**
    
    迁移技术是指允许按计划将虚拟机从一台主机计算机移到另一台主机计算机的任何技术。这种移动也可能是在资源负载平衡中发生的自动移动，但与系统中的故障无关。只要虚拟机从不通过在磁盘上持续的保存状态启动，那么支持就会受迁移。也就是说，通过在网络上传输状态和虚拟机内存移动虚拟机并且不会有感觉得到的停机时间的技术都可以与 Exchange 一起使用。第三方虚拟机管理程序供应商必须提供迁移技术支持，同时 Microsoft 也会提供对在此配置中使用的 Exchange 的支持。

硬件虚拟化要求

