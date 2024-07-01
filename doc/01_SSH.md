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
