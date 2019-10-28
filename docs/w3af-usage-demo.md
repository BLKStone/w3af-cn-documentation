# 使用 w3af 扫描测试站

测试目标站:
`http://testphp.vulnweb.com/`

## 主动扫描模式

启动 w3af 并进行配置

```bash
root@ubuntu-yellowradio:/home/team/devlop/w3af/extras/docker/scripts# ./w3af_console_docker
w3af>>>
w3af>>> profiles
w3af/profiles>>> use fast_scan
The plugins configured by the scan profile have been enabled, and their options configured.
Please set the target URL(s) and start the scan.
w3af/plugins>>> output text_file
w3af/plugins>>> output html_file
w3af/plugins>>> output
|-----------------------------------------------------------------------------------|
| Plugin name     | Status  | Conf | Description                                    |
|-----------------------------------------------------------------------------------|
| console         | Enabled | Yes  | Print messages to the console.                 |
| csv_file        |         | Yes  | Export identified vulnerabilities to a CSV     |
|                 |         |      | file.                                          |
| email_report    |         | Yes  | Email report to specified addresses.           |
| export_requests |         | Yes  | Export the fuzzable requests found during      |
|                 |         |      | crawl to a file.                               |
| html_file       | Enabled | Yes  | Generate HTML report with identified           |
|                 |         |      | vulnerabilities and log messages.              |
| text_file       | Enabled | Yes  | Prints all messages to a text file.            |
| xml_file        |         | Yes  | Print all messages to a xml file.              |
|-----------------------------------------------------------------------------------|
w3af/plugins>>> output config text_file
w3af/plugins>>> set output_file output-w3af.txt
w3af/plugins/output/config:text_file>>> set output_file /root/w3af-shared/output.txt
w3af/plugins/output/config:text_file>>> set http_output_file /root/w3af-shared/output-http.txt
w3af/plugins/output/config:text_file>>> set verbose True
w3af/plugins/output/config:text_file>>> view
|----------------------------------------------------------------------------------|
| Setting          | Value                             | Modified | Description    |
|----------------------------------------------------------------------------------|
| output_file      | /root/w3af-shared/output.txt      | Yes      | File name      |
|                  |                                   |          | where this     |
|                  |                                   |          | plugin will    |
|                  |                                   |          | write to       |
| verbose          | True                              | Yes      | Enable if      |
|                  |                                   |          | verbose output |
|                  |                                   |          | is needed      |
| http_output_file | /root/w3af-shared/output-http.txt | Yes      | File name      |
|                  |                                   |          | where this     |
|                  |                                   |          | plugin will    |
|                  |                                   |          | write HTTP     |
|                  |                                   |          | requests and   |
|                  |                                   |          | responses      |
|----------------------------------------------------------------------------------|
w3af/plugins/output/config:text_file>>> save
The configuration has been saved.
w3af/plugins/output/config:text_file>>> back
The configuration has been saved.
w3af/plugins>>> output config html_file
w3af/plugins/output/config:html_file>>> set output_file /root/w3af-shared/report.html
w3af/plugins/output/config:html_file>>> set verbose True
w3af/plugins/output/config:html_file>>> view
|----------------------------------------------------------------------------------|
| Setting | Value                                               | Modified | Description |
|----------------------------------------------------------------------------------|
| output_file | /root/w3af-shared/report.html                       |      | File name   |
|       |                                                     |      | where this  |
|       |                                                     |      | plugin will |
|       |                                                     |      | write to    |
| verbose | True                                                |      | True if     |
|       |                                                     |      | debug       |
|       |                                                     |      | information |
|       |                                                     |      | will be     |
|       |                                                     |      | appended to |
|       |                                                     |      | the report. |
| template | /home/w3af/w3af/w3af/plugins/output/html_file/templates/complete.html |      | The path to |
|       |                                                     |      | the HTML    |
|       |                                                     |      | template    |
|       |                                                     |      | used to     |
|       |                                                     |      | render the  |
|       |                                                     |      | report.     |
|----------------------------------------------------------------------------------|
w3af/plugins/output/config:html_file>>> save
The configuration has been saved.
w3af/plugins/output/config:html_file>>> back
The configuration has been saved.
w3af/plugins>>> back
w3af>>> target
w3af/config:target>>> view
|----------------------------------------------------------------------------------|
| Setting   | Value           | Modified | Description                                 |
|----------------------------------------------------------------------------------|
| target_framework | unknown         |      | Target programming framework                |
|           |                 |      | (unknown/php/asp/asp.net/java/jsp/cfm/ruby/perl) |
| target    | http://testphp.vulnweb.com/ | Yes  | A comma separated list of URLs              |
| target_os | unknown         |      | Target operating system                     |
|           |                 |      | (unknown/unix/windows)                      |
|----------------------------------------------------------------------------------|
w3af/config:target>>> save
The configuration has been saved.
w3af/config:target>>> back
The configuration has been saved.
w3af>>> start
```

配置文件详情 `blkstone_fast_scan.pw3af`:

