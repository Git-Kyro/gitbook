## Terraform

通过声明式配置文件预览虚拟机、容器、存储空间和网络等资源，从源代码控制资源，从而为测试和生产环境保持理想的与配置状态

创建main.tf文件
terraform{} terraform 版本/ucloud插件版本/backend
provider "ucloud" {} 包含账户的public_key/private_key/project_id/云商区域，
也可以多个provider实例，分别定位不同的区域，可以在一组配置文件中同时操作不同区域、不同帐号的资源

真正定义云端基础设施代码分为三部分，data、resource、output
data: 利用data模型对云商进行查询，例如查询不同区域的镜像id，安全在的id
resource: 定义实例资源，例如实例配置信息，弹性ip，第一次开机初始化脚步
output: 自定义我们希望创建完成后需要输出的信息

data "ucloud_security_groups" "default"{}
resoure "ucloud_instance" "web"{}
resource "ucloud_eip" "web-eip"{}
output “eip”{}

terraform init 分析代码中的provider，并尝试下载provider插件到本地
terraform plan 预览代码即将发生的变更
terraform apply 根据代码计算出的目标状态和当前状态的差异来计算变更计划 
terraform destroy 清除资源

状态管理
terraform每次执行基础设施变更后的状态信息保存在一个状态文件中，默认清卡西保存在当前目标下的terraform.tfstate文件里面

tfstate是明文的，例如定义的rootPassword

tfstate管理方案-Backend，远程存储接口
Terraform remote backend分为两种
标准：支持远程章台存储和状态锁
增强：在标准的基础上支持远程操作（执行plan、apply等操作）

consul 用来充当terraform backend存储
当多人操作给你tfstate文件加锁，防止多人操作产生的偏差

terraform workspace隔离