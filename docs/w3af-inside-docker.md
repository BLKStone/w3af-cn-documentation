# docker 中的 w3af (w3af inside docker)

对于大多数使用案例，在 `docker` 内部 `w3af` 应该是透明的，此页面记录了一些相对复杂的混合使用案例。


## 端口和服务 (Ports and services)

一些 `w3af` 插件，例如 `crawl.spider_man`, `audit.rfi` 会启动 HTTP 代理服务。为了访问到这些插件提供的服务，需要将侦听地址设置为 `0.0.0.0`，并且要在让对应端口在 docker 宿主机上暴露出来。可以在 docker 辅助脚本 `extras/docker/scripts/w3af_console_docker` 中使用 `-p` 参数来实现。

可以看下这条 [commit](extras/docker/scripts/w3af_console_docker) 来获取有关暴露端口的更多信息。


## 与容器共享数据 (Sharing data with the container)

当你使用 `w3af_console_docker` 或 `w3af_gui_docker` 命令来启动 docker 容器时，会设置两个卷 (volume) 来映射你宿主机的磁盘。

* 宿主机的 `~/.w3af` 目录会被映射到容器的 `/root/.w3af` 。 `w3af` 通常使用这个目录来存储扫描配置和内部数据
* 宿主机的 `~/w3af-shared` 目录会被映射到容器的 `/root/w3af-shared`。 你可以使用这个目录来保存 `w3af` 的扫描结果，或者通过这个目录给 `w3af` 提供输入文件。


## 对容器进行调试 (Debugging the container)

容器运行了 SSH 守护进程，可以被用于同时运行 `w3af_console` 和 `w3af_gui` 。你可以使用 `root` 作为用户名， `w3af` 作为密码连接上一个运行的容器。 通常，您无需为此担心，因为 Docker 辅助脚本将为您连接到容器。

调试容器的另一种方法是运行 Docker 辅助脚本时带上 `-d` 参数：

```bash
$ sudo ./w3af_console_docker -d
root@a01aa9631945:~#
```

> 警告：
> 除非您真的知道自己在做什么，否则请勿将 `w3af` 的 `docker` 容器绑定到公共IP地址！任何人都可以使用硬编码的 SSH 密钥进入 docker 容器！
>

