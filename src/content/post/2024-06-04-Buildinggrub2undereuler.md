+++
title= "Buildinggrub2undereuler"
date = "2024-06-04T14:59:03+08:00"
description = "Buildinggrub2undereuler"
keywords = ["Technology"]
categories = ["Technology"]
+++
Using repo:     

```
[root@localhost yum.repos.d]#  cat openEuler_x86_64.repo 
#Copyright (c) [2019] Huawei Technologies Co., Ltd.
#generic-repos is licensed under the Mulan PSL v1.
#You can use this software according to the terms and conditions of the Mulan PSL v1.
#You may obtain a copy of Mulan PSL v1 at:
#    http://license.coscl.org.cn/MulanPSL
#THIS SOFTWARE IS PROVIDED ON AN "AS IS" BASIS, WITHOUT WARRANTIES OF ANY KIND, EITHER EXPRESS OR
#IMPLIED, INCLUDING BUT NOT LIMITED TO NON-INFRINGEMENT, MERCHANTABILITY OR FIT FOR A PARTICULAR
#PURPOSE.
#See the Mulan PSL v1 for more details.
[openEuler-everything]
name=openEuler-everything
baseurl=http://repo.huaweicloud.com/openeuler/openEuler-20.03-LTS/everything/x86_64/
enabled=1
gpgcheck=0
gpgkey=http://repo.huaweicloud.com/openeuler/openEuler-20.03-LTS/everything/x86_64/RPM-GPG-KEY-openEuler

[openEuler-EPOL]
name=openEuler-epol
baseurl=http://repo.huaweicloud.com/openeuler/openEuler-20.03-LTS/EPOL/x86_64/
enabled=1
gpgcheck=0

[openEuler-update]
name=openEuler-update
baseurl=http://repo.huaweicloud.com/openeuler/openEuler-20.03-LTS/update/x86_64/
enabled=1
gpgcheck=0
[root@localhost yum.repos.d]# pwd
/etc/yum.repos.d
# yum makecache
```
Install package:    

```
yum install -y elfutils-libelf-devel gcc gnu-efi gnu-efi-devel openssl-devel make git rpm-build
yum install -y rpmdevtools* tree
 rpmdev-setuptree
[root@localhost ~]# ls
anaconda-ks.cfg  rpmbuild
cd ~/rpmbuild/SOURCES
wget http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz
cd ~/rpmbuild/SPECS
vim hello.spec
```
content is:     

```
Name:     hello
Version:  2.10
Release:  1%{?dist}
Summary:  The "Hello World" program from GNU
Summary(zh_CN): GNU Hello World program
License:  GPLv3+
URL:      http://ftp.gnu.org/gnu/hello
Source0:  http://ftp.gnu.org/gnu/hello/%{name}-%{version}.tar.gz

BuildRequires:  gettext
Requires(post): info
Requires(preun): info

%description
The "Hello World" program, done with all bells and whistles of a proper FOSS
project, including configuration, build, internationalization, help files, etc.

%description -l zh_CN
The Hello World program contains all parts required by the FOSS project, including configuration, build, i18n, and help files.

%prep
%setup -q

%build
%configure
make %{?_smp_mflags}

%install
make install DESTDIR=%{buildroot}
%find_lang %{name}
rm -f %{buildroot}/%{_infodir}/dir

%post
/sbin/install-info %{_infodir}/%{name}.info %{_infodir}/dir || :

%preun
if [ $1 = 0 ] ; then
/sbin/install-info --delete %{_infodir}/%{name}.info %{_infodir}/dir || :
fi

%files -f %{name}.lang
%doc AUTHORS ChangeLog NEWS README THANKS TODO
%license COPYING
%{_mandir}/man1/hello.1.*
%{_infodir}/hello.info.*
%{_bindir}/hello

%changelog
* Thu Dec 26 2019 Your Name <youremail@xxx.xxx> - 2.10-1
- Update to 2.10
* Sat Dec 3 2016 Your Name <youremail@xxx.xxx> - 2.9-1
- Update to 2.9
```
Building:     

```
rpmbuild -ba hello.spec 
# tree ~/rpmbuild/*RPMS
/root/rpmbuild/RPMS
└── x86_64
    ├── hello-2.10-1.x86_64.rpm
    ├── hello-debuginfo-2.10-1.x86_64.rpm
    └── hello-debugsource-2.10-1.x86_64.rpm
/root/rpmbuild/SRPMS
└── hello-2.10-1.src.rpm

```

Download the source code and install:     

```
rpm -ivh grub2-2.02-73.oe1.src.rpm 
cd rpmbuild/SPECS/
yum install -y  bison bzip2-devel dejavu-sans-fonts device-mapper-devel flex freetype-devel gettext-devel help2man libusb-devel ncurses-devel pesign rpm-devel texinfo xz-devel
$ vim /xxx/xx/grub.macros
%{4}./grub-mkimage -O %{1} -o %{2}.orig                         \\\
        -p /EFI/%{efi_vendor} -d grub-core ${GRUB_MODULES}      \
%{4}./grub-mkimage -O %{1} -o %{3}.orig                         \\\
        -p /EFI/BOOT -d grub-core ${GRUB_MODULES}
```
Change `.orig` to ``, so you could mkimage, and also remove the latter lines which use `pesign`
