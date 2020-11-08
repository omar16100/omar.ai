---
title: "Install fstransform On CentOS"
date: 2020-11-08T15:13:32+08:00
draft: False
---

Install fstransform On CentOS

Download latest epel-release rpm from

<http://download-ib01.fedoraproject.org/pub/epel/7/x86_64/>

```Shell
sudo wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-12.noarch.rpm
```

Install epel-release rpm:

```Shell
rpm -Uvh epel-release-7-12.noarch.rpm
```

Install fstransform rpm package:

```Shell
yum install fstransform
```

## References

* [CentOS Documentation](https://centos.pkgs.org/7/epel-x86_64/fstransform-0.9.3-5.el7.x86_64.rpm.html)
