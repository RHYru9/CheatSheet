### 1. **Basics of the SMTP Protocol**
SMTP (Simple Mail Transfer Protocol) is the protocol used to send emails. When an email program connects to an SMTP server, it sends certain commands, such as:
- `HELO`: Introduces itself to the server.
- `MAIL FROM`: Specifies the sender's email address.
- `RCPT TO`: Specifies the recipient's email address.
- `DATA`: Starts the email content section.

Emails have two types of sender addresses:
- **MAIL FROM**: Used by the server to return the email in case of delivery problems.
- **From header**: Used by the email client to display who the sender is.

### 2. **The mail() Function in PHP**
The `mail()` function in PHP is used to send emails. It has several parameters:
- **$to**: The recipient's email address.
- **$subject**: The email subject.
- **$message**: The email body.
- **$additional_headers**: Additional headers (optional).
- **$additional_parameters**: Additional parameters for the `sendmail` program (optional).

The fifth parameter is particularly important because it can be used to inject extra commands into `sendmail`, which is the program used to send the email.

### 3. **Command Injection via the Parameter**
If a web application does not properly filter input, an attacker could inject malicious data into this fifth parameter. For example, if this parameter is filled with a value from an unfiltered GET variable, such as:

```php
$sender = $_GET['senders_email'];
mail($to, $subject, $body, $headers, "-f $sender");
```

An attacker could make a request like this:

```
http://webserver/vulnerable.php?senders_email=attacker@email extra_data
```

This could cause the `sendmail` command to be executed with unwanted arguments, such as:

```
/usr/sbin/sendmail -t -i -f attacker@email extra_data
```

### 4. **Security Issues**
The `mail()` function uses `escapeshellcmd()` to try to secure the input, but this function does not protect against all potential attacks. An attacker can still inject additional parameters into the `sendmail` command.

For example, if an attacker sets the parameter to:

```
attackere@remote -LogFile /tmp/output_file
```

This could cause `sendmail` to be executed with:

```
/usr/sbin/sendmail -t -i -f attackere@remote -LogFile /tmp/output_file
```

If the `-LogFile` argument is accepted by `sendmail`, it could write data to a log file that may contain sensitive information.

### 5. **Known Exploitation Vectors**
One of the first examples of this exploitation was identified by Sogeti/ESEC, which showed how the fifth parameter could be used to read or write arbitrary files. For instance, if an attacker injects:

```
-C/etc/passwd -X/tmp/output.txt
```

The executed command would be:

```
/usr/sbin/sendmail -i -t -C/etc/passwd -X/tmp/output.txt
```

This could lead to the contents of `/etc/passwd` (which contains user information) being copied into the file `/tmp/output.txt`, which could then be accessed by the attacker.
