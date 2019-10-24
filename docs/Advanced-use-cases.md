# 高级使用示例 (Advanced use cases)

## 针对复杂的 web 应用 (Complex Web applications)

一些 Web 应用程序使用浏览器端技术，例如 JavaScript, Flash 和 Java  applet，这些技术是浏览器可以理解的，但 w3af 仍然无法理解。

为了解决这个问题， w3af 提供了一个名为 `spider_man` 的插件，从而允许用户分析复杂的 Web 应用程序。该插件会启动一个 HTTP 代理，供用户用来浏览目标站点，在此过程中，该插件将从请求中提取信息并将其发送给已启用的  `audit` 插件。

> 说明:
> 当存在Javascript，Flash，Java applet 或任何其他浏览器端技术时，都可以使用 `spider_man` 插件。唯一的要求是用户使用`spider_man` 开启的 HTTP(s) 代理，之后手动浏览站点。
> 


> 注意:
> 关于如何在你的浏览器中设置 `w3af` 的 CA 证书，请参考 [CA 设置章节](http://docs.w3af.org/en/latest/ca-config.html)
> 

> 译注:
> spider_man 插件可以理解为被动扫描模式

一个简单的示例将阐明问题，让我们假设使用 `w3af` 是对网站进行扫描，然而在主页上找不到任何链接。在用户仔细检查结果之后，很明显主页上有一个 Java applet 菜单，所有其他部分都链接到该菜单。因此，用户在激活 `crawl.spider_man` 插件后再次运行 `w3af` ，现，使用 浏览器 和`spider_man` 代理 手动浏览站点。用户完成浏览后，`w3af`将继续进行所有艰苦的漏洞审计工作。


这是一个 `spider_man` 插件运行的示例：

```bash
w3af>>> plugins
w3af/plugins>>> crawl spider_man
w3af/plugins>>> audit sqli
w3af/plugins>>> back
w3af>>> target
w3af/target>>> set target http://localhost/
w3af/target>>> back
w3af>>> start
spider_man proxy is running on 127.0.0.1:44444 .
Please configure your browser to use these proxy settings and navigate the target site.
To exit spider_man plugin please navigate to http://127.7.7.7/spider_man?terminate .
```

现在，用户将其浏览器配置为使用 `127.0.0.1:44444` 作为 HTTP 代理，并浏览目标站点，当他完成对要审核的站点部分的所有功能使用时时，他导航到该站点 `http://127.7.7.7/spider_man?terminate` 将停止代理并完成插件。 之后 `audit.sqli` 插件会在识别的 HTTP 请求上运行。

## 忽略特定表单 (Ignoring specific forms)

`w3af` 允许用户使用 表单ID排除功能来配置要忽略的表单。当用户在先前（更简单）的排除模型中发现限制时才创建此功能，此前模型的限制是仅允许使用 URL 匹配来忽略表单。


可以使用以下格式提供的表单ID列表配置排除项：

```python
[{"action":"/products/.*",
  "inputs": ["comment"],
  "attributes": {"class": "comments-form"},
  "hosted_at_url": "/products/.*",
  "method": "get"}]
```

在这当中

* `action` 是与 form action 属性中的 URL 路径匹配的正则表达式。
* `inputs` 是 form inputs 的 list
* `attributes` 是 `<form>` 标签所包含的属性
* `hosted_at_url` 与表单所在页面的 URL 路径匹配的正则表达式
* `method` 是用于提交表单的 HTTP 方法。

假设用户希望忽略发送到某个特定 form action 路径的，且 `<form>` 标签中的 class 属性值是 "comments-form" 的表单，他可以用一下方式配置：

```python
[{"action":"/products/comments",
  "attributes": {"class": "comments-form"}}]
```

在表单ID忽略列表中可以指定多种形式，例如下面的配置将排除所有使用 POST 或 PUT 提交的表单：

```python
[{"method": "post"}, {"method": "put"}]
```

也可以使用以下方法忽略所有表单：

```python
[{}]
```

可以使用 `misc-settings` 菜单中的两个变量配置此功能：

* `form_id_list`: 包含上面说明的格式以匹配表单的字符串。
* `form_id_action`: 在默认形况下 的`w3af` 设置，会排除 `form_id_list` 匹配中的表单，但是当`form_id_action` 的值被设置为 `include` 时， `w3af` 会仅扫描 `form_id_list` 匹配的表单。


为了简化此设置的配置， `w3af` 将在 `debug` 输出中添加一行（请确保将 `verbose` 设置为 `true` 以在输出文件插件中看到这些行），其中包含每个已识别的表单的表单ID。

> 说明：
> 这个特性可以与 `non_targets` 插件的功能一起使用。 `w3af` 会仅将请求发送到同时满足两个过滤器的目标。
> 

## URL 变量

爬取 web 应用是一项富有挑战的任务。有些 web 应用拥有数千 URL, 而着其中部分 URL 有拥有一个或多个 HTML 表单。 让我们探索一个包含一千种产品的通用电子商务站点，每种产品以不同的 URL 显示，例如：

* `/products/title-product-A`
* `/products/another-product-title`
* `/review-comment?id=6631`


浏览到每个 URL 时，每个产品页包含三个 HTML 表单，一个用于将产品添加到购物车，另一个用于收藏产品，最后一个用于询问有关该产品的问题。每个表单的表单的 action 路径都是设置到当前的产品信息页。

Web 应用程序安全扫描的主要目标是用最少的 HTTP 请求数量来实现完整的测试覆盖率（所有应用程序代码都经过测试）。

`w3af` 需要能够有效地爬网站点，从而减少 HTTP 请求的数量以达到完整的测试范围。
可以做出一些假设：

1. 提交某一个收藏产品功能表单的服务端处理代码与提交另一个收藏产品功能的服务端处理代码是一样的。
2. 浏览不同产品 `/product/*` 页面，在不同产品下，服务端运行的代码也是基本一样的，并且总是展示那三个 HTML 表单
3. 请求 `/review-comment?id=*` 总会返回一条用户评价。

如果我们确信这些假设是真的，那么在扫描过程中，我们可以简单索取一些样本，而不是扫描每一个链接。可以使用 `misc` 设置中的以下选项配置要收集的样本数量：

1. `path_max_variants`: 限制要抽样爬取的产品页面数
2. `params_max_variants`： 限制具有相同路径和参数名称的页面数
3. `max_equal_form_variants`： 限制要抽样的具有相同参数但 URL 不同的表单的数量

`w3af` 的默认值应适合大多数站点，但高级用户可能需要在“扫描花费太多时间”；“未扫描应用程序的多个区域”；“调试日志显示许多包含"Ignoring ... simply a variant"消息”时，修改这些默认参数值。


