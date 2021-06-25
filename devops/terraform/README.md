# Terraforms KW

Photo: https://eadn-wc03-4064062.nxedge.io/cdn/wp-content/uploads/2020/11/terraform-config-files-e1605834689106.png
is_finished: Yes
is_subpage: No

มันคือเครื่องสำหรับการทำ infrastructure provisioning → ตัวที่เขียน IaC นั้นเอง ปกติถ้าทำการสร้างผ่าน UI จะต้องไปให้ถูกอันว่าสร้างอันไหนก่อนขั้นตอนเป็นอย่างไร

แต่ถ้าทำใน terraform คือแค่เขียน IaC ทั้งหมด เดี๋ยวมันจะไปสร้างให้เอง

สามารถกำหนดได้ว่าจะลง software อะไรแล้วทำ packer

# What is Difference between Ansible and Terraform ?

ทั้ง 2 เป็น infrastructure as code หมดเลย

### terraform

focus ไปที่ infrastructure provisioning แต่ก็สามารถ deploy app ก็ได้อยู่ ส่วนใหญ่เชี่ยวชาญว่าต้องสร้างอะไรบ้าง และจะสร้างให้ไวได้อย่างไร

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

## Output

terraform เก็บทุก attribute ของ resource ที่มีการสร้างขึ้น → สามารถเรียกดูได้

```scheme
resource "aws_instance" "ec2_instance" {
  ami           = "ami-0d058fe428540cd89"
  instance_type = "t2.micro"
}

output "public_ip_address" {
	value = aws_instance.ec2_instance.public_ip
}
```

การเข้าถึงก็เหมือน attribute ของ resource

```scheme
<resource_type>.<resource_name>.<attribute>
```

## State

ใช้ในการเก็บ state ของ infrastructure

terraform เก็บ state ไว้ใน terraform.tfstate และ terraform.tfstate.backup

ใช้ในการติดตามการเปลี่ยนของ infrastructure → สามารถเก็บไว้ใน version control ได้เพื่อใช้งานร่วมกัน → ระวังเรื่องการ conflict, race condition→ มี backend เข้ามาช่วยจัดการ

- s3 → ใช้ dynamoDB ใน locking mechanism
- consul
- terraform enterpise

### backend.tf

```scheme
terraform {
	backend "s3" {
		<configuration>	
	}
}
```

ต่อไปเวลามีการสั่ง terraform apply, destroy ไม่ชัวร์ว่า plan ด้วยหรือป่าว ก็จะมีการ require lock → เพื่อไม่ให้คนเข้าไปใช้ share resource พร้อมกัน

## Data source

ข้อมูลที่เปลี่ยนแปลงเรื่อย ๆ ตามวันเวลา เช่น เวอร์ชั่นของ AMI, availability zone, list of public ip เพื่อใช้สำหรับการ filter traffic

```vhdl
data "data_type" "data_name" {
	<configuration>
}

resource "aws_instance" "ec2_instance" {
  ami           = "ami-0d058fe428540cd89"
  instance_type = "t2.micro"
	<data_type_id>  = <data_type>.<data_name>.<id>
}

```

## Module

มีคนสร้างไว้ให้แล้วหรือสร้างเองเพื่อให้จัดการโปรเจ็คได้ดีขึ้น → การแยก env แล้วค่อยส่งตัวแปรเข้าไปใน module นั้นแทน

```vhdl
module "<module-name>" {
	source = "<URL or relative path to module>"
	<list of arguments>
}
```

สามารถส่งค่าข้าม module ได้เช่นกัน

```python
# file dev.tf
module "main_vpc" {
  source     = "../modules/vpc"
  ENV        = "dev"
  AWS_REGION = var.AWS_REGION
}

module "instances" {
  source         = "../modules/instances"
  ENV            = "dev"
  VPC_ID         = module.main_vpc.vpc_id
  PUBLIC_SUBNETS = module.main_vpc.public_subnets
}
```

ถ้าอยากได้ output จาก module นั้น ๆ 

```vhdl
output "<output_name>" {
	value = module.<module_name>.<output_name>
}
```

## version.tf

สามารถระบุ version ที่ต้องการได้

```python
terraform {
	required_providers {
		aws = {
			version = ">=3.26.0"
		}
	}
  required_version = ">= 0.12"
}
```

# Terraform Advanced Usage

## Conditionals

```vhdl
condition ? true : false
```

```vhdl
resource "aws_instance" "ec2_instance" {
  ami           = "ami-0d058fe428540cd89"
  instance_type = "t2.micro"

	vpc_id        = "${var.ENV == 'prod' ? var.test_vpc_id : var.prd_vpc_id}"
	# or
	vpc_id        = var.ENV == "prod" ? var.test_vpc_id : var.prd_vpc_id
}
```

## Built-in Functions

ชื่อ function จะคล้าย python มาก

```python
file(file_name)  # read content of file
basename(path)  # return file name

coalesce(string1, string2, ...)  # return fiest non empty string
coalescelist(list1, list2, ...)  # return first non empty list

element(list, index)  # return single element from given index
index(list, elem)  # return index from given element
lookup(map_var, key, [default])  # return map_var[key] if exist -> else default

format(format, args, ...)  # format("I have %o3d pens", 10)
join(string_to_join, list)  # " ".join(array) as python
split(string_to_split, list)  # "a,b,c,d,e".split(",")
merge(map1, map2)  # union map1 and map2
substr(string, offset, lenght)  #

lower(string)  # return all lowercase
upper(string)  # return all uppercase
replace(strign, search, replace)  # replace("abcd", "a", "f+") -> f+bcd

list(item1, item2, ...)  # to create list
map(key1, value1, key2, value2, ...)  # to create map
```

## For and Loop

เหมาะกับพวก repeat nested block

```python
# normal loop
[for char in ["a", "b", "c"]: upper(char)]  # return ["A", "B", "C"]
# dict loop
[for key, value in var.map_vat: value => key]  # swap key and value
```

```python
resource "aws_security_group" "secgroup" {
  name = "secgroup"
	
	dynamic "ingress" {  # create ingress nest block 
		for_each = [22, 443]
		content {
			from_port = ingress.value
			to_port   = ingress.value
			protocol  = "tcp"	
		}
	}
}
```

## State Manipulation

ใช้ในการจัดการ state file

```bash
terrafrom state list
terrafrom state mv  # move item in the sate or rename
terrafrom state pull  # pull current state and stdout the output
terrafrom state push  # overwrite state by pushing local file to state
terrafrom state replace-provider  # replace a provider in state
terrafrom state rm <state1> <state2> # remove item in state
terrafrom state show
```

### Why to use state manipulation ?

- upgrading between version ex. version 0.11 → 0.12 → 0.13
- rename resource without recreating it
- change a key in for_each
- stop managing the resource but you don't want to destroy → terrafrom state rm
- show attribute in the resource

## Object

```vhdl
variable "condition" {
	type        = list(
		object({
			field  = string
			values = list(string)
		})
	)
	default     = []
	description = "object for condition configuration"
}

resource "aws_lb_listener_rule" "alb_rule" {
	...
	dynamic "condition" {
		for_each = var.condition
		content {
			dynamic "host_header" {
				for_each = condition.value.fields == "host-header" ? [1] : [0]
				content {
					values = condition.value.values	
				}
			}
		}
	}
}
```