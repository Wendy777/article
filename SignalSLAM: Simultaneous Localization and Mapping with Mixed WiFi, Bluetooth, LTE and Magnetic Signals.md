# 融合WIFI 蓝牙 LTE 地磁的 实时室内定位 （原文链接：http://pan.baidu.com/s/1jHYfD6A 密码：jjlk）
关键词 众包
## 摘要
室内定位通常依赖于测量RF信号的集合，诸如来自WiFi的接收信号强度（RSS），结合信号指纹的空间图。使用4G LTE电话小型小区可以产生新的定位技术，
其范围有限但信号强度信息丰富，即参考信号接收功率（RSRP）。在本文中，我们建议组合可用的射频信号源来构建可用于本地化或用于网络部署优化的多模式信号映射。
我们主要依靠同时定位和映射（SLAM），它提供了一个解决方案来构建观测图的地图，而不知道观察者的位置。 SLAM最近已经扩展到在所谓的WiFi-SLAM中包含来自WiFi的
信号强度。与WiFi-SLAM并行，其他定位算法已经开发出来，利用惯性运动传感器和WiFi RSS或磁场幅度的已知映射。在我们的研究中，我们使用现成的智能手机可以获得的
所有测量数据，并且可以从几个在大楼中自由行走的实验者那里收集数据，包括带时间戳的WiFi和蓝牙RSS，4G LTE RSRP，磁场强度，室外GPS参考点，特定地标处的近场通信（NFC）读数以及基于惯性数据的行人航位推算。我们使用用户姿势的GraphSLAM优化的修改版本来解决所有用户的位置，其中集合了多模式信号相似度的绝对位置和成对约束的集合。我们证明，我们可以恢复用户的位置，从而同时为每个WiFi接入点和4G LTE小型蜂窝“从口袋里”生成密集的信号图。最后，我们使用选定的单一模式（例如仅生成WiFi和我们生成的WiFi信号映射）来演示本地化性能。

## I Introduction
室内定位和映射是普及计算的关键。在日常使用基于位置的服务（例如可以在室内GPS无效的环境中准确地定位智能电话）的同时，还需要优化电信网络的部署。For instance, 在部署4G LTE小型蜂窝网络时，精确定位出现重复障碍。这些蜂窝的覆盖范围有限（与典型的WiFi接入点覆盖范围相当），但可以安装在办公空间，居民区或公共场所的多个位置。在规划这种蜂窝的布局时，服务的成本和质量是重要的考虑因素，因此需要构建精确的4G LTE信号覆盖图。

除了提供具有额外带宽的通信链路之外，4G LTE小型蜂窝的另一个优势在于，它们可以与WiFi一起用作定位系统的输入。

基于位置的服务和网络部署优化通常都依赖于诸如以信号强度为特征的射频（RF）信号集合的测量，对于WiFi，可能通过参考信号接收强度，对于LTE，伴随空间位置。手动收集这些信号指纹的时间和努力可能是令人望而却步的，促使人们对使用移动机器人的自动化方法或从行人轨迹重建RF地图的方法进行研究。后者属于RF信号的同时定位和映射（SLAM）和无监督映射的一般类别，并在I-B部分中进行了综述。

While using autonomous WiFi mapping robots may prove impractical in some buildings as it involves a deployment cost [30], it is worth noting that the pedestrian position could be recovered thanks to a wearable system consisting of a color and depth (RGBD) camera and a computer running real-time vision-based odometry [44] or even a full SLAM algorithm using the RGBD camera combined with a laser range [11]. In our research, we have decided not to rely on any vision systems and to run a cruder version of SLAM from the user’s pocket, using only the sensors available on a smartphone.

### A. Objective: Crowd-sourced RF Mapping from the Pocket

我们建议通过利用记录在一个或多个现成的智能手机上的所有传感器信号，无需实验者的额外工作即可构建包括WiFi，4G LTE或蓝牙的多模式射频信号地图。Our goal is to enable “crowd-sourced RF mapping from the pocket”.在一个优选的情况下，用户将简单地收集RF信号，同时在建筑物中自由行走，照顾他们的日常活动。 同时，他们将收集带有时间戳的WiFi和蓝牙RSS，4G LTE RSRP，户外的GPS磁场强度，GPS参考点，特定地标的近场通信（NFC）标签或QR码读数和行人航位推算 PDR）基于惯性数据。

我们的系统依赖于状态空间模型，其中用户的轨迹是未知的，并且取决于来自行人运动模型的动力学和多传感器观察（包括WiFi或LTE信号）。 与现有的使用运动模型和WiFi在室内跟踪用户的传感器融合算法不同，我们不提前知道RF信号图：我们的目标是从数据库中重建它 一个自由移动的行人在口袋里穿着一个商业级的智能手机，人为干预有限或没有。

我们系统的两个组成部分是行人推测定位（见下一节限制智能手机PDR）和适用于RF信号数据的SLAM算法。 SLAM为在不知道（移动）观察者的位置的情况下构建观测图的挑战提供了解决方案。

我们研究的两个创新是使用姿态不变的PDR，让用户可以将手机放在裤子的任何口袋里，而我们的SLAM的修改版本叫做SignalSLAM，它通过绝对的收集来优化用户的姿势 包括WiFi RSS，蓝牙RSS，LTE RSRP或甚至磁场的大小的多模态信号相似性的位置和成对约束。

*1）Pedestrian Dead Reckoning and its Limitations*

