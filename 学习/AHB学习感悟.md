![[Pasted image 20260729032417.png]]
前两个信号为master发出命令“是谁”“干什么”，hwdata为具体要做的“具体动作”。
前三个周期可简单总结为：
    第一个周期，master发出命令“是谁”“干什么”  
    第二个周期，如果是写，master把数据放到hwdata上。收到命令的slave解读“要干什么”，如果可以立即处理就把ready拉高并作出行动。master发出第二个“是谁”“要干什么”。
    第三个周期，master检查ready信号，查看第一个周期发出的命令是否执行完毕。如果成功就在下一个周期发第二个“具体动作”。如果失败，就保持第一个的“具体动作”和第二个“是谁”“要干什么”不变。


#  “忽略已收到的地址和控制信号”是什么意思

这里的“收到”只是说：

> Slave 的输入引脚上能看到 `HADDR`、`HWRITE`、`HSIZE` 等电平。

它不代表这些信号一定构成有效请求。

AHB 的信号线每个周期都会有 0 或 1，不可能真正“没有值”。即使 `HTRANS=IDLE`，总线上仍可能显示：

```
HADDR  = 0x2000
HWRITE = 1
HSIZE  = 010
```

但因为：

```
HTRANS = IDLE
```

所以这些地址和控制信号没有传输意义。Slave 应理解为：

> 总线上虽然有这些数值，但 Master 并没有要求执行一次访问。

所谓“忽略”，就是 Slave 不应：

- 把 `HADDR` 锁存为一笔新请求；
- 根据 `HWRITE` 执行写操作；
- 发起存储器读操作；
- 修改内部寄存器或存储器；
- 把它计入一次有效访问；
- 更新突发传输计数器。



# IDLE 是什么意思

`IDLE` 表示：

> Master 当前没有要发起的有效总线访问。

例如：

```
HTRANS = IDLE
HADDR  = 0x1000
HWRITE = 1
```

这里的 `0x1000` 和 `HWRITE=1` 不能解释为“向 0x1000 写数据”，因为 `HTRANS=IDLE` 已经声明当前没有传输。

可以把 `HTRANS` 看成地址和控制信号的“有效标签”：

```
HTRANS = NONSEQ/SEQ：地址和控制有效
HTRANS = IDLE/BUSY ：地址和控制不构成有效访问
```

需要注意，AHB 是流水线总线。当前地址阶段为 `IDLE` 时，Slave 仍可能正在完成上一笔有效传输的数据阶段。

例如：

```
周期          T1                 T2
地址阶段      NONSEQ：读0x1000    IDLE
数据阶段      上一笔              返回0x1000的读数据
```

T2 的 `IDLE` 只表示“没有下一笔新地址传输”，不表示上一笔数据阶段不需要完成。


# select信号是由地址译码产生的，而不是直接由地址决定的，地址可以具体到slave里的某一个寄存器。

# MUX用来决定哪一个slave返回数据，MUX默认返回ready为1（防止第一笔地址拿不到），其他时间当slave返回ready为1时，MUX才会切换slave。地址译码器用来根据master的addr产生sel信号，选中某个slave。


# master的切换也要等到ready信号拉起。为了保证MUX已经把上一个地址锁存。

# SPLIT/RETRY 使用两周期响应，目的正是给 Master 一个周期，将已经广播的下一笔传输撤销，并把 `HTRANS` 改成 `IDLE`。这里 `HREADY=1` 的含义不是“传输 A 成功了”，而是： 传输 A 以 SPLIT 结果结束，当前 Master 可以退出总线。在 T4 时钟沿，Master 正式接收到完整的 SPLIT 响应。协议要求 Master 在收到 SPLIT 或 RETRY 后立即执行一个 `IDLE` 传输，以便总线交给另一个 Master。## 普通等待和 SPLIT 的区别

| 响应                      | Master 如何处理下一笔地址    |
| ----------------------- | ------------------- |
| `HREADY=0, HRESP=OKAY`  | 保持地址和控制不变，继续等待      |
| `HREADY=0, HRESP=SPLIT` | 识别特殊响应，取消下一笔传输      |
| `HREADY=1, HRESP=SPLIT` | 当前传输以 SPLIT 结束，让出总线 |
| `HREADY=1, HRESP=OKAY`  | 当前传输成功完成，正常进入下一笔    |

![[Pasted image 20260730135326.png]]



