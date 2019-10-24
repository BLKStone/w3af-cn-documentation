# 使用脚本自动化 w3af (Automation using scripts)

在开发 `w3af` 时，我们意识到需要快速简便的方法来一遍又一遍地执行相同的步骤，因此脚本功能诞生了。 `w3af` 可以使用 `-s` 参数运行脚本文件。脚本文件是文本文件， `w3af_console` 每行只有一个命令。示例脚本文件如下所示：

```bash
plugins
output text_file
output config text_file
set output_file output-w3af.txt
set verbose True
back
```

> 注意： 脚本非常适合使用 cron 对您的网站进行定期扫描！
> 示例脚本文件可以在 `scripts/` 目录内找到。
> 译注： https://github.com/andresriancho/w3af/tree/master/scripts

## VIM 语法文件


项目开发组还维护了一个 w3af 脚本的 [VIM 语法文件](http://www.vim.org/scripts/script.php?script_id=4567)
