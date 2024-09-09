+++
title= "AutoStartV2"
date = "2024-09-09T17:50:22+08:00"
description = "AutoStartV2"
keywords = ["Technology"]
categories = ["Technology"]
+++
source code is listed as following:       

```
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QPushButton, QHBoxLayout, QLabel, QRadioButton
from PyQt5.QtCore import Qt
import subprocess

class MyWidget(QWidget):
    def __init__(self):
        super().__init__()
        
        self.radiobuttons = []  # 在__init__中定义radiobuttons列表
        
        self.initUI()

    def initUI(self):
        self.setWindowTitle('IDV-Multi-kvm虚拟机管理器')
        self.showMaximized() # 初始化窗口大小并最大化

        layout = QVBoxLayout()
        self.setLayout(layout)

        radio_button_layout = QHBoxLayout()
        
        output = subprocess.check_output(['virsh', 'list','--all', '--name']).decode('utf-8').splitlines()
        
        for i, line in enumerate(output[:-1]):
            self.radiobuttons.append(QRadioButton(line))
            radio_button_layout.addWidget(self.radiobuttons[i])
        
        layout.addLayout(radio_button_layout)  # 将button添加到主界面中
        
        button_layout = QHBoxLayout()
        self.start_button = QPushButton('启动选中的虚拟机')
        self.start_button.setStyleSheet("background-color: green; color: white") # 设置背景颜色为绿色和字体颜色为白色
        self.start_button.setMinimumWidth(self.width() // 3) 
        self.start_button.setMaximumWidth(self.width() // 3)
        self.start_button.clicked.connect(self.start_vmmachine)

        
        shutdown_button = QPushButton('IDV物理机关机', clicked=lambda: self.shutdown())
        shutdown_button.setStyleSheet("background-color: red; color: white") # 设置背景颜色为红色和字体颜色为白色
        shutdown_button.setMinimumWidth(self.width() // 3) 
        shutdown_button.setMaximumWidth(self.width() // 3)
        self.start_button.clicked.connect(self.shutdown)
        
        close_button = QPushButton('退出IDV管理程序', clicked=self.close)
        close_button.setStyleSheet("background-color: blue; color: white") # 设置背景颜色为蓝色和字体颜色为白色
        close_button.setMinimumWidth(self.width() // 3) 
        close_button.setMaximumWidth(self.width() // 3)
        
        button_layout.addWidget(self.start_button)
        button_layout.addWidget(shutdown_button)
        button_layout.addWidget(close_button)

        layout.addLayout(button_layout)  # 将button添加到主界面中
        
        self.show()

    def start_vmmachine(self):
        #selected_radio_button = [i for i in self.findChildren(QRadioButton)][0]
        for i, button in enumerate(self.findChildren(QRadioButton)):
            if button.isChecked():
                selected_radio_button = [i for i in self.radiobuttons][i]
                vm_name = selected_radio_button.text()
                print("********")
                print(vm_name)
                subprocess.run(['virsh', 'start', vm_name])
                break

    def shutdown(self):
        subprocess.run(['shutdown', '-h', 'now'])
        

if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = MyWidget()
    sys.exit(app.exec_())

```
qemu hook changes:       

```
$ cat qemumodified_hook 
#!/bin/bash
OBJECT="$1"
OPERATION="$2"
INSTANCE="instance-00000001"
#INSTANCE="win10"
if [[ $OBJECT == "$INSTANCE" || $OBJECT == "ubuntu2404" || $OBJECT == "UOS" || $OBJECT == "zkfd" || $OBJECT == "Kylin"   || $OBJECT == "Win10" || $OBJECT == "Win11" ]]; then
        case "$OPERATION" in
                "prepare")
                /bin/vfio-startup.sh 2>&1 | tee -a /var/log/libvirt/custom_hooks.log
                ;;
            "release")
                /bin/vfio-teardown.sh 2>&1 | tee -a /var/log/libvirt/custom_hooks.log
                ;;
        esac
fi

```
