# Learning Linux Forensics: Tracking the 'regmod' Module

## Introduction (Jedha Cybersecurity Course)
As part of my training in the **Jedha Cybersecurity course**, I performed my first investigation into the Linux Kernel. The goal was to find a "hidden" module that shouldn't be there and understand how it works. This report is a detailed log of the commands I used and what I learned as a beginner in this field.

## Step 1: Hunting the Secret Module
I started by looking for modules that didn't have a digital signature. Since `dmesg` gave me a "Permission Denied" error at first, I learned that I must use `sudo` to get administrative privileges.

I used this long command to filter everything:

```bash
lsmod | awk '{print $1}' | grep -v "Module" | xargs -I {} sh -c 'modinfo {} | grep -q "signer" || echo {}'
Discovery: The module name is regmod.

Step 2: Investigating the Module's "ID Card"
Once I had the name, I used modinfo to see its metadata. This is like looking at the module's passport.

Bash
modinfo regmod
What I found:

Author: Anne Onyme (The last name is Onyme).

Location: It is stored in /lib/modules/6.11.0-21-generic/extra/regmod.ko.

Dependencies: It uses the bluetooth module to function (which is suspicious for a "regular" module).

Parameter found: I saw a field called unlock_code. This looked like a password was needed.

### Step 3: Finding the Password (Reverse Engineering)
I searched for the word "open" because the hints suggested a code like "open...". 

```bash
strings /lib/modules/6.11.0-21-generic/extra/regmod.ko | grep "open"
Result: I found the word opensesame. This was the "magic word"!

Step 4: Cracking the Code and Getting the Flag
Now I had to load the module using that specific password to see what happened.

Loading the module with the parameter:

Bash
sudo insmod /lib/modules/6.11.0-21-generic/extra/regmod.ko unlock_code=opensesame
Checking the logs: I used dmesg to read the system messages and saw: [ 3246.708144] regmod: UNLOCKED! flag_part_three: treasure_hunt}

Unloading the module: The instructions said to check the logs again when removing it:

Bash
sudo rmmod regmod
sudo dmesg | tail -n 5
[ 3669.321164] regmod: flag_part_two: _modules_

Final Result
By combining the metadata and the log messages, I reconstructed the full flag: Jedha{Kernel_modules_treasure_hunt}

What I Learned
How to use sudo for restricted system commands.

How to list and inspect kernel modules (lsmod, modinfo).

How to load and unload modules with parameters (insmod, rmmod).

How to use strings to find hidden information inside binary files.

How to read system logs using dmesg.

Note: This project was completed as part of the Jedha Cybersecurity bootcamp
