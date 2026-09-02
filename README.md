# Lab-2-web-application-and-security-testing

## Part 1 and Part 2 - Web Basics & Application Setup

### Step 1: Confirm the Application URL
- **Target URL:** `http://192.168.136.136/dvwa/`
- **Objective:** Access the authorized DVWA path on the target web server in Firefox to begin application-layer testing.
- **Result:** Successfully accessed the DVWA application login/home page on the isolated host-only network.

![Step 1 - DVWA Home Page](https://github.com/user-attachments/assets/790a7123-c727-40b6-a7f3-06359e2cde1c)

### Step 2: Set DVWA Security to Low
- **Objective:** Configure DVWA security controls to the lowest level to establish a baseline for predictable application testing.
- **Action:** Selected **DVWA Security** from the left navigation menu, set the dropdown value to **Low**, and submitted the form.
- **Result:** Confirmed that the application security level is active as `Low`.

![Step 2 - Security Level Set to Low](https://github.com/user-attachments/assets/242c604d-8e97-47e4-ad9f-a8c138909256)

### Step 3: Open Network Tools
- **Objective:** Observe standard HTTP request and response attributes in browser Developer Tools prior to using an intercepting proxy.
- **Action:** Opened Developer Tools (**F12**), selected the **Network** tab, and reloaded `http://192.168.136.136/dvwa/security.php`.
- **Recorded Request Details:**
  - **Method:** `GET`
  - **URL:** `http://192.168.136.136/dvwa/security.php`
  - **Status Code:** `200 OK`

![Step 3 - Network Tools GET Request](https://github.com/user-attachments/assets/4584ce89-4ebd-4ec2-b182-bc344e4116cd)

### Step 4: Identify Cookies
- **Objective:** Understand how session state and security parameters are tracked without exposing sensitive session token values.
- **Action:** Inspected the request headers for `security.php` in Firefox Developer Tools to locate the **Cookie** field.
- **Recorded Cookie Names:**
  - `PHPSESSID`
  - `security`
    
![Step 4 - Identified Cookie Names](https://github.com/user-attachments/assets/f2132a06-2e5d-4fc4-9485-9b7c435f745a)

## Part 4 - Burp Suite

### Step 5: Start Burp
- **Command:**
  ```bash
  burpsuite 
 ![burpsuite](https://github.com/user-attachments/assets/41ff5a58-2e47-463e-81bd-3300c7081d48)

 ### Step 6: Open Burp Browser
- **Objective:** Launch Burp's embedded browser to capture HTTP/HTTPS traffic automatically without manual proxy or CA certificate configuration.
- **Action:** Navigated to **Proxy** → **Intercept** → **Open Browser**.
- **Result:** Embedded Chromium browser opened successfully and is connected to the proxy.

![Step 6 - Burp Embedded Browser Launched](https://github.com/user-attachments/assets/fc44c296-f2ea-4148-91fb-e4f83ac0ea9e)

### Step 7: Intercept a Harmless Request
- **Objective:** Examine raw HTTP request components intercepted in transit by Burp Suite before they reach the web server.
- **Action:** Toggled **Intercept on**, navigated to `http://192.168.136.136/dvwa/`, inspected the captured request line and headers, then clicked **Forward**.
- **Recorded Request Details:**
  - **Method:** `GET`
  - **Path:** `/dvwa/login.php`
  - **Host Header:** `192.168.136.136`
  - **User-Agent:** `Mozilla/5.0 (X11; Linux x86_64; ...)`
  - **Cookie Header:** Contains `security` and `PHPSESSID` parameters (secret token values redacted).

![Step 7 - Intercepted GET Request in Burp](https://github.com/user-attachments/assets/c71ef1bf-0077-4e40-85fe-e41e954a67a4)

## Part 5 - DVWA File Upload

### Step 8: Create a Harmless File
- **Command:**
  ```bash
  echo 'ICDFA beginner upload test' > icdfa-upload-test.txt
  ls -l icdfa-upload-test.txt

![step 8](https://github.com/user-attachments/assets/608299da-0910-4630-9c39-a1e45ed4beab)

### Step 9: Open File Upload Module
- **Objective:** Locate and access the target File Upload feature in DVWA to prepare for baseline vulnerability testing.
- **Action:** Selected **Upload** from the left navigation sidebar menu inside the DVWA web application interface.
- **Observed Behavior:** The web application successfully rendered the `Vulnerability: File Upload` panel containing file selection (`Browse...`) and submission (`Upload`) controls.

![Step 9 - DVWA File Upload Interface](https://github.com/user-attachments/assets/b6cd59ad-9bfc-48fe-9096-9866727b05cb)

### Step 10: Normal Upload
- **Objective:** Establish baseline application upload behavior by performing an unintercepted submission of a safe, plain-text file.
- **Action:** Selected `/home/cybermom/icdfa-upload-test.txt` using the file chooser and clicked **Upload** without modifying request headers.
- **Recorded Observations:**
  - **Status:** The application accepted and stored the file without file type or extension validation errors.
  - **Response Message:** `../../hackable/uploads/icdfa-upload-test.txt successfully uploaded!`
  - **Inferred Server Location:** `http://192.168.136.136/dvwa/hackable/uploads/icdfa-upload-test.txt`

![Step 10 - Successful Baseline File Upload](https://github.com/user-attachments/assets/b6cd59ad-9bfc-48fe-9096-9866727b05cb)


### Step 11: Intercept Upload
- **Objective:** Intercept and analyze a `multipart/form-data` POST request during a file upload operation.
- **Action:** Enabled **Intercept is on**, selected `icdfa-upload-test.txt`, submitted the upload form, and inspected the HTTP request body in Burp Suite proxy.
- **Observed Request Structure:**
  - **HTTP Method:** `POST`
  - **Path:** `/dvwa/vulnerabilities/upload/`
  - **Field Name:** `uploaded`
  - **Filename:** `icdfa-upload-test.txt`
  - **MIME Type (Content-Type):** `text/plain`
- Captured Headers and Form Data:
  ```http
  POST /dvwa/vulnerabilities/upload/ HTTP/1.1
  Host: 192.168.136.136
  Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryf7HoW7CJsMXaqroj

  Content-Disposition: form-data; name="uploaded"; filename="icdfa-upload-test.txt"
  Content-Type: text/plain
  ```

![Step 11 Intercepted POST Request](https://github.com/user-attachments/assets/9ea19b6c-7e15-4dfc-9500-a617994b7f97)


### Step 12: Extension Handling
- **Command Executed:**
  ```bash
  cp icdfa-upload-test.txt icdfa-upload-test.log
  ls -l icdfa-upload-test.*
  ```

![Step 12 screenshot](https://github.com/user-attachments/assets/9c54ed11-bd41-46af-b44e-106b20c22ce5)

- **Objective:** Evaluate whether modifying the file extension (from `.txt` to `.log`) affects application-side file validation controls while maintaining identical file contents.
- **Observed Result:**
  - **Status:** File accepted without validation restrictions or errors.
  - **Response Message:** `../../hackable/uploads/icdfa-upload-test.log successfully uploaded!`
  - **Target Path:** `http://192.168.136.136/dvwa/hackable/uploads/icdfa-upload-test.log`
- **Analysis:** Comparative analysis shows the application processes `.log` files identically to `.txt` files under the Low security setting, confirming an absence of file extension whitelisting or blacklisting controls.

### Step 13: MIME Handling
- **Objective:** Evaluate how the target application validates client-supplied `Content-Type` headers and demonstrate why MIME validation alone is insufficient for file security.
- **Action:** Intercepted the submission of `icdfa-upload-test.txt` in Burp Suite Proxy and modified the `Content-Type` header parameter from `text/plain` to `image/jpeg` before forwarding the payload to the server.
- **Observed Result:**
  - **Status:** File accepted without error.
  - **Response Message:** `../../hackable/uploads/icdfa-upload-test.txt successfully uploaded!`
- **Security Assessment & Risk:** 
  - The application accepts non-image file content as long as the HTTP request header asserts a valid image MIME type (`image/jpeg`).
  - **Vulnerability:** Relying on user-controlled HTTP headers allows attackers to easily bypass front-end or basic MIME-type filters. Proper security implementations must perform server-side content verification (e.g., inspecting file magic bytes) rather than trusting client-supplied headers.

![Step 13 - Intercepted MIME Modification in Burp](https://github.com/user-attachments/assets/85b661be-db78-45d7-9931-3403edf2a76a)
![Step 13 - Successful Upload Response](https://github.com/user-attachments/assets/c156fc93-0a5a-46fe-ac1f-c7d7b4a73fbe)

### Step 14: Storage Location Analysis
- **Objective:** Evaluate the web accessibility and directory execution permissions of the path used to store uploaded assets.
- **Action:** Disabled Burp Proxy Intercept to allow the `GET` request to complete, retrieving the direct URL: `http://192.168.136.136/dvwa/hackable/uploads/icdfa-upload-test.txt`.
- **Observed Result:**
  - **Status:** HTTP `200 OK`.
  - **Behavior:** The file was served directly from the web root and rendered raw text within the browser session.
- **Security Implications:**
  - **Direct Accessibility:** Storing user-uploaded files inside a publicly accessible web root directory (`/hackable/uploads/`) enables unauthenticated retrieval of uploaded content.
  - **Risk:** If executable scripts are uploaded to a web accessible directory without execution restrictions, an attacker could trigger code execution on the web server.
  - **Remediation:** Store uploaded assets outside the web root directory, serve them via dedicated application handlers with access controls, and mount storage directories with `noexec` options.

![Step 14 - Web Accessible Storage Location Verification](https://github.com/user-attachments/assets/caf032b8-a376-4604-81cc-c64d39b7c3cc)

### Step 15: Run Nikto
- **Command Executed:**
  ```bash
  nikto -h [http://192.168.136.136](http://192.168.136.136)
  Objective: Execute an automated web server scan using Nikto to evaluate server configurations, identify exposed administrative paths, detect outdated components, and verify missing HTTP security headers.

Selected Findings and Technical Analysis
1 Outdated Web Server and Service Versions (Apache/2.2.8, PHP/5.2.4):

Observation: Server: Apache/2.2.8 (Ubuntu) DAV/2 / PHP/5.2.4-2ubuntu5.10 appears to be outdated.

Security Risk: Disclosing exact, legacy version numbers allows malicious actors to cross-reference known Public CVEs (such as remote code execution or buffer overflow vulnerabilities) specific to old Apache and PHP releases.

2 Active HTTP TRACE Method (XST Risk):

Observation: HTTP TRACE method is active and replies which suggests the host is vulnerable to XST.

Security Risk: The HTTP TRACE method echoes back client request headers. When combined with Cross-Site Scripting (XSS), attackers can leverage Cross-Site Tracing (XST) to bypass HttpOnly cookie protections and leak session tokens.

3 Information Disclosure via Administrative Files (/phpinfo.php):

Observation: Output from the phpinfo() function was found.

Security Risk: Rendering phpinfo() reveals sensitive internal system state details, including environmental variables, direct server paths, loaded modules, and PHP compilation settings.

4 Directory Browsing Enabled (/doc/, /icons/, /test/):

Observation: Directory indexing found. / The /doc/ directory is browsable.

Security Risk: When directory indexing is left enabled without a default index.php or index.html file, the server exposes file directory trees, allowing attackers to discover unlinked assets or configuration files.

5 Missing Security Response Headers (X-Content-Type-Options, Content-Security-Policy):

Observation: Suggested security header missing: x-content-type-options / content-security-policy

Security Risk: Missing X-Content-Type-Options: nosniff permits browsers to perform MIME-type sniffing (exacerbating file upload risks), while omitting Content Security Policy (CSP) headers leaves the application vulnerable to script injection and XSS vectors.

![Nikto Scan Output](https://github.com/user-attachments/assets/fc269076-50d2-4286-8e24-a43392333f19)

![Nikto Scan Output](https://github.com/user-attachments/assets/594d3135-a65c-4325-a97c-01a18d285e21)

### Step 16: Save Nikto Output
- **Commands Executed:**
  ```bash
  nikto -h [http://192.168.136.136](http://192.168.136.136) -output lab2-nikto.txt
  mv lab2-nikto.txt.txt lab2-nikto.txt
  ls -l lab2-nikto.txt

Objective: Preserve automated scan results to ensure auditability, enable offline analysis and maintain persistent evidence for technical reporting.

Verification: Verified file persistence using ls -l lab2-nikto.txt, confirming the scan output file was successfully created and populated (12,978 bytes).

Security Reporting Value: Saving vulnerability assessment output in standardized text formats provides verifiable evidence for security reports, supports comparative analysis following remediation efforts, and prevents data loss from terminal buffer clears.

![ls -l lab2](https://github.com/user-attachments/assets/6fe4f5a4-7c31-4771-9831-026acc94e4c7)

## Part 8 - DIRB (Content Discovery)

### Step 17: Run DIRB
- **Command Executed:**
  ```bash
  dirb [http://192.168.136.136](http://192.168.136.136)
  Objective: Perform automated web content discovery using wordlist enumeration (common.txt) to locate unlinked, hidden, or sensitive web directories and endpoints on the target server.

Key Findings Analysis
1 http://192.168.136.136/phpMyAdmin/ (CODE:200|SIZE:4145):

Details: Exposes an interactive database administration web portal.

Risk: Directly exposes the database backend to web traffic, allowing potential password brute-forcing or exploitation of known phpMyAdmin vulnerabilities.

2 http://192.168.136.136/phpinfo.php (CODE:200|SIZE:48107):

Details: Returns full output from the PHP phpinfo() function.

Risk: Discloses sensitive server configuration details, including internal file paths, module versions, database connection parameters, and server environment variables.

3 http://192.168.136.136/dav/ (DIRECTORY IS LISTABLE):

Details: WebDAV directory endpoint with directory listing enabled.

Risk: Exposes raw filesystem contents and may allow unauthorized file downloads, directory browsing, or file writes depending on WebDAV HTTP method permissions (PUT/DELETE).

4 http://192.168.136.136/phpMyAdmin/setup/ (CODE:200|SIZE:8619):

Details: Exposed configuration and setup utility for phpMyAdmin.

Risk: Unsecured setup interfaces allow attackers to reconfigure database settings, manipulate server parameters, or read administrative state data.

5 http://192.168.136.136/twiki/ (CODE:200|SIZE:782):

Details: Unlinked web application directory for TWiki content management software.

Risk: Unlinked secondary applications increase the attack surface and frequently contain unpatched software vulnerabilities independent of the primary application.

![screenshot](https://github.com/user-attachments/assets/77319dcf-53a4-4589-8372-502574f3947c)

### Step 18: Relate DIRB to Uploads
- Objective: Bridge automated path discovery results with the file upload findings to evaluate storage exposure risks.
- Correlation Analysis:
  - Storage Architecture Exposure: DIRB findings confirm that web root structures on the target application expose directories and files directly to client requests.
  - Validation of Upload Risks: This corroborates the observation from Step 14 where uploaded files (`icdfa-upload-test.txt`) were immediately accessible via `http://192.168.136.136/dvwa/hackable/uploads/`.
  - Security Concluding Insight: Automated content discovery demonstrates that hidden or unlinked upload repositories can be mapped or inferred, reinforcing the requirement to isolate user-uploaded content outside the web root directory.


### Step 19: Save DIRB Output
- **Commands Executed:**
  ```bash
  dirb [http://192.168.136.136](http://192.168.136.136) -o lab2-dirb.txt
  ls -l lab2-dirb.txt
Objective: Save automated content discovery findings directly to a structured text file for persistent evidence retention, offline verification, and reporting.

Verification: Verified file persistence using ls -l lab2-dirb.txt, confirming the scan output file was successfully created and populated (6,623 bytes).

Security Reporting Value: Saving directory enumeration logs preserves raw scan data for security documentation, audit verification, and subsequent vulnerability mapping.

![screenshot](https://github.com/user-attachments/assets/ce3c49d2-4662-4388-98bd-422a74b1ce26)

Part 9 - Mutillidae Input Mapping

Step 20: Open Mutillidae
- Target URL: http://192.168.136.136/mutillidae/
- Objective: Access the OWASP Mutillidae II web application to begin input vector mapping and security analysis under a secondary vulnerable platform.
- Observed Result:
  - Status: HTTP 200 OK
  - Interface Details: Rendered the OWASP Mutillidae II main index page (Version 2.1.19, Security Level 0).

![Step 20 - Mutillidae Homepage Interface](https://github.com/user-attachments/assets/d3a1cb30-00ae-4c48-b033-602bbfce4b53)


Step 21 Intercepted HTTP Request Verification
- **Endpoint:** `/mutillidae/index.php`
- **Captured Method:** `GET`
- **Extracted Parameters:**
  - `page=user-info.php` (Target page identifier)
  - `username=admin` (User-controlled query parameter)
  - `password=12345` (User-controlled authentication credential parameter)
  - `user-info-php-submit-button=View+Account+Details` (Form action submission trigger)
- **Security Implications:** Parameters are directly exposed in cleartext within the HTTP request line. Transmitting credentials (`password=12345`) via HTTP `GET` query strings exposes sensitive values to server access logs, browser history entries, proxy caches, and HTTP referer logs.

![Step 21 - Intercepted GET Request in Burp Suite](https://github.com/user-attachments/assets/f588f710-a73e-4bb8-bda2-0c422be24971)

### Step 22: Establish Baseline
- **Objective:** Submit expected, benign input values to establish a baseline for standard application behavior, HTTP status codes, response length, and rendering prior to conducting injection testing.
- **Input Payload:** `username=admin` & `password=12345`
- **Baseline Observation:**
  - **HTTP Response Status:** `200 OK`
  - **Observed Behavior:** The application successfully processes the valid input request, returning a standard HTTP 200 response and displaying the expected user details for `admin` without database syntax errors or exceptions.
- **Methodology Value:** Establishing an explicit baseline provides a comparative standard for detecting anomalies (such as SQL syntax errors, application exceptions, response length fluctuations, or unauthorized data disclosure) during subsequent injection attempts.

![Step 22 - Baseline Burp Suite HTTP History Log](https://github.com/user-attachments/assets/9022bb41-8d2c-41ee-9adc-aa46b576853b)








