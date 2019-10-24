# w3af 常见使用场景 (Common use cases)

由于存在多个配置设置，因此框架有时很难找到如何执行特定任务的方法，此页面说明了如何使用 `w3af` 执行一些常见的用例。

## 仅扫描单个目录 (Scanning only one directory)


审核站点时，通常只对特定目录中的URL感兴趣。为了完成此任务，请按照下列步骤操作：

* 将 target URL 设置为 `http://domain/directory/`
* 启用全部 audit 插件
* 启用 `crawl.web_spider` 插件
* 在 `crawl.web_spider` 插件设置中，将 `only_forward` 设置为 `True` 
* 

使用此配置，爬虫插件将仅提供 `/directory` 目录下的 URL 。然后，审计局插件将仅扫描该目录内的 URL。

## 保存 URL 并将其用作其他扫描的输入 (Saving URLs and using them as input for other scans)

爬取 URL 可能是一个扫描任务中最耗费时间的过程，在某些情况下，这需要人工干预（spider man 插件）。为了保存在扫描过程中找到的所有URL，可以使用 `output.export_requests` 插件将URL写入用户配置的路径。

使用该 `import_results` 插件可以加载已保存的数据，该插件将读取所有信息并将其提供给 `w3af core`。




