![[Pasted image 20260527004136.png]]
![[Pasted image 20260530223112.png]]
最外层是tb_top，在top中例化if、dut，然后通过run_test把test也包含进来，test中有env。再连dut和if、连if和test中的某个部件（在这里是env中的if0）。至此把所有组件连接起来。

![[Pasted image 20260528001714.png|434]]
UVM是一个框架，编译时要引入uvm库

![[Pasted image 20260528002455.png]]

同一个phase要等所有的component执行完才进入下一个阶段

![[Pasted image 20260528010254.png]]

每个组件的run是并行执行的，而组件内部的细分run_phase阶段是串行执行的

![[Pasted image 20260528011643.png]]

执行run_test()；后，通过test_name找到test_name类，通过类中的uvm_root build 成uvm_test_top。然后再一级一级的例化下面的component。
通常component中负责左边的phase，test中负责右边的phase。

工厂模式好比让一家工厂做某个产品，你把a、b、c产品的图纸都给它，然后你想让它做哪个产品，它就会做哪个产品。
### 你想让它做哪个，它就做哪个 $\rightarrow$ UVM 的“覆盖” (`Override`)

这是你比喻中最传神的地方。

在传统的面向对象里，如果流水线上写死了 `product_a p = new();`，那它就只能永远生产 a。

而在 UVM 的工厂机制里，流水线上写的是：

代码段

```
// 厂长（底层环境）只下达了生产“普通包”的指令
p = base_packet::type_id::create("p"); 
```

这时候，作为“大Boss”的测试用例（Test 居），在不惊动流水线工人的情况下，偷偷给工厂下了一道命令（覆盖）：

> 📢 **“从现在开始，凡是要生产 `base_packet`（普通包）的地方，工厂一律按 `advanced_packet`（高级包）的图纸来生产！”**

代码段

```
// 一行命令，狸猫换太子
set_type_override_by_type(base_packet::get_type(), advanced_packet::get_type());
```

接下来，神奇的事情发生了：底层流水线工人完全不知道这件事，他们还是继续执行 `create("p")`，但工厂在后台自动把图纸掉包了，最后吐出来的全是**高级包**。

![[Pasted image 20260528014533.png]]
报告信息可分为by ID or by type

![[Pasted image 20260528014729.png]]
UVM_LOW  UVM_MEDIUM、、、是啰嗦程度。

![[Pasted image 20260528015018.png]]

![[Pasted image 20260528015209.png]]

![[Pasted image 20260528015241.png]]

![[Pasted image 20260528015442.png]]

![[Pasted image 20260528015559.png]]

![[Pasted image 20260528015634.png]]

![[Pasted image 20260528015745.png]]

![[Pasted image 20260528015839.png]]

![[Pasted image 20260528020300.png]]

![[Pasted image 20260528020317.png]]

![[Pasted image 20260528020405.png]]


![[Pasted image 20260604002229.png]]

![[Pasted image 20260604002452.png]]

![[Pasted image 20260604002750.png]]

![[Pasted image 20260604003733.png]]

![[Pasted image 20260604003755.png]]

![[Pasted image 20260604004313.png]]

![[Pasted image 20260604004546.png]]

![[Pasted image 20260604005401.png]]

![[Pasted image 20260604005544.png]]

![[Pasted image 20260604011245.png]]

![[Pasted image 20260604011345.png]]

![[Pasted image 20260604011634.png]]

![[Pasted image 20260604011837.png]]

![[Pasted image 20260604012013.png]]

![[Pasted image 20260604012023.png]]

