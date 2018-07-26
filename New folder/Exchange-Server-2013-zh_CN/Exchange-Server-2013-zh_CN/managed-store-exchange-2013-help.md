﻿---
title: '托管存储: Exchange 2013 Help'
TOCTitle: 托管存储
ms:assetid: efdaf80b-335c-491c-8eb5-1fafd297e8a2
ms:mtpsurl: https://technet.microsoft.com/zh-cn/library/Dn792020(v=EXCHG.150)
ms:contentKeyID: 62607052
ms.date: 01/11/2018
mtps_version: v=EXCHG.150
ms.translationtype: HT
---

# 托管存储

 

_**适用于：** Exchange Server 2013_

_**上一次修改主题：** 2014-07-14_

从 Exchange Server 4.0 到 Exchange Server 2010 的所有 Exchange Server 早期版本均支持信息存储进程单个示例 (Store.exe) 在邮箱服务器角色上运行。此单个存储示例托管服务器上的所有数据库：主动、被动、延迟和恢复。在以前的 Exchange 体系结构中，邮箱服务器上托管的不同数据库之间还有少量（如果有的话）的隔离。单个邮箱数据库存在的问题之一是，可能会对其他所有数据库产生负面影响，其邮箱损坏产生的崩溃可能影响为所有数据库驻留在服务器上的用户提供的服务。

Exchange 早期版本中单个存储示例的另一挑战是，虽然可扩展存储引擎 (ESE) 可很好地扩展到 8-12 个处理器核心，但除此之外，跨处理器通信和缓存同步问题均阻碍扩展。假如今天的服务器要大得多，具有 16 个或更多的可用核心系统，这将意味着在管理 ESE 的 8-12 个核心的相关性和对非存储进程使用其它核心（例如，助手、Search Foundation、托管可用性等）上将不得不面对管理方面的挑战。此外，之前的体系结构限制了存储进程的规模。

随着 Exchange 服务器自身的发展，Store.exe 进程经过这些年也已经取得了长足的进步，但作为一个单独的进程，最终的可扩展性是有限的，它代表了一个单一故障点。由于这些限制，Store.exe 不再存在于 Exchange 2013，取而代之的是托管存储。

## 托管存储

在 Exchange Server 2013 中，托管存储是信息存储（亦称存储）的名称。托管存储使用可提供存储进程隔离和更快数据库故障转移的控制器/工作进程模型。托管存储还包括一种新静态数据库缓存机制，该机制替代了 Exchange Server 早期版本中的动态缓冲算法。托管存储使用的多进程模型中，每个装入的数据库均有一个单个存储服务控制器进程（这种情况下为 Microsoft.Exchange.Store.Service.exe，亦称 MSExchangeIS）和一个工作进程（这种情况下为 Microsoft.Exchange.Store.Worker.exe）。装入数据库时，新的工作进程被实例化为仅服务该数据库。当数据库被卸除时，则该数据库的工作进程将终止。

例如，如果您的服务器上装入了 40 个数据库，将为托管存储运行 41 个进程，每个数据库对应一个进程，剩余的一个进程对应存储服务进程控制器。

存储服务进程控制器非常薄且可靠，但如果存储服务进程控制器死亡或终止，则其所有工作进程均将死亡（这些工作进程将检测到服务控制器进程已消失并退出）。存储过程控制器监视服务器上所有存储工作进程的运行状况。强制或意外终止 Microsoft.Exchange.Store.Service.exe 导致所有主动数据库副本即时故障切换。托管存储还与 Microsoft Exchange 复制服务 (MSExchangeRepl.exe) 和活动管理器紧密集成。控制器进程、工作进程和复制服务一起工作可以提供更高的可用性和可靠性：

  - Microsoft Exchange 复制服务进程 (MSExchangeRepl.exe)
    
      - 负责对存储发布装入和卸除操作
    
      - 启动存储、可扩展存储引擎 (ESE) 和托管可用性响应器报告的存储恢复操作或数据库故障
    
      - 检测意外数据库故障
    
      - 为管理任务提供管理界面

  - 存储服务进程/控制器 (Microsoft.Exchange.Store.Service.exe)
    
      - 根据从复制服务接收到的装入和卸除操作管理每个工作进程的生存期
    
      - 处理从 Windows 服务控制管理器传入的请求
    
      - 当检测到存储工作进程问题时，记录失败项目（如挂起或意外退出）
    
      - 在响应故障转移事件中终止存储工作进程

  - 存储工作进程 (Microsoft.Exchange.Store.Worker.exe)
    
      - 负责在数据库上为邮箱执行 RPC 操作
    
      - 工作进程内的 RPC 终结点实例是数据库的 GUID
    
      - 为数据库提供数据库缓存

