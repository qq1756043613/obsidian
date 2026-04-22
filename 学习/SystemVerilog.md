VCS 编译命令
例：vcs -full -sverilog test.sv -R
vcs:                           仿真器可执行程序，位置$VCS_HOME/bin/vcs
-full64                      支持64-bit模式
-sverilog                  支持SystemVerilog
-test.sv                     源码，-f filelist
-R                             编译完成后执行，不加-R则仅编译，./simv
-+v2k                       Verilog-200*
-ntb_opts uvm         把整个UVM的框架、底层吃进来
(native testbench options)