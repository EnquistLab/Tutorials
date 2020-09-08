# How to set up passwordless SSH login

## Contents

[Introduction](#introduction)  
[What are SSH keys?](#what)  
[Where are my SSH keys?](#where)  
[Generate private-public key pair on the local machine](#generate-local)  
[Copy the public key to the remote server](#copy-remote)  
[Usage](#usage)  
[Notes](#note)  

<a name="introduction"><\a>
## Introduction

This tutorial decribes how to set up SSH key-based authentication (passwordless login) between your local machine and a remote server. Once set up, this allows you to log into the remote server without entering a password. The main advantage of passwordless authentication is that it enables you to automate transactions between the two machines--for example, transferring files--without the need for the human intervention needed to enter passwords.

This tutorial assumes that both local and remote machines are Linux or Mac (which share more-or-less identical directory structures, at least with respect to SSH).

Adapted from: https://linuxize.com/post/how-to-setup-passwordless-ssh-login/.

<a name="what"><\a>
## What are SSH keys?

SSH keys come in pairs: a public key and a private key. The public key is intended to be shared with other machines. It is used by that machine to request access to your machine. Your machine then matches up that key with the corresponding private key to confirm that it is indeed authentic. If authentic, the connection is allowed; if any issues are encountered in pairing the public with the private key, the connection is refused.

<a name="where"><\a>
## Where are my SSH keys

SSH keys are specific to a machine+user combination. Your SSH keys live in your home directory in (hidden) directory `.ssh`. To list all directories in your home directory, including hidden directories and files, use the -a option ("all") with the `ls` command. I also add the -l option ("long") for detailed vertical display:

```
cd ~
ls -la
```

To list the contenst of your .ssh directory:

```
ls -al ~/.ssh/*
```

Or you can simply cd into .ssh and list the contents from there.

<a name="generate-local"><\a>
## Generate private-public key pair on the local machine

### 1. Check for existing SSH key pair

Use the following command to check if any SSH keys are present in your .ssh directory. This command only checks for public keys. The corresponding private key will have the same name, minus the `.pub` suffix:

```
ls -al ~/.ssh/*.pub
```
If there are existing keys, you can use those and skip the next step.

### 2. Generate a new SSH key pair

The following command will generate a new 4096 bits SSH key pair with your email address as a comment:

```
ssh-keygen -t rsa -b 4096 -C "your_email@domain.com"
```

Respond to the questions, noting the following:
* Press Enter to accept the default file location and file name.
* Do NOT enter a passphrase. This defeats the whole purpose of passwordless authentication.

To be sure that the SSH keys are generated you can list your new private and public keys with:

```
ls ~/.ssh/id_*
```

Output:

```
/home/yourusername/.ssh/id_rsa /home/yourusername/.ssh/id_rsa.pub
```

<a name="copy-remote"><\a>
## Copy the public key to the remote server

The final step is to copy your public key to the remote server. One approach is to copy the contents the public key (e.g., is_rsa.pub) and send it to the remote server admin. Or simply send the file. However, the simplest and most secure way is to do it yourself, as follows:

```
ssh-copy-id remote_username@server_ip_address
```

You will be prompted to enter the remote_username password:

```
remote_username@server_ip_address's password:
```

Once the user is authenticated, the public key will be appended to the remote user authorized_keys file and connection will be closed.

If for some reason the ssh-copy-id utility is not available on your local computer you can use the following command to copy the public key:

```
cat ~/.ssh/id_rsa.pub | ssh remote_username@server_ip_address "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

<a name="usage"><\a>
## Usage

One your SSH keys have been set up, you login to the remote machine as usual, except that now password is requested:

```
ssh remote_username@server_ip_address
```

Similarly, files are copied to the remote machine using the usual scp or rsync, without a password request.

<a name="notes"><\a>
## Notes
* You can have multiple SSH key pairs specific to certain machine or uses (for example, for connecting to different servers or for passwordless commits to different GitHub repositories. Not particularly hard, but the details are beyond the scope of this tutorial. For more details, try `man ssh-keygen`, also this discussion: https://stackoverflow.com/questions/2419566/best-way-to-use-multiple-ssh-private-keys-on-one-client.

