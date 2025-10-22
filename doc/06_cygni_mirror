# How to sync a GitHub repository when working on cygni

## 1) Create a mirror on a normal *server with internet access*

`git clone --mirror git@github.com:ORG/REPO.git ~/path-to-repo/repo.git`

Note: .git suffix is needed to indicate the repo is a mirror and not a normal clone

```bash
git -C ~/git/repo.git config fetch.prune true
git -C ~/git/repo.git config fetch.pruneTags true
```

## (facultative) 2) Make the mirror read-only and enforce security

This step is required to make sure nothing leaves the server and goes to GitHub by mistake

- Make the mirror read-only

`git -C ~/git/repo.git config receive.denyAll true`

- Activate the update hook (bash scripts automatically called upon specific `git` actions)

```bash
cat > ~/git/repo.git/hooks/update <<'EOF'
#!/bin/sh
echo "Read-only mirror: pushes are disabled." >&2
exit 1
EOF
chmod +x ~/git/repo.git/hooks/update
```
This creates an `update` script that raises an error whenever you try to update the mirror

## 3) Sudo-less automatically sync the mirror with GitHub

This can be done through either `crontab` or web servers (e.g. with python flask).
Web server setup is not supported yet, but it would be better than `cron` jobs, because `cron` jobs are scheduled every X minutes. 
This is not practical if you are actively debugging and updating the repo across servers...

To set up a crontab job *on the server with the mirror*, open a pseudo vim editor with

`crontab -e`

Add the following line to the file:

`*/5 * * * * git -C $HOME/path-to-repo/repo.git remote update --prune && git -C $HOME/path-to-repo/repo.git gc --auto`

Note that it is better to substitute the $HOME variable with the actual home path. 
Crontab will open a new terminal to run these commands; the same env as you would have on a regular terminal is not guaranteed.

## 4) Setup remote forwarding from cygni to the normal server

Add the following entry to your `~/.ssh/config` file (or ask ChatGPT to provide you with the equivalent set of commands ;))

```
Host cygni-gw
        HostName cygni-gw.esat.kuleuven.be
        User gsarda

Host cygni-fwd
        HostName tauceti5.cygni
        ProxyCommand ssh -q -W %h:%p cygni-gw
        User gsarda
        RemoteForward 2222 localhost:22
```

