---
title: C++ GUI Qt4编程-阅读笔记
date: 2012-06-10 09:27:38
tags:
- cpp
---

由  王宇 原创并发布 ： 

## 第一章Qt入门 

### 1.1 HellowQt 

qmake-project

qmakehello.pro

make

### 1.2 建立连接 

QObject::connect(button,SIGNAL(clicked()),&app,SLOT(quit());

### 1.3 窗口部件的布局

### 1.4 使用参考文档：QtAssistant 

<!--more-->

## 第二章创建对话框 

### 2.1 子类化QDialog

### 2.2 深入介绍信号和槽 

connect(sender,SIGNAL(signal),receiver,SLOT(slot))

一个信号可以连接多个槽

多个信号可以连接同一个槽

一个信号可以与另外一个信号相连接

连接可以被移除 

### 2.3 快速设计对话框 

QtDesigner

主程序：UI::GoToCellDiaglogui;QDialog*dialog=newQDialog();ui.setUi(dialog);dialog->show();

### 2.4 改变形状的对话框 

扩展对话框

扩展对话框通常只显示简单的外观，但是他还有一个切换按键，可以让用户在对话框的简单外观和扩展外观之间来回切换

多页对话框：QTabWidget

2.5动态对话框 

动态对话框就是在程序运行是使用的从Qt设计师的.ui文件创建而来的那些对话框。动态对话框不需要通过uic把.ui文件转换成C++代码，相反它是程序运行的时候使用QUiLoader类载入该文件的

QUiLoaderuiLoader;QFilefile("sortdialog.ui");QWidget*sortDialog=uiLoader.load(&file);

2.6内置的窗口部件类和对话框类 

Qt的按键部件

QPushButton、QToolButton、QCheckBox、QRadioButton

Qt的显示窗口部件

QLabel、QLCDNumber、QProgreeBar、QTextBrowser

Qt的输入窗口部件

QTextEdit、QSpinBox、QDoubleBox、QComboBox、QDataEdit、QTimeEdit、QDateTimeEdit、QScrollBar

Qt单页容器窗口部件

QGroupBox、QFrame、QTabWidget、QToolBox

Qt的项目视图窗口部件

QListView、QTreeView、QTableView

Qt的反馈对话框

QMessageBox、QInputDialog、QProgressDialog、QErrorMesage

Qt的颜色对话框和字体对话框

QColoerDialogQFontDialog

Qt文件对话框和打印对话框

QFileDialog、QPrintDialog

第三章创建主窗口 

3.1子类化QMainWindow

3.2创建菜单和工具栏

3.3设置状态栏

3.4实现File菜单

3.5使用对话框

3.6存储设置

3.7多文档

3.8程序启动动画 

第四章实现应用程序的功能

第五章创建自定义窗口部件 

5.1自定义Qt窗口部件 

通过继承子类的方法

5.2子类化QWidget 

许多自定义窗口部件都是对现有窗口部件的简单组合，不论它们是内置的Qt窗口部件，还是其他一些部件，通过对现有窗口部件的组合构建而成的自定义窗口部件

5.3在Qt设计师中集成自定义窗口部件 

改进法是最快捷和简单的方法

改进法的缺点：在QTDesigner中，无法对自定义窗口部件中的那些特定属性进行访问，并且也无法对这个窗口部件自身进行绘制

插入法需要创建一个插件库，QtDesigter可以在运行时加载这个库，并且可以利用该库创建窗口部件的实例

5.4双缓冲 

是一种图形用户界面编程技术，它包括把一个窗口部件渲染到一个脱屏像素映射中以及把这个像素映射复制到显示器上

第六章布局管理 

布局类：QHBoxLayout、QVBoxLayout、QGridLayout、QStractLayout 

6.1在窗口中摆放窗口部件 

绝对位置法：窗口部件分配固定的大小和位置

人工布句法：给定的大小尺寸总是可以和窗口的大小成比例

布局管理法：QHBoxLayout、QVBoxLayout、QGridLayout 

6.2分组布局 

QStackedLayout类可以对一组子窗口部件进行摆放，或者对它们进行“分页”，而且一次只显示其中一个，而把其他的子窗口部件或者分页都隐藏起来

6.3切分窗口 

QSplitter就是一个可以包含一些其他窗口部件的窗口部件。例如资源管理器中的左右结构

6.4滚动区域 

QScrollArea类提供了一个可以滚动的视口和两个滚动条

6.5停靠窗口和工具栏 

停靠窗口是指一些可以停靠在QMainWindow中或是浮动为独立窗口

四个停靠窗口区域：

中央窗口部件的上部、下部、左侧、右侧

6.6多文档界面 

第七章事件处理 

事件是有窗口系统或者Qt自身产生的，用以响应所发生的各类事情

不应该混淆“事件”和“信号”这两个概念。一般情况下，在使用窗口部件的时候，信号是十分有用的；而在实现窗口部件时，事件则是十分有用的 

例如，当使用QPushButton时，我们对于它的clicked()信号往往更为关注，而很少关心促成发射该信号的底层鼠标或者键盘事件

7.1重新实现事件处理器

7.2安装事件过滤器 

5个级别的事件处理和事件过滤方法

1、重新实现特殊的事件处理器

2、重新实现QObjec::event();

3、在QObject中安装事件过滤器

4、在QApplication对象中安装事件过滤器

5、子类化QApplication并且重新实现notify()

  7.3处理密集时的响应保持 


  第八章二维图形 

  Qt的二维图形引擎是基于QPainter类的

  几何形状：点、线、矩形、椭圆、饼状图等

  高级特性：反走样（文字和图形边缘）、像素混合、渐变填充和矢量路径

  QPainter可以画在“绘图设备”上，例如：QWideget、QPixmap、QImage；也可以用来打印文件和创建PDF

  可以使用OpenGL命令来替代QPainter

  8.1用QPainter绘图 

  三个主要工具：画笔、画刷、字体 

  画笔用来画线和边缘。它包括颜色、宽度、线形、拐点风格及连接风格

  画刷用来填充几个形状的图案

  字体用来绘制文字。

  QPainterPath类可以通过连接基本的图形元素来确定任意的矢量形状：直线、椭圆、多边形、弧形和其他的绘制路径

  绘制路径是基本的图元，从这个意义上来说，任何图形或图形组合都可以用绘制路径描述

  8.2坐标系统变换 

  理论上，像素的中心取决于半像素坐标。

  如果告诉QPainter绘制一个像素。例如：（100,100）它会相应地在两个方向做+0.5的偏移，使得像素点得中心位置在（100.5,100,5）

  这一差别初看起来理论性很强，但它在实践中却很重要。首先只有当反走样无效时（默认情况）才偏移+0.5；如果反走样有效，并且我们试图在（100,100）的位置绘制一个黑色的像素，实际上QPainter会为（99.5,99.5）（99.5,100.5）（100,5,99.5）(100.5,100.5)四个像素点着浅灰色，给人的印象是一个像素正好位于四个像素的重合处

  如果不需要这种效果，可以通过指定半像素坐标或者通过偏移QPainter（+0.5,+0.5）来避免这种效果的出现

  世界变换是在窗口-视口转换之外使用的变换矩阵。它允许移动、缩放、旋转或者拉伸绘制的项

  8.3用QImage高质量绘图 

  绘图时，我们可能需要面对速度和准确率的折中问题

  Qt受限于平台的内在支持：

  在X11上，类似反走样以及对分数坐标的支持只有当X服务器上存在X渲染扩展时才有效

  在MacOSX上，内置的走样绘图引擎使用与X11和Windows不同的算法绘制多边形，绘制结果也稍有不同

  Qt图形引擎的一个特别强大的特性是它支持复合模式

  默认的复合模式是QImage::CompositionMode_SourceOver,这意味着源像素（正在绘制的像素）被混合在目的像素（已存在的像素）上，这样，源图像的透明部分给我们以透明效果

  8.4基于项的图形视图 

  如果需要处理从几个到几万的项时，而且要求用户能够点击、拖动、和选取项，Qt的视图类提供了对这一个问题的解决方案

  Qt的视图体系包括一个由QGraphicScene充当的场景和一些由QGraphicItem的子类充当

  8.5打印 

  Qt中的打印与在QWidget、QPixmap或者QImage上的绘制非常相似，它包含以下步骤： 

  1、创建一个当作绘制设备的QPrinter.

  2、弹出一个QPrinterDialog对话框，以允许用户选择打印机并且设置一些选项

  3、创建一个在QPrinter上操作的QPainter

  4、使用QPainter绘制一页

  5、调用QPainter::newPage（）来进行下一页的绘制

  6、重复步骤4和步骤5，直到所有页都被打印为止

  第九章拖放 

  拖放是在一个应用程序内或者多个应用程序之间传递信号的一种直观的现代操作方式

  9.1使拖放生效 

当用户把一个对象拖动到这个窗口部件上时，就会调用dragEnterEvent()

当用户在窗口部件上放下一个对象时，就会调用dropEvent()

QWidget也提供dragMoveEvent()和dragLeaveEvent() 

  9.2支持自定义拖动类型

  9.3剪贴板处理技术 

  第十章项视图类

  第十一章容器类 

  容器类通常是用于在内存中存储给定类型的许多项的模板类

  既可以使用Qt容器也可以使用STL容器

  Qt容易的主要优点是它们在所有的平台上在运行时都表现得一致，并且它们都是隐含共享的

  隐含共享或者称为“写时复制”是一个能够把整个容器作为不需要太多运行成本的值来传递的最优化过程

  Qt容器的另一个主要特征就是易于使用的迭代器类，这是从java中得到的灵感，它们可以利用QDataStream变成数据流，而且它们通常可以使可执行文件中的代码量比相应的STL类中的要少

  最后在Qt/EmbededLinux支持的一些硬件平台上，通常只能使用Qt容器

  Qt提供了：QVector<T>、QLinkedList<T>、QList<T>、QMap<K,T>、QHash<K,T> 

  QString、QByteArrag和QVariant与容器有很多相似之处

  11.1连续容器 

  QVector<T>是一种与数组相似的数据结构，它可以把项存储到内存中相邻近的位置

  向量与普通C++数组的区别在于：向量知道自己的大小并且可以被重新定义大小

  对比较大的向量来说，在QVector<T>的开头或者中间插入项，或者在这些位置去除项，都是非常耗时的，因此Qt提供了QLinkedList<T>这是一种把项存储到内存中不相邻位置的数据结构（链表）

  QList<T>连续容器是一个“数组列表”，结合了单一类中QVector<T>和QLinkedList<T>的最重要的优点 

  除非我们想在一个极大的列表中执行插入或者要求列表中的元素都必须占据连续的内存地址，否则QList<T>通常是最合适采用的多用途容器类

  QStringList类是被广泛用于Qt应用编程接口的QList<QString>,还提供一些特别的函数，以使得这种类对字符串的处理方式更通用

  11.2并联容器 

  并联容器可以保存任意多个具有相同类型的项，且它们由一个键索引：主要提供：QMap<K,T>QHash<K,T>

  QMap<K,T>是一个以升序键顺序存储键值对的数据结构，这种排列使它可以提供良好的查找和插入性能以及键序的迭代

  QHash<K,T>是一个在哈希表中存储键值对的数据结构，与QMap<K,T>相比，它对K的模板类型有不同的要求，而且它提供了比QMap<K,T>更快的查找功能

  11.3通过算法 

  <QAlgorithms>类在容器上实现基本算法的一套全局模板函数

  qFind(list.begin(),list.end(),"key"); 

  qFill组装一个容器

qCopy()

qSort()、qStableSort()

qDeleteAll()

qSwap() 

  11.4字符串、字节组和变量 

  QString、QByteArray、QVariant使用隐含共享来最优化内存和速度 

  QString不仅仅是为用户界面，更多的是为数据结构所用

QString转换为constchar*,可以使用toAscii()或toLatin1()

  QByteArray上调用data()或constData()时，返回的字符串属于QByteArray对象。这就意味着不必为内存泄漏而担心，Qt将为我们重新收回内存

  另一方面，必须注意不要太长时间地使用指针

  第十二章输入与输出 

  从文件或者其他设备读取或者写入数据几乎是每个应用程序共有的特点

  Qt通过QIODevice为输入输出提供了极佳的支持

  QIODevice是一个封装能够读写字节块“设备”的强有力的提取器

  QFile

  QTemporaryFile

  QBuffer

  QProcess

  QTcpSocket

  QUdpSoclet

  QSslSocket 

  QProcess、QTcpSocket、QUdpSocket和SslSocket都是顺序存储设备，这意味着所存储的数据从第一个字节开始到最后一个字节为止只能读取一次

  QFile、QTemporayFile和QBuffer则是随机存取设备，因此可以从任意位置多次读取字节位所存储的数据

  QDataStream用来多写二进制数据，QTextStream用来读写文本数据

  QFile使存取单个文件变得简单

  QProcess类允许启动外部程序并通过标准输入、输出以及标准错误通道与外部程序交互

  网络与XML的读写，都是非常重要的主题

  12.1读取和写入二进制数据 

  Qt中载入和保存二进制数据的最简单方式是通过实例化一个QFile打开文件，然后通过QDataStream对象存取它

  QDataStream提供了一种与运行平台无关的存储格式，它不仅支持QList<T>和QMap<K,T>等Qt容器类，还支持整型和双精度等基本的C++类，以及其他许多Qt数据类型

  12.2读取和写入文本 

  Qt提供了QTextStream类读写纯文本文件以及如HTML、XML和源代码等其他文件格式的文件 

  QText Stream

  第十三章数据库



  QSqlDatabase QSqlQuery QSqlTableModel QTableView QSqlTableModel


  第十四章多线程 

  QT hread QMutex QReadWriteLock QSemaphore QWaitCondition

  第十五章网络

  第十六章XML



  QXmlStreamReader QXmlSimpleReader QXmlStreamWriter

  第十七章提供在线帮助

  第十八章国际化

  第十九章自定义外观

  第二十章三维图形





  第二十一章创建插件

  第二十二章应用程序脚本
