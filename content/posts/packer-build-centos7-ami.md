---
title: "Packer Build Centos7 Ami"
date: 2020-08-10T18:41:00+08:00
description: "packer基于iso制作centos7 aws ami镜像"
tags: ["packer"]
categories: ["devops"]
keywords: ["packer"]
draft: false
isCJKLanguage: true
---

在mac上用packer基于iso制作centos7 aws ami镜像

## install visualbox & packer 

visualbox: 6.0 

packer: 1.6.1

packer安装比较简单，省略安装过程


## 生成kickstart文件

先用visualbox的虚拟机手动安装一遍操作系统，根据生成的/root/anaconda-ks.cfg稍微改动生成自己要的kickstart文件

默认用户centos密码centos


```
mkdir http
cp ./centos7.8-ks.cfg ./http
```


ks.cfg
```
#version=DEVEL
# System authorization information
auth --enableshadow --passalgo=sha512
# Use CDROM installation media
cdrom
# Use graphical install
text
# Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=sda
# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8 --addsupport=zh_CN.UTF-8

# Network information
network  --bootproto=dhcp --device=eth0 --onboot=on --ipv6=auto --activate
network  --hostname=localhost.localdomain

# Root password
rootpw --iscrypted $6$coVd7Fm.ScyzVI0Z$/N26LC6qv.Pm7fOaZRww7YO4gNaQlQUyfnR1vNCigHXx./uTDegDgeOvLh7KZ0M6mfBqs.f1BEEKu9zvTRz2b1
# System services
services --enabled="chronyd"
# System timezone
timezone Asia/Shanghai --isUtc
user --name=centos --password=$6$WULB6LDSYWKEQKLC$WUsTHeafI/8QW0t9b9kvWGCMgfntdSTR.DLSqMHw4.5TiH6i92qwmBXyJ5KiotpitKrc1WWI.1vWATP4kMJz.0 --iscrypted --gecos="centos"
# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
# Partition clearing information
clearpart --none --initlabel
# Disk partitioning information
part / --fstype="ext4" --ondisk=sda --size=8191

%packages
@^minimal
@core
chrony
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

%post
echo "centos        ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers.d/centos
echo "Defaults:centos !requiretty"                 >> /etc/sudoers.d/centos
chmod 0440 /etc/sudoers.d/centos
%end
```

## aws

### 创建  s3  artisan-packer-image-s3


### 创建 import role

没有创建相关role的话，可以上次镜像文件到S3，但不能转成ami

trust-policy.json 

```
{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": { "Service": "vmie.amazonaws.com" },
         "Action": "sts:AssumeRole",
         "Condition": {
            "StringEquals":{
               "sts:Externalid": "vmimport"
            }
         }
      }
   ]
}
```

```
aws iam create-role --role-name vmimport --assume-role-policy-document "file:///jumen/work/aws/vmimport/trust-policy.json"
```


role-policy.json

用的中国大陆宁夏区域，所以s3的ARN比较特殊，以aws-cn开头,不是默认的aws

```
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket" 
         ],
         "Resource": [
            "arn:aws-cn:s3:::artisan-packer-image-s3",
            "arn:aws-cn:s3:::artisan-packer-image-s3/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket",
            "s3:PutObject",
            "s3:GetBucketAcl"
         ],
         "Resource": [
            "arn:aws-cn:s3:::artisan-packer-image-s3",
            "arn:aws-cn:s3:::artisan-packer-image-s3/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "ec2:ModifySnapshotAttribute",
            "ec2:CopySnapshot",
            "ec2:RegisterImage",
            "ec2:Describe*"
         ],
         "Resource": "*"
      },
     {
       "Effect": "Allow",
       "Action": [
         "kms:CreateGrant",
         "kms:Decrypt",
         "kms:DescribeKey",
         "kms:Encrypt",
         "kms:GenerateDataKey*",
         "kms:ReEncrypt*"
       ],
       "Resource": "*"
     },
    {
      "Effect": "Allow",
      "Action": [
        "license-manager:GetLicenseConfiguration",
        "license-manager:UpdateLicenseSpecificationsForResource",
        "license-manager:ListLicenseSpecificationsForResource"
      ],
      "Resource": "*"
    }
   ]
}
```


