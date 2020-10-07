<h1 align="center">enum4linux-ng</h1>
<p align="center">
<strong>A next generation version of enum4linux</strong>
</p>
<p align="center">
<img src="https://img.shields.io/badge/python-3.6-blue.svg"/>
<img src="https://img.shields.io/badge/python-3.7-blue.svg"/>
<img src="https://img.shields.io/badge/python-3.8-blue.svg"/>
<img src="https://img.shields.io/badge/License-GPLv3-green.svg"/>
</p>

enum4linux-ng.py is a rewrite of Mark Lowe's (former Portcullis Labs now CiscoCXSecurityLabs) enum4linux.pl, a tool for enumerating information from Windows and Samba systems, aimed for security professionals and CTF players. The tool is mainly a wrapper around the Samba tools `nmblookup`, `net`, `rpcclient` and `smbclient`.

I made it for educational purposes for myself and to overcome issues with enum4linux.pl. It has the same functionality as the original tool. Other than the original tool it parses all output of the Samba tools and allows to export all findings as YAML or JSON file. The idea behind this is to allow other tools to import the findings and further process them. It is planned to add new features in the future.

## Features
- support for YAML and JSON output
- colored console output
- ldapsearch und polenum are natively implemented
- support for legacy SMBv1 connections
- auto detection of IPC signing support
- 'smart' enumeration will automatically disable tests which would otherwise fail
- IPv6 support (experimental)

## Credits
I'd like to give credit to Mark Lowe for creating the original enum4linux.pl. In addition, I'd like to thank and give credit to Wh1t3Fox for creating polenum.
It was lots of fun reading your code. :)

## Legal note
If you use the tool: Don't use it for illegal purposes.

