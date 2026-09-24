---
layout: post
title: Windows Backups From Windows
---

# Overview

There are various ways to take backups in Windows. I will try to divide the article into practical uses for backups for regular Windows editions and Windows Server editions generally. This article will not review paid backup services, but focus on generally available tools and practices. 


# Windows Endpoints

## Windows 7 - 11: Backup and Restore (Windows 7)

Windows 7 came with a backup and restore utility named `Backup and Restore`. In later versions of the utility it was renamed  `Backup and Restore (Windows 7)`. This utility can enable file and image backups for a single Windows instance. The tool can also be used to create a system repair disc if you have an optical disk drive or DVD writer/reader. 

Backups can be rendered to locally connected drives or to network drives. Backups to network drives will give a warning that network backups cannot be securely protected. Destination and source drives connected locally must be in NTFS format. ExFAT and other FAT filesystems will not work with the utility.

Management of backups is done by the machine that enabled the backup. Individual files and versions can be restored through the tool. The image backup can be used to restore the protected system if it stops working. 

To access this tool go to `Control Panel`. In Windows 11 you will need to change the view to `Large icons` or `Small icons`.

Be careful when using this utility. An old _How to Geek_ [article](https://www.howtogeek.com/1838/using-backup-and-restore-in-windows-7/) warns that the (recommended) backup settings will not backup program files, FAT formatted files, Recycle Bin data, or temp files greater than 1 GB. 

You should also test the backup initially and periodically to ensure later backups are free of corruption. 

Windows officially deprecated the Windows 7 backup and restore utility. They continue to make it available but they do not issue updates. The newer tool for file versioning is `File History` and has existed since at least Windows 8. `File History` cannot be turned on if a Windows 7 Backup schedule exists, see this Microsoft [article](https://learn.microsoft.com/en-us/windows/compatibility/windows-7-backup-and-restore-deprecated).

The `Backup and Restore (Windows 7)` utility can still be useful for creating a full image backup. VSS is used to create the backup, according to a Windows Forums [post](https://windowsforum.com/windows-news.4/legacy-backup-and-restore-windows-7-still-useful-but-deprecated.377298/). VSS is the backbone of most live Windows backups. You should verify that the image works by either restoring to an unused disk, or converting a copy of the backup image to a VM disk and boot. You can also mount the image as a drive and run integrity checks, such as `chdsk`. If the protected device had integrity issues, backup your files and run repairs before Image backups.

You cannot restore individual files from the image backup using this tool. The backup utility restricts functionality to restoration of the whole image. Mounting the image can be done to extract files. 

You may notice that the system image backup tool has a warning that only one system image per computer can be stored at the destination location. If you want another backup on the destination location, rename the folder where the previous system image is stored. There is a reference below if you encounter issues or need further guidance. The size of the system image maybe as large as your used drive space. The image will contain all partitions such as EFI, C:, and recovery partitions. 
### Sources 
- [Howtogeek.com](https://www.howtogeek.com/1838/using-backup-and-restore-in-windows-7/)
- [WindowsForum.com](https://windowsforum.com/windows-news.4/legacy-backup-and-restore-windows-7-still-useful-but-deprecated.377298/)
- [Windows7 Backup Deprecated](https://learn.microsoft.com/en-us/windows/compatibility/windows-7-backup-and-restore-deprecated) 
- [Creating Multiple Image Backups](https://athomecomputer.co.uk/store-more-than-one-system-image-on-drive/)

## Windows File History

`File History` was Window's next tool for backups. This tool does not take image backups and is only used for file backups. The default protected locations are Libraries (Documents, Music, Pictures, Videos), Desktop, Contacts, and Favorites. Default settings run backups every hour. The default retention policy is `Forever`. These settings can be adjusted in the advanced settings to other preselected values.

`File History` has wider support for destination devices. FAT device, NTFS, and network drives are supported. Microsoft recommends use of an external storage device. 

To turn on and manage `File History` settings, first go to Control Panel. Next, select large or small icons in the view settings. Then click on `File History`. Setup is fairly simple. Ensure that a supported device is connected. `File History` will automatically scan for connected drives that can be used. 

I had difficulty using a samba share as a backup location. I don't know if it was my custom setup or a lack of compatibility with samba. The `File History` error occurred at the .NET Runtime level. It appears to be an access or permissions issue. You may have to do some troubleshooting if you intend to use a network share as your backup location.

>Default Behavior: Restoring a file or folder from a previous version will overwrite the current version of the file or folder 

To restore a file or folder you can inspect the properties of the item and go to the version tab. There you will see previous versions. The default restore feature will overwrite the current file or folder. You can preview a previous version with the `Open` button. You can also expand the `Restore` button to restore to a different location. I would recommend copying the current folder or file to another location first incase the expand feature misregisters the click and restores.

You may notice that there are already different versions of your files under the Previous Versions tab. If you use Windows 11, it is likely that `System Protection` is activated on your device. `System Protection` takes automatic snapshots of your Windows device before updates and at other intervals. These snapshots are restore points that can be used if a critical error occurs. You can also take manual snapshots. 

### Sources
- [Windows Support File History Backup and Restore](https://support.microsoft.com/en-us/windows/experience/backup-recovery/backup-and-restore-with-file-history)

## System Restore and System Protect

These two utilities are used in conjunction and are enabled by default on Windows 11. As previously stated, `System Protect` creates snapshots of the system before any updates creating restore points if an update fails. `System Protect` may take other snapshots occasionally. 

These snapshots are taken using VSS and are stored as shadow copies. The shadow copies are typically stored on the same drive in shadow storage. 

To access these tools you can go to Control Panel, set `View by` to small or large icons, select `Recovery`. Selecting `Open System Restore` will start the `System Restore` utility where you can restore to a previous restore point/snapshot. Selecting `Configure System Restore` will bring up the settings menu for `System Restore` and `System Protect`. From this menu you can create a manual restore point. 

Settings for `System Protect` and `System Restore` are very limited. There are command line tools that can be used to adjust VSS storage settings and location such as `vssadmin`. I would not recommend using these tools to beginners.

## Windows 11 and OneDrive

In recent years, Windows has pushed use of OneDrive. This can be a convenient file backup.  

OneDrive is a free service, but has only a 5 GB limit for the free license. I include it in this list because Microsoft 365 subscriptions contain higher limits for OneDrive for both personal and business subscriptions. Most subscriptions allow 1 TB of cloud storage per account on multiple devices. OneDrive is often enabled during account creation on Windows 11 when an email is used to sign in or when Microsoft 365 Office apps are activated. Your OneDrive is signed in if you agreed to sign in everywhere when activating Microsoft 365. 

OneDrive does automatic backups of user folders and files such as Desktop, Documents, Pictures, Videos, and Music.

If you reinstall Windows 11 or purchase a new Windows 11 device you will be prompted to sign-in to a Microsoft account. You will then be asked if you want to restore your files from OneDrive. 

If you cancel your Microsoft 365 subscription you may loss access to your cloud stored data.
### Sources
- [One Drive Subscription Details](https://www.microsoft.com/en-us/microsoft-365/onedrive/free-online-cloud-storage/#tabspillbar1_tab0)

## Robocopy

Robocopy is a command line tool that has been part of Windows for decades. It has had a few iterations. Earlier versions of robocopy on systems such as Windows Server 2012 R2, Windows 7 and earlier encountered errors when file path length exceeded 256 characters. This error was caused by early Windows path limitations. Modern versions of robocopy have fixed this issue. 

Robocopy can be used to archive data to another drive attached locally or on the network. Robocopy will by default use a delta between the files in the source and destination locations to determine which files need to be copied. This is convenient when only a few files have changed since your last backup. Robocopy does support compression. Robocopy can be slow but you can enable multi-threading for faster performance. Robocopy has many optional features. Robocopy does not do versioning. 

You can use robocopy in a batch file to use along side other tools to refine you own backup scheme.  

Robocopy is useful for file level backups but will not preserve installed programs, drivers, or OS. 

You can learn more about robocopy from Microsoft's official documentation. It is a useful tool for general copying too.

### Sources
- [Robocopy](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy)

# Windows Server

## Windows Server Backup

Windows Server Backup has existed since Windows Server 2008. It can be used to backup other Windows servers or Windows endpoints. The service uses VSS to take backups and can store backups on a separate drive. The drive could be local, external, or a network share.

Windows Server Backup has the backup options `Full Server (all volumes)`. `Critical volumes`, `Noncritical volumes`. The service can be used for bare metal restores, volume restores, or file level restoration.

The Sever Backup role improved significantly in Windows Server 2012. The 2012 version saw increased storage limits and supported larger target volumes. Another upgrade was improved support of Hyper-V virtual machine backups which allowed for easier restorations.

Most enterprises prefer to use backup service providers or third-party backup software rather than the Windows Server Backup role. There is general distrust of this service. Reliability issues, unfriendly user controls, default limit of one backup per system on network shares and more concerns can be found reported online. 

If you want to experiment with this feature, Microsoft does have some documentation. I was unable to find an official page on the Windows Server Backup role that is still updated. Some third parties have articles on Windows Server Backup like HostandTech.com. There is a link to their article in the sources of this section.

Most Windows Server versions have an evaluation licensed ISO that allows for a 180 days free trial. Most of these ISO's can be found easily on official Microsoft sites. The Windows Server 2025 evaluation version is locked behind a sign up form.  

### Sources
- [Microsoft Server Backup 2008](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754572(v=ws.10))
- [Microsoft Server Backup 2012](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj614621(v=ws.11))
- [Third Party Guide on Microsoft Server Backup](https://hostandtech.com/kb/windows-server/windows-server-backup-and-recovery-guide/)

# General Backup Practices

## 3, 2, 1 Rule

This is an old backup practice. It is good to known and consider while making your own backup plan and policy. Different sources explain the rule with slight variations. The rule is as follows:

- 3 copies
- 2 types of storage devices
- 1 offsite location

In other words, have 3 copies of important data. Store the data on at least 2 different types of storage devices or mediums. Keep one copy at an offsite location. 

The multiple copies mitigate the risk of accidental deletion. The different types of storage allow recovery if one device breaks, becomes corrupted, or becomes otherwise inaccessible. An offsite copy is useful incase the primary site experiences a disaster such as a flood or fire and the data is lost. 

The 3,2,1 rules does not specify when backups are taken or the type of backups. Generally, it is assumed that each of the 3 copies can stand alone incase 2 of the copies are lost. 

It is also a good idea to have an offline backup. A ransomware attack can encrypt or destroy the current data and online backups. In this case, online backups refers to both internet connected and/or locally connected backups.  An offline backup is usually an external drive or other media that is stored off. Offline backups are often taken at longer intervals than online backups. A series of offline storage devices may also be used where device A is used to take the first backup, then the device B is used to take the next backup. This reduces the likelihood that multiple or all offline backups become compromised. 

Some storage medias are better for offline backups than others. SSDs and flash drives store data through a series of cells that use high and low charges. If these devices are left without power for too long, data can become corrupted or lost. I would not expect an SSD to maintain data for more than a year without power. Other factors such as usage (writes to the cells) and climate can affect the cells charge retention. Hard drives do not have this issue because data is stored magnetically. You will have to store hard drives away from strong magnates and away from the elements. Tapes also do not have a power issue but are expensive. 

### Sources

- [Kingston's 3.2.1 Rule](https://www.kingston.com/en/blog/data-security/321-data-backup-method)
- [Acronis 3,2,1 Rule](https://www.acronis.com/en/blog/posts/backup-rule/)
- [More on SSDs](https://storedbits.com/how-do-ssds-retain-data-without-power/)


## How Often Should You Backup?

The short answer: As often as you can afford to lose. 

When determining how often to backup files or create a whole image (full backup) of your computer you should consider how often you make changes and how many days you can be set back. Maybe you don't make changes very often to local files and changes are minor. A weekly or monthly backup may be appropriate. Maybe you make a good amount of changes everyday and you can afford a day's set back. Then you can backup once a day. Maybe you make many changes a day that are all very important. Then an hourly, or sub hourly backup schedule may be appropriate. 

File backups are the easiest type of backup to restore. It could take seconds, minutes. or a few hours for very large files. Taking file backups can also be quicker when tracking files for changes.

Image backups consume more time to take and restore. They can bog down system resources. Full backups typically take the longest. Incremental and differential backups are shorter when taken. Restoring an image backup can take hours or days. Bare metal restores (restoring the Operatings System, drives, and data from an image) requires writing all the data of the backup to the original drive(s). An image restore from a full backup takes less time than an image restore from differential or incremental backups. For differential or incremental backup restoration, a full backup must be restored first. Then the series of differential or incremental backups must be restored in order to preserve data integrity. If one of the differential or incremental backups is corrupted in the series you will likely waste time and have to restore before the corrupted entry in the series.

## Backups and Encryption

Not all backup solutions support encryption. As far as I'm aware, none of the above discussed tools support encryption. The main purpose of backups is to make information available when needed. Ensuring the data is recoverable is prioritized by many backup tools. 

Encrypted backups can create frustration when keys are not properly stored or available for decryption. If you have encrypted backups make sure that you have the decryption keys or passwords securely stored, but available incase of a disaster. Storing the decryption keys or passwords on only the protected system (the system being protected by backups) is a recipe for disaster.  

Encrypted backups typically only secure the backup when it is at rest. Encrypted backups can either be achieved by encrypting the drive the backups are stored on or encrypting individual backup files. The backup needs to be decrypted to access, use, or change the data. A destination drive will need to be decrypted when backups are taken. 

Encrypted backups are more attractive when you store your backups on a third-party cloud. This could prevent unauthorized access of your personal data if the third-party is compromised. I would recommend encrypting or password protecting any sensitive files on your system regardless. 

# Last Words
Hopefully this article can be useful to someone. There are many free and opensource backup solutions and paid solutions. If you found the above solutions archaic, there are plenty of others. What every you decide on as your backup solution good luck. Do some due diligence on researching risks of the solution, and remember to do at least occasional checks on the backup. 