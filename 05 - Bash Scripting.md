# What is #Bash Scripting ?
- Bash scripting is essentially a plain-text file that contains a series of commands that are executed as if they had been typed in a terminal prompt.
- Generally end in **.sh**
- Begin with **#!/bin/bash** (shebang adn absolute path to the interpreter) and must have executable perms
	- Use **chmod +x** to make a script executable

# Variables
Named places to temporarily store data. Declare the variable in _name=value_ notation and precede it with a **$** to replace the variable with its value.
- **firstName=Good**
- **lastName=Hacker**
- **echo $firstName $lastName**
	- Good Hacker

## Command Substitution
We can also use variables in _command substitution_ by setting the value of a variable to the result of a command or program.
- **user=$(whoami)**
- **echo $user**
	- kali
Alternatively, you can accomplish this with backticks :
![[Pasted image 20221021153011.png]]

## Arguments with Bash Scripting
We can supply command-line #arguments and use them in our scripts
- _Note_ : Bash has a few special variables : 
	- ![[Pasted image 20221021153432.png]]
- So if you have a script that echoes "The first two arguments are $1 and $2" you can run it from the command line with arguments.  For example 
	- "**./script.sh we win**" will output --> "The first two arguments are we and win"

## User Input with Bash Scripting
use the "**read**" command to capture interactive user input while a script is running.
![[Pasted image 20221021160411.png]]
- you can modify the behavior of read with the **-p** and **-s** prompts, which allow us to specify a promt and make user input silent, respectively.
![[Pasted image 20221021160927.png]]

## If, Else, Elif Statements 
``if [ <some test> ]
``then
	``<perform some action>
``fi

- The above syntax represents the conditional.  The square brackets reference the test command and allow the use of all operators allowed by the test command. Common operators include [[[pwk-93521-841889_courseMaterials.pdf](file:///C:/Users/a890259/OneDrive%20-%20Atos/Bureau/pwk-93521-841889_courseMaterials.pdf)]]:
![[Pasted image 20221021161548.png]] , pg 116, PWK




