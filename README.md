# Penetration Testing & Forensic Audit Lab (Kali, Ubuntu, Metasploit, Nmap, Hydra, ***MITRE ATT&CK***)

**Overview of the lab:** In this lab, I built a controlled virtual network environment to simulate real-world cyberattacks and defense strategies. Using **Kali Linux** as the attack machine, I targeted an **Ubuntu** server and a **Metasploitable 2** instance. This project isn't just about hacking, it's also about understanding the entire lifecycle of a security incident, from the first scan to the final forensic log analysis.

> **Note on Network Configuration:** All IP addresses documented in this report are internal, non-routable RFC 1918 private addresses. All three virtual machines were set to a **Host-Only Adapter**, creating a private, isolated "sandbox" network. This ensures that all testing stays securely within the lab and does not interact with the internet or real-world systems.

### Lab Objectives:
* **Network Reconnaissance:** Using **Nmap** to discover active services and identify potential open doors on the network.
* **Credential Testing:** Performing a brute-force attack on **SSH** services to demonstrate the risks of weak password policies.
* **Vulnerability Exploitation:** Utilizing the Metasploit Framework to identify and exploit a backdoor in a legacy service (`vsftpd`).
* **Digital Forensics:** Auditing Linux system logs (`auth.log`) to identify **indicators of compromise (IoC)** and reconstruct the attack timeline.
* **Framework Alignment:** Mapping all technical actions to the ***MITRE ATT&CK*** framework to meet industry documentation standards.

### Technical Stack:
* **Nmap:** Used for Network Reconnaissance to discover active devices and identify open ports (entry points).
* **Hydra:** Used for Brute-Force Testing to identify weak or easily guessable passwords on network services like SSH.
* **Metasploit Framework:** Used for Vulnerability Validation to confirm if a system can be successfully compromised through known exploits.
* **Linux System Logs (`auth.log`):** Used for Forensic Analysis to track attacker activity and identify Indicators of Compromise (IoC).
* **VirtualBox / VMware:** Used to build an Isolated Sandbox Environment to safely simulate cyberattacks without impacting real networks.

---

## Phase 1: Finding the IP Addresses

This is a preparational phase. Because all three virtual machines were configured on the same **Host-Only Adapter** prior to starting, their IP addresses fall within the same local subnet. 

I ran the `ip addr` command in the terminal of each machine to identify their local IP addresses, which are noted below:

<img width="975" height="448" alt="image" src="https://github.com/user-attachments/assets/8560cf7c-ff16-4f70-94d1-211f8be126e8" />


* **Metasploitable:** `192.168.56.101`
* **Kali Linux:** `192.168.56.102`
* **Ubuntu Server:** `192.168.56.103`

Alternatively, I also discovered all of them at once from my Kali terminal using **Nmap**. 

I ran the following command on Kali to perform a "Ping Sweep":

`sudo nmap -sn 192.168.56.0/24`

This command sends a tiny "Are you there?" message to every possible address on the private network. It successfully found the exact same results for all three machines, confirming that my attack machine could see the targets.

<img width="975" height="204" alt="image" src="https://github.com/user-attachments/assets/1a085120-e003-4ee1-b88d-a824c6014554" />

## Phase 2: Setting Up the Vulnerability on the Ubuntu VM

With the preparation done, it was time to move into the exploitation phase. 

Before attacking, I needed to create an intentional vulnerability on my Ubuntu VM, essentially leaving a "weak door" open. To simulate poor credential management, I decided to create a new user account with a guessable password.

Initially, I tried to use my first name for the new user, but the terminal threw a fatal error because the machine itself was already named after my first name. That was a great practical lesson in Linux system administration!

Instead, I used my last name and added the user by typing the following command in the Ubuntu terminal:

`sudo adduser chowdhury`

When the system prompted me to set a password for the new user, I intentionally set it to something weak and guessable: `legendsafayet`.

<img width="975" height="343" alt="image" src="https://github.com/user-attachments/assets/86b7516c-dfc6-448a-8f9b-e4ca460e3129" />

I bypassed the optional user information prompts (Full Name, Room Number, etc.) by hitting `Enter` and typed `Y` to confirm the setup.

At this point, I switched to my Kali VM and ran the following command to initiate the brute-force attack:

`hydra -l chowdhury -p legendsafayet 192.168.56.103 ssh`

**Command Breakdown:**
* `-l chowdhury`: Tells Hydra the target username is `chowdhury`.
* `-p legendsafayet`: Tells Hydra to test the password `legendsafayet`.
* `ssh`: Tells Hydra to execute the attack against the Secure Shell (SSH) service.

<img width="975" height="356" alt="image" src="https://github.com/user-attachments/assets/3d3617fd-5073-428f-ad92-ea1f4806668a" />


Once Hydra confirmed `1 of 1 target successfully completed` (highlighted in the second yellow box from top), I manually verified the vulnerability. I typed the following into my Kali terminal to log in using the discovered credentials:

`ssh chowdhury@192.168.56.103`

**The Result:** After entering the password `legendsafayet`, my terminal prompt changed from `safayet@kaliVM` to `chowdhury@ubuntuVM` (marked with a yellow circle in the image). This provided clear evidence that I had successfully breached and was inside the target machine.

## Phase 3: Metasploitable Backdoor

After the SSH attack, I moved on to exploit a famous backdoor in the `vsftpd` service using the Metasploit Framework. I launched the framework by typing `msfconsole` in my Kali terminal and searched for the specific vulnerability using the following command:

`search vsftpd`

I targeted the backdoor in the Metasploitable machine's FTP service by typing:

