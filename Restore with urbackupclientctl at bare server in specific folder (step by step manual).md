# Restore with urbackupclientctl at bare server in specific folder (step by step manual)
[I wrote it at the UrBackup forum](https://forums.urbackup.org/t/restore-from-command-line-specific-file-in-folder/8762/39?u=dmitrius7).


## Preparatory actions

Disable scheduled backups for this client (options: interval for incremental/full file/image backups). It is necessary that when we install the client on the bare server, the backup does not start.

Stop urbackupclientbackend.service at the client (client at production server);
`systemctl stop urbackupclientbackend.service`

Download preconfigured client for Linux from web interface;

For restore files at the bare server you have to install the client. If you have new had (empty hdd) you can boot from linux live CD;

## Install preconfigured client for Linux to the bare server

Upload preconfigured client for Linux to the bare server and install it;


`sh UrBackup Client*.sh`
  
if you install client from preconfigured disturb by default the option "INTERNET_ONLY=true". 
If this client is not internet client you should disable it:

`sed -i "s/INTERNET_ONLY=true/INTERNET_ONLY=false/g" /etc/default/urbackupclient`
`systemctl restart urbackupclientbackend.service`

Check this client online

## Find backup at the CLI (urbackupclientctl)

List the folders to back up. Here we need to get the folder name - NAME column

    urbackupclientctl list-backupdirs
    PATH  NAME   FLAGS
    ----- ------ ----------------------------------------------
    /     rootfs follow_symlinks,symlinks_optional,share_hashes
    /boot boot   follow_symlinks,symlinks_optional,share_hashes

Display a list of backups for roots (folder name). 
Here we need to **find out the backupid of the required backup**. The higher the backupid, the higher the backup is. For convenience, we are guided by the web interface.

    urbackupclientctl browse -d rootfs
    the last(newest) backup:
    [{
    "access": 1595290141,
    "backupid": 158,
    "backuptime": 1595302492,
    "creat": 0,
    "dir": true,
    "mod": 1595288771,
    "name": "rootfs"
    }

    previous backup (older):
    ,{
    "access": 1595204059,
    "backupid": 148,
    "backuptime": 1595219417,
    "creat": 0,
    "dir": true,
    "mod": 1595204059,
    "name": "rootfs"
    }
    ]

For example we want to restore **rootfs/etc**.
Check. Display the contents of the backup:

    urbackupclientctl browse -d rootfs/etc -b 148
    [{
    "access": 1595202309,
    "creat": 0,
    "dir": true,
    "mod": 1588804354,
    "name": "ImageMagick-6"
    }
    ,{
    "access": 1595202309,
    "creat": 0,
    "dir": true,
    "mod": 1552059292,
    "name": "NetworkManager"
    }
    ,{
    "access": 1595202309,
    "creat": 0,
    "dir": true,
    "mod": 1583507894,
    "name": "X11"
    }
    and more...

We see that backup "-d rootfs/etc -b 148" is correct.


## Restore process

Start restoring to `/mnt`

Important! The / mnt folder **must be completely empty and have 777 permissions**. Otherwise, UrBackup will write the error `No permissions to create files in "/ mnt"`.

    urbackupclientctl restore-start -d rootfs/etc -b 148  -m /etc  -t  /mnt -o
    options:
    -d Path of folder/file to restore inside the backup;
     -b backupid from which to restore files/folders;
    -m Map from local output path of folders/files to a different local path (original path where files was at the client from backup was created);
    -t Map to local output path of folders/files to a different local path (path where we want to save restored files instead original path);
-o Consider other file systems when removing files/directories not in backup. (Actually I do not fully understand what this option means, if anyone knows, please write);

**Now files should be restored successfully!**

## Return the client settings back (at the production server)
After we have restored the files, the client on the bare server is no longer needed.
Remove the client on the bare server.

`sudo uninstall_urbackupclient`


Start urbackupclientbackend.service at the client (client at production server);
`systemctl start urbackupclientbackend.service`

Enable scheduled backups (options: interval for incremental/full file/image backups) at the web interface!

#### Finally we restored backup with urbackupclientctl at bare server!
