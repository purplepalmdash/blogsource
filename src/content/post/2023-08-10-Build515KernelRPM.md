+++
title= "Build515KernelRPM"
date = "2023-08-10T21:59:03+08:00"
description = "Build515KernelRPM"
keywords = ["Technology"]
categories = ["Technology"]
+++
Using a docker instance:    

```
$ dnf config-manager --set-enabled crb
$ yum install -y vim rpm-build python3-devel elfutils-devel  openssl-devel perl-generators pesign yum-utils bc bison bpftool dwarves flex gcc gcc-c++ git-core hmaccalc kmod m4 make net-tools perl-devel gcc-plugin-devel
$ vim /etc/yum.repos.d/kernellongterm.repo
[copr:copr.fedorainfracloud.org:kwizart:kernel-longterm-5.15]
name=Copr repo for kernel-longterm-5.15 owned by kwizart
baseurl=https://download.copr.fedorainfracloud.org/results/kwizart/kernel-longterm-5.15/epel-9-$basearch/
type=rpm-md
skip_if_unavailable=True
gpgcheck=1
gpgkey=https://download.copr.fedorainfracloud.org/results/kwizart/kernel-longterm-5.15/pubkey.gpg
repo_gpgcheck=0
enabled=1
enabled_metadata=1
$ yum makecache
```

As mock, do following:    

```
$ yumdownloader --source kernel-longterm
$ ls
kernel-longterm-5.15.124-200.el9.src.rpm
$ rpm -Uvh kernel-longterm-5.15.124-200.el9.src.rpm
$ ls
kernel-longterm-5.15.124-200.el9.src.rpm  rpmbuild
```
Replace to `5.15.113-1`, then modify as following:    

```
$ cp linux-5.15.tar.xz ./rpmbuild/SOURCES/linux-5.15.tar.xz
$ vim rpmbuild/SPECS/kernel.spec
Line 135? 
# Do we have a -stable update to apply?
#%define stable_update 124
%define stable_update 113

%define rpmversion %{kversion}.%{stable_update}
%define patchversion 5.15
#%define pkgrelease 200
%define pkgrelease 1
1400 # released_kernel with possible stable updates
1401 # This is special because the kernel spec is hell and nothing is consistent
1402 #xzcat %{SOURCE5000} | patch -p1 -F1 -s
1403 #xzcat %{SOURCE5000} | patch -p1 -F1 -s
1404 git commit -a -m "Stable update"
1405 
1406 # Note: Even in the "nopatches" path some patches (build tweaks and compile
1407 # fixes) will always get applied; see patch defition above for details
1408 
1409 #git am %{patches}                                                                                                                                                                                                                                      
1410 #git am %{patches}

```

