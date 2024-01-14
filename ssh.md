# SSH CONTENTS

1. [install SSH](#1-install-ssh)
2. [Connect to the Server](#2-connect-to-the-server)
3. [Client .ssh Directory Structure](#3-client-ssh-directory-structure)
4. [Client Config File Structure](#4-client-config-file-structure)
5. [Create SSH Keys to Connect to Server without a Password](#5-create-ssh-keys-to-connect-to-server-without-a-password)
6. [SSH Agent](#6-ssh-agent)
7. [Server side SSH](#7-server-side-ssh)
8. [Running Commands on Remote Servers with SSH](#8-running-commands-on-remote-servers-with-ssh)
9. [Using sshpass](#9-using-sshpass)

## 1 Install SSH
Check if ssh is installed on client.
```cli
    which ssh
```
Install ssh.
```cli
    sudo apt search openssh-client

    sudo apt install openssh-client
```

## 2 Connect to the Server
Connect to server. You will be promted for password first time connecting.
```cli
    ssh <user-name>@<ip-address or fqdn>
```

Connect to server using a different port than 22.
```
    ssh -p 2222 <username>@<ip-address or fqdn>
```

Connect to server that is configured in the config file.
```
    ssh server1
```

To follow an ssh connection inside /var/log/.
```cli
    tail -f auth.log
```

Get verbose information while connecting to the server.
The command will also displey the keys that are being used.
```
    ssh -v <username>@<ip_address or fqdn>
```

## 3 Client .ssh Directory Structure
```
.ssh
    known_hosts
    config
```

## 4 Client Config File Structure
Configure the clients config file.
```
    Host server1
        Hostname 172.105.37.26
        Port 22
        User root
        IdentityFile ~/.ssh/server1_key

    Host server2
        Hostname 172.105.37.25
        Port 2222
        User ciaran
        IdentityFile ~/.ssh/server2_key

    # adding the identity file is optional
```

## 5 Create SSH Keys to Connect to Server without a Password
generate a default ssh key. # will create a default key in your .ssh directory.
will also prompt you for an optional passphrase for the key.
will create a private and public key.
private keys should have user permissions only.
```
    ssh-keygen
```

Connect to server to add the public ssh key.
```
    ssh <username>@<ip_address or fqdn>
```

Create a .ssh directory in the home dirctory if it does not exist.
```
    mkdir .ssh
```

Inside the .ssh directory create a file called authorized_keys.
```
    touch authorized_keys
```
Paste the public key inside the authorized_keys file.
the next time you connect to this server you will not require a password.

Copy the public ssh key to the remote server.
```
    ssh-copy-id -i ~/.ssh/ip_rsa.pub <username>@<ip_address or fqdn>
```

Make custom ssh keys for each remote server.
```
    ssh-keygen -t ed25519  -C "acme"
```
* will be prompted for a filename.
* should give key a meaningfull filename.
* will be prompted for a passphrase for the key.
* the comment will be at the end of the public key.
* the comment can be deleted from the key and it will still work.

## 6 SSH Agent
Use an ssh agent so you only have to enter pasphrasses one time in a terminal session.
```
    ps aux | grep ssh-agent
```

If the ssh-agent is not running start the agent as follows.
```
    eval "$(ssh-agent)"
```

Add the private key to the ssh-agent. You will be prompted for a passphrase one time.
```
    ssh-add ~/.ssh/acme_id_ed25519
``` 

## 7 Server side SSH
Check if sshd is installed on the server. sshd is ssh daemon.
```
    which sshd
```

If installed check if it is running.
```
    systemctl status sshd
```

Structure of the remote servers /etc/ssh/ directory.
```
/etc/ssh/
    ssh_config
    ssh_config.d
    sshd_config
    sshd_config.d
```
* Contains the fingerprint public and private keys for the server.
* Also contains a ssh_config file which is overridden by the clients ssh config file.
* The sshd_config file is used to configure the servers ssh service.

Configure the sshd_config file.

You can change the default port number to something other than 22.
```
#Port 22 
```

You can set if you want to allow root login but prohibit password loging.
```
#PermitRootLogin prohibit-password
```

You can set if you want to allow root login with password.
```
#PermitRootLogin yes
```

You can change password authentication to disallow password authentication.
```
#PasswordAuthentication yes
```

After changing the sshd_config file restart the sshd service.
```
    systemctl restart sshd
```

The .ssh directory should contain an authorized_keys file for client public keys to be added to.
```
.ssh
    authorized_keys
```

## 8 Running Commands on Remote Servers with SSH
```sh
#!/bin/bash

# takes an argument of usernames file and creates them on the remote server

HOST="192.168.8.10"
USERNAME="root"


if [ -f "$1" ]; then
    readarray -t users < "$1"

    cmd_str=""
    for user in "${users[@]}"; do
        echo "$user"
        cmd_str="${cmd_str} useradd -m ${user};"
    done
    echo $cmd_str
    ssh "${HOST}@${USERNAME}" "$cmd_str"
else
    echo "$1 file does not exist"
fi
```

## 9 Using sshpass
Install sshpass
```cli
sudo apt install sshpass
```

Use sshpass with -p option to login to server with server password.
```cli
sshpass -p serverpassword ssh username@hostname
```

Ignore the server ssh fingerprint prompt.
```cli
sshpass -p serverpassword ssh -o StrictHostKeyChecking=no username@hostname
```

Use sshpass with -f option to login using a file. The server password must be on the
first line of the file.
```cli
echo servaerpassword > pass_file
chmod 0400 pass_file
sshpass -f pass_file ssh username@hostname
```

Ignore the server ssh fingerprint prompt.
```cli
echo servaerpassword > pass_file
chmod 0400 pass_file
sshpass -f pass_file ssh -o StrictHostKeyChecking=no username@hostname
```

Use sshpass -e option to pass server password as an enviromental variable.
```cli
SSHPASS='serverpassword' sshpass -e ssh username@hostname
```

Ignore the server ssh fingerprint prompt.
```cli
SSHPASS='serverpassword' sshpass -e ssh -o StrictHostKeyChecking=no username@hostname
```

Use sshpass with scp.
```cli
sshpass -p <server-password> scp <filename> username@hostname:/home/server1/
```

Using sshpass with scp where the servers ssh password is in a pass file.
```cli
sshpass -f <password-file> scp <filename> username@hostname:/home/server1/
```

Using sshpass with scp to copy a file from the remote server where the ssh password is in a pass file.
```cli
sshpass -f "pass_file" scp username@hostname:/home/server1/server_file.txt /home/ciaran/
```