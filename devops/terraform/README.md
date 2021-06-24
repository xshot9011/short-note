# Terraforms Basic KW

Photo: https://eadn-wc03-4064062.nxedge.io/cdn/wp-content/uploads/2020/11/terraform-config-files-e1605834689106.png
is_finished: No
is_subpage: No

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

# Terraform Basic

terraform จะอ่านทั้งหมด *.tf อยู่แล้ว → ไม่สนชื่อไฟล์นั้นเลย

## File type and usage

### *.tf

เอาไว้เก็บ IaC ที่เขียนขึ้นมา

### terraform.tfvars หรือ *.auto.tfvars

เมื่อทำการสร้างโปรเจ็คเกี่ยวกับ [gitignore.io](http://gitignore.io) จะเห็นว่ามี *.tfvars → เป็นไฟล์ที่เอาไว้เก็บ secret หรือตัวแปรที่ไม่มีค่า default

กรณี secret → เช่น access_key, private_key หรือใช้เพื่อให้ไม่ต้องป้อนค่าอีกตอนรัน terraform apply, plan, destroy

## การประกาศตัวแปร

```scheme
variable "<variable_name>" {
	type        = <type>
	default     = <value>
	description = "<description>"
}
```

### Example

```scheme
# there are string, number, bool
variable "var1" {
	type        = string
	default     = "Hello World !"
	description = "This is the var1 description"
}

# var.var3["key"]
variable "var3" {
	type        = map(string)
	default     = {
		key = "value"
	}
}

# only the same type
# var.var4[index]
# element, slice are allowed -> google
variable "var4" {
	type        = list
	default     = [1, 2, 3, 4, 5]
}
```

list → ชนิดเดียวกัน
tuple → ต่างชนิดได้
object → เหมือน map(type) แต่มีได้หลาย type

> ที่ควรใช้หลัก ๆ ก็คือ string, number, bool, map, list