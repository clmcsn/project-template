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

Note: this step assumes you know how ssh works, ssh keys and ssh config file. If you don't, be aware the following guide works only if server<->cygni have mutual authentication setup!

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

In cygni then add to the `~/.ssh/config`

```
Host esat-fwd
        HostName localhost
        User gsarda
        Port 2222
```

## 5) Pin GitHub’s host key (stops “Host key verification failed”)

This step is magic, if `crontab` fails for the error above: remove the known_hosts file and create a new one. 
Add the github.com to it:
`ssh-keyscan -H github.com >> ~/.ssh/known_hosts`

You might need to add a dedicated deploy SSH key to allow `crontab` to update the mirror, but I am not sure at this moment in time.
Update on the topic might follow.

## 6) Setup the repo on cygni

Clone the repo from the mirror:

`git clone esat-fwd:~/path-to-repo/repo.git ~/path-to-clone/repo`

I suggest keeping the paths the same as in the regular server and the cygni (if some setup scripts hardcode absolute paths in generated files 🙃).
Config the new repo as (ChatGPT suggested):

```
git remote set-url origin esat-fwd:~/path-to-repo/repo.git
git fetch --prune --tags
git branch --set-upstream-to=origin/main main   # adjust if not 'main'
# optional client-side guard: disable any push URL
git remote set-url --push origin DISABLED
```

## 7) Test if the setup works (e.g., pulling a silly commit)
