# Linux command

using: MobaXTerm (program)

## Install zsh 

-> better playing with command line

prerequisite

```txt
git
curl or wget
```

```bash
sudo apt-get update
sudo apt-get install zsh
zsh --version  # verify installtion
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 1. Install plugin

follow below instruction

https://github.com/romkatv/powerlevel10k.git

### 2. Plugin Configuration

inside folder plugin configuration in ~/.zshrz

```txt
...
plugins=(git
        kubectl
        kubectlx)
...
```

## SSH key gen

```bash
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519 -C "<email_address>"
```

- -o : Save the private-key using the new OpenSSH format rather than the PEM format. Actually, this option is implied when you specify the key type as ed25519.
- -a: It’s the numbers of KDF (Key Derivation Function) rounds. Higher numbers result in slower passphrase verification, increasing the resistance to brute-force password cracking should the private-key be stolen.
- -t: Specifies the type of key to create, in our case the Ed25519.
- -f: Specify the filename of the generated key file. If you want it to be discovered automatically by the SSH agent, it must be stored in the default `.ssh` directory within your home directory.
- -C: An option to specify a comment. It’s purely informational and can be anything. But it’s usually filled with <login>@<hostname> who generated the key.

### To set defualt ssh-key

```bash
# inside ~/.ssh/config
IdentityFile /home/<user>/.ssh/<private_key>
```

### chmod to current user in window

```bash
icacls .\<private_key> /inheritance:r
icacls .\<private_key> /grant:r "%username%":"(R)"
```

## Remove service in linux

```bash
export service=YOUR_SERVICE_NAME; systemctl stop $service && systemctl disable $service && rm -rf /etc/systemd/system/$service && rm -rf /usr/lib/systemd/system/$service && systemctl daemon-reload && systemctl reset-failed
```

# git command

## ถ้าเกิดเผลอใส่ข้อมูลลงไปใน git แล้วอยากลบทิ้ง

จากตัวอย่างนี้คือการลบ .env file

```bash
git filter-branch --index-filter "git rm -rf --cached --ignore-unmatch .env" HEAD
git push --force
```

## If you want to insert file with text in server that there's no text editor

```bash
nano
> 'nano' command not found
cat > ./.kube/config << EOL
this is line 1 msg
this is line 2 msg
...
this is line 3 msg
EOL

cat ./.kube/config
> the msg will appear here 
```

## git using ssh

```bash
ssh-keygen
cat ~/.ssh/<public_key>
```

Then copy public key and attach to github repo or account

```bash
# connect to git repo
ssh -vT git@github.com [-i <path_to_private_key>]
```
