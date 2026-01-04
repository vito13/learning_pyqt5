# 环境

## 安装QT

- 需要从官网下载在线安装包并挂接镜像安装
```
qt-online-installer-windows-x64-4.10.0.exe --mirror https://mirrors.ustc.edu.cn/qtproject
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

## 资源文件

就是将ico等res类型的文件内容以二进制形式写入文本文件里（如.py）,然后使用程序加载对应的资源名称，案例见Demo2_5Resource

## 信号 signal

- 本质是“某件事发生了”，就会产生一个信号
- 可以当成是个“广播”，如“我被点了！”、“当前项变了！”、“状态被勾选了！”，在代码里就类似triggered()、clicked()、currentItemChanged()、stateChanged()，不同对象发不同信号
- Action、Button、Tree、Checkbox等都可以发出信号

| 发信号的对象  | 常见信号   | 含义     |
| ----------- | -------------------- | ------ |
| QAction     | triggered()          | 行为被触发  |
| QPushButton | clicked()            | 按钮被点   |
| QTreeWidget | currentItemChanged() | 当前节点变化 |
| QCheckBox   | stateChanged()       | 勾选状态变化 |

- 发信号也可以手动调用，[见案例 自定义信号槽](##自定义信号槽)

```
def mouseDoubleClickEvent(self, event):
    self.doubleClicked.emit()
```

## Action

- QAction 是一个“行为（Action）对象”，它代表的是“用户要做的一件事”，比如删除、打开、保存、放大、缩小
- 如点菜单、点工具栏、按快捷键可能都是同一个QAction，会发出相同的信号，并且QAction也可以自主发出信号
- 使用.triggered.connect()设置对应的槽函数，也可以使用语法糖达到进行自动connect的效果

```
from PyQt5.QtWidgets import QApplication, QMainWindow, QAction

app = QApplication([])
win = QMainWindow()

action_exit = QAction("退出", win)
action_exit.setShortcut("Ctrl+Q")  # 设置快捷键
action_exit.setStatusTip("退出应用程序")
action_exit.triggered.connect(app.quit)  # 设置槽函数

menu = win.menuBar().addMenu("文件")
menu.addAction(action_exit)

win.show()
app.exec_()

这里 QAction 可以同时出现在菜单和工具栏，也可触发同一个槽函数。
```


## triggered

- QAction.triggered 是 QAction 的“被激活”信号，无论菜单、工具栏还是快捷键触发，都会发送这个信号
- triggered 会 默认传递一个 bool，表示 QAction 是否被选中（主要针对 checkable=True 的动作），对于非 checkable 的 QAction，参数一般无意义，但仍然可以接收
- 调用action_bold.trigger() 可以在代码中手动触发动作

| 功能        | 使用方式                             | 参数           | 说明              |
| --------- | -------------------------------- | ------------ | --------------- |
| 连接槽函数     | `action.triggered.connect(slot)` | `bool`（可选）   | 点击菜单/工具栏/快捷键时触发 |
| checkable | `action.setCheckable(True)`      | `True/False` | 触发信号时传递当前选中状态   |
| 手动触发      | `action.trigger()`               | 无            | 在代码里模拟用户点击动作    |


没有使用参数的案例

```
from PyQt5.QtWidgets import QApplication, QMainWindow, QAction

app = QApplication([])
win = QMainWindow()

# 创建一个 QAction
action_exit = QAction("退出", win)
action_exit.setShortcut("Ctrl+Q")

# 连接 triggered 信号
action_exit.triggered.connect(lambda checked=False: print("触发退出"))

# 菜单栏添加
menu = win.menuBar().addMenu("文件")
menu.addAction(action_exit)

win.show()
app.exec_()
```

使用参数的案例

```
from PyQt5.QtWidgets import QApplication, QMainWindow, QAction

app = QApplication([])
win = QMainWindow()

# 可选中 QAction
action_bold = QAction("加粗", win, checkable=True)

def on_bold_triggered(checked):
    if checked:
        print("文字加粗")
    else:
        print("取消加粗")

action_bold.triggered.connect(on_bold_triggered)

menu = win.menuBar().addMenu("格式")
menu.addAction(action_bold)

