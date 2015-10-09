<properties 
	pageTitle="流分析（流式数据分析）简介 | Windows Azure" 
	description="了解 Azure 流分析，这是一种完全托管的云服务，可以帮助你分析物联网 (IoT) 实时提供的流式数据。" 
	keywords="big data analytics,cloud service,internet of things,managed service,stream processing,streaming analytics,streaming data"
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="08/19/2015" 
	wacn.date="09/15/2015"/>


# 简要介绍 Azure 流分析，一种完全托管的云服务，适用于流式数据分析

Azure 流分析是一种完全托管的服务，可以在云中通过流式数据进行低延迟、高度可用、可伸缩且复杂的事件处理。它是一种大数据分析服务，适用于物联网 (IoT)，可以让开发人员将数据流与历史记录或引用数据结合起来，轻松快捷地进行数据分析以获得业务见解。

## 动机与概述

在现今这个时代，每天都有大量的数据在缆线上高速流动。那些能够实时处理这种流式数据并采取相应行动的组织可以极大地改进效率，让自己在市场中始终处于卓尔不群的地位。各行各业都可以找到进行实时流式分析的方案：由金融服务公司提供的个性化实时股票交易分析和提醒；实时检测欺诈行为；数据和身份保护服务；对物理对象（物联网，简称 IoT）中嵌入的传感器和激励器所生成的数据进行可靠的引入和分析；Web 点击流分析；当客户体验在某个时间范围内出现下降的趋势时，客户关系管理 (CRM) 应用程序会进行提醒。为了在竞争激烈的现代商业环境中获得成功，企业需要寻找最灵活、最可靠且最经济有效的方式来执行此类实时事件流数据分析。

构建流处理系统是一项复杂的任务。关联和聚合之类的流式处理操作不仅需要能够有效地实施，而且需要可伸缩性和容错能力。此外还有其他操作方面的挑战，包括部署、调试和监视方面的挑战。与构建和维护此类解决方案相关联的成本在快速增长。大型企业不得不支付昂贵的费用来完成自定义构建的解决方案，而小型公司则通常由于准入障碍和高昂的成本而错过了这种机会。流分析在云中提供专用于流分析的托管服务，可以轻而易举地解决上述困难。

只需在 Azure 门户中轻轻点击几下，客户就可以使用类似 SQL 的语言来指定转换方式并监视其整个流式处理管道的规模/速度，从而创作出流分析作业。该服务可以轻松地将每秒处理的事件数从几千字节扩大到千兆字节或更多。现在市场中提供的大多数其他的流式处理解决方案需要客户编写复杂的自定义代码，而使用流分析，客户只需编写简单且熟悉的声明性 SQL。

流分析提供了一系列运算符，从简单的筛选器到复杂的相关性，不一而足。定义基于时间的开窗操作（例如开窗聚合），或者将多个流关联起来以检测某类模式（例如序列），或者甚至将当前条件与历史值和模型进行对比，这些操作都可以通过一组简单且类似于 SQL 的流分析查询语言运算符在数分钟内完成。指定流式处理管道很简单，只需配置其输入和输出，然后提供类似于 SQL 的查询来描述所需的转换即可。虽然这足以应对大多数简单的案例，不过在规模扩大以后，随着方案复杂性的提高，需要对流分析进行配置。可以由用户来决定管道每一步所需的计算能力，目的是达到理想的峰值吞吐量。

目前，流分析会直接连接到 Azure 事件中心以引入流，并会连接到 Azure Blob 服务以获取历史数据。使用 Azure 事件中心的各种输入和输出接口，可以轻松地将流分析与其他数据源和处理引擎进行集成，而不会失去计算的流处理本质。可以将数据从流分析写入 Azure 存储空间，在该空间中，可以随后再次将数据处理成一系列事件，或将数据用于其他形式的通过 Azure HDInsight 进行的批处理分析。流分析利用了 Microsoft Research 在开发可调性极强且适用于时间敏感型处理的流式处理引擎方面多年的工作成果，并进行了语言集成，因此可以直观地指定各种语句。根据设计，流分析可以充分利用各种开源的 Hadoop、YARN 和 REEF，适合进行外展型处理。


## 关键功能

### 易于使用
流分析支持简单的声明性查询模型，用于描述各种转换。为了优化使用性能，我们选择了一种 SQL 变体作为域特定语言 (DSL)，并且不需要客户接触流处理系统中高度复杂的技术内容。使用流分析查询语言，你可以轻松快捷地实现各种临时功能，包括临时联接、开窗聚合、临时筛选器，以及其他常见的运算（例如联接、聚合、投影和筛选）。