## 静态数据库缓存算法

数据库缓存算法被称为动态缓冲区分配，由 Exchange Server 5.5 推出并用于 Exchange 2000 Server 中的信息存储、Exchange Server 2003、Exchange Server 2007 和 Exchange Server 2010，但未用于 Exchange 2013。Exchange 2013 使用更简单明了的算法来确定数据库缓存。发生故障转移时，托管存储不再能动态地重新分配数据库间的缓存，这极大地简化了内部缓存管理。而是根据本地数据库副本数和 *MaximumActiveDatabases* 值为每个数据库缓存（如每个存储工作进程）分配内存（如已配置）。如果 *MaximumActiveDatabases* 的值大于当前数据库副本数，则根据数据库副本数计算缓存。

Exchange 2013 使用的静态算法根据物理 RAM 为每个存储工作进程的 ESE 缓存分配内存。这被称为数据库的“最大缓存目标”。25% 的总服务器内存分配给了 ESE 缓存。这被称为“服务器缓存大小目标”。

> [!NOTE]
> 使用 Active Directory 中 <em>InformationStore</em> 对象的 <em>msExchESEParamCacheSizeMax</em> 属性可以覆盖服务器缓存大小目标和分配给 ESE 缓存存储的内存量（为在所有存储进程中进行分配所配置的值为 32 KB 页面数量）。


此缓存的静态量分配到主动和被动副本。只有当为主动数据库副本服务时，存储工作进程才会将最大缓存目标分配给存储工作进程。20% 的最大缓存目标分配给被动数据库副本。存储保留剩余部分，并在数据库从被动转为主动时将剩余部分分配给工作进程。

只在启动存储时计算最大缓存目标。因此，如果添加或删除数据库或数据库副本，则必须重启存储控制器服务 (MSExchangeIS) 以对缓存进行相应调整。如果不重启服务，与重启服务前创建的数据库相比较，新创建的数据库将具有较小的缓存大小目标。在这种情况下，数据库缓存大小目标总数将可能超过服务器缓存大小目标，直到重启 MSExchangeIS。

## 数据库缓存计算示例

以下为基于邮箱服务器内存和数据库配置的数据库缓存计算示例

**示例 1**

在此示例中，邮箱服务器内存为 48 GB，托管两个主动数据库和两个被动数据库。此外，未配置 *MaximumActiveDatabases* 参数。在此配置中，每个主动数据库副本工作进程的数据库缓存量为 3 GB，每个被动数据库副本工作进程的数据库缓存量为 0.6 GB。下面是获取这些值的方式。

若要获取服务器缓存大小目标，请将内存量乘以 25%：

48 GB X 25% = 12 GB

若要获取数据库最大缓存目标，请用主动和被动数据库总数除以服务器缓存大小目标：

12 GB / 4 个数据库 = 3 GB

若要确定被动数据库副本使用的内存量，请将数据库最大缓存目标乘以 20%：

3 GB X 20% = 0.6 GB

分配给服务器缓存大小目标的 12 GB 内存中，数据库工作进程使用 7.2 GB，信息存储为两个被动数据库副本保存 4.8 GB 以防被动数据库变为主动副本。在这种情况下，它们将使用 3 GB 的最大缓存目标。

**示例 2**

在此示例中，邮箱服务器内存仍为 48 GB，托管两个主动数据库和两个被动数据库；但是 *MaximumActiveDatabases* 参数值被配置为 2。在此配置中，每个主动数据库副本工作进程的数据库缓存量为 5 GB，每个被动数据库副本工作进程的数据库缓存量为 0.2 GB。下面是获取这些值的方式。

若要获取服务器缓存大小目标，请将内存量乘以 25%：

48 GB X 25% = 12 GB

若要获取数据库最大缓存目标，请用主动数据库总数和被动数据库总数乘以 20% 再除以服务器缓存大小目标：

12 GB / (2A + (2P X 20%)) = 5 GB

若要确定被动数据库副本使用的内存量，请将数据库最大缓存目标乘以 20%：

5 GB X 20% = 1 GB

分配给服务器缓存大小目标的 12 GB 内存中，数据库工作进程使用 12 GB，信息存储不为两个被动数据库副本保存内存，因为此配置中被动数据库不会变为主动副本（因为 *MaximumActiveDatabases* 配置的值为 2，且服务器上已有 2 个主动数据库副本）。
