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

https://matthewfl.com/unPacker.html 

**Encoding and decoding Base64**

`echo Yash | base64` # this command can be used for encoding in base64

`echo WWFzaA== | base64 -d ` # this command can be used for decoding base64 string

**Encoding and decoding HEX**

`echo Yash | xxd -p` # For encoding the data into HEX

`echo 59617368 | xxd -p -r` # For Decoding the HEX string

**Caesar/Rot13 Encoding and Decoding**

`echo https://www.hackthebox.eu/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'` # For encoding

`echo uggcf://jjj.unpxgurobk.rh/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'` # For Decoding

