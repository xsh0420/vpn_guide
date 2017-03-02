# 搭建VPN
## 新建 EC2 Instance
注意选择security group

[Connecting to Your Linux Instance Using SSH](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

```
ssh -i /path/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
```

## 安装 OpenVPN
[OpenVPN Downloads](https://openvpn.net/index.php/open-source/downloads.html)

### Linux (using RPM package)

1. Install `rpm-build`

  ```
  sudo yum install rpm-build
  sudo yum install openssl-devel
  sudo yum install lzo-devel
  sudo yum install pam-devel
  ```

2. Build your own RPM file, if you don't have it:

  ```
  rpmbuild -tb openvpn-[version].tar.gz
  ```

3. Once you have the .rpm file, you can install it with the usual

    rpm -ivh openvpn-[details].rpm

or upgrade an existing installation with

    rpm -Uvh openvpn-[details].rpm
