# Learning Linux Forensics: Tracking the regmod Module
## Introduction (Jedha Cybersecurity Course)

As part of my training in the Jedha Cybersecurity course, I performed my first investigation into the Linux Kernel.
The goal was to find a hidden kernel module that should not normally be present and to understand how it works.

This report documents the commands I used and what I learned as a beginner in Linux forensics and kernel analysis.

## Step 1: Hunting the Secret Module

I started by looking for kernel modules that did not have a digital signature.
Initially, running dmesg returned a Permission Denied error, which taught me that administrative privileges are required for kernel-level inspection.

To identify unsigned modules, I used the following command:

```bash
lsmod | awk '{print $1}' | grep -v "Module" | xargs -I {} sh -c 'modinfo {} | grep -q "signer" || echo {}'```

### Discovery

The suspicious module identified was named regmod.

## Step 2: Investigating the Module's "ID Card"

Once the module name was known, I inspected its metadata using modinfo.
This is similar to checking a module’s passport.

modinfo regmod

### Findings

Author: Anne Onyme

Location: /lib/modules/6.11.0-21-generic/extra/regmod.ko

Dependencies: Uses the bluetooth module (unexpected for a regular kernel module)

Parameter: unlock_code (suggesting a password-protected behavior)

## Step 3: Finding the Password (Reverse Engineering)

Hints suggested the password might start with "open...".
I searched for readable strings inside the kernel object file:

strings /lib/modules/6.11.0-21-generic/extra/regmod.ko | grep "open"

#### Final Result

The string opensesame was discovered — the module’s magic word.

## Step 4: Cracking the Code and Getting the Flag

With the password identified, I loaded the module while passing the required parameter.

sudo insmod /lib/modules/6.11.0-21-generic/extra/regmod.ko unlock_code=opensesame

Checking Kernel Logs
sudo dmesg | tail

# Observed output:

[ 3246.708144] regmod: UNLOCKED! flag_part_three: treasure_hunt}

## Step 5: Unloading the Module

Additional information appeared when the module was removed.

sudo rmmod regmod
sudo dmesg | tail -n 5

# Observed output:

[ 3669.321164] regmod: flag_part_two: _modules_

# Final Result

By combining all recovered parts, the full flag was reconstructed as:

Jedha{Kernel_modules_treasure_hunt}
