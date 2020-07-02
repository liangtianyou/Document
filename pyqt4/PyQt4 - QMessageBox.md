---
title: PyQt4 - QMessageBox

date: 2020-06-25 15:58

tags:
    - PyQt4
    - QMessageBox
---
QMessageBox是一个常用的模态对话框，用来显示一些信息类消息，可以让用户通过点击任意一个标准按钮来响应。每个标准按钮都有一个预定义的标题、角色，并且返回一个预定义的十六进制数字。
## 1 QMessageBox的函数
函数名     | 说明
---------- |-----------
setIcon()               | 显示预定义的图标
setText()               | 设置主消息的文字
setInformativeText()    | 显示额外信息
setDetailText()         | 显示一个Details按钮
setTitle()              | 显示自定义标题
setStandardButtons()    | 标准按钮及其对应的十六进制数字是QMessageBox.Ok 0x00000400、QMessageBox.Open 0x00002000、QMessageBox.Save 0x00000800、QMessageBox.Cancel 0x00400000、QMessageBox.Close 0x00200000、QMessageBox.Yes 0x00004000、QMessageBox.No 0x00010000、QMessageBox.Abort 0x00040000、QMessageBox.Retry 0x00080000、QMessageBox.Ignore 0x00100000。
setDefaultButton()      | 设置默认按钮。当回车时它会发射clicked信号
setEscapeButton()       | 当按下Escape键时，该按钮会发出clicked信号

## 2 示例
```python
#-*- coding: utf-8 -*-
import sys
from PyQt4.QtCore import *
from PyQt4.QtGui import *

class (QWidget):
    def __init__(self, parent=None):
        super(window, self).__init__(parent)
        self.init()

    def init(self):
        #创建布局
        layout = QHBoxLayout()
        button = QPushButton()
        button.setText(u"点我弹出窗口B")
        button.clicked.connect(self.showB)
        layout.addWidget(button)
        self.setLayout(layout)
        self.setWindowTitle(u"窗口A")

    def showB(self):
        msgdialog = QMessageBox()
        msgdialog.setIcon(QMessageBox.Information)

        msgdialog.setText(u"这是一个信息窗口")
        msgdialog.setInformativeText(u"这是个描述信息")
        msgdialog.setWindowTitle(u"信息弹出框")
        msgdialog.setDetailedText(u"按钮点开查看详情！")
        msgdialog.setStandardButtons(QMessageBox.Ok | QMessageBox.Cancel)
        msgdialog.buttonClicked.connect(self.msgButton)
        whatf = msgdialog.exec_()
        print("press button:",whatf)

    def msgButton(self,button):
        print("my press button:",button.text())

def main():
    app = QApplication(sys.argv)
    ex = window()
    ex.show()
    sys.exit(app.exec_())

if __name__ == '__main__':
    main()
```
