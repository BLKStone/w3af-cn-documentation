# w3af 检测规则 - blind_sqli

## 说明

w3af 的 blind_sqli 检测插件位于 [`w3af/w3af/plugins/audit/blind_sqli.py`](https://github.com/andresriancho/w3af/blob/master/w3af/plugins/audit/blind_sqli.py)。

入口函数

```python
def audit(self, freq, orig_response, debugging_id):
    """
    Tests an URL for blind SQL injection vulnerabilities.
    :param freq: A FuzzableRequest
    :param orig_response: The HTTP response associated with the fuzzable request
    :param debugging_id: A unique identifier for this call to audit()
    """
    #
    #    Blind SQL injection response diff
    #
    bsqli_resp_diff = BlindSqliResponseDiff(self._uri_opener)
    bsqli_resp_diff.set_eq_limit(self._eq_limit)
    bsqli_resp_diff.set_debugging_id(debugging_id)

    test_iterator = self._generate_response_diff_tests(freq, bsqli_resp_diff)

    self._send_mutants_in_threads(func=self._find_response_diff_sql,
                                  iterable=test_iterator,
                                  callback=lambda x, y: None)

    #
    #    Blind SQL injection time delays
    #
    bsqli_time_delay = BlindSQLTimeDelay(self._uri_opener)
    bsqli_time_delay.set_debugging_id(debugging_id)

    test_iterator = self._generate_delay_tests(freq, bsqli_time_delay)

    self._send_mutants_in_threads(func=self._find_time_delay_sql,
                                  iterable=test_iterator,
                                  callback=lambda x, y: None)
```

w3af 的盲注有两种检测方式，一种是基于 HTTP 响应差别 的 SQL 盲注检测； 另一种是 基于 HTTP 响应延时的 SQL 盲注检测。

## 基于 HTTP 响应差别 的 SQL 盲注检测

Blind SQL injection response diff

payload 生成函数

```python
def _get_statements(self, mutant, exclude_numbers=None):
    """
    Returns a list of statement tuples.
    """
    res = {}
    exclude_numbers = exclude_numbers or []

    rnd_num = int(rand_number(2, exclude_numbers))
    rnd_num_plus_one = rnd_num + 1

    num_dict = {'num': rnd_num}

    # Numeric/Datetime
    true_stm = '%(num)s OR %(num)s=%(num)s OR %(num)s=%(num)s ' % num_dict
    false_stm = '%i AND %i=%i ' % (rnd_num, rnd_num, rnd_num_plus_one)
    res[self.NUMERIC] = (true_stm, false_stm)

    # Single quotes
    true_stm = "%(num)s' OR '%(num)s'='%(num)s' OR '%(num)s'='%(num)s" % num_dict
    false_stm = "%i' AND '%i'='%i" % (rnd_num, rnd_num, rnd_num_plus_one)
    res[self.STRING_SINGLE] = (true_stm, false_stm)

    # Double quotes
    true_stm = '%(num)s" OR "%(num)s"="%(num)s" OR "%(num)s"="%(num)s' % num_dict
    false_stm = '%i" AND "%i"="%i' % (rnd_num, rnd_num, rnd_num_plus_one)
    res[self.STRING_DOUBLE] = (true_stm, false_stm)

    return res
```

使用的 payload 大致如下

```python
一般会随机一个 2位数字
rnd_num = 19
rnd_num_plus_one = 20
num_dict = {'num': rnd_num}

# 数字型盲注
# Numeric/Datetime
true_stm = '%(num)s OR %(num)s=%(num)s OR %(num)s=%(num)s ' % num_dict
false_stm = '%i AND %i=%i ' % (rnd_num, rnd_num, rnd_num_plus_one)
res[self.NUMERIC] = (true_stm, false_stm)

真条件语句
19 OR 19=19 OR 19=19
假条件语句
19 AND 19=20

# 单引号字符串盲注
# Single quotes
true_stm = "%(num)s' OR '%(num)s'='%(num)s' OR '%(num)s'='%(num)s" % num_dict
false_stm = "%i' AND '%i'='%i" % (rnd_num, rnd_num, rnd_num_plus_one)
res[self.STRING_SINGLE] = (true_stm, false_stm)

真条件语句
19' OR '19'='19' OR '19'='19
假条件语句
19' AND '19'='20

# 双引号字符串盲注
# # Double quotes
true_stm = '%(num)s" OR "%(num)s"="%(num)s" OR "%(num)s"="%(num)s' % num_dict
false_stm = '%i" AND "%i"="%i' % (rnd_num, rnd_num, rnd_num_plus_one)
res[self.STRING_DOUBLE] = (true_stm, false_stm)

真条件语句
19" OR "19"="19" OR "19"="19
假条件语句
19' AND '19'='20

```


基于 HTTP 响应差别的盲注检测，基本思路如下，在需要 fuzz 的目标参数中提供两种情况。其中一种情况迫使数据库 DMBS 返回一个条件为真的结果，另一种情况迫使数据库DBMS 返回一个条件为假的结果。然后比较在两种不同情况下，目标服务器的 HTTP 响应是否有区别。


```python
true_statement = statement_tuple[0]
false_statement = statement_tuple[1]
send_clean = self._uri_opener.send_clean
debugging_id = self.get_debugging_id()

mutant.set_token_value(true_statement)
true_response, body_true_response = send_clean(mutant,
                                               debugging_id=debugging_id,
                                               grep=True)

mutant.set_token_value(false_statement)
false_response, body_false_response = send_clean(mutant,
                                                 debugging_id=debugging_id,
                                                 grep=False)

if body_true_response == body_false_response:
    msg = ('There is NO CHANGE between the true and false responses.'
           ' NO WAY w3af is going to detect a blind SQL injection'
           ' using response diffs in this case.')
    self.debug(msg, mutant=mutant)
    return None
```

如果 真条件探测语句 与 假条件探测语句 的 HTTP 响应一致，就可以认为不存在。

在某些条件下，由于个性化推荐内容/动态广告的等原因，本质上 "真条件探测语句的 HTTP 响应" 与 "假条件探测语句的 HTTP 响应" 是 "一致" 的，但是两个响应的字符串并不完全相等。

为了检测这种"一致"， `w3af` 引入了另一个检测 `equal_with_limit` 。 大致是比较两个 "响应字符串"，比较他们的[相对距离(relative_distance)](https://github.com/andresriancho/w3af/blob/master/w3af/core/controllers/misc/fuzzy_string_cmp.py#L154-L170)是否大于阈值(0.6)，如果相对距离大于阈值，即认为这两个 HTTP 响应是一致的。



```python
compare_diff = False

msg = 'Comparing body_true_response and body_false_response.'
self.debug(msg,
           statement_type=statement_type,
           mutant=mutant,
           response_1=true_response,
           response_2=false_response)

if self.equal_with_limit(body_true_response,
                         body_false_response,
                         compare_diff):
    #
    # They might be equal because of various reasons, in the best
    # case scenario there IS a blind SQL injection but the % of the
    # HTTP response body controlled by it is so small that the equal
    # ratio is not catching it.
    #
    self.debug('Setting compare_diff to True', mutant=mutant)
    compare_diff = True
```

之后 `w3af` 也会比较 "真条件探测语句的 HTTP 响应" 与 "语法错误探测语句的 HTTP 响应" (`a'b"c'd"`) 是否 "一致" ，如果两者是一致的就会认为不存在盲注。

此外为了减少误报， `w3af` 还会尝试试探这个注入点是否是一个搜索引擎。

```python
# Check if its a search engine before we dig any deeper...
search_disambiguator = self._remove_all_special_chars(true_statement)
mutant.set_token_value(search_disambiguator)
search_response, body_search_response = send_clean(mutant,
                                                   grep=False,
                                                   debugging_id=debugging_id)

# If they are equal then we have a search engine
msg = 'Comparing body_true_response and body_search_response.'
self.debug(msg,
           statement_type=statement_type,
           mutant=mutant,
           response_1=true_response,
           response_2=search_response)

if self.equal_with_limit(body_true_response,
                         body_search_response,
                         compare_diff):
    return None

# Now a nice trick from real-life. In some search engines when
# searching for `46" OR "46"="46" OR "46"="46` we get only a
# couple of results, which I assume is because the search
# engine is trying to search for more terms.
#
# Removing the special characters will make w3af search for
# `46  OR  46   46  OR  46   46`, which yields many results in
# the application's search engine, which I assume is because the
# search engine just needs to match objects with 46 / OR.
#
# So, this means that the responses ARE different, but they came
# from a search engine. The check above is NOT going to catch that
# and will yield a false positive.
#
# If this is not a search engine, or is a search engine with a blind
# sql injection, the result with `46" OR "46"="46" OR "46"="46` should
# be have a larger HTTP response body: "all results" should be there.
#
# If it is a search engine, then the result for the search string
# without special characters will be larger.
if len(body_search_response) * 0.8 > len(body_true_response):
    msg = 'Search engine detected using response length, stop.'
    self.debug(msg,
               statement_type=statement_type,
               mutant=mutant,
               response_1=true_response,
               response_2=search_response)
    return None
```


对 真/假条件语句 重发再次验证，减少误报

```python
# Verify the injection!
statements = self._get_statements(mutant)
second_true_stm = statements[statement_type][0]
second_false_stm = statements[statement_type][1]

mutant.set_token_value(second_true_stm)
second_true_response, body_second_true_response = send_clean(mutant,
                                                             grep=False,
                                                             debugging_id=debugging_id)

mutant.set_token_value(second_false_stm)
second_false_response, body_second_false_response = send_clean(mutant,
                                                               grep=False,
                                                               debugging_id=debugging_id)

msg = 'Comparing body_second_true_response and body_true_response.'
self.debug(msg,
           statement_type=statement_type,
           mutant=mutant,
           response_1=true_response,
           response_2=second_true_response)

if not self.equal_with_limit(body_second_true_response,
                             body_true_response,
                             compare_diff):
    return None

msg = 'Comparing body_second_false_response and body_false_response.'
self.debug(msg,
           statement_type=statement_type,
           mutant=mutant,
           response_1=false_response,
           response_2=second_false_response)

if not self.equal_with_limit(body_second_false_response,
                             body_false_response,
                             compare_diff):
    return None
```

## 基于 HTTP 响应延时的 SQL 盲注检测

Blind SQL injection time delays

payload 生成函数

```python
def get_delays(self):
    """
    :return: A list of statements that are going to be used to test for
             blind SQL injections. The statements are objects.

             IMPORTANT: Note that I need this function that generates
             unique instances of the delay objects! Adding this to a list
             that's defined at the class level will bring threading issues
    """
    return self.DELAYS
```

具体 payload

```python
DELAYS = [
    # MSSQL
    ExactDelay("1;waitfor delay '0:0:%s'--"),
    ExactDelay("1);waitfor delay '0:0:%s'--"),
    ExactDelay("1));waitfor delay '0:0:%s'--"),
    ExactDelay("1';waitfor delay '0:0:%s'--"),
    ExactDelay("1');waitfor delay '0:0:%s'--"),
    ExactDelay("1'));waitfor delay '0:0:%s'--"),

    # MySQL 5
    #
    # Note: These payloads are better than "1 or SLEEP(%s)" since they
    #       do not call SLEEP for each row in the table
    #
    # Payloads are heavily based on the ones from SQLMap which can be found
    # at xml/payloads/05_time_blind.xml
    #
    ExactDelay("1 AND (SELECT * FROM (SELECT(SLEEP(%s)))foo)"),
    ExactDelay("1 OR (SELECT * FROM (SELECT(SLEEP(%s)))foo)"),

    # Single and double quote string concat
    ExactDelay("'+(SELECT * FROM (SELECT(SLEEP(%s)))foo)+'"),
    ExactDelay('"+(SELECT * FROM (SELECT(SLEEP(%s)))foo)+"'),

    # These are required, they don't cover the same case than the previous
    # ones (string concat).
    ExactDelay("' AND (SELECT * FROM (SELECT(SLEEP(%s)))foo) AND '1'='1"),
    ExactDelay('" AND (SELECT * FROM (SELECT(SLEEP(%s)))foo) AND "1"="1'),
    ExactDelay("' OR (SELECT * FROM (SELECT(SLEEP(%s)))foo) OR '1'='2"),
    ExactDelay('" OR (SELECT * FROM (SELECT(SLEEP(%s)))foo) OR "1"="2'),

    # MySQL 4
    #
    # MySQL 4 doesn't have a sleep function, so I have to use
    # BENCHMARK(1000000000,MD5(1)) but the benchmarking will delay the
    # response a different amount of time in each computer which sucks
    # because I use the time delay to check!
    #
    # In my test environment 3500000 delays 10 seconds
    # This is why I selected 2500000 which is guaranteed to (at least) delay
    # 8 seconds; and I only check the delay like this:
    #
    #    response.get_wait_time() > (original_wait_time + self._wait_time-2)
    #
    # With a small wait time of 5 seconds, this should work without
    # problems... and without hitting the ExtendedUrllib timeout !
    #
    # TODO: Need to implement variable_delay.py (modification of ExactDelay)
    #       and use the following there:
    #
    # ExactDelay("1 or BENCHMARK(2500000,MD5(1))") )
    # ExactDelay("1' or BENCHMARK(2500000,MD5(1)) or '1'='1") )
    # ExactDelay('1" or BENCHMARK(2500000,MD5(1)) or "1"="1') )

    # PostgreSQL
    ExactDelay("1 or pg_sleep(%s)"),
    ExactDelay("1' or pg_sleep(%s) and '1'='1"),
    ExactDelay('1" or pg_sleep(%s) and "1"="1'),

    # TODO: Add Oracle support
    # TODO: Add XXXXX support
    # TODO: https://github.com/andresriancho/w3af/issues/12385
]
```

关键检测逻辑在[exact_delay_controller.py](https://github.com/andresriancho/w3af/blob/master/w3af/core/controllers/delay_detection/exact_delay_controller.py#L76-L149)

```python
def delay_is_controlled(self):
    """
    All the logic/magic is in this method. The logic is very simple:
        * Try to delay the response in 4 seconds, if it works
        * Try to delay the response in 1 seconds, if it works
        * Try to delay the response in 6 seconds, if it works
          (note that these delays are actually determined by DELAY_SECONDS)
        * Then we have found a vulnerability!
    We go up and down and change the amount of seconds in order to make sure
    that WE are controlling the delay and there is no other external factor
    in place.
    """
    DELAY_SECONDS = [8, 4, 9, 5, 14]
```

就如同注释所描述的尝试各种不同时间的延迟，如果都成功了，说明确实存在盲注。

```python
original_wait_time
原始请求的等待时间
delay 是 payload 中等待的时间

# Upper bound is the highest number we'll wait for a response, it
# doesn't mean that it is the highest delay that might happen on
# the application.
#
# # So, for example if the application logic (for some reason) runs
# our payload three times, and we send:
#
#   sleep(10)
#
# The delay will be of 30 seconds, but we don't want to wait all
# that time (using high timeouts for HTTP requests is *very* bad when
# scanning slow apps).
#
# We just wait until `upper_bound` is reached
delta = original_wait_time * 0.25
upper_bound = original_wait_time + delta + delay * 2

# The lower_bound is the lowest number of seconds we require for this
# HTTP response to be considered "delayed".
#
# I tried with different variations of this, the first one included
# original_wait_time and delta, but that failed in this scenario:
#
#   * RTT (which defines original_wait_time) is inaccurately high: 3 seconds
#     instead of 1 second which was the expected result. This could be
#     because of outliers in the measurement
#
#           https://github.com/andresriancho/w3af/issues/16902
#
#   * lower_bound is then set to original_wait_time - delta + delay
#
#   * The payload is sent and the response is delayed for 4 seconds
#
#   * The delay_for method yields false because of the bad RTT
lower_bound = delay

```

对于某个特定延时的检测， `w3af` 设定了一个上下边界 `upper_bound` 和 `lower_bound`。 设置 `upper_bound` 的目的主要在于不让扫描器等待过长的时间.举个例子，假如我们发送的 payload 是类似 sleep(10) 的语句，但是 web 应用的业务逻辑可能执行了3 遍这个过程 (虽然概率比较低，但是确实可能存在类似的场景)，最终导致的延时结果是延迟 30 秒。让扫描器为了检测这个 URL 等待 30 秒是不划算的，因此只要时间到达了 `upper_bound`， HTTP 直接响应超时，此时直接认为注入存在。



