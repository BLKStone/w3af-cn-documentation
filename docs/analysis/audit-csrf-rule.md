# w3af 检测规则 - csrf

## 说明


从 audit 入口开始，首先会判断 web 应用是否做 `Referer/Origin` 检查；之后扫描器会检查 web 应用中是否存在一些常见的 CSRF Token。当 web 应用既没有 `Referer/Origin` 检查，又没有一些常见的 CSRF Token 时， `w3af` 就会认为这个请求存在 CSRF 漏洞。


```python
def audit(self, freq, orig_response, debugging_id):
    """
    Test URLs for CSRF vulnerabilities.
    :param freq: A FuzzableRequest
    :param orig_response: The HTTP response associated with the fuzzable request
    :param debugging_id: A unique identifier for this call to audit()
    """
    if not self._is_suitable(freq, orig_response):
        return

    #
    # Referer / Origin check
    #
    # IMPORTANT NOTE: I'm aware that checking for the referer header does
    # NOT protect the application against all cases of CSRF, but it's a
    # very good first step.
    #
    # In order to exploit a CSRF in an application
    # that protects using this method an intruder would have to identify
    # other vulnerabilities, such as XSS or open redirects, in the same
    # domain.
    #
    # TODO: This algorithm has lots of room for improvement
    if self._is_origin_checked(freq, orig_response, debugging_id):
        om.out.debug('Origin for %s is checked' % freq.get_url())
        return

    # Does the request have CSRF token in query string or POST payload?
    if self._find_csrf_token(freq):
        return

    # Ok, we have found vulnerable to CSRF attack request
    msg = 'Cross Site Request Forgery has been found at: %s' % freq.get_url()

    v = Vuln.from_fr('CSRF vulnerability', msg, severity.MEDIUM,
                     orig_response.id, self.get_name(), freq)

    self.kb_append_uniq(self, 'csrf', v)
```

`Referer/Origin` 检查的过程就是将原始请求的 Referer 字段替换为一个无效值 (http://www.w3af.org/),之后对比原始请求与变异请求分别对应的响应，如果两个 HTTP 响应相似度很高， `w3af` 则认为应用程序没有做 `Referer/Origin` 检查。
 

```python
def _is_origin_checked(self, freq, orig_response, debugging_id):
    """
    :return: True if the remote web application verifies the Referer before
             processing the HTTP request.
    """
    fake_ref = 'http://www.w3af.org/'

    mutant = HeadersMutant(copy.deepcopy(freq))
    headers = mutant.get_dc()
    headers['Referer'] = fake_ref
    mutant.set_token(('Referer',))

    mutant_response = self._uri_opener.send_mutant(mutant, debugging_id=debugging_id)

    if not self._is_resp_equal(orig_response, mutant_response):
        return True

    return False
```

在请求的 querysting 和 post_data 的位置检查常框架的 CSRF token， 还对 CSRF Token 的值做了一些限制（长度、香农信息熵等）。


```python
COMMON_CSRF_NAMES = (
    'csrf_token',
    'CSRFName',                   # OWASP CSRF_Guard
    'CSRFToken',                  # OWASP CSRF_Guard
    'anticsrf',                   # AntiCsrfParam.java
    '__RequestVerificationToken', # AntiCsrfParam.java
    'token',
    'csrf',
    'YII_CSRF_TOKEN',             # http://www.yiiframework.com/
    'yii_anticsrf'                # http://www.yiiframework.com/
    '[_token]',                   # Symfony 2.x
    '_csrf_token',                # Symfony 1.4
    'csrfmiddlewaretoken',        # Django 1.5
)

def _find_csrf_token(self, freq):
    """
    :return: A tuple with the first identified csrf token and value
    """
    post_data = freq.get_raw_data()
    querystring = freq.get_querystring()

    for token in chain(post_data.iter_tokens(), querystring.iter_tokens()):

        if self.is_csrf_token(token.get_name(), token.get_value()):

            msg = 'Found CSRF token %s in parameter %s for URL %s.'
            om.out.debug(msg % (token.get_value(),
                                token.get_name(),
                                freq.get_url()))

            return token.get_name(), token.get_value()
```