虽然我们以多种方式扩展了 SQL 的语义，使之对 SQL 用户来说更为直观，但我们仍然严格遵循 SQL 的语法标准，目的是方便实施和工具的使用。有关我们的查询语言的其他信息，请参阅“后续步骤”部分。

### 可伸缩性
流分析具有很强的事件吞吐量处理能力，最大吞吐量为 1GB/秒。与 Azure 事件中心集成以后，该解决方案就可以每秒引入数百万来自已连接设备、点击流、日志文件等的事件。为此，我们利用了事件中心的分区功能，每个分区产生的吞吐量为 1MB/秒。流分析支持在分区的同时，对这些引入的事件进行水平方向和垂直方向的处理。用户可以在查询定义中将计算分成多个逻辑步骤，每个步骤都可以进一步细分成在数据层面来说属于并行的执行元素。一段时间后，流分析会根据事件引入速率、处理的复杂程度，以及为了让用户对其工作负荷进行相应的自定义而预计会产生的延迟情况自动进行伸缩。有关如何实现可伸缩性的更多详细信息，请参阅“后续步骤”部分的链接。

### 可靠性、可重复性和快速恢复
流分析是云中的一种托管服务，可用于在出现节点故障的情况下，通过内置的恢复功能和检查点功能防止数据丢失并确保业务连续性。这些功能是通过流分析客户端定位模型体系结构提供的。该服务旨在保留相关状态，以便系统能够更好地从节点故障中恢复，并可对重新计算和缓存输出进行优化，以便有效地应对下游故障情况。

由于能够在内部维持相关状态，因此在可以对事件进行存档并在未来对其重新处理的情况下，该服务可以提供可重复的结果，即可以获得相同的结果。此功能允许客户在特定情况下（例如需要进行根本原因分析和假设情况分析）回过头来研究计算结果。将这些功能组合起来，就可以快速地从处理节点故障中进行恢复，快速地重新处理丢失的状态。

### 低延迟
托管服务的另一优势是低延迟。流分析目前已进行优化，其端到端可观察延迟为次秒级。因此，系统可以连续地进行高吞吐量处理。通信模型为拉取式的自适应批处理模型，可以根据已配置的超时和大小限制来运行。因此，在达到限制之前，将会对事件和其他记录进行批处理。即使吞吐量很高，系统的延迟也可以很低。

### 引用数据
用户可以通过流分析来指定和使用引用数据。这些引用数据可以是历史数据，也可以是在一段时间内不怎么更改的数据。系统简化了引用数据的使用，将其视同其他传入事件流，可以与其他实时引入的事件流进行联接以执行各种转换。有关如何构建和使用引用数据的其他信息，请参阅“后续步骤”部分。



## 选择 Azure 流分析的商业动机

### 低成本
作为一种云服务，流分析已经过优化，用户只需很少的成本即可运行和维护各种实时分析解决方案。该服务旨在让你能够根据使用情况即付即用。使用情况取决于已处理事件的数量，以及在群集中预配的处理相应流式分析作业所需的计算能力。

### 快速实现
有了流分析，你就有了一个高度可伸缩的实时事件处理解决方案，没有硬件成本，也没有其他前期成本，不需要进行耗时的安装或设置。Microsoft Azure 作为云服务提供商，将为你负责所有这一切。只需通过 Azure 门户订阅此服务，你就可以在数分钟内启动并运行该解决方案。门户中的用户界面十分友好，为你提供分步的快速入门指南，引导你完成输入源的配置和测试，以及输出存储的配置和设置。查询编辑器十分易用，提供自动完成功能和建议功能。

### 快速开发
为外展型分布式系统开发分析功能时，可以使用流分析来减少冲突，降低复杂性。开发人员只需使用基于 SQL 的查询语言描述其所需要的转换即可，然后系统就会自动处理相关事项，根据规模、性能和复原能力来分发工作负荷，不需要进行程序性的编程，后者在多数流处理解决方案中是不可或缺的。流分析包含各种内置功能并可使用用户友好型门户，因此符合那些高级开发人员的需要，可以轻松快捷地进行实时处理，不需操心基础结构采购、日常维护和缩放等事项。

## 方案和用例

### 适用于物联网 (IoT) 的实时分析
由于设备变得越来越智能，而且越来越多的设备具有通信功能，因此，如何利用这些设备生成的数据以及从这些设备收集的数据，商业界和用户群体对此的看法并不是一成不变的，而是在一直发展的。可以预计的是，随着可用的数据越来越海量化，我们将可以快速地组合和处理这些数据，从而更深刻地了解我们所处的环境以及我们平常所使用的设备。可以通过售货机这样一个示例来描述标准的 IoT 方案：

