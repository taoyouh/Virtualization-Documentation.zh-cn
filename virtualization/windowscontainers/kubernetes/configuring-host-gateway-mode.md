# <a name="host-gateway-mode"></a>主机网关模式 #
Kubernetes 网络的其中一个可用选项是*主机网关模式*，此模式要求在所有节点上配置 Pod 子网之间的静态路由。


## <a name="configuring-static-routes--linux"></a>配置静态路由 | Linux ##
为此，我们使用 `iptables`。 将 `$CLUSTER_PREFIX` 变量替换（或设置）为所有 Pod 将使用的简短格式的子网：

```bash
$CLUSTER_PREFIX="192.168"
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING ! -d $CLUSTER_PREFIX.0.0/16 \
              -m addrtype ! --dst-type LOCAL -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1
```

这只设置 Pod 的基本 NATing。 现在，我们需要使目标为 Pod 的所有流量都流经主要接口。 同样，根据需要替换 `$CLUSTER_PREFIX` 变量；如果适用，一并替换 `eth0`：

```bash
sudo route add -net $CLUSTER_PREFIX.0.0 netmask 255.255.0.0 dev eth0
```

最后，我们需要针对**每个节点**添加下一跃点网关。 例如，如果第一个节点是位于 `192.168.1.0/16` 上的 Windows 节点，则：

```bash
sudo route add -net $CLUSTER.1.0 netmask 255.255.255.0 gw $CLUSTER.1.2 dev eth0
```

必须在群集中的每个节点*上**为*群集中的每个节点添加类似的路由。


<a name="explanation-2-suffix"></a>
> [!Important]  
> 网关的子网是 `.2`，这**仅**适用于 Windows 节点。 对于 Linux，它可能始终为 `.1`。 出现此异常的原因是 `.1` 地址被保留为桥接主机网络与虚拟 Pod 网络的网络适配器的网关。


## <a name="configuring-static-routes--windows"></a>配置静态路由 | Windows ##
为此，我们使用 `New-NetRoute`。 [此存储库](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/AddRoutes.ps1)中存在一个可用的自动脚本 `AddRoutes.ps1`。 你将需要知道 *Linux 主机*的 IP 地址，以及 Windows 节点的*外部*适配器的默认网关（不是 Pod 网关）。 然后：


```powershell
$url = "https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/windows/AddRoutes.ps1"
wget $url -o AddRoutes.ps1
./AddRoutes.ps1 -MasterIp 10.1.2.3 -Gateway 10.1.3.1
```
