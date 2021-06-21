# Terraforms Basic KW

มันคือเครื่องสำหรับการทำ infrastructure provisioning → ตัวที่เขียน IaC นั้นเอง ปกติถ้าทำการสร้างผ่าน UI จะต้องไปให้ถูกอันว่าสร้างอันไหนก่อนขั้นตอนเป็นอย่างไร

แต่ถ้าทำใน terraform คือแค่เขียน IaC ทั้งหมด เดี๋ยวมันจะไปสร้างให้เอง

สามารถกำหนดได้ว่าจะลง software อะไรแล้วทำ packer

# What is Difference between Ansible and Terraform ?

ทั้ง 2 เป็น infrastructure as code หมดเลย

### terraform

focus ไปที่ infrastructure provisioning แต่ก็สามารถ deploy app ได้อยู่ ส่วนใหญ่เชี่ยวชาญว่าต้องสร้างอะไรบ้าง และจะสร้างให้ไวได้อย่างไร

### ansible

focus ไปที่ configuration tool พวกการ configure, deploy application, install/update software

## Conclusion

DevOps Engineer ก็ใช้งานทั้งสองเลย ใช้งานมันร่วมกัน

# What is Terraform is used for ?

1. สร้าง infrastructure
2. เปลี่ยนแปลง infrastructure ที่เราสร้าง การ reconfigure  -→ การจัดการ infrastructure
3. สร้างให้เหมือนเดิมแต่ต่างกันที่ ENV ได้ → การ replicate infrastructure

# How does Terraform work

มันเชื่อมต่อกับพวก AWS, GCP หรืออื่น ๆ ยังไงแล้วทำงานยังไง

## Terraform Architecture

1. terraform core → ตัวนี้ต้องการ 2 อย่างเพื่อที่จะให้มันสามารถทำงานได้
    1. tf-config ที่กำหนกว่าจะสร้างอะไรบ้าง
    2. state ที่เก็บ state ของ infrastructure ว่าตอนนี้เป็นอย่างไร

     ก็คือมันเอา 2 อันด้านบนเนี่ยยัดเข้ามาเพื่อสร้าง plan → สิ่งที่มันจะต้องทำ (created, updated, destroyed) เพื่อให้ meet desired state ที่กำหนดขึ้น

2. providers → ให้เราสามารถสร้าง resource อื่น ๆ ได้

หลัก ๆ ก็คือเมื่อสร้าง plan เสร็จแล้วใช้ provider เพื่อ execute plan นั้น ๆ