```
aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document "file:///jumen/work/aws/vmimport/role-policy.json"
```


## packer

centos_7.08_64_8G.json 

```
{
    "description": "Bare CentOS 7 prepped for AMI import",
    "variables": {
        "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
        "aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
        "iso_url": "https://mirrors.aliyun.com/centos/7.8.2003/isos/x86_64/CentOS-7-x86_64-Minimal-2003.iso",
        "iso_checksum": "659691c28a0e672558b003d223f83938f254b39875ee7559d1a4a14c79173193",
        "iso_checktype": "sha256",
        "ssh_username": "centos",
        "ssh_password": "centos",
        "vm_name": "CentOS7.8-Minimal",
        "disk_size": "8192",
        "memory": "4192",
        "cpus": "2"
    },
    "builders": [{
        "type": "virtualbox-iso",
        "guest_os_type": "RedHat_64",
        "iso_url": "{{user `iso_url`}}",
        "iso_checksum": "{{user `iso_checktype`}}:{{user `iso_checksum`}}",
        "ssh_username": "{{user `ssh_username`}}",
        "ssh_password": "{{user `ssh_password`}}",
        "ssh_wait_timeout": "10000s",
        "shutdown_command": "sudo -S shutdown -P now",
        "vm_name": "{{user `vm_name`}}",
        "disk_size": "{{user `disk_size`}}",
        "output_directory": "output-{{user `vm_name`}}",
        "http_directory": "http",
        "boot_command": ["<tab> text ks=http://{{.HTTPIP}}:{{.HTTPPort}}/centos7.8-ks.cfg<enter><wait>"],
        "format": "ova",
        "vboxmanage": [
            ["modifyvm", "{{.Name}}", "--memory", "{{user `memory`}}"],
            ["modifyvm", "{{.Name}}", "--cpus", "{{user `cpus`}}"]
        ]
    }],
    "post-processors": [{
        "type": "amazon-import",
        "access_key": "{{user `aws_access_key`}}",
        "secret_key": "{{user `aws_secret_key`}}",
        "region": "cn-northwest-1",
        "s3_bucket_name": "artisan-packer-image-s3",
        "license_type": "BYOL",
        "tags": {
            "Description": "packer amazon import "
        }
    }]

}
```


```
export AWS_ACCESS_KEY_ID=XXX
export AWS_SECRET_ACCESS_KEY=YYY
packer validate centos_7.08_64_8G.json 
packer build centos_7.08_64_8G.json 
```

## reference

http://work.haufegroup.io/automate-ami-with-packer/

https://github.internet2.edu/docker/packer-centos-7/blob/master/aws-centos7-base.json

https://github.com/allyunion/packer-aws-centos7/blob/master/aws-centos7-base.json

https://github.com/irvingpop/packer-chef-highperf-centos-ami/blob/master/create_base_ami/packer.json

https://github.com/allyunion/packer-aws-centos7

http://technologist.pro/devops/infrastructure-as-code-create-linux-rhelcentos-base-images-using-packer



## 问题

1. Waiting for SSH to become available


刚开始用的visulbox no-gui模式，无法定位问题，后面直接用GUI模式安装


2.

* Post-processor failed: Failed to start import from s3://artisan-packer-image-s3/packer-import-1597042426.ova: retry count exhausted. Last err: InvalidParameter: The service role vmimport provided does not exist or does not have sufficient permissions
	status code: 400, request id: 4456c33c-4174-4ee0-b7fd-6b784ab20167

aws s3 转为 ami 需要特定role

3.

An error occurred (MalformedPolicyDocument) when calling the PutRolePolicy operation: Partition "aws" is not valid for resource "arn:aws:s3:::artisan-packer-image-s3".


中国宁夏区域 特殊： 前缀为aws-cn

