# Bug 报告机制 (Bug reporting)

该框架正在持续开发中，我们可能会在尝试实现新功能时引入错误和回归。我们使用持续集成以及重型单元测试和集成测试来避免大多数此类情况，但其中一些仅能覆盖我们的用户。

## 良好的错误报告实践 (Good bug reporting practices)

如果您使用的是最新版本的框架，并且发现了一个错误，请[报告该错误](https://github.com/andresriancho/w3af/issues/new)， 包括以下信息：

* 复现它的详细步骤
* 预期输入和预期输出
* Python traceback (如果有的话)
* 这个命令 `./w3af_console --version` 的输出
* 日志文件 (Verbose 选项设置为 `True`)

报告可能与您的环境有关的安装错误和问题时，最好包含[详细的系统信息](https://gist.githubusercontent.com/andresriancho/9873639/raw/adaff04e2ffe95dfd0b0069a294297107249f7b3/collect-sysinfo.py)。

```bash
user@box:~/w3af$ wget http://goo.gl/eXpPDl -O collect-sysinfo.py
user@box:~/w3af$ chmod +x collect-sysinfo.py
user@box:~/w3af$ ./collect-sysinfo.py
```

这将生成一个名为 `/tmp/w3af-sysinfo.txt` 的文件，您可以将该文件包含在错误报告中。

## 确保使用最新版本 (Making sure you’re on the latest version)

`w3af` 通常由我们的用户以两种不同的方式安装：

* `apt-get install w3af` 或者其他类似方法
* `git clone git@github.com:andresriancho/w3af.git`

使用操作系统软件包管理器进行安装是最简单的方法，但通常会安装软件的旧版本，而该软件将无法进行`update.rst` 的安装。对于报告错误，我们建议您w3af从我们的存储库中安装最新版本。

建议从 git 仓库克隆到您家中的目录，并允许自动更新，以确保您始终使用最新的。


`w3af` 使用 `--version` 命令行参数很容易获得特定版本：

```
user@box:~/w3af$ ./w3af_console --version
w3af - Web Application Attack and Audit Framework
Version: 1.5
Revision: 4d66c2040d - 17 Mar 2014 21:17
Branch: master
Local changes: Yes
Author: Andres Riancho and the w3af team.
user@box:~/w3af$
```

命令的输出很容易理解，但是为了以防万一：

`Version: 1.5`： w3af 版本号
`Revision: 4d66c2040d - 17 Mar 2014 21:17`： 如果存在此行，说明您是从我们的 git 仓库克隆安装的。 `4d66c2040d` 是 git 已知的最近提交的 SHA1 ID。 
`Branch: Master`: git 仓库使用的分支。在大多数情况下，该值应为 `master` 或 `develop`。
`Local changes: Yes`: 指示您是否已手动修改过 `w3af` 的源代码

只是为了确保您使用的是最新版本，请确保在目录中运行该目录运行`git pull` 时，返回的信息是 `Already up-to-date.` ：

```bash
user@box:~/w3af$ git pull
Already up-to-date.
```

现在您可以报告错误了！

## 基本调试 (Basic debugging)

当您想知道框架在做什么时，最好的方法是启用 `text_file` 输出插件，确保将 verbose 配置设置设置为 `True` 。这将生成一个非常详细的输出文件，可用于深入了解 `w3af` 的内部结构。

```bash
plugins
output text_file
output config text_file
set verbose True
back
```

## 漏报 (False Negtives)

如果 `w3af` 无法识别您手动验证的漏洞，请确保：

* 识别该漏洞的 `audit` 审计插件已启用
* 使用基本调试，确保 `w3af` 找到与漏洞相关的 URL 和 参数。果您没有在日志中看到该消息，请确保已启用 `crawl.web_spider` 插件。


对于漏报案例的报告和[报告bug错误案例](https://github.com/andresriancho/w3af/issues/new)是一样的，需要之前的全部信息。



## 误报 (False Postives)

没有人喜欢误报，不到一分钟的时间，您就会从“该站点容易受到SQL注入攻击”的肾上腺素转变为“否，误报”。对你的心脏不好。

对于误报案例的报告和[报告bug错误案例](https://github.com/andresriancho/w3af/issues/new)也是一样的，包含尽可能多的信息，请记住，我们必须验证误报，编写单元测试然后进行修复。


## 常见问题

经过多年的 `w3af` 开发，我们发现了一些常见问题，这些问题虽然不是bug，但却使我们的用户感到烦恼，并且很常见，可以包含在本节中。


### 过期的配置文件

当用户从旧的 `w3af` 版本迁移到新的版本，并且用户目录中存储的配置文件与最新版本不兼容时，就会出现这些问题之一。 `w3af` 将尝试打开旧的配置文件并失败，用户将看到类似以下内容：

![](http://docs.w3af.org/en/latest/_images/profile-error.png)

该错误是不言自明的：“您尝试加载的配置文件已过时”，但是缺少用户可以执行的一些“快速操作”，以避免看到此错误。如果您不关心旧的配置文件，请：

```bash
user@box:~/$ rm -rf ~/.w3af/profiles/
```

下次运行 `w3af` 时，它将把默认配置文件复制到用户的主目录。

对于真正关心旧版本配置文件的用户，我建议您使用以下步骤手动迁移它们：

* 备份您的 profile 配置文件
* 从主目录中删除他们 ((~/.w3af/profiles/))
* 打开 profile 文件，并且用文件编辑器修改以迁移他们
* 打开 `w3af` 并且新建一些插件配置
* 保存新的插件配置