## Run
1. [Install dependencies](#Installing-dependencies) (various options)
2. ```$ git clone https://github.com/cddmp/enum4linux-ng && cd enum4linux-ng```
3. Run, e.g. ```$ ./enum4linux-ng.py -As 10.10.10.182 -oY enum.yaml```

If you prefer a Docker based installation, an example run can be found [below](#Docker).

### Demo
This demonstrates a run against Windows Server 2012 R2 standard installation. A user 'Tester' with password 'Start123!' was created. Firewall access was allowed. Once the enumeration is finished, I scroll up so that the results become more clear. Since no other enumeration option is specified, the tool will assume ```-A``` which behaves similar to enum4linux ```-a``` option. User and password are passed in. The ```-oY``` option will export all enumerated data as YAML file for further processing in ```enum.yaml```. The tool automatically detects at the beginning that LDAP is not running on the remote host. It will therefore skip any further LDAP checks which would normally be part of the default enumeration.

![Demo](https://github.com/cddmp/misc/blob/master/screencasts/enum4linux-ng/demo1.gif)

The second demo shows a run against Metasploitable2. This time the ```-A``` and ```-C``` option are used. While the first one behaves similar to enum4linux  ```-a``` option, the second one will enable enumeration of services. This time no credentials were provided. The tool automatically detects that it needs to use SMBv1. No YAML or JSON file is being written. Again I scroll up so that the results become more clear.

![Demo](https://github.com/cddmp/misc/blob/master/screencasts/enum4linux-ng/demo2.gif)

### Usage
```
ENUM4LINUX - next generation

usage: enum4linux-ng.py [-h] [-A] [-As] [-U] [-G] [-Gm] [-S] [-C] [-P] [-O] [-L] [-I] [-R] [-N] [-w WORKGROUP] [-u USER] [-p PW] [-d] [-k USERS] [-r RANGES] [-s SHARES_FILE] [-t TIMEOUT]
                        [-v] [-oJ OUT_JSON_FILE | -oY OUT_YAML_FILE | -oA OUT_FILE]
                        host

This tool is a rewrite of Mark Lowe's enum4linux.pl, a tool for enumerating information from Windows and Samba systems. It is mainly a wrapper around the Samba tools nmblookup, net,
rpcclient and smbclient. Other than the original tool it allows to export enumeration results as YAML or JSON file, so that it can be further processed with other tools. The tool tries to
do a 'smart' enumeration. It first checks whether SMB or LDAP is accessible on the target. Depending on the result of this check, it will dynamically skip checks (e.g. LDAP checks if LDAP
is not running). If SMB is accessible, it will always check whether a session can be set up or not. If no session can be set up, the tool will stop enumeration. The enumeration process can
be interupted with CTRL+C. If the options -oJ or -oY are provided, the tool will write out the current enumeration state to the JSON or YAML file, once it receives SIGINT triggered by
CTRL+C. The tool was made for security professionals and CTF players. Illegal use is prohibited.

positional arguments:
  host

optional arguments:
  -h, --help         show this help message and exit
  -A                 Do all simple enumeration including nmblookup (-U -G -S -P -O -N -I -L). This option is enabled if you don't provide any other option.
  -As                Do all simple short enumeration without NetBIOS names lookup (-U -G -S -P -O -I -L)
  -U                 Get users via RPC
  -G                 Get groups via RPC
  -Gm                Get groups with group members via RPC
  -S                 Get shares via RPC
  -C                 Get services via RPC
  -P                 Get password policy information via RPC
  -O                 Get OS information via RPC
  -L                 Get additional domain info via LDAP/LDAPS (for DCs only)
  -I                 Get printer information via RPC
  -R                 Enumerate users via RID cycling
  -N                 Do an NetBIOS names lookup (similar to nbstat) and try to retrieve workgroup from output
  -w WORKGROUP       Specify workgroup/domain manually (usually found automatically)
  -u USER            Specify username to use (default "")
  -p PW              Specify password to use (default "")
  -d                 Get detailed information for users and groups, applies to -U, -G and -R
  -k USERS           User(s) that exists on remote system (default: administrator,guest,krbtgt,domain admins,root,bin,none). Used to get sid with "lookupsid known_username"
  -r RANGES          RID ranges to enumerate (default: 500-550,1000-1050)
  -s SHARES_FILE     Brute force guessing for shares
  -t TIMEOUT         Sets connection timeout in seconds, affects -L, -P, service and session checks (default: 5s)
  -v                 Verbose, show full samba tools commands being run (net, rpcclient, etc.)
  -oJ OUT_JSON_FILE  Writes output to JSON file (extension is added automatically)
  -oY OUT_YAML_FILE  Writes output to YAML file (extension is added automatically)
  -oA OUT_FILE       Writes output to YAML and JSON file (extensions are added automatically)
```

## Installing dependencies
The tool uses the samba clients tools, namely:
- nmblookup
- net
- rpcclient
- smbclient

These should be available for nearly all Linux distributions. The package is typically called `smbclient`, `samba-client` or something similar.

In addition, you will need the following Python packages:
- ldap3
- PyYaml
- impacket

Some examples for specific Linux distributions installations are listed below. Alternatively, distribution-agnostic ways (python pip, python virtual env and Docker) are possible.

### Linux distribution specific
#### ArchLinux

```console
#  pacman -S smbclient python-ldap3 python-yaml impacket
```
#### Fedora/CentOS/RHEL
(tested on Fedora Workstation 31)

```console
# dnf install samba-common-tools samba-client python3-ldap3 python3-pyyaml python3-impacket
```

#### Kali Linux/Debian/Ubuntu 
(tested on Kali Linux 2020.1, recent Debian versions like Buster should work)

```console
# apt install smbclient python3-ldap3 python3-yaml python3-impacket
```

### Linux distribution-agnostic
#### Python pip
Depending on the Linux distribution either `pip3` or `pip` is needed:

```console
$ pip install pyyaml ldap3 impacket
```

Alternative:

```console
$ pip install -r requirements.txt
```

Remember you need to still install the samba tools as mentioned above.

#### Python virtual environment
```console
$ git clone https://github.com/cddmp/enum4linux-ng
$ cd enum4linux-ng
$ python3 -m venv venv
$ source venv/bin/activate
$ pip install -r requirements.txt
```

Remember you need to still install the samba tools as mentioned above.
#### Docker
```console
$ git clone https://github.com/cddmp/enum4linux-ng
$ docker build enum4linux-ng --tag enum4linux-ng
```
Once finished an example run could look like this:
```console
$ docker run -t enum4linux-ng -As 1.2.3.4 
```
