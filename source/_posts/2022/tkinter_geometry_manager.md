---
title: Python tkinter 布局管理
date: 2022-02-16 23:56:21
tags: [Python, tkinter, GUI]
categories: Python
---

本文讲述如何使用 tkinter 的布局管理 (被称作 layout managers 或 geometry managers)。tkinter 有三种布局管理方式：
- pack
- place
- grid

> 注意，这三种布局管理在同一个 master window 里一定不该被混用！

布局管理有以下功能：

- 在屏幕上排列控件，包括确定组件的大小和位置
- 注册窗口控件到底层窗口系统
- 管理控件在屏幕上的显示

虽然控件自己也可以指定大小和对齐方式等信息， 但最终的控件大小及位置还是由布局管理决定的。

<!-- more -->

# Pack 布局管理器
Pack布局管理器按行或列打包控件。您可以使用fill，expand和side等选项来控制此布局管理器。

管理器处理在同一master widget中打包的所有窗口小部件。打包算法很简单，但有点不太好用文字描述;想象一块弹性材料，中间有一个非常小的矩形孔。对于每个窗口控件，按照打包的顺序，布局管理器使孔足够大以容纳窗口控件，然后将其放置在给定的内边缘（默认为上边缘）。然后它重复所有小部件的过程。最后，当所有窗口小部件都被打包到孔中时，管理器计算所有窗口小部件的边界框，使master widget足够大以容纳所有窗口小部件，并将它们全部移动到主窗口。

pack 是三种布局管理器中最容易使用的。我们可以用pack方法声明控件之间的相对位置，而不必精确地指定控件在屏幕上的位置。pack 布局管理器会自动处理好控件的这些细节。虽然pack更容易使用，但相较于place和grid，这种布局管理器的灵活性也受到限制。对于简单的应用程序，pack肯定是首选。以下几种情况比较适合使用pack布局管理器：
- 将控件放在frame（或任何其他容器控件）中，并让它填充整个frame
- 从上到下逐个放置一些控件
- 并排放置一些控件

示例：

```python
import tkinter as tk

if __name__ == "__main__":
    root = tk.Tk()
    
    tk.Label(root, text="Red Sun", bg="red", fg="white").pack()
    tk.Label(root, text="Green Grass", bg="green", fg="black").pack()
    tk.Label(root, text="Blue Sky", bg="blue", fg="white").pack()

    root.mainloop()
```

![img](20170114201534466.png)

如果需要创建更复杂的布局，通常需要使用额外的Frame控件对控件进行分组。您也可以使用grid布局管理器。



## fill 选项：控制填充
在上面那个例子里, 我们简单的将三个 Label 控件 pack 到父控件`root`上，没有使用任何属性。 因此，pack必须决定以哪种方式来排列这些 Label 控件。可以看到，pack默认使用`pack(side=tkinter.TOP)`方式进行布局，即从上到下依次放置，并水平居中。 同时，我们也发现 pack 默认会将 Label 控件的大小设置为文本的大小。如果你想让这些控件和其父控件一样宽, 可以使用`fill=tkinter.X`属性，使其水平横向填充父控件：

```python
import tkinter as tk

root = tk.Tk()
w1 = tk.Label(root, text="Red Sun", bg="red", fg="white")
w1.pack(fill=tk.X)
w2 = tk.Label(root, text="Green Grass", bg="green", fg="black")
w2.pack(fill=tk.X)
w3 = tk.Label(root, text="Blue Sky", bg="blue", fg="white")
w3.pack(fill=tk.X)

root.mainloop()
```



![img](20170114202405976.png)



## padding 选项：控件边距 



Pack 可以在四个方面控制控件边距: 内边距, 外边距, 水平边距, 垂直边距:



**padx** - 设置水平方向的外边距



```python
from Tkinter import *



root = Tk()



w = Label(root, text="Red Sun", bg="red", fg="white")



w.pack(fill=X,padx=10)



w = Label(root, text="Green Grass", bg="green", fg="black")



w.pack(fill=X,padx=10)



w = Label(root, text="Blue Sky", bg="blue", fg="white")



w.pack(fill=X,padx=10)



mainloop()
```



![img](20170114202920393.png)



**pady** - 设置竖直方向的外边距



```python
from Tkinter import *



root = Tk()



w = Label(root, text="Red Sun", bg="red", fg="white")



w.pack(fill=X,pady=10)



w = Label(root, text="Green Grass", bg="green", fg="black")



w.pack(fill=X,pady=10)



w = Label(root, text="Blue Sky", bg="blue", fg="white")



w.pack(fill=X,pady=10)



mainloop()
```