```ini
[profile]
description = Profile generated using the console UI.
name = blkstone_fast_scan

[crawl.web_spider]
only_forward = False
follow_regex = .*
ignore_regex =

[audit.xss]
persistent_xss = True

[audit.sqli]

[output.html_file]
template = %ROOT_PATH%/plugins/output/html_file/templates/complete.html
output_file = /root/w3af-shared/report.html
verbose = True

[output.text_file]
verbose = True
output_file = /root/w3af-shared/output.txt
http_output_file = /root/w3af-shared/output-http.txt

[output.console]
verbose = False

[grep.symfony]
override = False

[grep.file_upload]

[grep.wsdl_greper]

[grep.form_autocomplete]

[grep.strange_parameters]

[grep.svn_users]

[grep.private_ip]

[grep.motw]

[grep.code_disclosure]

[grep.blank_body]

[grep.path_disclosure]

[grep.strange_http_codes]

[grep.http_auth_detect]

[grep.credit_cards]

[grep.dom_xss]

[grep.html_comments]

[grep.http_in_body]

[grep.dot_net_event_validation]

[grep.ssn]

[grep.error_500]

[grep.meta_tags]

[grep.password_profiling]

[grep.click_jacking]

[grep.directory_indexing]

[grep.lang]

[grep.get_emails]
only_target_domain = True

[grep.hash_analysis]

[grep.error_pages]

[grep.strange_reason]

[grep.user_defined_regex]
single_regex =
regex_file_path = %ROOT_PATH%/plugins/grep/user_defined_regex/empty.txt

[grep.strange_headers]

[grep.objects]

[grep.oracle]

[grep.feeds]

[grep.analyze_cookies]

[misc-settings]
fuzz_cookies = False
fuzz_form_files = True
fuzz_url_filenames = False
fuzz_url_parts = False
fuzzed_files_extension = gif
fuzzable_headers =
form_fuzzing_mode = tmb
stop_on_first_exception = False
max_discovery_time = 120
interface = ppp0
local_ip_address = 172.17.0.2
non_targets =
msf_location = /opt/metasploit3/bin/

[http-settings]
timeout = 0
headers_file =
basic_auth_user =
basic_auth_passwd =
basic_auth_domain =
ntlm_auth_domain =
ntlm_auth_user =
ntlm_auth_passwd =
ntlm_auth_url =
cookie_jar_file =
ignore_session_cookies = False
proxy_port = 8080
proxy_address =
user_agent = w3af.org
rand_user_agent = False
max_file_size = 400000
max_http_retries = 2
max_requests_per_second = 0
always_404 =
never_404 =
string_match_404 =
url_parameter =


```

## 被动扫描模式


