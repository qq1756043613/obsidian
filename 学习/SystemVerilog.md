VCS 编译命令
例：vcs -full -sverilog test.sv -R
vcs:                           仿真器可执行程序，位置$VCS_HOME/bin/vcs
-full64                      支持64-bit模式
-sverilog                  支持SystemVerilog
-test.sv                     源码，-f filelist
-R                             编译完成后执行，不加-R则仅编译，./simv
-+v2k                       Verilog-200*
-ntb_opts uvm         把整个UVM的框架、底层吃进来
-kdb                         给 verdi 用的，编译一个源码出来，给 verdi 做源码分析
-l                              把编译结果写道 log 文件里面
(native testbench options)


![[Pasted image 20260422105207.png|291]]
![[Pasted image 20260422105625.png|664]]

![[Pasted image 20260422181926.png]]

只有 Wire 类型能够支持多驱驱动（通常代码没有多驱）

![[Pasted image 20260422225648.png]]

![[Pasted image 20260423004105.png]]

定长数组在编译时已经在内存中确定好长度。

![[Pasted image 20260423005210.png]]
![[Pasted image 20260423005344.png]]

![[Pasted image 20260423010359.png]]

![[Pasted image 20260423010643.png]]

关联数组
![[Pasted image 20260423011531.png]]

#### 场景 A：用字符串查找整数

代码段

```
int age_map [string]; 
```

- **存什么？** `int`（整数）。
    
- **怎么找？** 用 `string`（字符串）。
    
- **操作：** `age_map["Tom"] = 25;` —— 这里存入的是 **25**。
    
### 1. 数量与清理方法

- **`num()` 或 `size()`**: 返回关联数组中当前存储的元素总数。
    
- **`delete([index])`**:
    
    - 如果提供了索引（key），则删除该特定条目。
        
    - 如果没有提供参数，则清空整个数组。
        

### 2. 存在性检查

- **`exists(index)`**: 检查给定的索引是否存在。
    
    - 如果存在返回 `1`；
        
    - 如果不存在返回 `0`。
        

### 3. 遍历与索引寻址

关联数组由于键值通常是非连续的（如散列或随机 ID），因此需要专门的导航方法：

- **`first(index)`**: 获取数组中的**第一个**索引，并将其赋值给变量 `index`。如果数组为空，则返回 `0`。
    
- **`last(index)`**: 获取数组中的**最后一个**索引。
    
- **`next(index)`**: 获取指定索引 `index` 之后的下一个索引。
    
- **`prev(index)`**: 获取指定索引 `index` 之前的前一个索引。
    

> **注意**：在使用 `first`、`next` 等方法时，返回的是逻辑顺序（通常根据键的大小排序）。


transaction 事务，可以认为是一包数据，和package不一样，一般认为package大一些，transaction小一些。

![[Pasted image 20260423013349.png]]
在正常变量声明前加一个rand，就变成了随机变量。

![[Pasted image 20260423014150.png]]


**过程描述**

![[Pasted image 20260423210238.png|689]]

![[Pasted image 20260423211757.png]]
可以把不耗时的task定义为void function

![[Pasted image 20260424122701.png]]

**自动存储**
![[Pasted image 20260424125529.png]]
静态函数创建的是静态变量，函数执行完后创建的变量不会销毁，但每次传入的参数是可以改变的。

**静态变量**
![[Pasted image 20260424130644.png]]
在 SystemVerilog 的 **静态存储**（默认情况）中，局部变量的初始化遵循以下诡异的规则：

- **编译期初始化**：`int data = ini << 2;` 中的赋值动作发生在**仿真时间 0 刻之前**（编译阶段）。
    
- **只执行一次**：无论你调用多少次 `task check`，这行赋值语句只在程序开始前跑一遍。
    
- **变量状态**：因为在仿真开始前 `ini` 的值还是默认的 `0`（或者未定义），所以 `data` 在仿真还没开始时就被设为了 $0 \ll 2 = 0$。

当你把 `program` 或 `task` 声明为 `automatic` 时，规则发生了彻底改变：

- **运行时初始化**：`automatic` 变量是在 **任务被调用时** 动态分配在“栈”上的。
    
- **每次调用都执行**：每当程序执行到 `check` 任务，系统会实时地在栈上划出一块空间，并**立刻执行**那行赋值语句 `data = ini << 2`。
    
- **实时性**：此时 `ini` 已经被你在 `initial` 块中改成了 `1`。因此，赋值动作会实时计算 $1 \ll 2$，结果就是 **4**。
    
- **销毁与重来**：任务结束，这个 `data` 连同它的值一起从栈上销毁。下次调用再重新计算。

**OOP**
![[Pasted image 20260425112709.png]]

![[Pasted image 20260425112852.png]]

