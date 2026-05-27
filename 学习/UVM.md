![[Pasted image 20260527004136.png]]


![[Pasted image 20260528001714.png|434]]
UVM是一个框架，编译时要引入uvm库

![[Pasted image 20260528002455.png]]

同一个phase要等所有的component执行完才进入下一个阶段

![[Pasted image 20260528010254.png]]

每个组件的run是并行执行的，而组件内部的细分run_phase阶段是串行执行的

![[Pasted image 20260528011643.png]]

执行run_test()；后，通过test_name找到test_name类，通过类中的uvm_root build 成uvm_test_top。然后再一级一级的例化下面的component。

工厂模式好比让一家工厂做某个产品，你把a、b、c产品的图纸都给它，然后你想让它做哪个产品，它就会做哪个产品。