+++
title= "OnBuildxpumanager"
date = "2023-12-13T09:33:02+08:00"
description = "OnBuildxpumanager"
keywords = ["Technology"]
categories = ["Technology"]
+++
### environment
vagrant vm, for using this vm we could reach out the internet(through gfw).     

### Steps
Get the source code and prepare the code changes:    

```
# apt install -y git build-essential
# git clone  https://github.com/intel/xpumanager.git
# cd xpumanager
# vim ./core/src/vgpu/precheck.cpp +72    
    } else if (cmdRes.output().find("vmx") != std::string::npos) {
        /*
        *   VMX flag detected by lscpu
        */
        result->vmxFlag = true;
    } else {
        result->vmxFlag = true;
        //result->vmxFlag = false;
        //std::string msg = "No VMX flag, Please ensure Intel VT enabled in BIOS";
        //strncpy(result->vmxMessage, msg.c_str(), msg.size() + 1);
    }
# vim builder/Dockerfile.builder-ubuntu
    make -j && make install && \
---->
    make -j8 && make install && \
```
Build the build docker image, save the iidfile:    

```
$ sudo docker build --build-arg BASE_VERSION=$BASE_VERSION --build-arg http_proxy=$http_proxy --build-arg https_proxy=$https_proxy --iidfile /tmp/xpum_builder_ubuntu_$BASE_VERSION.iid -f builder/Dockerfile.builder-ubuntu .
$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
<none>       <none>    8beea6fc722f   9 minutes ago   1.92GB
ubuntu       22.04     b6548eacb063   11 days ago     77.8MB
$ cp /tmp/xpum_builder_ubuntu_22.04.iid ~
```
Using this docker image for building the deb file(xpumanager):     

```
sudo docker run --rm \
    -v $PWD:$PWD \
    -u $UID \
    -e CCACHE_DIR=$PWD/.ccache \
    -e CCACHE_BASEDIR=$PWD \
    $(cat /tmp/xpum_builder_ubuntu_$BASE_VERSION.iid) $PWD/build.sh
cp /home/vagrant/xpumanager/build/xpumanager_1.2.25_20231213.023315.251edc28~u22.04_amd64.deb ~
```
Build the xpu-smi:     

```
rm -fr build
sudo docker run --rm \
    -v $PWD:$PWD \
    -u $UID \
    -e CCACHE_DIR=$PWD/.ccache \
    -e CCACHE_BASEDIR=$PWD \
    $(cat /tmp/xpum_builder_ubuntu_$BASE_VERSION.iid) $PWD/build.sh -DDAEMONLESS=ON
cp /home/vagrant/xpumanager/build/xpu-smi_1.2.25_20231213.023748.251edc28~u22.04_amd64.deb ~
```

### verification
Install:    

```
# sudo apt-get install -y ./xpu-smi_1.2.25_20231213.023748.251edc28~u22.04_amd64.deb
# ls /dev/dri/
by-path  card0  card1  renderD128  renderD129
```