win.show()
app.exec_()
```

## 槽 slot

- 是个被调的函数，类似回调，但更安全更可靠
- 信号都会有对应的槽函数进行处理
- 使用编辑器可以设置默认提供的槽函数，但自定义的槽需要在代码里手动设置

### 关于@pyqtSlot

@pyqtSlot 不是“写不写都行的装饰器”，而是 PyQt 进入 Qt 元对象系统、获得类型、性能、线程保障的钥匙。

| 维度 | 不使用 @pyqtSlot | 使用 @pyqtSlot |
|---|---|---|
| 是否必须 | 否 | 否（但强烈推荐） |
| 槽类型 | 普通 Python 函数 | Qt 元对象系统中的槽 |
| 类型信息 | 无 | 有（强类型） |
| 信号匹配 | 运行期模糊匹配 | 连接期就可校验 |
| 重载支持 | 不支持 | 支持（int / str 等） |
| 执行性能 | 较慢（Python 回调） | 较快（Qt 直达槽） |
| 线程语义 | 不明确 | 明确（自动 QueuedConnection） |
| 跨线程安全 | 需开发者保证 | Qt 自动保证 |
| 高频信号 | 不推荐 | 推荐 |
| Qt/C++ 可见性 | 不可见 | 可见 |
| Qt Designer / QML | 不支持 | 支持（部分场景） |
| 大型工程可维护性 | 一般 | 高 |

| 使用场景 | 是否推荐 @pyqtSlot |
|---|---|
| 按钮 clicked | 可不用 |
| 自定义信号 | 必须 |
| 信号重载 | 必须 |
| 跨线程通信 | 必须 |
| 高频信号（timer / slider） | 强烈推荐 |
| Demo / 临时代码 | 可不用 |
| 工程级项目 | 统一使用 |

| C++ Qt | PyQt |
|---|---|
| slots: | @pyqtSlot |
| 强类型槽 | 强类型槽 |
| Meta-Object | Meta-Object |
| 编译期检查 | 运行期检查 |

### 自动绑定槽

下面两句作用一样


- 手动设置槽

```
self.ui.actTree_DeleteItem.triggered.connect(
    self.on_actTree_DeleteItem_triggered
)

```
- 自动绑定，会在setupUi() 里被 connectSlotsByName() 扫描符合“on_xxx_signal”的会自动绑定
```
@pyqtSlot()
def on_actTree_DeleteItem_triggered(self):

语法糖：on_xxx_signal
on_<对象objectName>_<信号名>(参数)
注意objectName的里面是可以包含有下划线的...

合格的名命：
def on_actTree_DeleteItem_triggered(self):
def on_actTree_ScanItems_triggered(self):
def on_treeFiles_currentItemChanged(self, current, previous):
def on_actZoomFitH_triggered(self):


用户点击菜单
↓
QAction(actTree_DeleteItem)
        │
        │ triggered()
        ▼
Qt 自动找到，执行你写的代码
on_actTree_DeleteItem_triggered()
```
```
  def on_btnCalculate_clicked(self):  ##"计算总价"按钮
      num=int(self.ui.editCount.text())
      price=float(self.ui.editPrice.text())
      total=num*price
      self.ui.editTotal.setText("%.2f" %total)

   @pyqtSlot(int)    ##"数量"SpinBox
   def on_spinCount_valueChanged(self,count):
      price=self.ui.spinPrice.value()
      self.ui.spinTotal.setValue(count*price)

   @pyqtSlot(float)     ##"单价" DoubleSpinBox
   def on_spinPrice_valueChanged(self,price):
      count=self.ui.spinCount.value()
      self.ui.spinTotal.setValue(count*price)

```

自动连接是“约定优于配置”的快捷方式，不是类型安全机制。参数一复杂，就该放弃它

| 情况 | 自动连接是否生效 | 是否安全 |
|---|---|---|
| 参数完全匹配 | 是 | ✅ |
| 槽参数少于信号 | 是 | ⚠（可控） |
| 槽参数多于信号 | 否 | ❌ |
| 参数类型不匹配 | 连接成功 | ❌（运行期风险） |
| 信号重载 | 不稳定 | ❌ |

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

## QActionGroup

- QActionGroup 是 一组 QAction 的容器
- 主要用于互斥行为：一组 QAction 中只能选中一个（类似单选按钮）
- 方便统一管理多 QAction 的状态（比如启用/禁用）
- 注意QActionGroup 本身不是 UI 元素，不显示，只有 QAction 被添加到菜单、工具栏等控件中才显示


| 类            | 类型 | 可用场景      | 主要功能                  | 特点                    |
| ------------ | -- | --------- | --------------------- | --------------------- |
| QAction      | 对象 | 菜单、工具栏、按钮 | 封装操作（文本、图标、快捷键、槽）     | 可以被多个控件共享             |
| QActionGroup | 容器 | 菜单、工具栏、按钮 | 管理一组 QAction（互斥或批量控制） | 本身不显示，支持互斥（exclusive） |

下面的案例演示互斥的菜单栏选项

```
from PyQt5.QtWidgets import QApplication, QMainWindow, QAction, QActionGroup

app = QApplication([])
win = QMainWindow()

menu = win.menuBar().addMenu("显示模式")

# 创建互斥的 QActionGroup
group = QActionGroup(win)
group.setExclusive(True)  # 互斥

action_percent = QAction("百分比", win, checkable=True)
action_value = QAction("数值", win, checkable=True)

group.addAction(action_percent)
group.addAction(action_value)

menu.addAction(action_percent)
menu.addAction(action_value)

