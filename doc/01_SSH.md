# SSH

## What is SSH?

SSH (Secure Shell) is a protocol to send commands to a computer over a network in a safe/secure way.
In other words, SSH is used to control a machine from remote. 
As SSH is based on cryptography, the user needs to authenticate to send/receive commands. There are different ways to do so:
 1. Use user and password (slow and not suggested)
 2. SSH RSA keys
 3. SSH Certificate (need to be issued by the system administration of the institution server farm)

## SSH RSA keys

Please search and learn what an ssh key is and what is its purpose (you also need it for GitHub!)
In short you can do (if your pc is a Linux system):

```bash
ssh-keygen
cat ~/.ssh/id-rsa.pub | ssh r<number>@ssh.esat.kuleuven.be "cat >> ~/.ssh/authorized_keys"
```

Extra: what does the last command do?

### ProxyCommand in VS Code

To bypass a server-client access point you can configure a new host in the `.ssh/config` file. This file as its own sintax and you can search for more info online.
Below an example:

```bash
Host esat
    HostName ssh.esat.kuleuven.be
    User <user>

Host eridani5
    HostName eridani5.esat.kuleuven.be
    User <user>
    ProxyCommand ssh -q -W %h:%p esat
```

Note: this will work only if the public key is the default `~/.ssh/id_rsa.pub`
