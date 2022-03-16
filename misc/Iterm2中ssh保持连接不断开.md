# Iterm2中ssh保持连接不断开



## 通过客户端ssh参数配置



通过配置 ServerAliveInterval 来实现，在 ~/.ssh/config 中加入： ServerAliveInterval=60

ServerAliveInterval 60 #表示ssh客户端每隔60秒给远程主机发送一个no-op包，no-op是无任何操作的意思，这样远程主机就不会关闭这个SSH会话。



`vim ~/.ssh/config`

```config
Host *
    ServerAliveInterval 60
```

我觉得60秒就好了，而且基本去连的机器都保持，所以配置了*，如果有需要针对某个机器，可以自行配置为需要的serverHostName。