action_percent.triggered.connect(lambda: print("百分比模式"))
action_value.triggered.connect(lambda: print("数值模式"))

action_percent.setChecked(True)  # 默认选中
win.show()
app.exec_()
```

# UI

## self.ui

- 通过designer创建的ui可通过self.ui.xxx进行访问
- 代码里动态创建的控件，则直接self.__xxx即可，即内部的变量使用2个下划线

## QPushButton

- checkable = true是可以按下与弹起的按钮
- 当一个 QPushButton，checkable = true、autoExclusive = true，且和其他同类按钮处在同一个父对象下，它们会自动组成一个“互斥组”。即同一时间只能有一个按钮处于 checked 状态，类似radio

# Model View

把“数据”和“怎么显示、怎么操作”彻底分开。

## Model

- 只负责数据从哪来（内存 / 文件 / 数据库）、有多少行、多少列、某一行某一列是什么值、数据能不能改
- 不关心显示成列表还是树、用什么颜色、用户点没点
- 三种基础 Model如下

| 类型             | 结构        | 典型 View      | 适合场景   |
| -------------- | --------- | ------------ | ------ |
| **ListModel**  | 一维（行）     | `QListView`  | 简单列表   |
| **TableModel** | 二维（行 × 列） | `QTableView` | 表格数据   |
| **TreeModel**  | 层级结构      | `QTreeView`  | 树 / 目录 |

### ListModel 列表模型

```
0
1
2
3
...
```
- 只实现“行”，只有“行”，没有“列”，每一行就是一个 item
- 典型用途：文件列表、下拉选项、日志列表、简单菜单
- 常见 Model: QStringListModel（最简单）、QStandardItemModel（只用一列）
- 不能表达多字段数据
```
model = QStringListModel(["A", "B", "C"])
view = QListView()
view.setModel(model)
```

### TableModel 表格模型
```
行\列 | 姓名 | 年龄 | 职位
-------------------------
0     | 张三 |  20  | 开发
1     | 李四 |  22  | 测试
```
- 行 + 列的二维结构，每个单元格都有明确坐标 (row, column)
- 典型用途：Excel 风格数据、配置表、统计结果、参数列表
- 常见 Model：QStandardItemModel、自定义 QAbstractTableModel
- 不能表达父子关系
```
model = QStandardItemModel(2, 3)
model.setHorizontalHeaderLabels(["姓名", "年龄", "职位"])
```
### TreeModel 树模型

```
root
 ├─ A
 │   ├─ A1
 │   └─ A2
 └─ B
     └─ B1
```
- 父子关系、每个节点本身也是“一行数据”、每个节点仍然可以有多列
- 常见 Model：QStandardItemModel、QFileSystemModel、自定义 QAbstractItemModel
- 典型用途：文件树、目录结构、组织架构、配置层级
- TreeModel 本质上比 TableModel 多了 3 个问题：
   - 某个 item 的 parent 是谁
   - 有多少 child
   - child 在 parent 下的 row

### Model Index

- 是指向 Model 中某一个数据项的位置描述，类似句柄
- QModelIndex 里装了row, column, parent, model，可理解成树结构里的“绝对地址”，即在某个父节点下，第 row 行、第 column 列的那个数据项

   | 信息         | 含义         |
   | ---------- | ---------- |
   | `row()`    | 在父节点下的第几行  |
   | `column()` | 第几列        |
   | `parent()` | 父节点是谁      |
   | `model()`  | 属于哪个 Model |
- View和Model 都可获取 Index，但不同的model里Index内容也不一样
   ```
   index = view.currentIndex()
   index = model.index(row, column, parentIndex)
   ```
- 很多函数的参数不是数据，而是index。槽函数也有会使用index
   ```
   def on_treeView_clicked(self, index):
      print(index.row(), index.column())
   ```

### Item Role

- 一个数据项（item）不是一个值，而是一组「属性集合」；每个属性通过一个 Role 来区分。
- View 会根据不同场景要不同 role，同一个 index，不同 role，返回不同数据
   - 显示时 → DisplayRole
   - 编辑时 → EditRole
   - 绘制图标 → DecorationRole
   - 排序时 → UserRole
- 不是每个 Role 都必须存在、没设置的 Role，Model 可以返回空

| 属性   | Role                  |
| ---- | --------------------- |
| 显示文本 | Qt::DisplayRole       |
| 编辑值  | Qt::EditRole          |
| 图标   | Qt::DecorationRole    |
| 文字颜色 | Qt::ForegroundRole    |
| 背景色  | Qt::BackgroundRole    |
| 对齐方式 | Qt::TextAlignmentRole |
| 是否可选 | Qt::UserRole + n      |

```
model.setData(index, "文件名", Qt.DisplayRole)
model.setData(index, QIcon("file.png"), Qt.DecorationRole)
model.setData(index, "原始值", Qt.EditRole)
model.setData(index, "这是一个文件", Qt.ToolTipRole)
model.setData(index, Qt.AlignCenter, Qt.TextAlignmentRole)

