# 运行 w3af (Running w3af)

`w3af` 有两个用户界面，控制台用户界面和图形用户界面。本用户指南将重点放在控制台用户界面上，在该界面上更容易解释框架的功能。要启动控制台用户界面，请执行以下操作：

```bash
$ ./w3af_console
w3af>>>
```

通过此提示，您将能够配置框架和插件设置，启动扫描并最终利用漏洞。此时，您可以开始键入命令。您必须学习的第一个命令是 `help` （请注意，命令区分大小写）：

```bash
w3af>>> help
|----------------------------------------------------------------|
| start         | Start the scan.                                |
| plugins       | Enable and configure plugins.                  |
| exploit       | Exploit the vulnerability.                     |
| profiles      | List and use scan profiles.                    |
| cleanup       | Cleanup before starting a new scan.            |
|----------------------------------------------------------------|
| help          | Display help. Issuing: help [command] , prints |
|               | more specific help about "command"             |
| version       | Show w3af version information.                 |
| keys          | Display key shortcuts.                         |
|----------------------------------------------------------------|
| http-settings | Configure the HTTP settings of the framework.  |
| misc-settings | Configure w3af misc settings.                  |
| target        | Configure the target URL.                      |
|----------------------------------------------------------------|
| back          | Go to the previous menu.                       |
| exit          | Exit w3af.                                     |
|----------------------------------------------------------------|
| kb            | Browse the vulnerabilities stored in the       |
|               | Knowledge Base                                 |
|----------------------------------------------------------------|
w3af>>>
w3af>>> help target
Configure the target URL.
w3af>>>
```

上面显示的帮助中说明了主菜单命令。每个菜单的内部结构将在本文档的后面部分看到。正如您已经注意到的，该 `help` 命令可以带有一个参数，如果可用，将显示该命令的详细帮助，例如 `help keys`。

关于控制台UI的其他有趣的事情是自动补全功能（键入'plu'，然后键入TAB）和命令历史记录（键入某些命令后，使用向上和向下箭头导航历史记录）。

要进入配置菜单，只需键入其名称并按Enter，您将看到提示如何更改，并且您现在处于该上下文中：

```bash
w3af>>> http-settings
w3af/config:http-settings>>>
```

所有配置菜单都提供以下命令：

* `help`
* `view`
* `set`
* `back`

这是 `http-settings` 菜单中这些命令的用法示例：

```bash
w3af/config:http-settings>>> help
|-----------------------------------------------------------------|
| view  | List the available options and their values.            |
| set   | Set a parameter value.                                  |
| save  | Save the configured settings.                           |
|-----------------------------------------------------------------|
| back  | Go to the previous menu.                                |
| exit  | Exit w3af.                                              |
|-----------------------------------------------------------------|
w3af/config:http-settings>>> view
|-----------------------------------------------------------------------------------------------|
| Setting                | Value    | Description                                               |
|-----------------------------------------------------------------------------------------------|
| url_parameter          |          | Append the given URL parameter to every accessed URL.     |
|                        |          | Example: http://www.foobar.com/index.jsp;<parameter>?id=2 |
| timeout                | 15       | The timeout for connections to the HTTP server            |
| headers_file           |          | Set the headers filename. This file has additional headers|
|                        |          | which are added to each request.                          |
|-----------------------------------------------------------------------------------------------|
...
|-----------------------------------------------------------------------------------------------|
| basic_auth_user        |          | Set the basic authentication username for HTTP requests   |
| basic_auth_passwd      |          | Set the basic authentication password for HTTP requests   |
| basic_auth_domain      |          | Set the basic authentication domain for HTTP requests     |
|-----------------------------------------------------------------------------------------------|
w3af/config:http-settings>>> set timeout 5
w3af/config:http-settings>>> save
w3af/config:http-settings>>> back
w3af>>>
```

总而言之，该 `view` 命令用于列出所有可配置参数及其值和说明。该 `set` 命令用于更改值。最后，我们可以执行 `back` 或按 CTRL + C返回上一级菜单。可以使用以下示例获取有关每个配置参数的详细帮助：

`help parameter`

```bash
w3af/config:http-settings>>> help timeout
Help for parameter timeout:
===========================
Set low timeouts for LAN use and high timeouts for slow Internet connections.

w3af/config:http-settings>>>
```

`http-settings` 和 `misc-settings` 配置菜单用于设置由所述框架所使用的系统范围的参数。所有参数都有默认值，在大多数情况下，您可以保留它们的原样。 `w3af` 其设计方式使初学者可以运行它而不必学习很多内部知识。

它也足够灵活，可以由知道他们想要什么并且需要更改内部配置参数来完成任务的专家进行调整。


## 用 GTK 用户界面运行 w3af (Running w3af with GTK user interface)

该框架还有一个图形用户界面，您可以通过执行以下命令开始：

```bash
$ ./w3af_gui
```

图形用户界面使您可以执行框架提供的所有操作，并采用一种更加简便快捷的方式来开始扫描和分析结果。

> 注意： GUI具有不同的第三方依赖性，可能需要您安装额外的 OS 和 python 软件包。
> 


## 插件配置 (Plugin configuration)

使用 `plugins` 配置菜单配置插件。

