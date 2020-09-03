# Copying files to unix servers

## Contents

[Introduction](#introduction)  
[From Linux/Mac](#linux)  
[From Windows](#windows)  

<a name="introduction"></a>
## Introduction

This gives a quick overview of some lightweight and reliable command-line tools you can used to transfer files to the BIEN servers. If you have passwordless authentication set up on the BIEN server, you can run these commands without having to enter a password, which makes them ideal for automated use in scripts. The BIEN server administrators (current Brad Boyle and Nick Outin) can help you set up passwordless authentication.

<a name="linux"></a>
## From Linux/Mac

### scp (secure copy)

Copy `myfile` to my home directory

```
scp /path/to/file/myfile boyle@ceiba.ncea.ucsb.edu 
```

Copy `myfile` to subdirectory `data` in my home directory:

```
scp /path/to/file/myfile boyle@ceiba.ncea.ucsb.edu:data/ 
```
Notes:
* If you haven’t set a passwordless SSH login to the remote machine, you will be asked to enter the user password.
* Brad or Nick can help you set up passwordless authentication.

If you are transferring a bunch of files, or multiple files insides directories and subdirectories, you should tar & compress them first, then transfer with scp and uncompress on the other end.

#### Example: copy directory `mybigdirectoryfulloffiles/` to directory `data/` in my home directory on ceiba:

Compress and transfer from the source machine:

```
tar -czf mybigdirectoryfulloffiles.tar.gz mybigdirectoryfulloffiles
scp mybigdirectoryfulloffiles.tar.gz boyle@ceiba.ncea.ucsb.edu:data/ 
```

Uncompress on the target machine:

```
cd ~/data
tar -xzf mybigdirectoryfulloffiles.tar.gz
```

### rsync

Rsync transfers files by parts. It can resume transfers when connections are interrupted. It can transfer entire directories, and is commonly used to back up files because it compares and copies only the parts that have changed. To work properly, rsync needs to be installed on both machines. All our servers have rsync installed.

If you have passwordless authentication set up, basic rsync commands look much like scp.

On the source machine:

```
rsync -a myfile boyle@ceiba.ncea.ucsb.edu:data/
```

If the transfer will take a very long time, run the command in `screen`.

If the connection is weak or intermittent, wrapping rsync in the following bash script will allow your transfer to repeating resume on its own until the transfer is complete. You must have passwordless authentication for this to work.

```
#!/bin/bash

# Parameters
SOURCEFILE="path/to/you/file"
REMOTE_USER="remote_server_user_name"
SERVER_ADDRESS="remote_server_address_or_IP"
DEST_DIR="destination_directory_no_trailing_slash"

# Main
while [ 1 ]
do
    rsync -avz --partial ${SOURCEFILE} ${REMOTE_USER}@${SERVER_ADDRESS}:${DEST_DIR}/
    if [ "$?" = "0" ] ; then
        echo "rsync completed normally"
        exit
    else
        echo "Rsync failure. Backing off and retrying..."
        sleep 180
    fi
done
```

<a name="windows"></a>
## From Windows

### pscp
If you are running PowerShell, you can use executable pscp.exe, which is roughly equivalent to scp:

```
pscp -pw password /path/to/file/myfile boyle@ceiba.ncea.ucsb.edu:data/ 
```
* This sends the password as plain text (not recommended!)

As a safer option, if you have passwordless authentication set up on the remote machine, you can do this instead:

```
pscp -i /path/to/file/myfile boyle@ceiba.ncea.ucsb.edu:data/ 
```

### Git BASH (for rsync)

I haven't used it, but apparently there is a bash-shell simulator for Windows: Git BASH (https://gitforwindows.org/). See discussion at https://superuser.com/a/915733/1136490.

Here's an example from the above superuser post:

```
bash -c "rsync -avz -P --stats --timeout=60 --exclude Downloads . my_remote_linux_computer@128.95.155.200:/media/my_remote_linux_computer/LaCie/My\\ Documents"
```
