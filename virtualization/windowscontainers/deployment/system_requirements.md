# Windows 容器要求

**这是初步内容，可能还会更改。**

此指南列出 Windows 容器主机的要求。

## 支持的操作系统映像

提供 Windows Server Technical Preview 4 并带有两个容器操作系统映像、Windows Server Core 和 Nano 服务器。 并非所有配置都支持这两个操作系统映像。 下表详细介绍所支持的配置。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td><center>**主机操作系统**</center></td>
<td><center>**Windows Server 容器**</center></td>
<td><center>**Hyper-V 容器**</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 完整用户界面</center></td>
<td><center>Core 操作系统映像</center></td>
<td><center>Nano 操作系统映像</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Core 操作系统映像</center></td>
<td><center> Nano 操作系统映像</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Nano 操作系统映像</center></td>
<td><center>Nano 操作系统映像</center></td>
</tr>
</table>

## Hyper-V 容器要求

如果 Windows 容器主机将在 Hyper-V 虚拟机上运行，并且还将托管 Hyper-V 容器，则需要启用嵌套虚拟化。 嵌套的虚拟化具有以下要求：

- 至少 4 GB RAM 可用于虚拟化的 Hyper-V 主机。
- 在物理和虚拟主机上具有 Windows Server 2016 Technical Preview 4 或 Windows 10 版本 10565。
- 带有 Intel VT-x 处理器（此功能目前只适用于 Intel 处理器）。
- 容器主机虚拟机还需要至少 2 个虚拟处理器。





