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

