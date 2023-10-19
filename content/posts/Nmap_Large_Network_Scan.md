+++
title = "Large Network Scan with Nmap and Masscan"
date = "2023-10-19"
+++

# Nmap and Masscan optimized scan for large networks

## Nmap

This could lead to Network saturation in a large scan and also not accurate results due to excessive `min-rate` value, but for a single IP it really improve the scan time:

```sh
# TCP
sudo nmap -p- --min-rate 10000 IP -Pn --open -n -oN open-ports.txt

# UDP (slow even with the huge min-rate)
sudo nmap -p- --min-rate 10000 -sU IP -Pn --open -n -oN open-ports.txt

# UDP top 100
sudo nmap -p- --min-rate 10000 --top-ports 100 -sU IP -Pn --open -n -oN open-ports.txt

# Show open ports in line
cat open-ports.txt | grep / | sed "s/\/.*//" | tr '\n' ',' | sed '$s/,$//'

# Then perform ServiceVersion -sV and Script -sC Scan
sudo nmap -v -p ports IP -Pn -n -sV -sC
# UDP
sudo nmap -v -p ports IP -Pn -n -sU -sV -sC
# stealth
sudo nmap -v -p ports IP -Pn -n -sS
```

## Large Network

If you need to scan large networks lets say in order of CIDR `/8`, `/16`, `/24`, it's a good idea, if you don't want to wait for days, to adapt the scan options to the target Network. Even if the option `-T` in Nmap could help by setting it to `-T4` or `-T5` some parameter if correctly adjusted could really imporve the time needed to complete the process.

> this scan method does not meant to avoid detection or being quiet, the goal is to get results as fast as possible!

### Important parameters

* `-n`: NO DNS resolution

* `-Pn`: NO host discovery

* `--open`: show only open ports

* `--min-rate`: As mentioned before it's important not to exceed on this parameter because it will lead to Network instability and false results or even freeze the scan, so if for a single IP you could try with `10000` for having an ultra fast scan, it a process of trial and error for individuate the correct value in each situation. Here for fictional purposes I use `5000` as demonstrational value.

* `-min-rtt-timeout` and `-max-rtt-timeout`:

`--min-rtt-timeout` and `--max-rtt-timeout`: To correctly set them run a quick ping if you can or figure out average network response times using another suitable method. Once you have an average ICMP echo time, you can add `25ms` for `--min-rtt-timeout` and `100ms` for `--max-rtt-timeout`. Based on the ping results, with an average of about `230ms` ping time, (in example) you could set minimum at `255ms` and maximum at `330ms`.

* `--min-hostgroup`:

`--min-hostgroup`: This parameter will speed things up by grouping the targets. Values for this should be decided in each situation and typical values that I used goes from `128` up to `512` or `1024` for very large network.

* `--max-retries`: 0

* `--max-scan-delay`: delay between packets = 0

* `-p`: Here ports parameter when specified is set to most common ports for common services that may be open on the target if the scan is performed ON the lists of live Hosts(here we are blindly thinking that some of this ports may be open on some of those targets but some other may be missing). My suggestion is to adapt the command to each Host found Up when the main scan is finshed, also for further and best analysis of all the targets when an interesting Host is found allways perform a custom scan on each of those Hosts

Nmap discovery command used against `/16` ranges and larger:

```sh
# Network Sweep to find Live hosts first

# Skeleton command Stealth scan -sS
sudo nmap -Pn -n -sS -p [ports] --min-rtt-timeout 30ms --max-rtt-timeout 100ms \
--max-retries 0 --max-scan-delay 0 --min-hostgroup [choose-value] \
--min-rate [find-optimal-value] -v --open -iL inputfile -oA output

# ping - find the corrects values for min/max-rtt-timeout
# fictional values used here
ping -c 5 IP

# Tweaked
sudo nmap -Pn -n -sS -p 21,22,25,53,80,111,137,139,445,443,5900,8080 --min-rtt-timeout 255ms \
--max-rtt-timeout 330ms --max-retries 0 --max-scan-delay 0 --min-hostgroup 128 \
--min-rate 5000 -v --open -iL targets.txt -oA Custom
#
grep "Status: Up" Custom | cut -d " " -f2 | tee UpHosts.txt
#
sudo nmap -Pn -sS -p- --min-rtt-timeout 255ms --max-rtt-timeout 330ms --max-retries 1 \
--max-scan-delay 0 --min-hostgroup 128 --min-rate 5000 -v --open \
-iL UpHosts.txt -oA Custom_Full_Port
```