value = model.data(index, Qt.DisplayRole)


View
 ↓  (QModelIndex)
Model (QStandardItemModel)
 ↓
Item (QStandardItem)
 ↓
Role → Value

```

### QStandardItem

- 是个多属性的单个数据项，支持多 role、支持多列、支持 parent / child（树结构）、一个自带字典（role→value）的节点对象
- 可被当成：表格里的一个单元格、树里的一个节点、列表里的一行

```
item = QStandardItem()
item.setText("main.cpp")                  # DisplayRole
item.setIcon(QIcon("file.png"))           # DecorationRole
item.setToolTip("源文件")                  # ToolTipRole
item.setData("/src/main.cpp", Qt.UserRole)


item.text()
item.setText()
item.data(role)
item.setData(value, role)
item.appendRow(child)
item.child(row, column)
item.parent()
```

### QStandardItemModel

- 一个基于 QStandardItem 的通用 Model 实现（类比资源管理器样式，每一行都可以是棵树）
- 即能按行操作，也可按树操作
- 一个 现成的 Model 实现、自动处理
   - QModelIndex
   - rowCount / columnCount
   - index() / parent()
   - data() / setData()
   - Model ↔ View 通知刷新

```
model = QStandardItemModel()
model.setHorizontalHeaderLabels(["名称", "类型"])

root = model.invisibleRootItem()

# 第 1 行（根节点下）
folder = QStandardItem("src")
type1  = QStandardItem("folder")
root.appendRow([folder, type1])

# 第 2 行（根节点下）
file1 = QStandardItem("readme.md")
type2 = QStandardItem("file")
root.appendRow([file1, type2])

# 给“src”这一行，再加子行（树）
child = QStandardItem("main.cpp")
childType = QStandardItem("file")
folder.appendRow([child, childType])

```

- 适用于：
   - 数据量不巨大
   - 需要快速做 UI
   - 树结构
   - 原型 / 工具类项目
   - 不想写一堆模板代码
- 不适用于：（应使用 QAbstractItemModel / QAbstractTableModel）
   - 百万级数据
   - 严格内存控制
   - 数据本身已经是复杂结构
   - 性能瓶颈明显


### QItemSelectionModel

- Model只管数据、View只管怎么画，所以需要专门管理「当前项、选中项」的QItemSelectionModel对象，注意当前和选中的概念是不同的

   | 概念       | 含义   |
   | -------- | ---- |
   | current  | 光标所在格子，current 可以不在 selected 里|
   | selected | 被选中的N个格子，selected 可以有多个，但 current 只有一个 |

- 常用信号
   - currentChanged：单元格焦点变化（适合更新按钮状态、检测是否有脏数据）
   - currentRowChanged：行变了（适合做表单切换、加载照片、Mapper 更新）
   - selectionChanged：选中的内容变了（多选场景、拖选、框选时候）

- SelectionModel基于view，调用self.ui.tableView.setSelectionModel(self.selModel)进行绑定，同一SelectionModel可绑到多个view里

## View

- 只负责整体数据如何呈现，如行高、列宽、滚动条、选中了哪一行
- 常见View如QListView、QTableView、QTreeView

## Delegate

- Delegate 是 View 的“画笔”和“编辑器”，也是 Role 真正被“用起来”的地方。
- 从 model 拿 role 数据，根据 role 决定绘制和编辑方式，负责三件事

   | 职责           | 说明      |
   | ------------ | ------- |
   | paint        | 怎么画     |
   | sizeHint     | 多大      |
   | createEditor | 用什么控件编辑 |

- 默认 Delegate：文本 → QLabel，编辑 → QLineEdit，但也可以改用下拉框、复选框、进度条、按钮
- 何时一定要写 Delegate
   - 单元格显示不是纯文本、进度条、按钮、星级、彩条
   - 编辑方式不是默认的、下拉框、日期、范围限制
   - 同一数据，多种显示风格
- 常见 Delegate 类型：QStyledItemDelegate（推荐）、QItemDelegate（老）

```
class ProgressDelegate(QStyledItemDelegate):
    def paint(self, painter, option, index):
        value = index.data(Qt.UserRole)

        opt = QStyleOptionProgressBar()
        opt.rect = option.rect
        opt.minimum = 0
        opt.maximum = 100
        opt.progress = value
        opt.text = f"{value}%"
        opt.textVisible = True

        QApplication.style().drawControl(
            QStyle.CE_ProgressBar, opt, painter
        )
