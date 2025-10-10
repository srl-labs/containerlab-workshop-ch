# Install SRL-GPT and use AskAI

In this lab, you will install SRL-GPT, an AI assistant application installed on Nokia’s SR Linux OS that connects to OpenAI 4o large language model to let you ask natural-language questions about network configuration, logs, and state. Once installed, you’ll interact with it via prompts prefixed with askai (e.g. askai Which interfaces are up? or askai Show me configured IP interfaces), making it easier to learn SR Linux concepts, explore configuration details, and retrieve operational information without relying on traditional CLI syntax.

# Streaming Telemetry Lab

> [!CAUTION]
> Before proceeding, please destroy all other labs to free up the memory  
> `clab des -c -a` - destroys all running labs and removes their respective lab dirs

Start with copying the Containerlab lab-examples to the working directory:

```bash
cd ~/clab-workshop_snr
cp -r /etc/containerlab/lab-examples /home/user/clab-workshop_snr/
```

## Deploy the srl02 lab example

```bash
clab dep -t ~/clab-workshop_snr/lab-examples/srl02
```

Check if the lab is running

```bash
clab ins -a
```

## Deploying the SRL-GPT app on clab-srl02-srl1

The SRL-GPT app can be installed with a single command 'sudo apt update && sudo apt install -y srlgpt' that is executed in the SR Linux bash.

```bash
bash sudo apt update && sudo apt install -y srlgpt
```

Check if the askai-server is running:

```bash
show system application askai-server
```
Output should look like this:
```
A:admin@srl1# show system application askai-server
  +--------------+------+---------+---------+--------------------------+
  |     Name     | PID  |  State  | Version |       Last Change        |
  +==============+======+=========+=========+==========================+
  | askai-server | 5677 | running | 1.0.106 | 2025-10-05T16:55:37.505Z |
  +--------------+------+---------+---------+--------------------------+
```

Logout and log back is onto `clab-srl02-srl1` 

```bash
A:admin@srl1# quit
```
```
Connection to clab-srl02-srl1 closed.
```
```bash
ssh clab-srl02-srl1
```

## Configure the askai-server parameters

First, retrieve the openai key which you can find on the server by running the following command on the host server:

```bash
cat ~/keys/openai_key.txt
```

```bash
enter candidate
askai-server openai-key "xxxxxxxxxxxxxxxxxxxxxxxx"
askai-server model-version gpt-4
askai-server dynamic-state true
commit stay
```

## Ask the first question and accept the terms

Now that the askai-server parameters are configured, you can start asking questions using natural language

```
A:admin@srl1# askai who are you?
```

Accept the terms of the end user aggreement
```
Please go through all terms of end user agreement and Press any key to continue...

Do you accept the terms (yes/no): yes
```

Use AskAi to learn SR Linux.
