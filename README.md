# git command 

## ถ้าเกิดเผลอใส่ข้อมูลลงไปใน git แล้วอยากลบทิ้ง

จากตัวอย่างนี้คือการลบ .env file

```bash
git filter-branch --index-filter "git rm -rf --cached --ignore-unmatch .env" HEAD
git push --force
```