```

# event

## event对象

- 事件是一个对象（基类QEvent 及其子类），描述的是「发生了什么事」，由 Qt 内部事件循环自动产生并分发
   
   | 常见事件举例   | 对应事件类          |
   | ------ | -------------- |
   | 鼠标按下   | `QMouseEvent`  |
   | 键盘按键   | `QKeyEvent`    |
   | 窗口重绘   | `QPaintEvent`  |
   | 窗口大小变化 | `QResizeEvent` |
   | 定时器触发  | `QTimerEvent`  |
   | 窗口关闭   | `QCloseEvent`  |
- 所有QObject都有QObject::event(QEvent *event)，内部根据类型调用不同事件的handler，QObject仅处理“非界面事件”，比如：定时器、自定义事件、对象销毁，QWidget则增加了鼠标、键盘、绘制、窗口显示 / 隐藏等处理
   ```
   QmyLabel
   ↓
   QLabel::event()
      ↓
   QWidget::event()
      ↓
   QObject::event()
   ```
- 用户可重载具体的事件处理函数进行自定义处理，不重载的话，每个具体事件也都有默认的处理，其实也可重载父类的event做类似hook的效果，或者在子类里调用父类的handler

   ```
   操作系统鼠标按下
        ↓
   Qt 生成 QMouseEvent
         ↓
   Qt 找到被点击的 QWidget
         ↓
   调用 QWidget::event(event)
         ↓
   event() 判断：这是鼠标按下事件
         ↓
   event() 内部调用 mousePressEvent(event)
   ```
- 事件处理函数总是返回book，T则代表处理了，无需继续传递给上层了

## event与signal

- 系统先产生event，调用对应的具体handler，handler里emit signal、接着调用对应的槽
- 槽才是真正进行业务处理的地方，完美解决了event与业务的解耦，实现了信号化封装

   ```
   [ 操作系统 ]
         ↓
   [ QMouseEvent ]
         ↓
   [ QLabel::event() ]
         ↓
   [ QmyLabel::mouseDoubleClickEvent ]
         ↓
   [ emit doubleClicked() ]
         ↓
   [ QmyWidget::do_doubleClicked ]
   ```
- event对应的handler只有一个，但可以emit多个signal，每个signal也可以有多个槽进行处理（正确的设计思想）
- 下面的案例演示了
   - Qt 检测到鼠标双击
   - Qt 调用 QLabel::event()
   - event() 识别出是 MouseButtonDblClick
   - Qt 调用你重写的 mouseDoubleClickEvent
   - 事件处理函数里没直接写业务逻辑，而是emit doubleClicked()，这一步完成了：“底层事件 → 高层信号” 的转换

   ```
   import sys

   from PyQt5.QtWidgets import  QApplication, QWidget,QLabel

   from PyQt5.QtCore import  pyqtSignal

   from PyQt5.QtGui import    QMouseEvent, QFont


   ## 自定义标签类，自定义了doubleClicked()信号
   class QmyLabel(QLabel): 
      doubleClicked = pyqtSignal()  ##自定义信号
      
      def mouseDoubleClickEvent(self,event):    ##双击事件的处理
         self.doubleClicked.emit()
         

   class QmyWidget(QWidget):
      def __init__(self, parent=None):
         super().__init__(parent) 
         self.resize(280,150)          
         self.setWindowTitle("Demo5_2,事件与信号")

         LabHello = QmyLabel(self) 
         LabHello.setText("双击我啊")

         font = LabHello.font()
         font.setPointSize(14)  
         font.setBold(True)     
         LabHello.setFont(font) 
         size=LabHello.sizeHint() 
         LabHello.setGeometry(70, 60, size.width(), size.height())
      
         LabHello.doubleClicked.connect(self.do_doubleClicked)

      def do_doubleClicked(self):  ##标签的槽函数
         print("标签被双击了")

      def mouseDoubleClickEvent(self,event): ##双击事件的处理
         print("窗口双击事件响应")
         
      
      
   ##  ============窗体测试程序 =================
   if  __name__ == "__main__":  
      app = QApplication(sys.argv) 
      form=QmyWidget() 
      form.show()
      sys.exit(app.exec_())
   ```

## eventFilter

- 对当前对象管理的子对象进行消息预处理的机制
- 可以做到不破环子对象的实现，且额外添加控制逻辑
- 使用时先定义过滤器函数（被重载的虚函数、返回T则截断了，见下代码），然后进行安装即可
- 一个过滤器可以监听多个对象，一个对象也可以被多个过滤器监听

```
import sys

from PyQt5.QtWidgets import  QApplication,  QWidget, qApp

from PyQt5.QtCore import  Qt,QEvent

##from PyQt5.QtGui import  QKeyEvent

from ui_Widget import Ui_Widget

class QmyWidget(QWidget): 
   def __init__(self, parent=None):
      super().__init__(parent)   #调用父类构造函数，创建窗体
      self.ui=Ui_Widget()        #创建UI对象
      self.ui.setupUi(self)      #构造UI界面

      self.ui.LabHover.installEventFilter(self)
      self.ui.LabDBClick.installEventFilter(self)
      qApp.processEvents()

