# 扫描 REST APIs (Scan REST APIs)

`w3af` 可用于识别和利用 REST API 中的漏洞。

扫描程序支持从使用 [Open API 规范记录](https://swagger.io/docs/specification/about/) 的 REST API 中提取端点和参数，这意味着 `w3af` 将能够以完全自动化的方式扫描这些 API。

当测试未使用 Open API 规范记录的 REST API 时，用户将不得不使用 `spider_man` 将与 REST API 调用关联的 HTTP 请求提供给框架。


## 扫描符合 Open API 规范的 REST API 应用

`crawl.open_api` 插件可用于识别 Open API 规范文档(`openapi.json`)的位置，并对其内容进行解析（通常 `openapi.json` 在 API 根目录中）。

解析 入口点 (endpoint), HTTP 头 (headers) 和 HTTP 参数 (parameter) 之后插件会将这些信息发送到 `w3af core` ，之后调用审计插件来识别漏洞。

使用此插件扫描 REST API 很容易，但是这里有一些提示：

* 如果您知道 Open API 规范文档(`openapi.json`) 的 URL，请将其包含在 `w3af` 的目标URL中，这将确保找到并扫描了该 API。
* 如果您有凭据信息，请在 `query_string_auth` 或 `header_auth` 中提供凭据信息，此信息将添加到与 REST API 关联的所有 HTTP 请求中。

即使您无法确定是否使用 Open API 规范记录了REST API，启用此插件也是一个好主意，因为该插件将找到该文档并创建一个信息性的发现，以确保用户可以手动对其进行检查。

## 将 HTTP 请求输入到 w3af

当 REST API 不存在 Open API 规范文档时，仅有的解决办法时手动输入这些 端点路径 和 参数信息 到扫描跨框架。

此过程可用于任何 REST API 的扫描，只需按照以下步骤将 HTTP 请求输入到 `w3af`:

1. 启用 `spider_man` 插件，你可以参考 “[高级使用示例](http://docs.w3af.org/en/latest/advanced-use-cases.html)” 章节。
2. 配置 REST API 客户端以通过 `127.0.0.1:44444` 的 HTTP 代理发送请求
3. 运行 REST API 客户端
4. 停止 `spider_man` 的代理 `curl -X GET http://127.7.7.7/spider_man?terminate --proxy http://127.0.0.1:44444`

> 说明:
> 由于这些 REST API 无法被爬取， `w3af` 只会审计被代理捕获的 HTTP 请求。用户教导 `w3af` 的全部 API 端点 和 参数信息的步骤是安全审计成功的关键。 


