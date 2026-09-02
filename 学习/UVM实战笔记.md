在UVM验证平台中,只要一个类使用 uvm_component_utils注册且此类被实例化了,那么这 个类的main_phase就会自动被调用。这句话是什么意思？
只要这个类是一个真正加入 UVM 组件层次结构中的 `uvm_component`，UVM 的 phase 调度机制就会在仿真运行到 `main_phase` 时，自动调用它的 `main_phase()`，不需要你手动调用。当uvm仿真进入main_phase时，会自动执行各个component的main_phase，而不需要手动的去显示调用。


UVM 的 task phase（如 `run_phase`、`main_phase`）通过 objection 机制判断“这个 phase 里是否还有事情没做完”。只要还有 objection 没有 drop，UVM 就继续停留在当前 phase；当所有 objection 都被撤销后，当前 phase 才结束并进入下一个 phase。
Objection 是 UVM task phase 的结束控制机制。当某个 component 调用 `raise_objection()` 时，表示当前 phase 中还有任务没有完成，UVM 不会结束该 phase；当各个 component 对应的 objection 都通过 `drop_objection()` 撤销，使 objection count 变为 0 后，当前 phase 才结束并进入下一个 phase。因此 objection 常用于保证 sequence、DUT 响应以及检查过程完成后再结束仿真阶段。


raise_objection语句必须在main_phase中第一个 消耗仿真时间的语句之前。


interface也是一个组件，里面包含了接口的信息，也需要向其输入必要的信号（如clk、rstn等）。使用时也要在top_tb
中实例化。只要把interface中包含的信号值改了，那么连接到interface上的dut的信号自然也会改变。
![[Pasted image 20260902130707.png|544]]


这个问题的终极 原因在于UVM通过run_test语句实例化了一个脱离了 top_tb层次结构的实例,建立了一个新的层次结构。


当UVM启动后，会自动执行build_phase。build_phase在new函数之后main_phase之前执行。在build_phase中主要通过 config_db的set和get操作来传递一些数据，以及实例化成员变量等。


类名 / 类型名、实例名、层次路径（full name）之间的关系：
`run_test()` 的参数决定顶层创建哪一种 test 类型，但顶层 test 的实例名固定为 `uvm_test_top`；其他 component 在 `create("实例名", parent)` 时，第一个参数决定实例名，第二个参数决定父子层次关系，而“父组件路径 + 当前实例名”共同构成该 component 的完整层次路径。