##  ==================event处理函数=================================
   def eventFilter(self,watched,event):
      if (watched==self.ui.LabHover):     #上面的QLabel组件
         if (event.type()==QEvent.Enter): #鼠标光标进入
            self.ui.LabHover.setStyleSheet("background-color: rgb(170, 255, 255);")
         elif (event.type()==QEvent.Leave):     #鼠标光标移出
            self.ui.LabHover.setStyleSheet("")
            self.ui.LabHover.setText("靠近我，点击我")
         elif (event.type()==QEvent.MouseButtonPress):   #鼠标按键按下
            self.ui.LabHover.setText("button pressed")
         elif (event.type()==QEvent.MouseButtonRelease): #鼠标按键释放
            self.ui.LabHover.setText("button released")

      if (watched==self.ui.LabDBClick):   #下面的QLabel组件
         if (event.type()==QEvent.Enter):
            self.ui.LabDBClick.setStyleSheet("background-color: rgb(85, 255, 127);")
         elif (event.type()==QEvent.Leave):
            self.ui.LabDBClick.setStyleSheet("") 
            self.ui.LabDBClick.setText("可双击的标签")
         elif (event.type()==QEvent.MouseButtonDblClick):   #鼠标双击
            self.ui.LabDBClick.setText("double clicked")

      return super().eventFilter(watched,event)
      ##      return True  
##  =============自定义槽函数===============================        

   
##  ============窗体测试程序 ================================
if  __name__ == "__main__":     
   app = QApplication(sys.argv)
   form=QmyWidget()            
   form.show()
   sys.exit(app.exec_())
```

## processEvents

- 当queue里很多event待处理时候常常会ui无反应，最好是设计为多线程来解决
- 另外也可以使用processEvents函数先对某类event进行处理，其提供了几个分类可以先处理
- 案例待完善

# 拖放

待完善

# 绘图

## Window、Viewport

- Window：逻辑坐标空间（logical coordinate system），默认尺寸是QWidget的逻辑大小，即widget.width() × widget.height()，可以理解为要绘制的内容的全景的尺寸
- Viewport：设备坐标空间（device coordinate system），默认尺寸是逻辑大小 × devicePixelRatioF()，渲染后输出到的窗口尺寸，与window有映射公式

## 映射公式

```
x_device = (x_window - window.left) * viewport.width / window.width + viewport.left
y_device = (y_window - window.top)  * viewport.height / window.height + viewport.top
```

## 坐标系变化

- 绘图时使用的都是逻辑坐标，即逻辑像素DIP，与设备无法（防止DPI问题，是为了抽象出不同设备，做到可移植，设备坐标只存在于 Qt 内部的最后一步）
- 在默认不做任何变化的情况下，QPainter 的坐标系以当前绘制控件（QWidget）自身的左上角为 (0, 0)。即Window的尺寸等于QWidget尺寸，甚至viewport也一样，都是1:1
   - 假如当前都是300*200，从原点画线到右下角将占满整个viewport
   - 仅调用setWindow(0, 0, 600, 400)，线条终点在viewport中心，类似拉远了相机，看到的视野变大了，实现了缩放，适用于整体的缩放
   - 仅调用setWindow(-150, -100, 300, 200)，线条起点来到了viewport中心，相当于平移了相机，适用于整体滚动操作
   - 仅调用setViewport(150, 100, 300, 200)，线条起点在viewport中心，虽然也是缩放了，但没有改变相机，而是把原占满viewport的内容缩小为原1/4，并且移动到了右下角的格子，即改变了到视口的映射，适用于画中画、分屏的效果
 
- 所有操作都不是在动世界，而是在动“怎么看世界”的相机
   | 概念       | 类比      |
   | -------- | ------- |
   | 平移       | 移动相机|
   | 缩放       | 拉近 / 拉远相机 |
   | 旋转       | 转相机     |




# QT SQL

## 分类

- QSqlDatabase   连接
- QSqlQuery      执行 SQL
- QSqlTableModel 表模型
- QDataWidgetMapper 表单映射
- QSqlRecord  一行记录

| Model                        | 是否可改 | 说明          |
| ---------------------------- | ---- | ----------- |
| QAbstractItemModel || SQL 抽象接口       |
| QStandardItemModel || 内存表            |
| **QSqlQueryModel**           | ❌ 只读SQL 结果集 | 任意复杂 SQL 查询结果 |
| **QSqlTableModel**           | ✅ 可改 | 自动 CRUD 表模型     |
| **QSqlRelationalTableModel** | ✅ 可改 | 单表 + 外键     |

## SQLite

QT自带SQLite

## QSqlQueryModel

- 关键词只有三个：SQL 查询、结果集、只读。即可以把“一条 SQL 查询结果”包装成 Qt 的 Model（只读）
   ```
   数据库
   ↓
   QSqlQuery  （执行 SQL）
   ↓
   QSqlQueryModel  （结果 → Model）
   ↓
   QTableView / QListView （到 Model 为止，不负责写库）
   ```
- 继承自QAbstractItemModel，有 rowCount / columnCount、有 index / data、有 headerData
- 它和 QSqlTableModel 的“本质区别”
  
   | 对比项     | QSqlQueryModel | QSqlTableModel |
   | ------- | -------------- | -------------- |
   | SQL 自由度 | ⭐⭐⭐⭐⭐          | ⭐              |
   | 是否可写    | ❌              | ✅              |
   | 多表      | ✅              | ❌              |
   | 自动 CRUD | ❌              | ✅              |
   | 本质      | 查询结果           | 表对象            |

案例演示了在内存建库、加载读取并呈现

```
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QTableView
from PyQt5.QtSql import QSqlDatabase, QSqlQuery, QSqlQueryModel

