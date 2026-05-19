# Penetration Testing & Forensic Audit Lab Using Kali, Ubuntu, Metasploit, Nmap, Hydra.

**Overview of the lab:** In this lab, I built a controlled virtual network environment to simulate real-world cyberattacks and defense strategies. Using **Kali Linux** as the attack machine, I targeted an **Ubuntu** server and a **Metasploitable 2** instance. This project isn't just about hacking, it's also about understanding the entire lifecycle of a security incident, from the first scan to the final forensic log analysis.

> **Note on Network Configuration:** All IP addresses documented in this report are internal, non-routable RFC 1918 private addresses. All three virtual machines were set to a **Host-Only Adapter**, creating a private, isolated "sandbox" network. This ensures that all testing stays securely within the lab and does not interact with the internet or real-world systems.

### Lab Objectives:
* **Network Reconnaissance:** Use **Nmap** to discover active services and identify potential open doors on the network.
* **Credential Testing:** Perform a brute-force attack on **SSH** services to demonstrate the risks of weak password policies.
* **Vulnerability Exploitation:** Utilize the Metasploit Framework to identify and exploit a backdoor in a legacy service (`vsftpd`).
* **Digital Forensics:** Audit Linux system logs (`auth.log`) to identify **indicators of compromise (IoC)** and reconstruct the attack timeline.
* **Framework Alignment:** Map all technical actions to the ***MITRE ATT&CK*** framework to meet industry documentation standards.

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

Before attacking, I needed to create an intentional vulnerability on my Ubuntu VM, essentially leaving a "weak door" open. To simulate poor credential management, I decided to create a new user account with a highly guessable password.

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
