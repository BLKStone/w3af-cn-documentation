# w3af 检测规则 - rfi

## 说明

首先从 audit 入口开始， 远程文件包含 (Remote File Inclusion) 漏洞的检测包含两个手段。
其一是使用 `http://w3af.org/rfi.html` 这个文件检测；其二是 `w3af` 自己再启动一个服务端，将需要远程包含的文件部署好。

```python
def audit(self, freq, orig_response, debugging_id):
        """
        Tests an URL for remote file inclusion vulnerabilities.
        :param freq: A FuzzableRequest
        :param orig_response: The HTTP response associated with the fuzzable request
        :param debugging_id: A unique identifier for this call to audit()
        """
        # The plugin is going to use two different techniques:
        # 1- create a request that will include a file from the w3af site
        if self._use_w3af_site:
            self._w3af_site_test_inclusion(freq, orig_response, debugging_id)

        # Sanity check required for #2 technique
        config_ok, config_message = self._correctly_configured()

        if not config_ok and not self._error_reported:
            # Report error to the user only once
            self._error_reported = True
            om.out.error(self.CONFIG_ERROR_MSG % config_message)
            return
        
        # 2- create a request that will include a file from a local web server
        self._local_test_inclusion(freq, orig_response, debugging_id)
        
        # Now that we've captured all vulnerabilities, report the ones with
        # higher risk
        self._report_vulns()
```

## 利用 w3af 网站检测远程文件包含

远程文件的地址是 `http://w3af.org/rfi.html` ， 内容如下：

```java
<% out.print("w3af"); out.print(" by Andres Riancho"); %>
```

将 HTTP 参数中能替换的部分都尝试替换成 `http://w3af.org/rfi.html` 来获取响应，会根据响应内容分别提供不同等级的告警。如果在响应中检测到 `w3af by Andres Riancho`，说明该远程包含是可执行代码的。
如果只检测到 `w3af` 和 `by Andres Riancho` ，说明虽然可以包含远程代码，但是不可执行。此外， `w3af` 还会尝试找一些错误信息。

```python
RFI_ERRORS = ('php_network_getaddresses: getaddrinfo',
                  'failed to open stream: Connection refused in'
                  'java.io.FileNotFoundException',
                  'java.net.ConnectException',
                  'java.net.UnknownHostException')   

def _analyze_result(self, rfi_data, mutant, response):
    """
    Analyze results of the _send_mutant method.
    """
    if rfi_data.rfi_result in response:
        desc = 'A remote file inclusion vulnerability that allows remote' \
               ' code execution was found at: %s' % mutant.found_at()

        v = Vuln.from_mutant('Remote code execution', desc,
                             severity.HIGH, response.id, self.get_name(),
                             mutant)

        self._vulns.append(v)

    elif rfi_data.rfi_result_part_1 in response \
    and rfi_data.rfi_result_part_2 in response:
        # This means that both parts ARE in the response body but the
        # rfi_data.rfi_result is NOT in it. In other words, the remote
        # content was embedded but not executed
        desc = 'A remote file inclusion vulnerability without code' \
               ' execution was found at: %s' % mutant.found_at()

        v = Vuln.from_mutant('Remote file inclusion', desc,
                             severity.MEDIUM, response.id, self.get_name(),
                             mutant)

        self._vulns.append(v)

    else:
        #
        #   Analyze some errors that indicate that there is a RFI but
        #   with some "configuration problems"
        #
        for error in self.RFI_ERRORS:
            if error in response and not error in mutant.get_original_response_body():
                desc = 'A potential remote file inclusion vulnerability' \
                       ' was identified by the means of application error' \
                       ' messages at: %s' % mutant.found_at()

                v = Vuln.from_mutant('Potential remote file inclusion',
                                     desc, severity.LOW, response.id,
                                     self.get_name(), mutant)

                v.add_to_highlight(error)
                self._vulns.append(v)
                break
```


## 利用本地服务器检测远程文件包含

利用本地服务器的文件来检测 RFI, 会在 w3af 上开启一个 HTTP 应用的服务端口，用于部署被远程包含的文件。使用的 payload 大致如下：

```php
# php
<?php echo "8PcokTUkv"; echo "oudVjYpIm"; ?>
# php
<? echo "8PcokTUkv"; echo "oudVjYpIm"; ?>
# jsp
<% out.print("8PcokTUkv"); out.print("oudVjYpIm"); %>
# asp
<% \n response.write("8PcokTUkv");\n response.write("oudVjYpIm"); \n %>

```