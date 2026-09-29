# Suricata and Fail2Ban Security Lab

[Русская версия](README.ru.md)

In this hands-on security lab, I used two virtual machines: Kali Linux as the testing machine and Debian as the monitored server. I used Suricata to monitor network traffic on Debian and observed Fail2Ban blocking the Kali machine after repeated unsuccessful SSH login attempts.

I performed all scans and login tests between my own virtual machines.

## Lab environment

![Lab topology](assets/topology.svg)

| Machine | Role | Address |
| --- | --- | --- |
| Debian | Suricata sensor and SSH server | `192.168.96.131` |
| Kali Linux | Nmap and Hydra test machine | `192.168.96.132` |

I ran both virtual machines in VMware Workstation Pro with their network adapters set to NAT. Initially, Debian could not connect to the network while configured for Host-only networking. I changed the adapters to NAT, after which connectivity worked. In my Kali screenshot, a three-packet ping to `8.8.8.8` succeeded with no packet loss.

I downloaded the Linux images from their official websites.

## Objectives

- Install Suricata and Fail2Ban on Debian.
- Configure Suricata to listen on Debian's `ens33` interface.
- Observe Suricata alerts during Nmap scans from Kali.
- Test SSH logins from Kali and observe Fail2Ban blocking the source address.

## Debian setup

I updated the package information and installed the monitoring and blocking tools:

```bash
apt update
apt install suricata fail2ban -y
systemctl status suricata
systemctl status fail2ban
```

I checked the network interface with:

```bash
ip a
```

The active interface on Debian was `ens33`. In `/etc/suricata/suricata.yaml`, I changed the interface under `af-packet` from the example value `eth0` to `ens33`:

```yaml
af-packet:
  - interface: ens33
```

My screenshots show Debian using `ens33` with address `192.168.96.131/24`. They also show Suricata 7.0.10 running in system mode with its service active.

I restarted Suricata after changing the interface. I also restarted networking during setup and checked connectivity with a ping:

```bash
systemctl restart suricata
systemctl restart networking
ping -c 3 8.8.8.8
```

I updated the Suricata rules and restarted the service:

```bash
sudo suricata-update
sudo systemctl restart suricata
```

I followed the alert log with:

```bash
tail -f /var/log/suricata/fast.log
```

`tail -f` first prints the existing last lines of the log file, then waits and prints new lines as Suricata writes them. Therefore, lines visible immediately after starting this command may be older events, not events caused by `tail` itself. In the captured output, some lines have timestamps from the previous day. The log also includes DHCP and package-management traffic; one APT-related alert is explicitly classified as not suspicious traffic.

## Nmap scans from Kali

With the Suricata alert log open on Debian, I ran the following scans from Kali against the Debian VM:

```bash
sudo nmap -sA 192.168.96.131
sudo nmap -sT 192.168.96.131
sudo nmap -sS 192.168.96.131
sudo nmap -sV 192.168.96.131
```

These options run an ACK scan (`-sA`), a TCP connect scan (`-sT`), a SYN scan (`-sS`), and service/version detection (`-sV`).

My Suricata log screenshots show traffic from `192.168.96.132` to `192.168.96.131`. During the TCP connect scan (`-sT`) and SYN scan (`-sS`), I observed alerts for MySQL (3306), Oracle (1521), Microsoft SQL Server (1433), PostgreSQL (5432), and a potential VNC scan (5800-5820). During service/version detection (`-sV`), I observed alerts for the same database ports and a potential VNC scan. These are signature alerts about scan traffic directed at those ports; they do not by themselves confirm that those services were installed or listening on Debian.

I did not see an alert in `fast.log` during the captured ACK scan (`-sA`). An ACK scan sends TCP packets with the ACK flag and is commonly used to examine packet filtering; it does not identify ports as open or closed. The lack of an alert means that no matching alert was written to this log during the captured scan. It does not prove that Suricata could not see the packets or classified them as harmless noise. I did not modify the Suricata rules; one possible explanation is that the rules in use did not generate an alert for those packets under the observed conditions.

## SSH and Fail2Ban test

I first tried Hydra before installing the SSH server:

```bash
sudo hydra -L user.txt -P pass.txt 192.168.96.131 ssh
```

That first attempt did not test Fail2Ban because Debian did not yet have an SSH server. After discovering this, I installed the OpenSSH server package and enabled and started the service:

