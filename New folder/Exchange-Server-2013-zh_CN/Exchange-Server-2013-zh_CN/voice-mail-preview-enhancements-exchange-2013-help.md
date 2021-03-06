﻿---
title: '语音邮件预览增强功能: Exchange 2013 Help'
TOCTitle: 语音邮件预览增强功能
ms:assetid: 1fcccec1-4edc-40b8-948c-111647d7d770
ms:mtpsurl: https://technet.microsoft.com/zh-cn/library/JJ150501(v=EXCHG.150)
ms:contentKeyID: 50490033
ms.date: 05/21/2018
mtps_version: v=EXCHG.150
ms.translationtype: MT
---

# 语音邮件预览增强功能

 

_**适用于：** Exchange Server 2013, Exchange Server 2016_

_**上一次修改主题：** 2012-07-05_

语音邮件预览是一种功能，可用接收他们使用 Microsoft Exchange Server 2010或Exchange Server 2013的语音邮件用户的统一邮件 (UM)。语音邮件预览提供录音的文本版本，从而增强了 UM 语音邮件功能。语音邮件文本将显示在 Microsoft Office Outlook Web 应用程序，Outlook 2010 中和其他电子邮件程序中的电子邮件。

## 语音邮件预览增强功能

在Exchange 2013，UM 包括几个Outlook Web App和Outlook客户端的用户界面的增强功能和改进的语音邮件预览增加可靠性和准确性。此外，通过 Microsoft 语音平台 (版本 11.0) 和统一通信管理 API (UCMA) 4.0 增强语法生成和语言支持提供了与语音相关的服务的一些增强功能。

统一的消息推出了Exchange 2010中的语音邮件预览功能。语音邮件预览使用自动语音识别 (ASR) 添加到语音邮件语音邮件音频文件的文本版本。尤其是在用它来录制音频段包括未知的语音和噪声的电话时，不完全准确，ASR。

某些组织要求以一致的方式无错误或附近的无错误的聊天记录，语音邮件，对于某些问题，如果不对所有，其用户。语音邮件预览合作伙伴计划可以帮助满足这些要求的此类组织。语音邮件预览合作伙伴计划专为Exchange 2010来改善语音邮件预览结果，但它不使用Exchange 2010客户由于开销和成本。为了解决这些问题， Exchange 2013包含语音邮件预览了以下增强功能 ︰

  - **改进了音频规范化**  音频的规范化是统一地增加 （或减少） 的过程的整个振幅音频信号，以便产生峰值振幅匹配指定的目标或标准。UM 规范化音频录音之前对它进行压缩并发送给用户。

  - **增强的语音识别**  通过收集语音邮件 （仅当Exchange客户选择共享此信息），可用于添加到语音引擎的单词和短语的语音邮件预览结果。这是通过设置`$true`使用**Set-UMMailbox** cmdlet 的*VoiceMailAnalysisEnabled*参数或上**Set-UMMailboxPolicy** cmdlet 将*AllowVoiceMailAnalysis*参数设置为`$true` 。此外， Exchange 2013 UM 使用更有效地从用户使用 Outlook Voice Access 创建的电子邮件线程的信息。这包括有关的参与者 （活动目录或个人联系人） 信息 （国家/地区、 城市、 公司） 和 Outlook Voice Access 用户的电话号码信息。

  - **语音邮件预览信心**  信心分数为统一消息直接相关的整体准确性的抄写到分配的编号。已调整使用的 UM 的可信度计算更准确地表示实际改写成文字消息的准确性。

  - **筛选** 检测和筛选攻击性词语，并将结果缓存并存储在用户邮箱中。

  - **隐藏文字预览**  如果语音邮件预览的信心分数低于给定阈值时，将隐藏的语音邮件预览文本。如果已隐藏文本，语音邮件将包括指出语音邮件的信心太低的结果将显示的文本。

  - **抄写性能**  语音邮件预览是 CPU 密集型操作，它需要大约两倍的时间来处理音频文件。如果生成语音邮件预览文本太长，CPU 节流停止处理预览。在Exchange 2010，UM 没有尝试任何超过 75 秒的语音邮件，理赔。在Exchange 2013中，转录是整个语音邮件，但如果超出 75 秒中不包含该消息的文本。

  - **配色方案**   由于对用于区分语音邮件预览的低、中和高可信度的颜色易混淆，已在 Exchange 2013 中删除了用于 Outlook Web App 和 Outlook 的配色方案。