根据文档 [Docker 中的 w3af](http://docs.w3af.org/en/latest/docker.html#ports-and-services) 中的描述。
可以参考如下 [commit](https://github.com/andresriancho/w3af/commit/a8e2f66e31d8ad4a769cd0e7c12c87559dd026f3)

w3af console 配置
禁用 `web_spider` 插件，仅启用 `spider_man` 插件

```bash
w3af/plugins>>> crawl !web_spider
w3af/plugins>>> crawl spider_man
w3af/plugins>>> crawl
|----------------------------------------------------------------------------------------------------------|
| Plugin name                  | Status  | Conf | Description                                              |
|----------------------------------------------------------------------------------------------------------|
| archive_dot_org              |         | Yes  | Search archive.org to find new pages in the target site. |
| bing_spider                  |         | Yes  | Search Bing to get a list of new URLs                    |
| content_negotiation          |         | Yes  | Use content negotiation to find new resources.           |
| digit_sum                    |         | Yes  | Take an URL with a number (index2.asp) and try to find   |
|                              |         |      | related files(index1.asp, index3.asp).                   |
| dir_file_bruter              |         | Yes  | Finds Web server directories and files by bruteforcing.  |
| dot_listing                  |         |      | Search for .listing files and extracts new filenames     |
|                              |         |      | from it.                                                 |
| find_backdoors               |         |      | Find web backdoors and web shells.                       |
| find_captchas                |         |      | Identify captcha images on web pages.                    |
| find_dvcs                    |         |      | Search Git, Mercurial (HG), Bazaar (BZR), Subversion     |
|                              |         |      | (SVN) and CVSrepositories and checks for files           |
|                              |         |      | containing                                               |
| genexus_xml                  |         |      | Analyze the execute.xml and DeveloperMenu.xml files and  |
|                              |         |      | find new URLs                                            |
| ghdb                         |         | Yes  | Search Google for vulnerabilities in the target site.    |
| google_spider                |         | Yes  | Search google using google API to get new URLs           |
| import_results               |         | Yes  | Import HTTP requests found by output.export_requests and |
|                              |         |      | Burp                                                     |
| oracle_discovery             |         |      | Find Oracle applications on the remote web server.       |
| phishtank                    |         |      | Search the phishtank.com database to determine if your   |
|                              |         |      | server is (or was)being used in phishing scams.          |
| phpinfo                      |         |      | Search PHP Info file and if it finds it will determine   |
|                              |         |      | the version of PHP.                                      |
| pykto                        |         | Yes  | A nikto port to python.                                  |
| ria_enumerator               |         | Yes  | Fingerprint Rich Internet Apps - Google Gears Manifest   |
|                              |         |      | files, Silverlight and Flash.                            |
| robots_txt                   |         |      | Analyze the robots.txt file and find new URLs            |
| sitemap_xml                  |         |      | Analyze the sitemap.xml file and find new URLs           |
| spider_man                   | Enabled | Yes  | SpiderMan is a local proxy that will collect new URLs.   |
| url_fuzzer                   |         | Yes  | Try to find backups, and other related files.            |
| urllist_txt                  |         |      | Analyze the urllist.txt file and find new URLs           |
| user_dir                     |         |      | Identify user directories like "http://test/~user/" and  |
|                              |         |      | infer the remote OS.                                     |
| web_diff                     |         | Yes  | Compare a local directory with a remote URL path.        |
| web_spider                   |         | Yes  | Crawl the web application.                               |
| wordnet                      |         | Yes  | Use the wordnet lexical database to find new URLs.       |
| wordpress_enumerate_users    |         |      | Finds users in a WordPress installation.                 |
| wordpress_fingerprint        |         |      | Finds the version of a WordPress installation.           |
| wordpress_fullpathdisclosure |         |      | Try to find the path where the WordPress is installed    |
| wsdl_finder                  |         |      | Find web service definitions files.                      |
|----------------------------------------------------------------------------------------------------------|
w3af/plugins/crawl/config:spider_man>>> set listen_address 0.0.0.0
w3af/plugins/crawl/config:spider_man>>> view
|----------------------------------------------------------------------------------------------------------|
| Setting        | Value   | Modified | Description                                                        |
|----------------------------------------------------------------------------------------------------------|
| listen_address | 0.0.0.0 | Yes      | IP address that the spider_man proxy will use to receive requests  |
| listen_port    | 44444   |          | Port that the spider_man HTTP proxy server will use to receive     |
|                |         |          | HTTP requests                                                      |
|----------------------------------------------------------------------------------------------------------|
w3af/plugins/crawl/config:spider_man>>> back
The configuration has been saved.
w3af/plugins>>> back


```

停止代理 `curl -X GET http://127.7.7.7/spider_man?terminate --proxy http://127.0.0.1:44444`

被动扫描配置项 `blkstone_passive.pw3af`

```ini
[profile]
description = Profile generated using the console UI.
name = blkstone_passive

[crawl.spider_man]
listen_address = 0.0.0.0
listen_port = 44444

[audit.xss]
persistent_xss = True

[audit.sqli]

[output.html_file]
template = %ROOT_PATH%/plugins/output/html_file/templates/complete.html
output_file = /root/w3af-shared/report.html
verbose = True

[output.text_file]
verbose = True
output_file = /root/w3af-shared/output.txt
http_output_file = /root/w3af-shared/output-http.txt

[output.console]
verbose = False

[grep.symfony]
override = False

[grep.file_upload]

[grep.wsdl_greper]

[grep.form_autocomplete]

[grep.strange_parameters]

[grep.svn_users]

[grep.private_ip]

[grep.motw]

[grep.code_disclosure]

[grep.blank_body]

[grep.path_disclosure]

[grep.strange_http_codes]

[grep.credit_cards]

[grep.dom_xss]

[grep.html_comments]

[grep.http_auth_detect]

[grep.dot_net_event_validation]

[grep.ssn]

[grep.error_500]

[grep.hash_analysis]

[grep.password_profiling]

[grep.click_jacking]

[grep.directory_indexing]

[grep.lang]

[grep.get_emails]
only_target_domain = True

[grep.meta_tags]

[grep.feeds]

[grep.error_pages]

[grep.strange_reason]

[grep.user_defined_regex]
single_regex =
regex_file_path = %ROOT_PATH%/plugins/grep/user_defined_regex/empty.txt

[grep.strange_headers]

[grep.objects]

[grep.oracle]

[grep.http_in_body]

[grep.analyze_cookies]

[misc-settings]
fuzz_cookies = False
fuzz_form_files = True
fuzz_url_filenames = False
fuzz_url_parts = False
fuzzed_files_extension = gif
fuzzable_headers =
form_fuzzing_mode = tmb
stop_on_first_exception = False
max_discovery_time = 120
interface = ppp0
local_ip_address = 172.17.0.2
non_targets =
msf_location = /opt/metasploit3/bin/

[http-settings]
timeout = 0
headers_file =
basic_auth_user =
basic_auth_passwd =
basic_auth_domain =
ntlm_auth_domain =
ntlm_auth_user =
ntlm_auth_passwd =
ntlm_auth_url =
cookie_jar_file =
ignore_session_cookies = False
proxy_port = 8080
proxy_address =
user_agent = w3af.org
rand_user_agent = False
max_file_size = 400000
max_http_retries = 2
max_requests_per_second = 0
always_404 =
never_404 =
string_match_404 =
url_parameter =

```



