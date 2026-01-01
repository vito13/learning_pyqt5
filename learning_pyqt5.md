# 环境

## 安装QT

- 需要从官网下载在线安装包或离线包
```
qt-online-installer-windows-x64-4.10.0.exe
```

## python 3.7

- 使用安装包
- 使用anaconda3（建议首选，自带常用库）
- 使用pycharm（可对多个py版本进行选择，并使用）

```
C:\Users\Administrator>where python3
C:\Users\Administrator\AppData\Local\Microsoft\WindowsApps\python3.exe

C:\Users\Administrator>where pip
C:\ProgramData\Anaconda3\Scripts\pip.exe

C:\Users\Administrator>where pip3
C:\ProgramData\Anaconda3\Scripts\pip3.exe
```

## pyqt 5.12

- 单独QT安装包
- 使用pycharm进行插件安装（推荐，菜单选择插件搜之安装即可）

```
C:\Users\Administrator>where pyuic5
C:\ProgramData\Anaconda3\Library\bin\pyuic5.bat
C:\ProgramData\Anaconda3\Scripts\pyuic5.exe

C:\Users\Administrator>where pyrcc5
C:\ProgramData\Anaconda3\Library\bin\pyrcc5.bat
C:\ProgramData\Anaconda3\Scripts\pyrcc5.exe

C:\Users\Administrator>where pylupdate5
C:\ProgramData\Anaconda3\Library\bin\pylupdate5.bat
C:\ProgramData\Anaconda3\Scripts\pylupdate5.exe
```

## hello world

运行后出现空白窗口即可

```
import sys
from PyQt5.QtWidgets import QApplication, QWidget
app = QApplication(sys.argv)
w = QWidget()
w.setWindowTitle("Hello World")
w.resize(300, 200)
w.show()
sys.exit(app.exec())


使用命令行或ide启动都可以
PS D:\pystart\Project1\qt> python .\mainqt.py
PS D:\pystart\Project1\qt> 
```

# 概念

## Qt Creator、Qt Designer

- Qt Designer是ui编辑器，生成XML的.ui 文件
- PYQT默认没有Qt Creator、但有Qt Designer

```
C:\Users\Administrator>where designer
C:\ProgramData\Anaconda3\Library\bin\designer.exe

Qt Creator   ←←←←←←←←←←←← 你主要用的
   │
   ├── 编辑 Python / C++ 代码
   ├── 运行 / 调试程序
   ├── 项目管理
   └── Qt Designer（集成在里面）
           │
           └── 生成 .ui 文件（界面描述）
```

## .ui 被使用的两种方式

- 方式一：ui → py（最常见，新手友好），优点简单好理解教程最多，缺点：改 UI 要重新生成

```
.ui  ──pyuic──▶  ui_mainwindow.py
                        │
                        └─ 你的逻辑代码
```

使用pyuic5将ui生成py

```
 D:\pystart\Project1\qt> pyuic5 -o .\fromhello.py .\fromhello.ui
```

使用如下代码加载窗体py，启动即可

```
import sys
from PyQt5.QtWidgets import QApplication, QMainWindow
from fromhello import  Ui_Dialog  # 这里导入生成的py文件里的类

class MyWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.ui = Ui_Dialog()
        self.ui.setupUi(self)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MyWindow()
    window.show()
    window.ui.label.setText("xxx") # 在此可以动态设置属性
    sys.exit(app.exec_())
```

- 方式二：运行时加载 ui（进阶），优点改 UI 不用重新生成更灵活，缺点对结构理解要求高
  
```
.ui
 │
 └── uic.loadUi("xxx.ui", self)
```

## 信号槽

- 类似回调、但更安全更可靠
- 使用编辑器可以设置默认提供的槽函数，但自定义的槽需要在代码里手动设置

### 自定义信号槽

- Human：信号的发送者
- Responsor：信号的接收者（槽）
- 演示了：自定义信号、同名重载信号、信号发射 emit、槽函数绑定 / 解绑、构造函数里发射信号的效果

```
## 自定义信号与槽的演示
import sys

from PyQt5.QtCore import QObject, pyqtSlot, pyqtSignal

class Human(QObject):
## 定义一个带str类型参数的信号
   nameChanged = pyqtSignal(str)
    
## overload型信号，两种参数，一种int，一种str
   ageChanged = pyqtSignal([int],[str])

   def __init__(self,name='Mike',age=10,parent=None):
      super().__init__(parent)   #调用父类构造函数
      self.setAge(age)
      self.setName(name)

   def   setAge(self,age):
      self.__age= age
      self.ageChanged.emit(self.__age)   #int参数信号
        
      if age<=18:
         ageInfo="你是 少年"
      elif (18< age <=35):
         ageInfo="你是 年轻人"
      elif (35< age <=55):
         ageInfo="你是 中年人"
      elif (55< age <=80):
         ageInfo="您是 老人"
      else:
         ageInfo="您是 寿星啊"
         
      self.ageChanged[str].emit(ageInfo)   #str参数信号
        
   def setName(self,name):
      self.__name = name
      self.nameChanged.emit(self.__name)


class Responsor(QObject):
   @pyqtSlot(int)
   def do_ageChanged_int(self,age):
      print("你的年龄是："+str(age))

   @pyqtSlot(str)
   def do_ageChanged_str(self,ageInfo):
      print(ageInfo)

#   @pyqtSlot(str)
   def do_nameChanged(self, name):
      print("Hello,"+name)

  
if  __name__ == "__main__":    ##测试程序
   print("**创建对象时**")    
   boy=Human("Boy",16)   
   resp=Responsor()

   boy.nameChanged.connect(resp.do_nameChanged)

   ## overload的信号如果都定义了槽函数，两个槽函数不能同名，连接时需要给信号加参数区分
   boy.ageChanged.connect(resp.do_ageChanged_int)        #缺省参数，int型
   boy.ageChanged[str].connect(resp.do_ageChanged_str)   #str型参数

   print("\n **建立连接后**")    
   boy.setAge(35)       #发射 两个ageChanged 信号
   boy.setName("Jack")  #发射nameChanged信号

   boy.ageChanged[str].disconnect(resp.do_ageChanged_str)   #断开连接
   print("\n **断开ageChanged[str]的连接后**")    
   boy.setAge(10)   #发射 两个ageChanged 信号    
    

```

## 资源文件

就是将ico等res类型的文件内容以二进制形式写入文本文件里（如.py）,然后使用程序加载对应的资源名称，案例见Demo2_5Resource