![img](20170114203036003.png)



**ipadx** - 设置水平方向的内边距



```python
from Tkinter import *



root = Tk()



w = Label(root, text="Red Sun", bg="red", fg="white")



w.pack()



w = Label(root, text="Green Grass", bg="green", fg="black")



w.pack(ipadx=10)



w = Label(root, text="Blue Sky", bg="blue", fg="white")



w.pack()



mainloop()
```



![img](20170114203323832.png)



**ipady** - 设置竖直方向的内边距



```python
from Tkinter import *



root = Tk()



w = Label(root, text="Red Sun", bg="red", fg="white")



w.pack()



w = Label(root, text="Green Grass", bg="green", fg="black")



w.pack(ipadx=10)



w = Label(root, text="Blue Sky", bg="blue", fg="white")



w.pack(ipady=10)



mainloop()
```



![img](20170114203423489.png)



上述四个属性的默认值都是 0.



## side选项：顺次放置控件



我们把上面那几个 Label 从左到右放在一排:



```python
from Tkinter import *

root = Tk()

w = Label(root, text="red", bg="red", fg="white")
w.pack(padx=5, pady=10, side=LEFT)
w = Label(root, text="green", bg="green", fg="black")
w.pack(padx=5, pady=20, side=LEFT)
w = Label(root, text="blue", bg="blue", fg="white")
w.pack(padx=5, pady=20, side=LEFT)

mainloop()
```



![img](20170114203536054.png)



如果把上述 side 属性的值都改为 RIGHT, 那么上面 Label 控件的排列顺序就反过来了:

![img](20170114204013789.png)



# Place 布局管理



Place 布局管理可以显式的指定控件的绝对位置或相对于其他控件的位置. 要使用 Place 布局, 调用相应控件的 place() 方法就可以了. 所有 tkinter 的标准控件都可以调用 place()

 方法.

下面是一个使用 Place 布局的例子: 为 Label 控件设置随机的背景色, 然后计算各个 Label 的背景色的亮度(灰度值), 如果其亮度小于 120, 则将其前景色(文字颜色, fg属性)设置为白色, 否则设为黑色. 这样做是为了避免使背景色和前景色过于接近而导致文字不易阅读.



```python
import Tkinter as tk



import random



    



root = tk.Tk()



# width x height + x_offset + y_offset:



root.geometry("170x200+30+30") 



     



languages = ['Python','Perl','C++','Java','Tcl/Tk']



labels = range(5)



for i in range(5):



   ct = [random.randrange(256) for x in range(3)]



   brightness = int(round(0.299*ct[0] + 0.587*ct[1] + 0.114*ct[2]))



   ct_hex = "%02x%02x%02x" % tuple(ct)



   bg_colour = '#' + "".join(ct_hex)



   l = tk.Label(root, 



                text=languages[i], 



                fg='White' if brightness < 120 else 'Black',bg=bg_colour)
   l.place(x = 20, y = 30 + i*30, width=120, height=25)
root.mainloop()
```



![img](20170114205223623.png)



# Grid 布局管理



Pack 作为首选的布局管理方式, 其运作方式并不是特别易于理解. 已经由 Pack 布局完成的设计也很难做出改变. Grid 布局在1996年作为另一种可供选择的布局方式被引入. Grid 布局方式易学易用, 但似乎大家还是习惯用 Pack.

Grid 在很多场景下是最好用的布局方式. 相比而言, Pack 布局在控制细节方面有些力不从心. Place 布局虽然可以完全控制控件位置, 但这也导致使用 Place 会比其他两种布局方式更加复杂.

Grid 把控件位置作为一个二维表结构来维护, 即按照行列的方式排列控件: 控件位置由其所在的行号和列号决定. 行号相同而列号不同的几个控件会被彼此上下排列; 列号相同而行号不同的几个控件会被彼此左右排列.

使用 Grid 布局的过程就是为各个控件指定行号和列号的过程. 不需要为每个格子指定大小, Grid 布局会自动设置一个合适的大小.

下面还是举个栗子吧:
```python
from Tkinter import *

colours = ['red','green','orange','white','yellow','blue']
r = 0
for c in colours:
    Label(text=c, relief=RIDGE,width=15).grid(row=r,column=0)
    Entry(bg=c, relief=SUNKEN,width=10).grid(row=r,column=1)
    r = r + 1
mainloop()
```



![img](20170114210830212.png)