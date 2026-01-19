# **BIUH智元机器人智元灵犀x1上位机软件使用指南**  

如果看完这篇文章还有疑问请移步智元机器人官网：https://www.zhiyuan-robot.com/DOCS/OS/X1-PDG
---------------------------
## **1 准备工作**

### **1.1 软件准备**

在自己电脑（windows系统）上下载上位机软件:https://www.zhiyuan-robot.com/file/ueditor/php/upload/file/20250114/x1/REF-CLI%20v1.0.3.exe

### **1.2 硬件准备**

USB-TYPEC线，L28推杆需要用4pinUSB线。
---
<p align="center">
<img src="https://github.com/liusihan54/ro/raw/main/webwxgetmsgimg%20(2).jpg" width="50%" alt="推杆线示意图">
</p>

## **2 调试**

*注意，所有的代码前面的设备号要换成实际的设备号，演示代码的设备号只是一个例子*

用USB-TYPECL将执行器连接到自己电脑上面。若连接成功，上位机软件会显示软件号，比如说已连接REF xxxxxxxxxxxx 设备作为ref0.若没有任何东西显示出来，则要检查执行器是否成功上电，上电成功正常会闪烁绿灯。若上电成功但仍然没有反应，则换一根数据线。

---
<p align="center">
<img src="https://github.com/liusihan54/ro/raw/main/webwxgetmsgimg.jpg" width="500" alt="连接成功后显示如下">
</p>

连接成功后输入对应的设备号，比如上面显示连接的设备为ref3，则输入ref3并按回车，会有一些执行器的基本参数输出。

若要查看执行器内部参数模式，则输入
```python
ref0.motor.config
```

执行器上电后默认为失能状态，也就是绿灯闪烁状态，我们可以通过以下指令进行状态切换：
```python
ref0.motor.request_state(1)
ref0.motor.request_state(0)
```
其中，state(1)为使能状态，绿灯常亮，state(0)为失能状态，绿灯闪烁。

确认完改执行器没有问题后可以进行设置ID操作，每个执行器的ID需不一样，以便区分不同的执行器。

```python
ref0.can_node_id
```
这个代码可以查看该执行器的节点ID

```python
ref0.can_node_id=0
```
这个代码可以设置该执行器的节点ID，后面的数字根据实际情况来定。

设置完节点ID就可以保存了

*注意，所以参数的更改都需要保存，而保存前需要使执行器处于失能状态*

```python
ref0.save_config()//保存所有参数，返回True则成功
```

### **2.1 L28推杆调试**

L28推杆调试特殊一点，需要用到4pinUSB线。

前面的使能，失能状态切换，设置ID都是同上操作。接下来讲一下L28推杆的特殊操作。

L28推杆上电后会自动收缩到零点，但是我们可以让它进行伸缩操作。
```python
ref0.motor.ctrl.set_pos(0) // 将L28运动至零点处
ref0.motor.ctrl.set_pos(1) // 将L28运动至2.53mm处
ref0.motor.ctrl.set_pos(5) // 将L28运动至12.63mm处
ref0.motor.ctrl.set_pos(9.5) // 将L28运动至24mm处
```
L28推杆至多运动到24mm处，也就是pos（9.5）。

但是上述代码只能让L28推杆伸缩到一个具体位置之后就不能动了。聪明的小伙伴肯定想到了我们还可以写一个循环函数，让L28推杆重复运动。
那么我来写个例子
```python
import time
for i in range(10):
  ref0.motor.ctrl.set_pos(0)
  time.sleep(3)
  ref0.motor.ctrl.set_pos(9.5)
  time.sleep(3)
```
这样L28推杆就会重复伸缩10次

### **2.2 OmniPicker调试**

OmniPicker调试是用到USB-TYPEC线，前面的使能，失能状态切换，设置ID都是同上操作。接下来讲一下OmniPicker的特殊操作。

正常情况下，OmniPicker上电后会自动闭合归零，那么我们其实也是可以通过代码来控制夹爪的闭合程度的。
```python
ref0.motor.ctrl.set_pos(0)//控制夹爪闭合
ref0.motor.ctrl.set_pos(0.5)//控制夹爪张开一半
ref0.motor.ctrl.set_pos(1)//控制夹爪全部张开
```
*注意，夹爪和L28推杆不同，夹爪pos()的最大值只有1*

那么同理，我们也可以写一个循环让夹爪一直开合
```python
import time
for i in range(10):
  ref0.motor.ctrl.set_pos(0) // 控制夹爪为闭合状态
  time.sleep(3)
  ref0.motor.ctrl.set_pos(0.5) // 控制夹爪张开一半
  time.sleep(3)
  ref0.moto.ctrl.set_pos(1)//控制夹爪全部张开
  time.sleep(3)
```
