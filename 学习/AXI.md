![[Pasted image 20260812154604.png]]

ACE 是 **AMBA Coherency Extensions**，中文通常叫 **AMBA 一致性扩展协议**。
CPU0                                  CPU1
 │                                          │
L1 Cache                          L1 Cache
 │                                          │
 └────── Interconnect ──┘
             │
           Memory
ace是为了解决多个cache之间的一致性问题。例如有cpu0和cpu1，它们各自有L1 cache 和 L2 cache，当cpu0读取内存中的值并修改后，cpu1可能并不知道。ace在普通axi基础上增加了与cache一致性有关的通道。