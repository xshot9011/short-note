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