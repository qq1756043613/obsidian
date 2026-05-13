[[APB]]
psel 、

alarm信号是只维持1s的信号。但是其能将awake信号拉高。awake信号在被清零前一直保持有效状态。不将alarm接到awake上的原因是，匹配是一个瞬时事件，而唤醒是一个需要所存的状态。

r't'c