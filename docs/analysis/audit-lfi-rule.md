# w3af 检测规则 - lfi

## 说明

lfi 就是本地文件包含漏洞 (LFI, local file inclusion) 。
从插件入口开始,基本上就是构造变异请求，分析响应的过程。

```python
def audit(self, freq, orig_response, debugging_id):
    """
    Tests an URL for local file inclusion vulnerabilities.
    :param freq: A FuzzableRequest
    :param orig_response: The HTTP response associated with the fuzzable request
    :param debugging_id: A unique identifier for this call to audit()
    """
    mutants = create_mutants(freq,
                             self.get_lfi_tests(freq),
                             orig_resp=orig_response)

    self._send_mutants_in_threads(self._uri_opener.send_mutant,
                                  mutants,
                                  self._analyze_result,
                                  grep=False,
                                  debugging_id=debugging_id)
```


在构造变异请求的过程中，有一点值得注意，假设被扫描的页面 URL 为 `http://host.tld/show_user.php?id=1`， `w3af` 会自动构造两个自包含的 payload 。例如， `http://host.tld/show_user.php?id=show_user.php` 和 `http://host.tld/show_user.php?id=/show_user.php` 。之后会添加一些常见的 LFI 检测 payload。 


```python
def get_lfi_tests(self, freq):
    """
    :param freq: The fuzzable request we're analyzing
    :return: The paths to test
    """
    #
    #   Add some tests which try to read "self"
    #   http://host.tld/show_user.php?id=show_user.php
    #
    lfi_tests = [freq.get_url().get_file_name(),
                 '/%s' % freq.get_url().get_file_name()]

    #
    #   Add some tests which try to read common/known files
    #
    lfi_tests.extend(self._get_common_file_list(freq.get_url()))

    return lfi_tests
```

常见的 LFI 攻击载荷如下， `w3af` 会根据知识库 (knowledgebase) 中目标的操作系统来优化 payload，尽可能减少请求数。



```python
def _get_common_file_list(self, orig_url):
    """
    This method returns a list of local files to try to include.

    :return: A string list, see above.
    """
    local_files = []

    extension = orig_url.get_extension()

    # I will only try to open these files, they are easy to identify of they
    # echoed by a vulnerable web app and they are on all unix or windows
    # default installs. Feel free to mail me (Andres Riancho) if you know
    # about other default files that could be installed on AIX ? Solaris ?
    # and are not /etc/passwd
    if cf.cf.get('target_os') in {'unix', 'unknown'}:
        local_files.append('/../' * 15 + 'etc/passwd')
        local_files.append('../' * 15 + 'etc/passwd')

        local_files.append('/../' * 15 + 'etc/passwd\0')
        local_files.append('/../' * 15 + 'etc/passwd\0.html')
        local_files.append('/etc/passwd')

        # This test adds support for finding vulnerabilities like this one
        # http://website/zen-cart/extras/curltest.php?url=file:///etc/passwd
        local_files.append('file:///etc/passwd')

        local_files.append('/etc/passwd\0')
        local_files.append('/etc/passwd\0.html')

        if extension != '':
            local_files.append('/etc/passwd%00.' + extension)
            local_files.append('/../' * 15 + 'etc/passwd%00.' + extension)

    if cf.cf.get('target_os') in {'windows', 'unknown'}:
        local_files.append('/../' * 15 + 'boot.ini')
        local_files.append('../' * 15 + 'boot.ini')

        local_files.append('/../' * 15 + 'boot.ini\0')
        local_files.append('/../' * 15 + 'boot.ini\0.html')

        local_files.append('C:\\boot.ini')
        local_files.append('C:\\boot.ini\0')
        local_files.append('C:\\boot.ini\0.html')

        local_files.append('%SYSTEMROOT%\\win.ini')
        local_files.append('%SYSTEMROOT%\\win.ini\0')
        local_files.append('%SYSTEMROOT%\\win.ini\0.html')

        # file:// URIs for windows , docs here: http://goo.gl/A9Mvux
        local_files.append('file:///C:/boot.ini')
        local_files.append('file:///C:/win.ini')

        if extension != '':
            local_files.append('C:\\boot.ini%00.' + extension)
            local_files.append('%SYSTEMROOT%\\win.ini%00.' + extension)

    return local_files
```

如果变异请求的响应中包含了 FILE_PATTERNS 的信息，且原始响应不包含该信息，`w3af` 就认为发现了一个 LFI 漏洞。

```python
FILE_PATTERNS = (
        "root:x:0:0:",
        "daemon:x:1:1:",
        ":/bin/bash",
        ":/bin/sh",

        # /etc/passwd in AIX
        "root:!:x:0:0:",
        "daemon:!:x:1:1:",
        ":usr/bin/ksh",

        # boot.ini
        "[boot loader]",
        "default=multi(",
        "[operating systems]",

        # win.ini
        "[fonts]",
)
```

如果上述规则未检测出漏洞， `w3af` 还会尝试去寻找一些可疑的报错信息，以辅助人工渗透。检测的具体报错信息如下

```python
FILE_OPEN_ERRORS = [# Java
                    'java.io.FileNotFoundException:',
                    'java.lang.Exception:',
                    'java.lang.IllegalArgumentException:',
                    'java.net.MalformedURLException:',

                    # PHP
                    'fread\\(\\):',
                    'for inclusion \'\\(include_path=',
                    'Failed opening required',
                    '<b>Warning</b>:  file\\(',
                    '<b>Warning</b>:  file_get_contents\\(',
                    'open_basedir restriction in effect']
```