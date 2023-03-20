> There are basic configuration information about local NAS, 10G backbone network and ways to launch everything properly ...
---

# The structure
The lab mini-network consists of the next nodes:
* QNAP Network Area Storage, domain name is NASTML, IP address is 172.16.32.174
* Servers (detailed information is on the servers page)
* TL-SX3008F JetStream 8-Port SFP+ 10GE Managed Switch

Both NAS and Servers' group are connected to the campus network 172.16.32.0/23 and could be connected to over the SSH protocol directly from the campus network, or over cs.uml.edu

Both NAS and Servers' group are connected over 10G NIC cards to the isolated network 192.168.0.0/25, which has no active routing over the internet and allows the Servers to access shared folders on NASTML
Switch has all the ports configured to default vlan 1. Ports 7 and 8 are aggregated and connected to NASTML over aggregated ports 4 and 5 correspondingly. NASTML is DHCP and DNS server of 192.168.0.0/25 network with the default gateway 192.168.0.2/25 and DHCP pool starting from 192.168.0.4/25


## Shared folders

NASTML has two logical storages
* SSDPool (RAID5 of 4 x SSD Samsung Evo 4TB SSD) total size of 10TB
* HDDPool (single 16TB HDD) total size of 13.64TB

There are next shared folders:
* shared_home (SSD_Pool)
* archive (HDD_Pool)
* Public (HDD_Pool) - supports quest access

# Access
We are using 2 protocols to connect to NASTML:
* SAMBA (SMB v2, v3, v4)
* NFS

Samba provides better speed and easy to manage user-access permissions, though those are not fully compatible with Unix POSIX permissions on the current version of the QNAP NASTML firmware.
So far it is recommended to use Samba connections only to read or to download some information from NASTML. It is NOT recommended to use Samba for shared_home folder at this stage since some of Ubuntu services require POSIX information.
This would be fixed with an upcoming firmware upgrade

## Ubuntu SAMBA

NASTML has a list of users with usernames and passwords, which might be different from those on the Servers.
Any machine on the campus can connect to NASTML shared folders using those over SAMBA protocol.
To do that from a machine using Ubuntu, you need to install CIFS-utils. You can run the following command to do so
```bash
sudo apt install cifs-utils # for the Ubuntu 18+ versions
sudo apt-get install cifs-utils # for the older versions of Ubuntu
```
Once it's done we can mount any of those folders using the next command to the specific path we choose.
```bash
#On the servers default location is /mnt/samba
sudo mount -t cifs-utils nastml.uml.edu/archive -o user='%username%',password='%passwd%' /mnt/%path% #using the domain name
sudo mount -t cifs-utils //172.16.32.174/archive -o user='%username%',password='%passwd%' /mnt/%path% #using the IP address
sudo mount -t cifs-utils //172.16.32.174/Public /mnt/%path% #Public folder has guest access
```
## Ubuntu NFS
NFS (Network File System) allows different servers to tralslate and inherit POSIX permissions over the file operations, which is required for sharing our home directories located on NASTML inbetween different servers.
That is done to create a unified environment for each user working on TextMachineLab Servers. That is why NASTML/shared_home only accessible from 192.168.0.0/25 network, so nothing could interfere with the environment
To connect to the NASTML/shared_home directory using NFS you do not need a username/password combination, those are resolved by servers' Ubuntu policies.

If you are reconfiguring some server, or planning to add a different one you need to install NFS services to Ubuntu first
```bash
sudo apt install nfs-common
```

After that we can mount our shared home directory as:
```bash
sudo mount -t nfs 192.168.0.2:/shared_home /mnt/shared_home
```

## Windows/MacOS

Windows and MacOS has built-in samba-compatible clients, and all you need to connect is the path 'nastml.uml.edu/archive' and a combination of a username and password.
To get those please access the current system administrator of the TextMachineLab

## Access restrictions
| Shared Folder | Guest Access | Samba Support  | NFS Support      | 172.16.32.0/23 Accessibility | 192.168.0.0/25 Accessibility |
| shared_home   | no           | yes            | only 192.168.0.0 | yes                          | yes                          |
| archive       | no           | yes            | no               | yes                          | yes                          |
| public        | yes          | yes            | no               | yes                          | yes                          |

# User management
In order for the different servers running under the same user in the same environment we have to keep POSIX user IDs the same for a single user on each machine.
So far here is the list of how we name them

Those home directories are stored at 'nastml.uml.edu/shared_home/%usermane%'
The path of home directory on each server is '/mnt/shared_home/%username%'

If you create a user, please make sure it has the same UID on the all fo the machines, before setting his home directory path to the shared folder


