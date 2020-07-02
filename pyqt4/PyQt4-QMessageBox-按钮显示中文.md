---
title: PyQt4 - QMessageBox - 按钮显示中文
date: 2020-06-26 16:41
tags:
    - PyQt4
    - QMessageBox
    - 中文按钮
---

直接调用 ```QMessageBox``` 上的 ```information```, ```warning``` 等方法, 生成的按钮都是英文的 ```Yes|No``` 等.
```python
    QMessageBox.information(self, u"执行结果", u"执行完成", QMessageBox.Yes | QMessageBox.No)
```

## 让按钮显示中文
```python
#!/usr/bin/env python
#-*- coding: utf-8 -*-

import sys
from PyQt4 import QtCore, QtGui

class ConfirmMsgBox(QtGui.QWidget):
    def __init__(self, parent=None):
        super(ConfirmMsgBox, self).__init__(parent)
        self.init()

    def init(self):
        # 创建布局
        layout = QtGui.QHBoxLayout()
        button = QtGui.QPushButton()
        button.setText(u"点我弹出窗口B")
        button.clicked.connect(self.showB)
        layout.addWidget(button)
        self.setLayout(layout)
        self.setWindowTitle(u"窗口A")

    def showB(self):
        msgdialog = QtGui.QMessageBox()
        msgdialog.setIcon(QtGui.QMessageBox.Information)

        msgdialog.setText(u"这是一个信息窗口")
        msgdialog.setInformativeText(u"这是个描述信息")
        msgdialog.setWindowTitle(u"信息弹出框")
        msgdialog.setDetailedText(u"按钮点开查看详情！")
        msgdialog.setStandardButtons(QtGui.QMessageBox.Ok | QtGui.QMessageBox.Cancel)
        buttonY = msgdialog.button(QtGui.QMessageBox.Ok)
        buttonY.setText(u'确定')
        buttonN = msgdialog.button(QtGui.QMessageBox.Cancel)
        buttonN.setText(u'取消')
        msgdialog.buttonClicked.connect(self.msgButton)
        whatf = msgdialog.exec_()
        print("press button:",whatf)

    def msgButton(self,button):
        print(u"点击按钮:",button.text())

def main():
    app = QtGui.QApplication(sys.argv)
    ex = ConfirmMsgBox()
    ex.show()
    sys.exit(app.exec_())

if __name__ == '__main__':
    main()
```
