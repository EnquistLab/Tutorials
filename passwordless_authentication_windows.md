# How to set up passwordless SSH login from Windows

## Contents

[Introduction](#introduction)  
[What are SSH keys?](#what)  
[How do I communicate with the remote machine??](#how)  
[Generate private-public key pair on the local machine](#generate-local)  
[Copy the public key to the remote server](#copy-remote)  
[Usage](#usage)  
[Notes](#note)  

<a name="introduction"></a>
## Introduction

This tutorial decribes how to set up SSH key-based authentication (passwordless authentication) between your local Windows machine and a remote Linux server. Passwordless authentication allows you to log into the remote server without entering a password. The main advantage of passwordless authentication is that it enables you to automate transactions between the two machines--for example, transferring files--without the need for the human intervention needed to enter passwords. Also, it is becoming more and more common to disable password-based authentication for security reasons; in such situations, passwordless authentication is your only option.

This tutorial assumes that the local machine is running Windows. If you are logging in to the remote server from a local machine running Mac OS or Linux, see alternative tutorial, passwordless\_authetication\_mac.

Adapted from https://stackoverflow.com/a/71422513/2757825.

<a name="what"></a>
## What are SSH keys?

SSH keys come in pairs: a public key and a private key. A private-public key pair is stored on your local machine, and the public key is shared with one or more remote machines. The public key is used by the remote machine to request a connection to your machine. The local machine then attempts to match the public key from the remote machine with a corresponding public key stored locally to confirm that the request is indeed authentic. If authentic, the connection is allowed; if the keys cannot be matched, the connection is refused.

<a name="how"></a>
## How do I communicate with the remote machine?

To communicate with the remote machine, you will need a Linux command line terminal emulator. Many options are available, some for free, some for a price. Windows Terminal (free) is probably your best option. It used to be a bare-bones, barely functional application, but has recently been upgraded. 

One of many sites that describe Windowns terminal emulators: https://www.maketecheasier.com/best-terminal-emulators-for-windows/

<a name="#generate-local"></a>
## Create the SSH key on your machine

First of all, we need to create a new key on the Windows machine, where we start the connection. Open you terminal application and issue the following command:

```
ssh-keygen -t rsa
```

Don't change the default path or try to remember where you saved the key, it will be used for the next command. Press enter another two times to avoid using a passphrase (you **don't** want to have to enter a passphrase--it will prevent you from running automated connections via scripts).

After that, if you haven't change the default path, the key will be created into `{USERPROFILE}\.ssh\id_rsa.pub`.

<a name="#copy-remote"></a>
## Copy the public key to the remote server

Now, you can usually use the command ssh-copy-id for installing the key on the remote host, but unfortunately this command is not available on Windows, so we have to install it using this command:

```
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh {REMOTE_HOST} "cat >> .ssh/authorized_keys"
```

or if your key is not in the default path:

```
type {RSA_KEY_PATH} | ssh {REMOTE_HOST} "cat >> .ssh/authorized_keys"
```

and replace the `{RSA_KEY_PATH}` with your RSA path.

Replace `{REMOTE_HOST}` with the remote host IP or Name (e.g., boyle@ceiba.nceas.ucsb.edu), launch the command, insert the password if required, and you're done!

<a name="#copy-remote-alt"></a>
## Copy the public key to the remote server (alternative method)

The above method for copying over your SSH key to the remote server only works if password-based login to the remote machine is allowed. If password-based login has already been disabled, then you will need to find you public SSH key on your local machine (see above for potential locations), copy the contents of the public key, paste it into an email and send to the administrator of the remote Linux machine. The administrator will then paste the public key to the `authorized_keys` file in you `.ssh/` directory on the remote machine.