```bash
w3af>>> plugins
w3af/plugins>>> help
|-----------------------------------------------------------------------------|
| list             | List available plugins.                                  |
|-----------------------------------------------------------------------------|
| back             | Go to the previous menu.                                 |
| exit             | Exit w3af.                                               |
|-----------------------------------------------------------------------------|
| output           | View, configure and enable output plugins                |
| audit            | View, configure and enable audit plugins                 |
| crawl            | View, configure and enable crawl plugins                 |
| bruteforce       | View, configure and enable bruteforce plugins            |
| grep             | View, configure and enable grep plugins                  |
| evasion          | View, configure and enable evasion plugins               |
| infrastructure   | View, configure and enable infrastructure plugins        |
| auth             | View, configure and enable auth plugins                  |
| mangle           | View, configure and enable mangle plugins                |
|-----------------------------------------------------------------------------|
w3af/plugins>>>
```

可以在此菜单中配置除 `attack` 插件以外的所有插件。让我们列出所有  `audit` 类型的插件：

```bash
w3af>>> plugins
w3af/plugins>>> list audit
|-----------------------------------------------------------------------------|
| Plugin name        | Status | Conf | Description                            |
|-----------------------------------------------------------------------------|
| blind_sqli         |        | Yes  | Identify blind SQL injection           |
|                    |        |      | vulnerabilities.                       |
| buffer_overflow    |        |      | Find buffer overflow vulnerabilities.  |
...
```

要启用 `xss` 和 `sqli` 插件，然后验证框架是否理解该命令，我们发出以下命令集：

```bash
w3af/plugins>>> audit xss, sqli
w3af/plugins>>> audit
|----------------------------------------------------------------------------|
| Plugin name        | Status  | Conf | Description                          |
|----------------------------------------------------------------------------|
| sqli               | Enabled |      | Find SQL injection bugs.             |
| ssi                |         |      | Find server side inclusion           |
|                    |         |      | vulnerabilities.                     |
| ssl_certificate    |         | Yes  | Check the SSL certificate validity   |
|                    |         |      | (if https is being used).            |
| un_ssl             |         |      | Find out if secure content can also  |
|                    |         |      | be fetched using http.               |
| xpath              |         |      | Find XPATH injection                 |
|                    |         |      | vulnerabilities.                     |
| xss                | Enabled | Yes  | Identify cross site scripting        |
|                    |         |      | vulnerabilities.                     |
| xst                |         |      | Find Cross Site Tracing              |
|                    |         |      | vulnerabilities.                     |
|----------------------------------------------------------------------------|
w3af/plugins>>>
```

或者，如果用户有兴趣确切了解插件的功能，那么他也可以运行以下 `desc` 命令：

```bash
w3af/plugins>>> audit desc xss

This plugin finds Cross Site Scripting (XSS) vulnerabilities.

One configurable parameters exists:
    - persistent_xss

To find XSS bugs the plugin will send a set of javascript strings to
every parameter, and search for that input in the response.

The "persistent_xss" parameter makes the plugin store all data
sent to the web application and at the end, request all URLs again
searching for those specially crafted strings.

w3af/plugins>>>
```

现在我们知道了此插件的功能，但让我们检查一下其内部：

```bash
w3af/plugins>>> audit config xss
w3af/plugins/audit/config:xss>>> view
|-----------------------------------------------------------------------------|
| Setting        | Value | Description                                        |
|-----------------------------------------------------------------------------|
| persistent_xss | True  | Identify persistent cross site scripting           |
|                |       | vulnerabilities                                    |
|-----------------------------------------------------------------------------|
w3af/plugins/audit/config:xss>>> set persistent_xss False
w3af/plugins/audit/config:xss>>> back
The configuration has been saved.
w3af/plugins>>>
```

具体 `xss` 插件的配置菜单还具有 `set` 命令， 用于更改参数值; 以及 `view`命令，用于列出现有参数值。在刚才的示例中，我们禁用了 `xss` 插件中的存储型型跨站脚本的检查。

## 保存配置

设置插件和框架配置后，就可以将这些信息保存到配置文件中：

```bash
w3af>>> profiles
w3af/profiles>>> save_as tutorial
Profile saved.
```

配置文件另存为中的文件 `~/.w3af/profiles/` 。可以加载保存的配置以运行新的扫描：

```bash
w3af>>> profiles
w3af/profiles>>> use fast_scan
The plugins configured by the scan profile have been enabled, and their options configured.
Please set the target URL(s) and start the scan.
w3af/profiles>>>
```



与其他用户共享配置文件可能会出现问题，因为它们包含插件配置引用的文件的完整路径，这将要求用户共享配置文件，引用的文件并手动编辑配置文件以匹配当前环境。为了解决此问题，添加了 `self-contained` 标志：

```bash
w3af>>> profiles
w3af/profiles>>> save_as tutorial self-contained
Profile saved.
```

一个 `self-contained` 的配置 (profile) 文件会打包所有的引用文件，因此可以很方便的与其他用户共享。

## 启动扫描 (Starting the scan)

配置所有所需的插件后，用户必须设置目标URL，最后开始扫描。目标选择是通过以下方式完成的：

```bash
w3af>>> target
w3af/config:target>>> set target http://localhost/
w3af/config:target>>> back
w3af>>>
```

最后，运行 `start` 以运行所有已配置的插件。

```bash
w3af>>> start
```

在扫描过程中，您可以随时点击 `<enter>` 以获取w3af核心的实时状态。状态行如下所示：
    

```bash
Status: Running discovery.web_spider on http://localhost/w3af/ | Method: GET.
```


