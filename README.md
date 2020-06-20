# Katsuna scripts

## Initialising various Katsuna environments from scratch

### 1. Client side (local):

* Acquire root access to the new server
* Install your public ssh key into the server's root account
* Set these settings into your **~/.ssh/config**:
```
Host katsuna-SERVER
    HostName XXX.XXX.XXX.XXX
    User root
    IdentityFile ~/.ssh/ID_YOUR_PRIVATE_KEY
    ForwardAgent yes
```

OR if you are using the same key for all Katsuna servers

```
Host *.katsuna.com
    User root
    IdentityFile ~/.ssh/ID_YOUR_PRIVATE_KEY
    ForwardAgent yes
```

**ForwardAgent** is a very important setting. This allows us to pull and push between the server and github without a new specific github user for the server, and, in the same time, protects us from keeping our private ssh keys onto the server.

For more details, check here: [Using SSH agent forwarding](https://developer.github.com/guides/using-ssh-agent-forwarding/#using-ssh-agent-forwarding)

* **Re-login** into the server:
```
ssh katsuna-SERVER
```

### 2. Server side (remote):

* Type these commands into the server
```
apt-get install -y git-core
git clone git@github.com:Katsuna/server_scripts /usr/local/bin/server_scripts
/usr/local/bin/server_scripts/katsuna-install
```

You should now re-login and the Katsuna installation scripts should be available!
```
