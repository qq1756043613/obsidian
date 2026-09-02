在UVM验证平台中,只要一个类使用 uvm_component_utils注册且此类被实例化了,那么这 个类的main_phase就会自动被调用。这句话是什么意思？
只要这个类是一个真正加入 UVM 组件层次结构中的 `uvm_component`，UVM 的 phase 调度机制就会在仿真运行到 `main_phase` 时，自动调用它的 `main_phase()`，不需要你手动调用。当uvm仿真进入main_phase时，会自动执行各个component的main_phase，而不需要手动的去显示调用。


UVM 的 task phase（如 `run_phase`、`main_phase`）通过 objection 机制判断“这个 phase 里是否还有事情没做完”。只要还有 objection 没有 drop，UVM 就继续停留在当前 phase；当所有 objection 都被撤销后，当前 phase 才结束并进入下一个 phase。


