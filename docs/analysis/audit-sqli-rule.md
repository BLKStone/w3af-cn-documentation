# w3af 检测规则 - sqli

## 说明

w3af 的 sqli 检测插件位于 [`w3af/w3af/plugins/audit/sqli.py`](https://github.com/andresriancho/w3af/blob/master/w3af/plugins/audit/sqli.py) 。

审计插件的入口函数一般都是 audit 函数， `freq` 是 FuzzableRequest，一个可以 fuzz 的请求， `orig_response` 是原始响应， `debugging_id` 是调试编号。

```python
def audit(self, freq, orig_response, debugging_id):
    """
    Tests an URL for SQL injection vulnerabilities.
    :param freq: A FuzzableRequest
    :param orig_response: The HTTP response associated with the fuzzable request
    :param debugging_id: A unique identifier for this call to audit()
    """
    mutants = create_mutants(freq, self.SQLI_STRINGS, orig_resp=orig_response)

    self._send_mutants_in_threads(self._uri_opener.send_mutant,
                                  mutants,
                                  self._analyze_result,
                                  debugging_id=debugging_id)
```

SQLI_STRINGS 是用于测试的 payload 。

```python
SQLI_STRINGS = (u"a'b\"c'd\"",
                u"1'2\"3")
```

通过 create_mutants 变异请求构造函数，扫描器会尝试在 URL 参数变量 (QSMutant) ， POST 请求 Data 变量 (PostDataMutant) ， 上传文件名称变量 (FileNameMutant)， URL路径参数变量 (URLPartsMutant) ， HTTP 请求头 (HeadersMutant)，JSON 数据(JSONMutant)，Cookie 数据(CookieMutant)，文件内容数据(FileContentMutant)，XMLRPC(XmlRpcMutant) 等位置插入 payload 。

之后再响应信息中检测 SQL 执行错误的信息。

```python
SQL_ERRORS_STR = (
    # ASP / MSSQL
    (r'System.Data.OleDb.OleDbException', dbms.MSSQL),
    (r'[SQL Server]', dbms.MSSQL),
    (r'[Microsoft][ODBC SQL Server Driver]', dbms.MSSQL),
    (r'[SQLServer JDBC Driver]', dbms.MSSQL),
    (r'[SqlException', dbms.MSSQL),
    (r'System.Data.SqlClient.SqlException', dbms.MSSQL),
    (r'Unclosed quotation mark after the character string', dbms.MSSQL),
    (r"'80040e14'", dbms.MSSQL),
    (r'mssql_query()', dbms.MSSQL),
    (r'odbc_exec()', dbms.MSSQL),
    (r'Microsoft OLE DB Provider for ODBC Drivers', dbms.MSSQL),
    (r'Microsoft OLE DB Provider for SQL Server', dbms.MSSQL),
    (r'Incorrect syntax near', dbms.MSSQL),
    (r'Sintaxis incorrecta cerca de', dbms.MSSQL),
    (r'Syntax error in string in query expression', dbms.MSSQL),
    (r'ADODB.Field (0x800A0BCD)<br>', dbms.MSSQL),
    (r"ADODB.Recordset'", dbms.MSSQL),
    (r"Unclosed quotation mark before the character string", dbms.MSSQL),
    (r"'80040e07'", dbms.MSSQL),
    (r'Microsoft SQL Native Client error', dbms.MSSQL),
    (r'SQL Server Native Client', dbms.MSSQL),
    (r'Invalid SQL statement', dbms.MSSQL),

    # DB2
    (r'SQLCODE', dbms.DB2),
    (r'DB2 SQL error:', dbms.DB2),
    (r'SQLSTATE', dbms.DB2),
    (r'[CLI Driver]', dbms.DB2),
    (r'[DB2/6000]', dbms.DB2),

    # Sybase
    (r"Sybase message:", dbms.SYBASE),
    (r"Sybase Driver", dbms.SYBASE),
    (r"[SYBASE]", dbms.SYBASE),

    # Access
    (r'Syntax error in query expression', dbms.ACCESS),
    (r'Data type mismatch in criteria expression.', dbms.ACCESS),
    (r'Microsoft JET Database Engine', dbms.ACCESS),
    (r'[Microsoft][ODBC Microsoft Access Driver]', dbms.ACCESS),

    # ORACLE
    (r'Microsoft OLE DB Provider for Oracle', dbms.ORACLE),
    (r'wrong number or types', dbms.ORACLE),

    # POSTGRE
    (r'PostgreSQL query failed:', dbms.POSTGRE),
    (r'supplied argument is not a valid PostgreSQL result', dbms.POSTGRE),
    (r'unterminated quoted string at or near', dbms.POSTGRE),
    (r'pg_query() [:', dbms.POSTGRE),
    (r'pg_exec() [:', dbms.POSTGRE),

    # MYSQL
    (r'supplied argument is not a valid MySQL', dbms.MYSQL),
    (r'Column count doesn\'t match value count at row', dbms.MYSQL),
    (r'mysql_fetch_array()', dbms.MYSQL),
    (r'mysql_', dbms.MYSQL),
    (r'on MySQL result index', dbms.MYSQL),
    (r'You have an error in your SQL syntax;', dbms.MYSQL),
    (r'You have an error in your SQL syntax near', dbms.MYSQL),
    (r'MySQL server version for the right syntax to use', dbms.MYSQL),
    (r'Division by zero in', dbms.MYSQL),
    (r'not a valid MySQL result', dbms.MYSQL),
    (r'[MySQL][ODBC', dbms.MYSQL),
    (r"Column count doesn't match", dbms.MYSQL),
    (r"the used select statements have different number of columns",
        dbms.MYSQL),
    (r"DBD::mysql::st execute failed", dbms.MYSQL),
    (r"DBD::mysql::db do failed:", dbms.MYSQL),

    # Informix
    (r'com.informix.jdbc', dbms.INFORMIX),
    (r'Dynamic Page Generation Error:', dbms.INFORMIX),
    (r'An illegal character has been found in the statement',
        dbms.INFORMIX),
    (r'[Informix]', dbms.INFORMIX),
    (r'<b>Warning</b>:  ibase_', dbms.INTERBASE),
    (r'Dynamic SQL Error', dbms.INTERBASE),

    # DML
    (r'[DM_QUERY_E_SYNTAX]', dbms.DMLDATABASE),
    (r'has occurred in the vicinity of:', dbms.DMLDATABASE),
    (r'A Parser Error (syntax error)', dbms.DMLDATABASE),

    # Java
    (r'java.sql.SQLException', dbms.JAVA),
    (r'Unexpected end of command in statement', dbms.JAVA),

    # Coldfusion
    (r'[Macromedia][SQLServer JDBC Driver]', dbms.MSSQL),

    # SQLite
    (r'could not prepare statement', dbms.SQLITE),

    # Generic errors..
    (r'Unknown column', dbms.UNKNOWN),
    (r'where clause', dbms.UNKNOWN),
    (r'SqlServer', dbms.UNKNOWN),
    (r'syntax error', dbms.UNKNOWN),
    (r'Microsoft OLE DB Provider', dbms.UNKNOWN),
)
```

如果原始响应中不包含 SQL 错误信息，而变异响应中包含 SQL 错误信息，则认为此处存在 SQL 注入漏洞。