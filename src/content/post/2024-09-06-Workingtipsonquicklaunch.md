+++
title= "Workingtipsonquicklaunch"
date = "2024-09-06T17:42:44+08:00"
description = "Workingtipsonquicklaunch"
keywords = ["Technology"]
categories = ["Technology"]
+++
`/opt/bgok_close.py`:    

```
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QPushButton, QVBoxLayout, QHBoxLayout, QLineEdit, QLabel
from subprocess import run

class MyWidget(QWidget):
    def __init__(self):
        super().__init__()
        
        self.initUI()

    def initUI(self):
        self.setWindowTitle('IDV-OSX-kvm虚拟机管理器')
        self.showMaximized() # 初始化窗口大小并最大化

        layout = QVBoxLayout()
        self.setLayout(layout)

        label = QLabel('IDV-OSX-kvm 虚拟机 ID:')
        layout.addWidget(label)

        self.id_input = QLineEdit('macOSrx550')
        layout.addWidget(self.id_input)

        button_layout = QHBoxLayout()
        button_layout.setContentsMargins(0, 0, 0, 50) # 调整按钮之间的间隔
        layout.addLayout(button_layout)
        
        start_button = QPushButton('启动IDV-OS-X虚拟机', clicked=self.start_virtmachine)
        start_button.setStyleSheet("background-color: green; color: white") # 设置背景颜色为绿色和字体颜色为白色
        start_button.setMinimumWidth(self.width() * 0.5) # 各占据屏幕宽度的50%
        start_button.setMaximumWidth(self.width() * 0.5)
        button_layout.addWidget(start_button)

        shutdown_button = QPushButton('IDV物理机关机', clicked=self.shutdown)
        shutdown_button.setStyleSheet("background-color: red; color: white") # 设置背景颜色为红色和字体颜色为白色
        shutdown_button.setMinimumWidth(self.width() * 0.5) # 各占据屏幕宽度的50%
        shutdown_button.setMaximumWidth(self.width() * 0.5)
        button_layout.addWidget(shutdown_button)

        close_button = QPushButton('退出IDV管理程序', clicked=self.close)
        close_button.setStyleSheet("background-color: blue; color: white") # 设置背景颜色为蓝色和字体颜色为白色
        close_button.setMinimumWidth(self.width() * 0.5) # 各占据屏幕宽度的50%
        close_button.setMaximumWidth(self.width() * 0.5)
        button_layout.addWidget(close_button)

    def start_virtmachine(self):
        id = self.id_input.text()
        run(['virsh', 'start', id])

    def shutdown(self):
        run(['shutdown', '-h', 'now'])

if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = MyWidget()
    sys.exit(app.exec_())

```

Auto start:      

```
$ cat /etc/xdg/autostart/vmmanager.desktop 
[Desktop Entry]
0=V
1=m
2=m
3=a
4=n
5=a
6=g
7=e
Name=Vmmanage
Exec=/usr/bin/python3 /opt/bgok_close.py %U
Terminal=false
Type=Application
Icon=Vmmanage
StartupWMClass=Vmmanage
Comment=Vmmanage
Categories=Utility;

```