售货机通常会将需要引入到系统中的数据（例如产品库存、状态和温度）发送到现场网关（如果售货机没有启用 IP 功能）或云网关（如果售货机启用了 IP 功能）。系统会对传入数据流进行处理和转换，以便计算出来的输出能够即时通过网关反馈回设备进行相应的操作。例如，如果售货机过热，则可能需要重启设备或者需要设备自动进行固件升级，不需人工干预。处理完的输出还可以根据事件触发其他警报和通知，自动安排技术人员进行维修。

随着收集和处理的数据越来越多，也可以通过机器学习在系统中形成相应的模式并从中进行学习。例如，可以根据馈送到系统中的实时事件流来预测售货机何时需要进行维修，也可以在要发生故障时安排技术人员进行预防性的维护。

这种由设备向处理系统发送信息并根据实时处理结果进行可能的操作的模式在 IoT 用例中可以说是很常见的。其他这样的方案还有：联网汽车、点击流分析、设备管理。为了优化此反馈回路以降低延迟并提高吞吐量，可以使用流分析从 Azure 事件中心引入数据，处理完以后再将其直接发送回事件中心，以便各个已订阅设备执行相应的操作。


![大数据分析和物联网：Azure 流分析是针对 IoT 的实时分析。][img.stream.analytics.scenario1]


### 通过仪表板进行遥测和日志分析
随着设备、计算机和应用程序数量的增长，企业在进行业务运营时常常需要监视和响应多变的业务需求，因此需要近实时地创建为数众多的各种分析。标准的遥测/日志分析方案可以通过在线服务或应用程序示例来描述，但该模式通常见于各种通过应用程序或设备遥测来收集数据并进行报告的企业。

该应用程序或服务会定期收集运行状况数据。收集的数据包括：表示应用程序或基础结构在某一时刻的最新状态的数据、用户请求日志，以及其他表示在应用程序内执行的操作或活动的数据。在过去，数据将保存到 blob 或其他类型的数据存储中供进一步的处理。

近来的趋势是对数据进行实时仪表板处理，因此除了将数据保存到 blob 或其他类型的存储进行历史分析外，客户还希望能够直接处理和转换传入数据流，这样就可以通过仪表板和/或通知将数据即时提供给最终用户进行必需的操作。例如，如果某个服务站点发生故障，这样就可以通知操作人员快速开展调查并解决问题。在多个这样的用例中，通常会由人来监视实时仪表板，其中会显示通过流分析对引入的数据进行处理后更新的数据集。
 
![Azure 流分析：通过仪表板进行流式数据遥测和日志分析。][img.stream.analytics.scenario2]

### 对需要将来进行处理的事件存档
企业期望执行的速度能够越来越快，执行的方式能够越来越灵活。企业和开发人员现在都会选择易用的云服务来满足其灵活性方面不断提高的需要，同时会寻求那种能够以近实时方式引入和处理其系统所生成的连续数据流的平台。目前，这些客户尚无法使用经过优化的托管服务以低延迟方式执行对数据存储的写入操作，因此最终丢失多组可以深入揭示业务运营情况的关键数据。标准的事件存档方案可以描述如下：

分布在全球各处的各种设备和平台的数据被推送到一个集中的数据收集器。将这些数据放置到中心位置以后，将会对其执行某种无状态转换，例如清除个人身份信息、添加地域标记、执行 IP 查找等。然后，已转换的数据将存档到 Blob 存储中，所采用的方式适合 HDInsight 或其他工具进行脱机处理。

![Azure 流分析事件存档，适用于未来进行的流处理。][img.stream.analytics.scenario3]
 
## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)

## 后续步骤
我们已经向你介绍了流分析，这是一种托管服务，适用于对物联网的数据进行流式分析。若要了解有关此服务的详细信息，请参阅：

- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)


<!--Image references-->
[img.stream.analytics.scenario1]: ./media/stream-analytics-introduction/Introduction-to-Azure-Stream-Analytics_01.png
[img.stream.analytics.scenario2]: ./media/stream-analytics-introduction/Introduction-to-Azure-Stream-Analytics_02.png
[img.stream.analytics.scenario3]: ./media/stream-analytics-introduction/Introduction-to-Azure-Stream-Analytics_03.png


<!--Link references-->
[stream.analytics.developer.guide]: /documentation/articles/stream-analytics-developer-guide
[stream.analytics.scale.jobs]: /documentation/articles/stream-analytics-scale-jobs
[stream.analytics.introduction]: /documentation/articles/stream-analytics-introduction
[stream.analytics.get.started]: /documentation/articles/stream-analytics-get-started
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301
 

<!---HONumber=69-->