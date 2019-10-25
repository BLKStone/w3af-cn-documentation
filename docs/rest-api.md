# REST API 简介 (REST API Introduction)

本文档部分是 `w3af` 的 REST API 服务的用户指南，其目的是为开发人员提供使用任何开发语言与 `w3af` 服务对接的知识。

我们建议您 在深入研究此 REST API 文档前之前，先通读 [w3af 用户指南](http://docs.w3af.org/)。


## 启动 REST API 服务

可以通过运行以下命令来启动 REST API：

```bash
$ ./w3af_api
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

或者也可以在 docker 容器中运行：

```bash
$ cd extras/docker/scripts/
$ ./w3af_api_docker
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

## 认证

可以通过在命令行设置一个 SHA-512 哈希过的密码 `-p <SHA512_HASH>` 来设置认证机制。
你也可以在配置文件中的 `password:` 字段进行设置。

Linux 和 Mac 用户可以用如下方式生成明文的 SHA512 哈希值

```bash
$ echo -n "secret" | sha512sum
bd2b1aaf7ef4f09be9f52ce2d8d599674d81aa9d6a4421696dc4d93dd0619d682ce56b4d64a9ef097761ced99e0f67265b5f76085e5b0ee7ca4696b2ad6fe2b2  -

$ ./w3af_api -p "bd2b1aaf7ef4f09be9f52ce2d8d599674d81aa9d6a4421696dc4d93dd0619d682ce56b4d64a9ef097761ced99e0f67265b5f76085e5b0ee7ca4696b2ad6fe2b2"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

在上面的示例中，用户只能使用 HTTP 基本身份验证以及默认的用户名 `admin` 和密码 `secret` 进行连接。

例如，使用以下 `curl` 命令

```bash
$ curl -u admin:secret http://127.0.0.1:5000
{
  "docs": "http://docs.w3af.org/en/latest/api/index.html"
}
```

请注意，即使使用基本身份验证，往返于REST API的流量也不会加密，这意味着身份验证和漏洞信息仍可能被具有“中间人”功能的攻击者嗅探到。

在公共可用 IP 地址上运行REST API时，建议采取其他预防措施，包括在 SSL 代理服务器（例如启用了mod_proxy 的 Pound, nginx, Apache）之后运行它。


## 配置文件格式

使用配置文件是可选的，并且是存储设置的方便位置，否则可以使用命令行参数指定设置。

配置文件为标准YAML格式，并接受命令行上的任何选项。示例配置文件如下所示：


```yaml
# This is a comment
host: '127.0.0.1'
port: 5000
verbose: False
username: 'admin'
# The SHA512-hashed password is 'secret'. We don't recommend using this.
password: 'bd2b1aaf7ef4f09be9f52ce2d8d599674d81aa9d6a4421696dc4d93dd0619d682ce56b4d64a9ef097761ced99e0f67265b5f76085e5b0ee7ca4696b2ad6fe2b2'
```

在上面的示例中，除 password 之外的所有其他值均是默认值，可以在不更改API运行方式的情况下从配置文件中省略。

## 通过 TLS/SSL 进行服务

`w3af` 的 REST API 使用 Flask 提供， Flask 可用于通过 `TLS/SSL` 传递内容。默认情况下，`w3af` 将生成一个自签名证书，并使用该自签名证书将 HTTPS 协议绑定到 5000 端口。

如果用户想禁用 https 功能，可以在命令行张加入 `--no-ssl` 参数。

如果高级用户想使用自己的 SSL 证书，它可以：

* 将 w3af 启动成 HTTP 模式，并且使用类似 nginx 的代理来处理 SSL 流量，将解密后的流量转发到 REST API。
* 将用户生成的 SSL 证书和密钥到 `/.w3af/ssl/w3af.crt` 和 `/.w3af/ssl/w3af.key` 并且使用 `--no-ssl` 参数来启动 `./w3af_api`

> 注意：
> 与在 `w3af-api` 中直接运行 SSL 相比，使用 `nginx` 来部署 `w3af` 为用户提供了贡多配置选项和安全性。
> 


## REST API 源码

REST API 的源码在 `[w3af/core/ui/api/](https://github.com/andresriancho/w3af/tree/master/w3af/core/ui/api/)` 目录下，可以根据您的兴趣选择进一步阅读。

## REST API 客户端

如果您写了 REST API 的其他客户端？可以让我们知道并链接到这里！

官方 [Python REST API 客户端](https://github.com/andresriancho/w3af-api-client)，也可以[在 pypi上获得](https://pypi.python.org/pypi/w3af-api-client)


## API 端点 


* [The `/scan/` resource](http://docs.w3af.org/en/latest/api/scans.html)
    * [Starting a scan](http://docs.w3af.org/en/latest/api/scans.html#starting-a-scan)
* [The `/kb/` resource](http://docs.w3af.org/en/latest/api/kb.html)
    * [List](http://docs.w3af.org/en/latest/api/kb.html#list)
    * [Knowledge base filter](http://docs.w3af.org/en/latest/api/kb.html#knowledge-base-filters)
    * [Details](http://docs.w3af.org/en/latest/api/kb.html#details)
* [The `/version` resource](http://docs.w3af.org/en/latest/api/version.html)
* [The `/traffic/` resource](http://docs.w3af.org/en/latest/api/traffic.html)
    * [Encoding](http://docs.w3af.org/en/latest/api/traffic.html#encoding)
* [The `/urls/` resource](http://docs.w3af.org/en/latest/api/urls.html)
* [The `/fuzzable-requests/` resource](http://docs.w3af.org/en/latest/api/urls.html#the-fuzzable-requests-resource)
    * [Encoding](http://docs.w3af.org/en/latest/api/urls.html#encoding)
* [The `/exceptions/` resource](http://docs.w3af.org/en/latest/api/exceptions.html)
    * [Reporting vulnerabilities](http://docs.w3af.org/en/latest/api/exceptions.html#reporting-vulnerabilities)