`use exploit/unix/ftp/vsftpd_234_backdoor`

Inside the Metasploit prompt, I initially attempted to set the payload using `set payload cmd/unix/interact`, but the framework returned a "not valid" error. I suspected this was due to the constraints of the isolated network environment I built for the lab. Deciding to push forward, I set the target IP and attempted to launch the attack:

`set RHOSTS 192.168.56.101`
`exploit`

<img width="975" height="415" alt="image" src="https://github.com/user-attachments/assets/aeca5a08-4014-41c3-9ba3-d3ab773f20ec" />

As seen in the terminal output above, the attack failed. I encountered a validation error with the default reverse payloads because of my isolated sandbox network. 

To bypass this issue, I pivoted to a **Bind Shell** configuration. This adjustment allowed me to establish a direct connection to the target without relying on a reverse connection back to my Kali machine, successfully granting me root access.

<img width="975" height="482" alt="image" src="https://github.com/user-attachments/assets/696b1d3c-0804-4d07-9f81-13acc5963b2b" />

Before successfully exploiting the backdoor, I encountered another issue, a **race condition** where the backdoor listener on port 6200 remained active from a prior failed session. I rebooted the Metasploitable VM to clear the hung state.

Once the target machine was back online, I successfully executed the attack. Here is the step-by-step breakdown of the final command sequence:

`use exploit/unix/ftp/vsftpd_234_backdoor`
`set payload cmd/unix/bind_ruby`
`set RHOSTS 192.168.56.101`
`exploit`

<img width="975" height="471" alt="image" src="https://github.com/user-attachments/assets/8e79314a-4c11-46c1-b251-021c95c4c21f" />

To verify the level of access I had achieved through the bind shell, I ran two simple commands within the new session:

* **`whoami`**: The system replied with `root`.
* **`hostname`**: The system replied with `metasploitable`.

This proved that I had successfully navigated the network-level configuration issues, bypassed the payload constraints, and gained full administrative control over the target server.

## Phase 4: Post-Incident Analysis & MITRE Mapping

In this final phase, I put on my "Security Analyst" hat to prove that the attacks I performed left a clear trail. By matching these logs to the MITRE ATT&CK framework, I've demonstrated how real-world threats are identified and categorized by security professionals.

### The Forensic Log Evidence

After the automated attack, I performed a deep-dive audit of the `/var/log/auth.log` file, using the `-a` flag with `grep` to bypass binary encoding.

Here is the exact command I used to filter the noise and isolate the SSH logs for the targeted user:

`grep -a "chowdhury" /var/log/auth.log | grep "sshd" | tail -n 15`

<img width="975" height="233" alt="image" src="https://github.com/user-attachments/assets/5387ba26-1cb0-4568-810d-a83a65983fbf" />

As we can see in the log output, multiple lines show `Accepted password for chowdhury from 192.168.56.102`. This is a clear **Indicator of Compromise (IoC)**, identifying successful SSH authentications for the user `chowdhury` originating from the attacker IP (`192.168.56.102`). 

The logs also show `pam_unix(sshd:session): session opened` and `session closed`. This confirms that my Kali machine successfully opened interactive sessions, verifying the transition from a password-guessing attempt to full remote access.

### Network Architecture & OPSEC

This lab was built using Host-Only Adapters in VirtualBox to create a completely isolated "sandbox" environment.

* **Protection:** This prevents "cross-contamination," ensuring that scanning and exploitation traffic cannot reach my home network or the public internet.
* **Stability:** It allows for the testing of "noisy" attacks (like the `vsftpd` backdoor) without triggering external security alerts or causing accidental damage to production-like systems.

### ***MITRE ATT&CK*** Mapping

To ensure this lab follows industry standards, I have mapped my activities to the ***MITRE ATT&CK*** **Framework**. This allows for a clear, standardized understanding of the tactics and techniques used during the simulation.

| Tactic (Phase) | Target | ***MITRE*** ID | Technique Name |
| :--- | :--- | :--- | :--- |
| **Initial Access** | Ubuntu | `T1110.001` | Brute Force: Password Guessing |
| **Execution** | Metasploitable | `T1210` | Exploitation of Remote Services |
| **Discovery** | Network | `T1046` | Network Service Discovery |

### Conclusion & Summary

This lab successfully demonstrated how a single open port or a weak password can lead to a full system compromise. By gaining remote shell access to both targets, I proved the effectiveness of common attack vectors like password guessing and legacy service exploitation.

However, the most important part of this project was the **defensive audit**. By analyzing the system logs, I was able to prove exactly when and how the attacker got in. This project highlights my ability to not only perform security testing but also to provide the forensic evidence needed to investigate and remediate a breach.

### Connect & Explore

**GitHub**: Thank you for following along! This lab is part of my ongoing commitment to mastering incident response and forensic analysis. I’m constantly adding new labs and documentation, check back soon for upcoming projects! Check out my existing works below: 
* [**Incident Response with Splunk**](https://github.com/safayetchowdhury/Incident-Response-with-Splunk)
*  [**SQL-Based Security Incident Investigation**](https://github.com/safayetchowdhury/SQL-based-Security-Incident-Investigation)
*  [**Wireshark-and-Nmap-Packet-Analysis-and-Network-Reconnaissance**](https://github.com/safayetchowdhury/Wireshark-and-Nmap-Packet-Analysis-and-Network-Reconnaissance)


**LinkedIn:** I’d love to **[connect](https://www.linkedin.com/in/chowdhurysafayet/)**! If you’re working in information security or are interested in these methodologies, let's discuss how we can improve security posture together.

