---
layout: post
title: "Creating A Local NAS With Samba"
---
# What is a NAS?
NAS stands for network attached storage. Essentially, a NAS is a place on your network where you can store files. Usually a NAS provides a centralized place to store files and can be accessed from multiple device and or accounts. It can be used to augment devices with low storage capacities, a destination for backups, a media hub, central storage for projects, or any other storage purpose. 

You can purchase a NAS or create your own. There are many providers and many ways to implement a NAS. A NAS can be as simple a dedicated computer with a shared folder or as complicated as specialized devices with RAID hot-swap drives serving "cloud drives" (web browser accessed storage service), FTP, SFTP, FTPS, NFS, SMB (the protocol used for Windows shared folders) and more.

What I want to focus on in this article is setting up your own NAS with Samba for easy compatibility with Windows devices.

# Samba
Samba is an open source server available on Linux. It allows Linux machines to serve SMB file shares like Windows. The Samba project also has tools that allow Linux users to access SMB shares. Samba is widely available and has a docker image on Docker Hub. To keep things simple I will not be using the docker image in this article. There are a lot of settings that will be confusing enough for a new user. 


## Security Considerations


Samba will tie-in with user accounts available on the Linux host. Samba will create its own password database. The user must be present on the Linux host before creating a Samba password. After you create a password for a user they become a Samba user. The local Linux password and the SMB password can be different. I recommend creating a separate unprivileged user for the purpose of network shares. Using different passwords for local and SMB use is another way to improve security. Default settings may try to sync the passwords.

The smb.conf file typically found in `/etc/samba/` is the main configuration file for Samba. There are 4 basic sections in the smb.conf file: `[global]`, `[homes]`, `[printers]`, and `[print$]`. Keep these default sections and adjust their settings as needed.  You will likely find other sections and settings commented out including settings for using Samba as an AD DC. 

Default min server protocol is SMB 2.02. SMB 2 does not encrypt data in transit.
SMB 3 allows the use of encryption in transit and is available. Encryption can be enabled or enforced either globally or per share with the `server smb encrypt` property. This can be configured in the  smb.conf. Consult `man smb.conf` for details. The default global setting for `server smb encrypt` will enable negotiation for encryption but will not turn on encryption globally or per share. This would enable an SMB client that requires an encryption to connect. SMB 3 is supported in Samba smbclient 4.1 and newer, Windows 8 and newer, and Windows Server 2012 and newer. You can also adjust the encryption algorithms if desired.  Enforcing encryption may slow down transfer rates. 

In the quick setup, I will show how to enforce encryption for all connections. If you wish to set the min server protocol higher you can, but that step will not be presented in the quick setup.

The default configuration for `[homes]` will allow all Samba users to see the home directories of other Samba users. These permissions can be tweaked. The default configuration only allows the owner of the directory to view content, but the usernames of other SMB users will be visible to authenticated users. To prevent this ensure that `valid users` equals `%S`, such that `valid users = %S`. This setting is already in some distribution of smb.conf. If the property is not present in your default config, you should set it.

Modern/Newer versions of Samba use NTLMv2 with NTLMSSP for authentication when in a workgroup and can be configured to use Kerberos if joined to a domain or if the domain controller features are used. You can find reference to this in the smb.conf [man](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html) under `ntlm auth` and in other parts of the manual.  

NTLMv2 is the standard Windows authentication protocol used in workgroups but is weak by today's standards of cryptography. The default key size is 56 bits. For the US, there is an optional key size of 128 bits, click [here](https://learn.microsoft.com/en-us/troubleshoot/windows-client/windows-security/enable-ntlm-2-authentication) for history and info on NTLMv2. You can learn a little about NTLMSSP [here](https://learn.microsoft.com/en-us/windows/win32/com/ntlmssp). 

