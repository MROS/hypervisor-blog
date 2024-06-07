## 縮寫

暫存器系列
- GPR = General Purpose Registers
- tp = thread pointer = x4 暫存器，指向 進程控制塊/thread local storage。
- sepc = supervisor exception program counter = 陷入 S 模式時，觸發 S 模式的指令的虛擬地址。

分頁系列
- gpt = guest page table

其他
- hart = hardware threads

## hypercraft 定義之結構

### PerCpu
應對到真實的 CPU （？）


## VM 啓動流程

爲什麼要抽象出 vcpu ？

PerCpu 跟 vcpu 的關係是？
