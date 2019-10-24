# w3af 的认证 / 登陆后扫描 (Authentication)

w3af 支持以下类型的身份验证方案：

* HTTP 基础认证
* NTLM 身份认证
* 基于表单的身份认证
* 设置 HTTP Cookie


如果用户提供了凭据， `w3af` 将确保使用活动的用户会话运行扫描。

HTTP 基本认证 和 NTLM 身份验证是 Web 服务器通常提供的两种 HTTP 级别身份验证，而表单和 cookie 身份验证方法则由应用程序本身提供。由用户确定与应用程序保持会话所需的身份验证方法，但是通常，对 HTTP 流量的快速检查将确定所需的身份验证方法。

## HTTP 基础认证 与 NTML 身份认证 (Basic and NTLM authentication)

要配置 HTTP 基础认证 或 NTLM 凭据，请打开 HTTP 设置菜单。本节中设置的配置将影响所有插件和其他核心库。

```bash
w3af>>> http-settings
w3af/config:http-settings>>> view
|--------------------------------------------------------------------------------------|
| Setting                | Description                                                 |
|--------------------------------------------------------------------------------------|
...
|--------------------------------------------------------------------------------------|
| ntlm_auth_url          | Set the NTLM authentication domain for HTTP requests        |
| ntlm_auth_user         | Set the NTLM authentication username for HTTP requests      |
| ntlm_auth_passwd       | Set the NTLM authentication password for HTTP requests      |
| ntlm_auth_domain       | Set the NTLM authentication domain (the windows domain name)|
|                        | requests. Please note that only NTLM v1 is supported.       |
|--------------------------------------------------------------------------------------|
...
|--------------------------------------------------------------------------------------|
| basic_auth_user        | Set the basic authentication username for HTTP requests     |
| basic_auth_passwd      | Set the basic authentication password for HTTP requests     |
| basic_auth_domain      | Set the basic authentication domain for HTTP requests       |
|--------------------------------------------------------------------------------------|
w3af/config:http-settings>>>
```

请注意 HTTP 基础认证和 NTLM 身份验证的两个不同的配置部分。输入您的首选设置，然后选择 `save`。现在，扫描器已准备好开始经过身份验证的扫描，下一步是启用特定插件并开始扫描。

> 注意
> NTLM 和 HTTP 基础认证通常需要带有 `\` 字符的用户名，需要在 w3af-console 中进行转义输入。例如，将 `domain\user` 作为用户名输入时，需要使用 `set basic_auth_user domain\\user` 。

## 表单认证

在最新的 w3af 版本中，表单身份验证发生了显着变化。从版本 1.6 开始，使用 `auth` 插件配置表单身份验证。框架中提供了两个身份验证插件：

* detailed
* generic

身份验证插件是一种特殊类型的插件，它负责在整个扫描过程中使会话保持活动状态。在开始扫描之前身份验证插件就会被调用（为了获得新的会话），将在扫描运行时每 5 秒调用一次这些插件（以验证当前会话是否仍然存在，并在需要时创建一个新的会话）。


本教程将说明如何配置具有以下选项的身份验证插件 `generic` ：

* `username` Web 应用程序的用户名
* `password` Web 应用程序的密码
* `username_field` 可以在登录 HTML 源代码中找到的用户名表单输入的名称。
* `password_field` 可以在登录 HTML 源中找到的密码表单输入的名称。
* `auth_url` 将用户名和密码发布到的网址。
* `check_url` 用于检查会话是否仍处于活动状态的URL，通常将其设置为Web应用程序用户的设置页面。
* `check_string` 一个字符串，如果在check_url的HTTP响应正文中发现该字符串证明该会话仍处于活动状态，通常将其设置为只能在用户的设置页面中找到的字符串，例如他的姓氏。

一旦配置了所有这些设置，建议仅启用 `crawl.web_spider` 和 `auth.generic` 来进行测试扫描，以验证是否可以爬取登陆后才能访问的链接。另外，请注意 `w3af` 的日志，因为如果身份验证过程出现任何问题，身份验证插件将创建日志条目。日志条目如下：

```text
Login success for admin/password
User "admin" is currently logged into the application
```

您预期看到的是配置是否成功以及类似以下消息：

```text
Can't login into web application as admin/password
```

这说明认证插件配置不正确，或者应用程序需要将更多参数发送到 `auth_url` ，在某些情况下，可以通过使用 `detailed` 插件来解决。


> 警告:
> 配置 `crawl.web_spider` 插件以忽略注销链接。这很重要，因为我们希望在扫描期间保持会话活动。
> 
> 注意:
> 创建新的身份验证插件很容易！可以通过克隆 detailed 认证插件来添加自定义身份验证类型。
> 



## 设置 HTTP Cookie (Setting HTTP Cookie)

对于表单身份验证不起作用的情况（可能与包含反 CSRF 令牌或 两因素身份验证 的登录表单有关）， `w3af` 为用户提供了一种设置一个或多个 HTTP  Cookie 的方法，以便在扫描期间使用。

您可以通过任何喜欢的方式捕获这些 cookie ：直接从浏览器中，使用Web代理， wireshark 等。

使用文本编辑器创建 [Netscape 格式的 cookie jar](http://www.cookiecentral.com/faq/#3.5) 文件，替换示例值：

```
# Netscape HTTP Cookie File
.netscape.com   TRUE    /   FALSE   946684799   NETSCAPE_ID 100103
```

创建文件后 cookie_jar_fil e，在 http-settings 菜单中进行设置以指向该文件。

> 警告
> 确保您创建的文件符合规范，Python的cookie解析器非常严格，如果发现任何错误，则不会加载cookie。
> 
> 最常见的错误是省略域名开头的点（请参阅 .netscape.com ），并使用空格代替制表符作为字段分隔符（上面的示例使用制表符，但 HTML 渲染器可能会将其替换为空格） 。




## 设置 HTTP 头 (Setting HTTP headers)

一些 Web 应用程序使用自定义 HTTP 标头进行身份验证，w3af 框架也支持此标头。

此方法将设置一个HTTP请求标头，该标头将添加到框架发送的每个HTTP请求中，请注意，使用此方法时不会对会话状态进行验证，如果会话无效，则扫描将继续使用无效会话（标头值）。


为了使用此方法，您首先必须：

* 使用您喜欢的文本编辑器创建一个文本文件，其内容如下 `Cookie: <insert-cookie-here>` ; 不带引号，并插入所需的会话 cookie。
* 然后，在 `w3af` 的 `http-settings` 配置菜单中，将 `headers_file` 配置参数设置为指向最近创建的文件。
* `save` 保存

即可将 `w3af` 扫描程序配置为对所有 HTTP 请求都使用 HTTP 会话cookie。