Useful guide and data source [RedSiege - BeyondT4](https://redsiege.com/blog/2022/07/beyondt4/)

Even if the above suggested techniqe from RedSiege works, the methods to find live hosts from `targets.txt` (that actually is already refined) is not spefied in the RedSiege article. His data are for a predefined list of 77 IPs (6s of scan) and for the whole network CIDR /16 = 65.000 IPs (15 min scan). It's up to you to find the better way to refine the whole network IPs to a small number. Eatherway the ip range in the commad could be provided:

```sh
sudo nmap -Pn -n -sS -p 21-23,25,53,111,137,139,445,80,443,3389,5900,8080,8443 \
--min-rtt-timeout 255ms --max-rtt-timeout 330ms --max-retries 0 --max-scan-delay 0 \
--min-hostgroup 128 --min-rate 5500 -v --open 10.11.1.0/16 -oA Custom
```

But doing so we are not fully optimizing the scan because it's better to find live hosts first and then perform the scan.

#### Ping sweep

One approch is to ping sweep the whole netwok earlyier to find first the live hosts in it and then proced with the previous suggested method:

```sh
# ping sweep
nmap -v -sn 10.11.1.1/16 -oG ping-sweep.txt
#
grep Up ping-sweep.txt | cut -d " " -f 2 | > live-hosts-list.txt
# then
sudo nmap -v -Pn -n -sS -p 21-23,25,53,111,137,139,445,80,443,3389,5900,8080,8443 \
--min-rtt-timeout 275ms --max-rtt-timeout 350ms --max-retries 0 --max-scan-delay 0 \
--min-hostgroup 128 --min-rate 5500 -v --open -iL live-hosts-list.txt -oA Scan_results
```

##### Internal Network dicovery

**Ping Gateway IP Addresses**

Let’s say internally, we got an IP address 192.168.57.111 netmask 255.255.255.0 with a default gateway of 192.168.57.1. It is a high probability that the rest of the network ranges would have been defined as /24 CIDR as well. In that case, a ping sweep for the range of `192.168.*.1` with a watch on the TTL would possibly reveal what the other network ranges are.

```sh
nmap -sn -v -PE 192.168.*.1
```

---


## Masscan

Another option is to use Masscan for an intial target approach due to its speed an then perform an accurate scan on each Host with Nmap to properly enumrate all services with all the Namp .

Add a few additional masscan options, including `--rate` to specify the desired rate of packet transmission, `-e` to specify the raw network interface to use, and `--router-ip` to specify the IP address for the appropriate gateway:

- `sudo masscan -p80 10.11.1.0/24 --rate=1000 -e tap0 --router-ip 10.11.0.1`

```sh
# 
masscan -iL target_ranges.txt -p 21,22,25,53,80,111,137,139,445,443,5900,8080 \
-oG raw_targets_list.txt --rate 3000 --open-only
#
awk '/Host/ {print $4}' raw_targets_list.txt | sort -uV > live_hosts

# Insane
masscan 10.1.1.1/16 -p 21,22,25,53,80,111,137,139,445,443,5900,8080 \
--rate 1000000 --open-only --http-user-agent \
"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0"\
 -oG output.txt

# Fast dsicovery only specific ports
sudo masscan target/16 -p 22 --rate 2000 -oG output.txt

# Multi Target
masscan target_1 target_2 -p 80,433 --rate 100000 --banners --open-only\
--http-user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0"\
--source-ip 192.168.100.200 -oL "output.txt"

# /24 network
masscan 10.1.1.1/24 -p 0-65535 --rate 1000000 --open-only --http-user-agent \
"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0"\
 -oJ output.json
```

---


#### SNMP - UDP (Bonus Tip)

If SNMP on UDP is active perform that scan to list all services/processes:

```sh
#
sudo nmap -p- --min-rate 10000 --top-ports 100 -sU IP -Pn --open -n -oN open-ports.txt
#
sudo nmap -v -p 161 IP -Pn --open -n -sU --script snmp-info.nse,snmp-processes.nse
```
