# Cheat Sheets Collection

My frequently used commands and knoledges here.

## Vim

### Enable paste mode

```
:set paste
```

## Server Management

### SSH

#### Generate key pair

```
ssh-keygen -t rsa -b 2048 -f hello-secure-key.pem
```

* copy public key to target server.

```
cat hello-secure-key.pem.pub | pbcopy
rm -f hello-secure-key.pem.pub
```

* copy private key for client machine(or server).

```
cat hello-secure-key.pem | pbcopy
```

#### ssh config file template

**do not forget make its permission set to 600.**

```
mkdir -p ~/.ssh/
touch ~/.ssh/config
chmod 600 ~/.ssh/config
```

* config file example following:

```
# ~/.ssh/config
# Prevents server session disconnected.
ServerAliveInterval 15
Host *
  ServerAliveInterval 120

Host yunocchi
  HostName yunocci.fully-qualified-domain-name.com
  Port 22
  User wnoguchi
  IdentityFile ~/.ssh/hello-secure-key.pem
```
