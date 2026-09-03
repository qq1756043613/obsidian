![[Pasted image 20260812154604.png]]

ACE 是 **AMBA Coherency Extensions**，中文通常叫 **AMBA 一致性扩展协议**。
CPU0                                  CPU1
 │                                          │
L1 Cache                          L1 Cache
 │                                          │
 └────── Interconnect ──┘
             │
           Memory
ace是为了解决多个cache之间的一致性问题。例如有cpu0和cpu1，它们各自有L1 cache 和 L2 cache，当cpu0读取内存中的值并修改后，cpu1可能并不知道。
ace在普通axi基础上增加了与cache一致性有关的通道。
AC Snoop Address 
CR Snoop Response 
CD Snoop Data

什么是snoop？
可以理解为：去问其他 Cache：“你那里有没有这个地址的数据？”。例如cpu0要读 0x1000，系统不一定直接去ddr，应为cpu1的cache里可能已经有这个地址，而且数据比dd更新。于是 Coherent Interconnect 可以向 CPU1：
Interconnect
     │
     │ AC: “你有没有 0x1000？”
     ▼
   CPU1 Cache
   
CPU1 回：
CR:
“有，而且我这里有最新数据。”

甚至通过：CD，把数据直接返回。

整个过程：
CPU0
 │
 │ Read 0x1000
 ▼
Coherent Interconnect
 │
 │ AC Snoop
 ▼
CPU1 Cache
 │
 ├── CR：我有这个数据
 │
 └── CD：数据是 20

于是cpu0得到20，而不是ddr里的旧数据10。