Kerberos fixes a lot of the authentication shortfalls that have not been addressed in NTLMv2. Kerberos continues to receives updates and is a more complicated than the basic challenge response in NTLMv1 and NTLMv2. Windows domain controllers will often leave NTLMv1 or NTLMv2 as a fall back protocol. NTLMv1 has been disabled by default in recent years but may still be prolific on older infrastructure. See this Microsoft [article](https://support.microsoft.com/en-us/topic/upcoming-changes-to-ntlmv1-in-windows-11-version-24h2-and-windows-server-2025-c0554217-cdbc-420f-b47c-e02b2db49b2e) for details.

NTLMv2 is also vulnerable to pass-the-hash attacks. This is usually only exploitable if a bad actor has admin access to a machine connected to the file share and or has stored the NT hash. Red Team Guide has a good writeup on pass-the-hash, and pass-the-ticket attacks. Pass-the-ticket is a similar exploit for Kerberos and follows a similar path as pass-the-hash. Their guide also contains proof of concepts, a simple test lab setup, references to tools, mitigation techniques and more resources. You can visit their guide [here](https://redteamguide.com/guides/pass-the-hash-pass-the-ticket-guide/).

Harding practices for Samba in a domain and as an AD DC can be found on the Samba wiki. See [Samba Security Documentation](https://wiki.samba.org/index.php/Samba_Security_Documentation) and [Hardening Samba As An AD DC](https://wiki.samba.org/index.php/Hardening_Samba_as_an_AD_DC).

You should weight the risks of using an SMB share and the sensitivity of content that will be stored before implementation.

## Quick Setup

1. Create a folder that will later be shared. Example: `/shared/myshare`
2. Create unprivileged Linux user: `adduser`  or `useradd` depending on your distro
3. Ensure the user has access to the share folder. Samba runs with root permissions on most distros.
4. Install Samba with the package manager. Look up how to install Samba on your particular distro. Some distros like Raspberry Pi OS, will require you to install multiple packages. 
5. Create a copy of the default config `/etc/samba/smb.conf` for later reference: `sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.old`
6. Open `/etc/samba/smb.conf` in an editor
7. In the `[global]` field section set or verify the following:
	1. Verify that the workgroup field is set to your workgroup, `WORKGROUP` is default for Windows
	2. `unix password sync = no` -maybe default, stops Samba for syncing the user's Linux password and SMB password 
	3. `server smb encrypt = required` - requires encryption for all connections
	4. `map to guest = never` - required for modern Windows clients to prompt for credentials
	5. `usershare allow guests = no` - prevents users from creating public shares (shares with guest access).
8. In the `[homes]` section:
	1. Verify `browseable = no`
	2. Verify `create mask = 0700` - only provides permissions to user
	3. Verify `directory mask = 0700` - only provides permissions to user
	4. Verify `valid users = %S` - ensures only the authorized user can visit their home directory
9. Review `[printers]` and `[print$]` - makes changes if any
10. Create your network share. 
	1. At the bottom of the file add `[<sharename>]` where `<sharename>` is the shared folder name. For example `[myshare]` will create the shared folder at `\\hostname\myshare`. 
	2. Under the new section set:
		1. `path = /path/to/shared/dir/on/host`
		2. `writeable = yes`
		3. `guest ok = no` - disables guest access
		4. `create mask = 0775` - or 0700 or any standard chmod permissions
		5. `directory mask = 0775` - or 0700 or any standard chmod permissions
11. After saving changes and exiting the editor, run `testparm` to check for errors.
12. Restart the service `sudo systemctl restart smbd`
13. Ensure ports 139 and 445 are open on the host firewall. If you use ufw `sudo ufw allow Samba`
14. Connect to the SMB share at `\\ipaddress\<sharename>` or `\\hostname\<sharename>` and provide your Samba credentials. In Windows you can type the path into the File Explorer or Run command. 



# Troubleshooting


## Windows 11 Won't Connect

 You will have to change the default config to prompt for credentials when connecting. The default config will send unrecognized usernames to login as guest. Windows will send the username of the current user on the first log on attempt for network shares. The SMB client for current versions of Windows 11 reject the connection because it cannot connect as a guest to network shares. This is a newer security feature of Windows. 

To fix this go to the smb.conf file and change the value of `map to guest` equal to `never`,
`map to guest = never`.

