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

## grep command

grep <word>

```bash
# option
## -c count the number of word
## -w match all word
## -n show line number 
## -i ignore case matching
## -l Displays list of a filenames(match gicen pattern) only .
grep -l 'linux' <filename>, ...  # find file that contain "linux"
# ^ the starting of the line, & the end of line
grep '^kbuet'  # find kbuet at the beginning of the line
grep '^&' # find the blank line
# using . mean any char if you want to find . use \.
grep 'k...t'  # find kxxxt
# . is for 1 char; [<src_char[-dest_char]>...] is for set of matched char
grep '[Gg][Gg][Ee][Zz]' # GGEZ, GgEZ, GGeZ, ... 
grep 'LAB[1-9]_[A-Z]'  # lab[1-9]_[FOLLOW WITH LETTER A-Z]
grep 'LAB[1-9]_[^A-Z]' # lab[1-9]_[NOT FOLLOW WITH LETTER A-Z]
# match all entire word
grep '\<k.l\>' # match all word start with k end with l
# character classes
## [[:alnum:]] – Alphanumeric characters.
## [[:alpha:]] – Alphabetic characters
## [[:blank:]] – Blank characters: space and tab.
## [[:digit:]] – Digits: ‘0 1 2 3 4 5 6 7 8 9’.
## [[:lower:]] – Lower-case letters: ‘a b c d e f g h i j k l m n o p q r s t u v w x y z’.
## [[:space:]] – Space characters: tab, newline, vertical tab, form feed, carriage return, and space.
## [[:upper:]] – Upper-case letters: ‘A B C D E F G H I J K L M N O P Q R S T U V W X Y Z’.
# min max or len
## {min}, {min,max}
grep '[[:digit:]]{1,3}'  # find the digit with have len 1-3 ex. 000, 111, 929
grep '[vV]{3}' # find set of [vV] with len 3  vVv or vvv or VVV ...
# u can use 'OR' with \|
grep '[[:digit:]]{1,3}\|word2'
# and with grep ... | grep ... 
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
