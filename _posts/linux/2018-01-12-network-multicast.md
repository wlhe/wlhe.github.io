---
layout: post
title:  网络编程——多播
date:   2018-01-12
categories: linux
tags: code linux 
---

## 网络编程多播——Multicast

 IP多播（也称多址广播或组播）技术，是一种允许一台或多台主机（多播源）发送单一数据包到多台主机（一次的，同时的）的TCP/IP网络技术。

 通俗点讲，多播也称组播，大概可以理解为分组广播的意思，是介于单播和广播之间的一种通信机制，使用多播方式，可以实现对局域网内一组特定的主机进行通信，对局域网节点分组，加入该分组即可接收该分组的消息，而未加入分组则收不到消息。

### 多播编程步骤
    1. 建立套接字接口  
    2. 设置套接字属性
    3. 加入特定的多播组
    4. 发送/接收信息
    5. 离开多播组
    6. 关闭套接字
    * 若只是发信息，可不用加入组，直接向该组发送信息即可

### 多播相关套接字选项
    1.  IP_MULTICASE_TTL
        设置超时时间， 范围是0~255
    2.  IP_MULTICAST_IF
        指定使用的网络接口，如果主机有多个网络接口，不设置该选项则使用默认接口发送接收，使用该选项可指定某特定网络接口发送和接收信息
    3. IP_MULTICAST_LOOP
        设置是否晕熏数据发送到本地loop地址
    4. IP_ADD_MEMBERSHIP/IP_DROP_MEMBERSHIP
        加入/离开特定的多播组， 操作一个struct ip_mreq结构体，里面包含带加入/离开组的信息
    
### 多播编程客户端示例：
（只发送）
```c
#define MULT_ADDR "224.0.0.250"
#define MULT_PORT 9999
#define MSG "multicast message"
int main()
{
    int sock;
    struct sockaddr_in mult_addr;
    sock = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock < 0) {
        return -1;
    }

    memset(&mult_addr, 0, sizeof(mult_addr));
    mult_addr.sin_family = AF_INET;
    mult_addr.sin_port = htons(MULT_PORT);
    mult_addr.sin_addr.s_addr = inet_addr(MULT_ADDR);

    socklen_t l = sizeof(mult_addr);
    unsigned char ttl = 20;
    l = sizeof(ttl);
    setsockopt(sock, IPPROTO_IP, IP_MULTICAST_TTL, (void *)&ttl, l);
    l = sizeof(mult_addr);

    while (1) {
        sendto(sock, MSG, sizeof(MSG), 0, (struct sockaddr *)&mult_addr, l);
        printf("SEND to : %s buf: %s\n", inet_ntoa(mult_addr.sin_addr), MSG);

        if (getchar() == 'q') {
            break;
        }
    }

    close(sock);
    return 0;
}
```

### 多播编程服务端示例：
（只接收）
```c
#define MULT_ADDR "224.0.0.250"
#define MULT_PORT 9999
int main()
{
    struct sockaddr_in addr;
    int sock;
    struct ip_mreq mreq;

    sock = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock < 0) {
        printf("failed to create udp socket\n");
        return -1;
    }

    memset(&addr, 0, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_port = htons(MULT_PORT);
    addr.sin_addr.s_addr = htonl(INADDR_ANY);
    if (bind(sock, (struct sockaddr *)&addr, sizeof(addr))) {
        printf("failed to bind socket\n");
        close(sock);
        return -1;
    }

    memset(&mreq, 0, sizeof(mreq));
    mreq.imr_multiaddr.s_addr = inet_addr(MULT_ADDR);
    mreq.imr_interface.s_addr = htonl(INADDR_ANY);

    char buf[64];
    socklen_t l = sizeof(addr);
    if (setsockopt(sock, IPPROTO_IP, IP_ADD_MEMBERSHIP, &mreq, sizeof(mreq)) < 0) {
        printf("failed to join multicast group\n");
        close(sock);
        return -1;
    }
    while (1) {
        int n = recvfrom(sock, buf, sizeof(buf), 0, (struct sockaddr *)&addr, &l);
        if (n > 0) {
            printf("RECIVE from %s => buf: %s\n", inet_ntoa(addr.sin_addr), buf);
        }
    }

    setsockopt(sock, IPPROTO_IP, IP_DROP_MEMBERSHIP, &mreq, sizeof(mreq));
    close(sock);
    return 0;
}
```
[**SOURCE CODE**](https://github.com/wlhe/network/tree/a7fad93c759963e5ce65c08ae96b99442e6af4bc/multicast)  
