# udprok
这是一款利用UDP打洞实现的TCP端口转发软件（先考虑只实现TCP转发，后面可以再考虑Tun/Tap的实现）
软件利用UDP协议来封装TCP数据，利用中转服务器实现UDP打洞使两边实现UDP直连，最后通过UDP通道实现TCP数据的通信。

## udprokd
程序作为中转服务器，将两端的udprok客户端的UDP直通

## udprok
端口转发程序，利用udprokd实现udp打洞后，与其他的udprok进行直通，以实现端口转发的目的

## 使用场景
客户机A上建有一个L2TP的VPNServer，并运行于NAT后方，客户机B上安装有一个L2TP的VPNClient，并且也处于另一个NAT后方。现在客户机B希望能连上客户机A的VPNServer。

### 部署结构
首先，客户机A上部署udprok，并以s模式运行。客户机B上部署udprok，并以c模式运行。另外部署一台公网服务器，安装udprokd作为打洞服务器。

### 运行流程
客户机A向udprokd发起udp请求，并告知希望与客户机B进行打洞。同时，客户机B也向udprokd发起请求，并告知希望与客户机A进行打洞。当udprokd收到双方希望打洞的请求后，将双方发起请求时的源ip地址和源端口号交换告知对方，对方在收到ip与端口号之后，双方采用新的ip与端口号进行通信。同时，为了防止打洞失效，双方都定时向打洞的端口上发送心跳包。同时s模式端主动向c模式端告知s模式端所监听的端口，c模式端也对该端口进行监听。

### 端口转发
当c模式端监听的端口有连接请求时，利用udp通道通知s模式端，s模式端向端口发出连接请求，若连接被接受，则告知c模式端，c模式端接受连接请求。当s/c模式端收到数据时，将数据通过udp通道发送到对端，对端通过建立端tcp描述符将数据发送出去。当s/c模式端收到断开连接时，利用udp通道通知对端，对端关闭建立的连接。

## udprokd流程
udprokd启动，分析参数，设置监听的端口，死循环读取数据

## udprok流程
