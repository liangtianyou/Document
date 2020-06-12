# PyQt4步长调节器QSpinBox使用
[TOC]
## 1 描述
QSpinBox是一个主要处理整数和离散值集合的步长调节器控件，它允许用户通过单击增减按钮或用键盘输入值来实现当前显示值的改变。它主要显示整数，也可以通过自定义显示一些离散的值集（周一~周日）

QSpinBox继承了QAbstractSpinBox，可以使用父类的各种API。

### 1.1  示例
```python
#!/usr/bin/env python
#-*- coding: utf-8 -*-

import os
import sys
from PyQt4 import QtCore, QtGui

class SetDialog(QtGui.QDialog):
    def __init__(self):
        super(SetDialog, self).__init__()
        self.setupUI()
        self.center()
        
    def center(self):
        screen = QtGui.QDesktopWidget().screenGeometry()
        size =  self.geometry()
        self.move((screen.width()-size.width())/2, (screen.height()-size.height())/2)
        
    def setupUI(self):
        self.setWindowTitle(u"设置")
        self.resize(400,200)
        self.setFixedSize(self.width(), self.height())
        self.setWindowFlags(QtCore.Qt.WindowCloseButtonHint)
        
        self.downNumLab = QtGui.QLabel(u"下载数量:",self)
        self.downNumSpin = QtGui.QSpinBox()
        self.downNumSpin.setValue(3)
        self.downNumSpin.setRange(1,5)
        
        self.upNumLab = QtGui.QLabel(u"上传数量:",self)
        self.upNumSpin = QtGui.QSpinBox()
        self.upNumSpin.setValue(3)
        self.upNumSpin.setRange(1,5)
        
        self.saveBtn = QtGui.QPushButton(u"保存")
        self.saveBtn.clicked.connect(self.saveBtnClicked)
        
        # 主布局
        self.mainLayout = QtGui.QGridLayout()
        
        self.mainLayout.addWidget(self.downNumLab,0,0)
        self.mainLayout.addWidget(self.downNumSpin,0,1)
        
        self.mainLayout.addWidget(self.upNumLab,1,0)
        self.mainLayout.addWidget(self.upNumSpin,1,1)
        
        self.mainLayout.addWidget(self.saveBtn,2,0,1,2)
        
        self.mainLayout.setHorizontalSpacing(10)
        self.mainLayout.setVerticalSpacing(5)
        self.mainLayout.setContentsMargins(80, 20, 80, 20)
        
        self.setLayout(self.mainLayout)
        
    def saveBtnClicked(self):
        downnum = self.downNumSpin.value()
        upnum = self.upNumSpin.value()
        print('下载数量：%s' % downnum)
        print('上传数量：%s' % upnum)
        self.accept()

if __name__ == '__main__':     
    app = QtGui.QApplication(sys.argv)
    ex = SetDialog()
    ex.show()
    sys.exit(app.exec_())
```
效果图：
![QSpinBox效果](../../static/images/pyqt4_qspinbox1.png)

## 2 常用接口
### 2.1 最大值和最小值
```python
QSpinBox.setMinimum()        #设定最大值
QSpinBox.setMaximum()        #设定最小值
QSpinBox.setRange(min,max)   #设定范围
QSpinBox.maximum()           #获取最大值
QSpinBox.minimum()           #获取最小值
```
### 2.2 数值循环
可以通过设置数值循环，在数据到最大值时可以直接转到最小值（反之也可以）。
```python
QSpinBox.setWrapping(True)        #启用数值循环
QSpinBox.wrapping()               #是否启用数值循环
```
### 2.3 步长设置
```python
QSpinBox.setSingleStep(1)         #设定步长
QSpinBox.singleStep()             #获取步长——>str
```
### 2.4 前缀和后缀和最小值特殊文本
加了前缀或后缀只改变数值，前后缀不变化。
```python
QSpinBox.setPrefix()             #设定前缀
QSpinBox.setSuffix()             #设定后缀
QSpinBox.prefix()                #获取前缀——>str
QSpinBox.suffix()                #获取后缀——>str
```
还可以在最小值时显示一个固定的字符，比如0元的时候定义为免费
```python
QSpinBox.setSpecialValueText('')                #设置最小值对应文本
QSpinBox.specialValueText()                     #获取最小值对应文本——>str
```
### 2.5 进制更改
```python
QSpinBox.setDisplayIntegerBase(int)                #设置进制
QSpinBox.displayIntegerBase()                      #获取进制——>str
```
这里设置的进制是不光常用的二进制、八进制和十进制的，任何一个整数是都可以的（还可以大于16，不过实际没什么意义，0-F后可以继续）
### 2.6 数值的设置和获取
```python
QSpinBox.setValue()          #设定值
QSpinBox.value()             #获取int的值
```
这里用的value获取的是int对应的值，不包含前后缀，最小值时不对应特殊文本。
## 3 自定义展示格式
我们可以通过重写下面的函数来实现自定义的格式显示
```python
class custSpinBox(QSpinBox):
    def textFromValue(self, v: int):
        pass
```
这个函数传递了当前的整数值。
```python
#!/usr/bin/env python
#-*- coding: utf-8 -*-

import sys
from PyQt4 import QtCore, QtGui

class custSpinBox(QtGui.QSpinBox):
    def textFromValue(self, v):   #自定义展示格式
        week = [u'星期日',u'星期一',u'星期二',u'星期三',u'星期四',u'星期五',u'星期六',]
        weekday = week[v]
        return weekday

class Window(QtGui.QWidget):
    def __init__(self):
        super(Window,self).__init__()
        self.UI_SetUp()
        
    def UI_SetUp(self):
        self.sb = custSpinBox(self)
        self.sb.resize(100,30)
        self.sb.move(100,100)
        self.sb.setRange(0,6)

if __name__ =='__main__':
    app = QtGui.QApplication(sys.argv)
    window = Window()
    window.show()
    sys.exit(app.exec_())
```
这样就显示的是周日到周六了。
![QSpinBox自定义效果](../../static/images/pyqt4_qspinbox2.png)
## 3 信号
继承父类：
```python
QSpinBox.editingFinished()
```
自定义信号：
```python
QSpinBox.valueChanged()
```
它传递有int类型的信号，还可以重构一下发送字符串类型的数据。
```python
QSpinBox.valueChanged[str]() 
```
发送字符串时是按照当前显示的状态发送的，包含有前后缀以及自定义的显示类型。






