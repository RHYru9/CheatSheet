---

### What is Email Injection?
Email injection is a type of attack where an attacker manipulates the input of a web form that sends an email. The goal is to trick the application into executing additional commands or sending emails that the attacker controls. This can lead to unauthorized actions, such as sending spam emails or exploiting server functionality.

### Example Scenario
Let’s say you have a contact form on your website where users can submit their email address and a message. The form might look something like this:

```html
<form action="send_email.php" method="post">
    Email: <input type="text" name="email">
    Message: <textarea name="message"></textarea>
    <input type="submit" value="Send">
</form>
```

When the form is submitted, the server-side script `send_email.php` processes the input. A simple PHP script might look like this:

```php
<?php
$email = $_POST['email'];
$message = $_POST['message'];
mail("recipient@example.com", "New message", $message, "From: $email");
?>
```

### How an Attacker Can Exploit This
If the application does not properly validate or sanitize the input, an attacker could enter an email address like:

```
attacker@example.com\nContent-Type: text/plain\nBcc: spam@example.com
```

This would allow the attacker to add extra headers to the email, potentially sending it to additional addresses without the knowledge of the recipient.

### Header Injection Example
In header injection, an attacker can manipulate the headers of the email being sent. If an attacker submits a value like:

```
attacker@example.com\r\nCc: malicious@example.com
```

Here’s what happens:
- The attacker uses a carriage return and newline (`\r\n`) to break out of the intended header and inject a new header.
- This could result in the email being sent to both the intended recipient and the malicious address in the Cc field.

### Body Injection Example
Body injection occurs when an attacker manipulates the body of the email sent by the application. For instance, if the attacker sends an input like:

```
attacker@example.com; echo "Malicious content" >> /var/www/html/malicious.txt
```

If the application were to execute the body content as a command (which it shouldn't), it could result in the addition of malicious content to a file on the server.

### Wildcard Abuse
Wildcard abuse occurs when attackers exploit the behavior of email processing systems that use wildcards to match patterns. For example, if the application accepts user input that gets processed in a way that interprets wildcards, an attacker could input:

```
attacker@example.com; rm -rf *
```

If the application is improperly configured and interprets this input as a command, it could result in deletion of files on the server. Wildcard characters can be used to match multiple files, leading to broader attacks.

### Why This is Dangerous
1. **Information Disclosure**: Attackers can potentially send emails to unintended recipients, leaking sensitive information.

2. **Email Spam**: By manipulating the email headers, attackers can send spam emails to many recipients without their knowledge.

3. **Server Compromise**: If the attacker can inject commands or manipulate file paths, they might exploit vulnerabilities to gain deeper access to the server.

### How to Protect Against Email Injection
To prevent email injection and header/body injection attacks, developers should follow these best practices:

1. **Input Validation**: Always validate and sanitize user inputs. For emails, use a regular expression to ensure they are in the correct format.

   ```php
   if (filter_var($email, FILTER_VALIDATE_EMAIL)) {
       // Valid email
   } else {
       // Invalid email
   }
   ```

2. **Use Prepared Statements**: If you’re inserting user inputs into a database or executing commands, use prepared statements to avoid injection.

3. **Limit User Input**: Limit the length of input fields to prevent overly long and potentially harmful inputs.

4. **Escape Special Characters**: Use functions like `escapeshellarg()` to escape input that is passed to shell commands.

5. **Avoid Direct Execution**: Do not directly execute user input as commands. If a command must be run, ensure that it is carefully constructed from validated input.

6. **Implement Error Handling**: Ensure that error messages do not disclose sensitive information about the server or its configuration.

7. **Use a Web Application Firewall (WAF)**: Implementing a WAF can help filter out malicious inputs and provide an additional layer of security against injection attacks.
8. ---


- [https://book.hacktricks.xyz/pentesting-web/email-injections](https://book.hacktricks.xyz/pentesting-web/email-injections)
