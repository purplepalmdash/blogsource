+++
title = "WorkingTipsOnRPMBuild"
date = "2017-06-21T11:33:40+08:00"
categories = ["Linux"]
keywords = ["Linux"]
description = "WorkingTipsOnRPMBuild"

+++
### Prerequisites
From docker images Centos:6.6.    

### Steps
#### Environment Preparation
Start the docker instance:    

```
$ sudo docker run -it centos:6.6 /bin/bash
```
Install the dev environment(C):  

```
# yum install -y rpm-build rpmdevtools vim gcc tar openssh-clients

```
Create the macro for rpmbuild, and setup the rpm build tree:   

```
# vim /root/.rpmmacros
%_topdir    /root/rpmbuild
# rpmdev-setuptree
```

#### C Project
Refers to:    

[https://blog.packagecloud.io/rpm/rpmbuild/packaging/2015/06/29/building-rpm-packages-with-rpmbuild/](https://blog.packagecloud.io/rpm/rpmbuild/packaging/2015/06/29/building-rpm-packages-with-rpmbuild/)    

#### Verification
Using a new docker instance, then you could verify your rpm installation and
uninstallation.   

### On Building wget
Refers to:   

[http://www.winseliu.com/blog/2016/04/04/rpm-build-your-package/](http://www.winseliu.com/blog/2016/04/04/rpm-build-your-package/)    

```
# sudo docker run -it centos:6.6 /bin/bash
# yum install -y which tree lrzsz tar gcc rpm-build which tree lrzsz tar gcc gnutls gnutls-devel
# mkdir -p /home/mywget
# cd /home/mywget
# mkdir BUILD RPMS SOURCES SPECS SRPMS
# cd /home/mywget/SOURCES
# wget wget's source code from ftp.gnu.org
eg. http://ftp.gnu.org/gnu/wget/wget-1.18.tar.gz
# rpmbuild --showrc
# rpm --eval "%{_topdir}"
# grep -i _topdir /usr/lib/rpm/rpmrc /usr/lib/rpm/redhat/rpmrc /usr/lib/rpm/macros /usr/lib/rpm/redhat/macros  | less
/usr/lib/rpm/macros:%_builddir          %{_topdir}/BUILD
/usr/lib/rpm/macros:%_rpmdir            %{_topdir}/RPMS
/usr/lib/rpm/macros:%_sourcedir         %{_topdir}/SOURCES
/usr/lib/rpm/macros:%_specdir           %{_topdir}/SPECS
/usr/lib/rpm/macros:%_srcrpmdir         %{_topdir}/SRPMS
/usr/lib/rpm/macros:%_buildrootdir              %{_topdir}/BUILDROOT
/usr/lib/rpm/macros:%_topdir            %{getenv:HOME}/rpmbuild
# cat ~/.rpmmacros 
%_topdir /home/mywget/rpm
``` 
Edit the `SPECS/wget.spec`:    

```
# this is a sample spec file for wget
  
%define _topdir /home/mywget
%define name    wget
%define release 2
%define version 1.18

%define _unpackaged_files_terminate_build 0

Summary:   GNU wget
License:   GPL
Name:      %{name}
Version:   %{version}
Release:   %{release}
Source:    %{name}-%{version}.tar.gz
Prefix:    /usr
Group:     Development/Tools

%description
The GNU wget program downloads files from the Internet using the command-line.

%prep
%setup -q

%build
./configure --sysconfdir=/etc
make

%install
make install prefix=$RPM_BUILD_ROOT/usr # or use DESTDIR=$RPM_BUILD_ROOT

%post
echo "hello world"

%preun
echo "bye"

%clean
rm -rf $RPM_BUILD_ROOT

%files
%defattr(-, root, root)
/usr/bin/wget
```
Build the package via:    

```
# rpmbuild -vv -bb --clean SPECS/wget.spec 
# tree
.
├── BUILD
├── BUILDROOT
├── rpm
│   ├── BUILD
│   ├── BUILDROOT
│   ├── RPMS
│   ├── SOURCES
│   ├── SPECS
│   └── SRPMS
├── RPMS
│   └── x86_64
│       ├── wget-1.18-2.x86_64.rpm
│       └── wget-debuginfo-1.18-2.x86_64.rpm
├── SOURCES
│   └── wget-1.18.tar.gz
├── SPECS
│   └── wget.spec
└── SRPMS
```
Verify:    

```
#  sudo docker run -it -v /home/dash/dockerv:/mnt centos:6.6 /bin/bash
# yum localinstall -y wget-1.18-2.x86_64.rpm
```

Binary files packaging using rpm is also very easy to adapt.   