class Demo(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("QSqlQueryModel 示例（内存数据库）")
        self.resize(400, 300)

        layout = QVBoxLayout(self)
        view = QTableView(self)
        layout.addWidget(view)

        # 1️⃣ 创建 SQLite 内存数据库（无需安装）
        db = QSqlDatabase.addDatabase("QSQLITE")
        db.setDatabaseName(":memory:")

        if not db.open():
            print("数据库打开失败")
            return

        # 2️⃣ 用 QSqlQuery 建表 & 插数据
        query = QSqlQuery()

        query.exec("""
            CREATE TABLE user (
                id   INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT,
                age  INTEGER
            )
        """)

        query.exec("INSERT INTO user (name, age) VALUES ('Alice', 20)")
        query.exec("INSERT INTO user (name, age) VALUES ('Bob', 30)")
        query.exec("INSERT INTO user (name, age) VALUES ('Charlie', 25)")

        # 3️⃣ 使用 QSqlQueryModel（只读）
        model = QSqlQueryModel()
        model.setQuery("SELECT id, name, age FROM user")

        # 设置表头（可选，但推荐）
        model.setHeaderData(0, 1, "ID")
        model.setHeaderData(1, 1, "姓名")
        model.setHeaderData(2, 1, "年龄")

        # 4️⃣ 绑定到 View
        view.setModel(model)



if __name__ == "__main__":
    app = QApplication(sys.argv)
    w = Demo()
    w.show()
    sys.exit(app.exec_())
```

## QSqlTableModel

- 相当于记录集的作用。负责和数据库直接打交道、执行SELECT / INSERT / UPDATE / DELETE等操作、记录“脏数据”（isDirty），可通过tabModel.setEditStrategy设置提交策略，如立即或非立即等


- 
```
打开库
self.DB=QSqlDatabase.addDatabase("QSQLITE") 
self.DB.setDatabaseName(dbFilename) 
self.DB.open()

打开表
self.ui.tableView.setModel(self.tabModel)
self.tabModel = QSqlTableModel(self, self.DB)
self.tabModel.setTable("employee")
self.tabModel.setEditStrategy(QSqlTableModel.OnManualSubmit)
self.tabModel.select()

获取字段名、设置字段名到view的header

设置model

```

## QDataWidgetMapper

- 作用是把数据表的字段与对应的控件建立映射，当前行变了后，SelectionModel发出currentRowChanged，需要手动调用mapper，这样控件内容就会自动更新了

```
self.mapper = QDataWidgetMapper()
self.mapper.setModel(self.tabModel)
self.mapper.addMapping(self.ui.dbSpinEmpNo, self.fldNum["empNo"])
self.mapper.addMapping(self.ui.dbEditName,  self.fldNum["Name"])
self.mapper.addMapping(self.ui.dbComboSex,  self.fldNum["Gender"])

def do_currentRowChanged(self, current, previous):
    self.mapper.setCurrentIndex(current.row())
```

- 通过 QSqlTableModel + View + Mapper，可以完整实现单表的增删改查，基本无需再手写 SQL。但一旦跨表、统计、性能、事务，就必须回到 QSqlQuery。

## QSqlRelationalTableModel

- 继承自QSqlTableModel，支持“外键关联”的表模型，使用setRelation设置
- 常用函数

   | 函数                                               | 功能                         |
   | ------------------------------------------------ | -------------------------- |
   | `setRelation(column, QSqlRelation(...))`         | 设置外键关联                     |
   | `relation(column)`                               | 获取 QSqlRelation 对象         |
   | `setJoinMode(QSqlRelationalTableModel.LeftJoin)` | 关联表查询方式，默认左连接              |
   | `submitAll()`                                    | 提交修改（增删改）                  |
   | `record(row)`                                    | 获取一行，仍然是 QSqlRecord（字段值原始） |
- 和 QSqlTableModel 的对比总结

   | 特性             | QSqlTableModel | QSqlRelationalTableModel       |
   | -------------- | -------------- | ------------------------------ |
   | 单表 CRUD        | ✔️             | ✔️                             |
   | 外键关联显示         | ❌              | ✔️                             |
   | 支持 Mapper      | ✔️             | ✔️                             |
   | 显示 ComboBox 编辑 | ❌              | ✔️ (需要 QSqlRelationalDelegate) |
   | 多表 JOIN / 聚合   | ❌              | ❌                              |
   | 手写 SQL         | 可选             | 可选                             |


```
model = QSqlRelationalTableModel()
model.setTable("employee")
model.setRelation(2, QSqlRelation("department", "deptId", "deptName"))
model.select()

view.setModel(model)
view.setItemDelegate(QSqlRelationalDelegate(view))
```

## QSqlQuery

- 它可以执行任意SQL，包括复杂查询 / JOIN / 聚合 / 自定义事务，直接与数据库交互，其实QSqlTableModel、QSqlQueryModel内部最终也调用 QSqlQuery 来执行 SQL，与QSqlTableModel 不同的是：

   | 类              | 优势                          | 劣势                |
   | -------------- | --------------------------- | ----------------- |
   | QSqlTableModel | 直接映射单表，Mapper 自动更新控件，易用     | 只能单表，复杂查询难        |
   | QSqlQuery      | 万能 SQL，支持复杂 JOIN / 聚合 / 子查询 | 需要手写 SQL，UI 不自动绑定 |

- QSqlQuery 的典型使用流程
   ```
   1️⃣ 创建 Query
   query = QSqlQuery(db)  # db 是 QSqlDatabase 对象
   2️⃣ 执行 SQL
   query.exec("SELECT id, name, salary FROM employee WHERE salary > 5000")
   3️⃣ 遍历结果集
   while query.next():
      id = query.value(0)       # 按列索引
      name = query.value("name")# 按字段名
      salary = query.value(2)
      print(id, name, salary)
   4️⃣ 获取字段元信息
   rec = query.record()  # QSqlRecord 对象
   print(rec.count())    # 字段数量
   print(rec.fieldName(1))  # 字段名
   5️⃣ 参数化 SQL（防止注入）
   query.prepare("INSERT INTO employee(name, salary) VALUES(?, ?)")
   query.addBindValue("张三")
   query.addBindValue(5000)
   query.exec()

   query.prepare("UPDATE employee SET salary=:s WHERE id=:i")
   query.bindValue(":s", 5500)
   query.bindValue(":i", 1)
   query.exec()
   6️⃣ 支持 DDL / DML
   query.exec("CREATE TABLE test(id INTEGER PRIMARY KEY, name TEXT)")
   query.exec("DELETE FROM employee WHERE id=10")
   ```

## QSqlRecord

- 代表数据库的一行记录的对象，只负责描述、存储一行数据（字段名、类型、值），被 Query 和 Model 用来读写数据库，也是 Mapper 修改数据时的中转站。类似这样：
   ```
   [
      字段0: name="id",    type=Int,    value=1
      字段1: name="name",  type=String, value="张三"
      字段2: name="age",   type=Int,    value=18
   ]
   ```
- 常用方法：
   - record.count()
   - record.fieldName(i)
   - record.indexOf("name")
   - record.value("name")
   - record.setValue("name", "李四")
   - record.isNull("age")
- 从哪里获取：
   - 从 QSqlQuery 里取
   ```
   query.exec("SELECT * FROM student")
   query.next()
   record = query.record()
   ```
   - 从 QSqlTableModel 里取
   ```
   record = model.record(row)
   ```
   - 用于“插入新行”
   ```
   record = model.record()
   record.setValue("name", self.editName.text())
   record.setValue("age", int(self.editAge.text()))

   if not model.insertRecord(-1, record):
      print(model.lastError().text())
   else:
      model.submitAll()
   ```

## QSqlField

- 是数据库字段的封装，它存储 字段名、类型、值、是否为空、是否只读 等信息。QSqlRecord是QSqlField 的集合（一行数据）。Mapper / TableModel / Record 都会用到它

- 核心属性
   ```
   field.name()  # 获取字段名
   field.setName("Salary")
   field.type()  # 返回 QVariant.Type，例如 Int, String
   field.setType(QVariant.Int)
   field.value()  # 返回当前值
   field.setValue(5000)  # 修改值
   field.isNull()
   field.setNull(True)
   field.isReadOnly()
   field.setReadOnly(True)
   ```
- 案例
   ```
   record = model.record(0)      # 第一行
   field = record.field("Salary") # 拿 Salary 字段
   print(field.value())           # 5000
   
   field.setValue(5500)
   record.setValue("Salary", field.value())

   获取字段名和类型
   record = model.record()
   for i in range(record.count()):
      field = record.field(i)
      print(field.name(), field.type())

   修改单个字段的值
   record = model.record(row)
   field = record.field("Salary")
   field.setValue(field.value() * 1.1)
   record.setValue("Salary", field.value())
   model.setRecord(row, record)
   ```