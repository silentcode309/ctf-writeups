# Directory Traversal Challenge – Juice Shop

Category: Directory Traversal  
Difficulty: Easy  
Platform: OWASP Juice Shop  
Date: 11-2025

## Summary
This challenge demonstrates a directory traversal vulnerability in the file-serving functionality of OWASP Juice Shop. The application exposes an endpoint that allows users to request files by name, but it fails to properly sanitize the file path. As a result, attackers can use `../` sequences to escape the intended directory and read arbitrary files on the server.

## Recon
Initial exploration identified a public file browser located at:

/ftp

The page loads files dynamically using a parameter, typically in the form:

/ftp?file=<filename>

This hinted at potential file‑path manipulation.

Basic tests were performed:

- Submitting a valid file → returned normally  
- Replacing the filename with traversal patterns such as `../` and `../../`  
- Observing server responses for clues (errors, partial matches, etc.)

## Vulnerability Discovery
A crafted payload successfully escaped the intended directory:

/ftp?file=../../../../../../etc/passwd

The server returned the full contents of `/etc/passwd`, confirming directory traversal. Additional tests with other system files also succeeded:

/ftp?file=../../../../../../../etc/hostname
/ftp?file=../../../../../../../proc/self/environ

Each returned evaluated file contents, demonstrating unrestricted file read capability.

## Exploitation
Once traversal was confirmed, multiple sensitive files were accessed to validate the extent of exposure. While no destructive or malicious actions were performed, these tests illustrated significant impact:

- Reading system user information  
- Reading environment variables  
- Reading other configuration files depending on permissions  

This proves an attacker could potentially escalate exposure by locating secrets, tokens, or system data.

## Root Cause
The backend endpoint concatenates untrusted user input directly into a filesystem path without:

- Normalizing input  
- Restricting allowed patterns  
- Sanitizing traversal sequences  
- Using safe library methods (e.g., path resolution with explicit root‑directory enforcement)

As a result, any crafted `../` sequence escapes the intended directory.

## Mitigation
To prevent directory traversal attacks:

- Sanitize and normalize all file paths before accessing them  
- Strip or block traversal sequences (`../`, `..\`, `%2e%2e/`, etc.)  
- Implement allowlists (only serve files from a predefined set)  
- Use safe path resolution libraries that enforce a strict root directory  
- Disable direct filesystem access from user-controlled parameters

## Tools
- Browser developer tools  
- Burp Suite Community Edition  
- OWASP Juice Shop (Docker instance)