```bash
apt install openssh-server -y
systemctl enable --now ssh
systemctl status ssh
```

My screenshots show the SSH service enabled and active, listening on port 22 on both IPv4 (`0.0.0.0`) and IPv6 (`::`).

On Kali, I prepared two text files: `user.txt` with five usernames and `pass.txt` with five passwords. I have not included their contents in this project. After SSH was running, I ran Hydra again against the Debian SSH service:

```bash
hydra -L user.txt -P pass.txt 192.168.96.131 ssh
```

On Debian, I followed the Fail2Ban log:

```bash
tail -f /var/log/fail2ban.log
```

I did not manually change the Fail2Ban configuration. Fail2Ban provides packaged filters, jail definitions, and ban actions; my screenshots show that the `sshd` jail was active with the systemd backend in this Debian installation. I did not inspect the exact configuration file that enabled the jail during the lab. The captured log reports `maxRetry` 5, `findtime` 600 seconds, and `bantime` 600 seconds. During the Hydra test, Fail2Ban logged repeated `Found 192.168.96.132` entries and then `Ban 192.168.96.132`. These are the values and events I observed; I did not edit the settings.

I can see further `Found` entries after the `Ban` line. They show that Fail2Ban continued to find matching SSH failure records in the system journal. My screenshots do not establish whether these entries came from connection attempts already in progress or how the firewall action was implemented. Hydra's captured output shows that the run finished with `0 valid password found`; it does not show a `Connection refused` error.

## Results

- I ran Suricata on Debian and observed scan-related alerts from Kali.
- Fail2Ban detected repeated SSH login failures from Kali and banned `192.168.96.132`.
- The log showed a configured ban time (`bantime`) of 600 seconds (10 minutes).

## Evidence

The screenshots below document the setup and observed results. Select a filename to view the full image.

| Stage | Evidence |
| --- | --- |
| VMware NAT configuration | [vmware-nat.png](evidence/vmware-nat.png) |
| Kali address and connectivity check | [kali-ip-connectivity.png](evidence/kali-ip-connectivity.png) |
| Debian interface and address | [debian-interface.png](evidence/debian-interface.png) |
| Suricata interface configuration | [suricata-interface-config.png](evidence/suricata-interface-config.png) |
| Suricata service status | [suricata-service-status.png](evidence/suricata-service-status.png) |
| Existing Suricata log entries | [suricata-fast-log-existing-events.png](evidence/suricata-fast-log-existing-events.png) |
| Nmap TCP connect scan (`-sT`) alerts | [nmap-tcp-connect-alerts.png](evidence/nmap-tcp-connect-alerts.png) |
| Nmap SYN scan (`-sS`) alerts | [nmap-syn-alerts.png](evidence/nmap-syn-alerts.png) |
| Nmap service detection (`-sV`) alerts | [nmap-service-detection-alerts.png](evidence/nmap-service-detection-alerts.png) |
| Fail2Ban service status | [fail2ban-service-status.png](evidence/fail2ban-service-status.png) |
| SSH service status | [ssh-service-status.png](evidence/ssh-service-status.png) |
| Fail2Ban `sshd` jail and observed parameters | [fail2ban-sshd-jail.png](evidence/fail2ban-sshd-jail.png) |
| Fail2Ban ban event | [fail2ban-ban-event.png](evidence/fail2ban-ban-event.png) |
| Hydra run result | [hydra-test-result.png](evidence/hydra-test-result.png) |
| Earlier Suricata status and alerts | [suricata-status-and-initial-alerts.png](evidence/suricata-status-and-initial-alerts.png) |
| Initial Fail2Ban log inspection | [fail2ban-initial-log-attempt.png](evidence/fail2ban-initial-log-attempt.png) |

## Notes

- My descriptions of alerts and the ban are based on the terminal screenshots I captured.
- I did not include the SSH usernames, passwords, or wordlist contents. The terminal screenshots do show the local shell account name `alex` in some prompts.
- My screenshots show Suricata log output associated with the `-sT`, `-sS`, and `-sV` scans. No alert appeared in `fast.log` during `-sA`. The screenshots do not include Nmap's own scan summaries or port-state results.
- In this lab, I practiced defensive monitoring and blocking in a two-VM training environment.

## References

- [Nmap Reference Guide: Port Scanning Techniques](https://nmap.org/book/man-port-scanning-techniques.html)
- [Suricata User Guide](https://docs.suricata.io/)

