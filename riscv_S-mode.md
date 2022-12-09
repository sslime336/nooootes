## riscv Supervisor Registers

## S 特权级

### Sstatus 系统状态寄存器

- SPP 表明进入S态之前的模式：user为0，其他为1

```assembly
csrr s1, sstatus
addi s0, s1, 1 << 8 # s0 = s1 + (1 << 8) 如果一开始 SPP 为 0(U-mode)，则现在为 1
bnez s0, _to_user # 如果不为 0，说明之前为 U-mode
```

### Sscratch 一个字的临时存储空间，一般用来辅助中断处理

### Stvec 中断跳转地址

```rust
pub fn init() {
    // save all registers
    extern "C" {
        fn __alltraps();
    }

    sscratch::write(0); // default trap from kernel
    unsafe {
        // setup stvec addr as __alltraps, which was impl in `trap.S`
        // `stvec::TrapMode::Direct` means we do not cross MMU.
        stvec::write(__alltraps as usize, stvec::TrapMode::Direct);
    }
    println!("trap init finished");
}
```

### Scause 中断或异常的原因

```rust
#[no_mangle]
fn trap_handler(trap_frame: &mut TrapFrame) -> &mut TrapFrame {
    let scause = scause::read();
    let _stval = stval::read();
    match scause.cause() {
        // Interrupts
        scause::Trap::Interrupt(UserSoft) => {}
        // ...
        scause::Trap::Interrupt(_) => {
            panic!("Unknown interrupt!");
        }

        // Exceptions
        scause::Trap::Exception(InstructionMisaligned) => {}
        // ...
        scause::Trap::Exception(_) => {
            panic!("Unknown exception!");
        }
    }

    trap_frame
}
```

### Sepc 发生中断时的位置  (pc)

- 需要记得手动 +4 Bytes(4 Bytes 为除精简指令集外，指令一般的长度，即移动到下一条指令，否则会一直陷入 trap)

| CSR 名  | 该 CSR 与 Trap 相关的功能                                    |
| ------- | ------------------------------------------------------------ |
| sstatus | `SPP` 等字段给出 Trap 发生之前 CPU 处在哪个特权级（S/U）等信息 |
| sepc    | 当 Trap 是一个异常的时候，记录 Trap 发生之前执行的最后一条指令的地址 |
| scause  | 描述 Trap 的原因                                             |
| stval   | 给出 Trap 附加信息                                           |
| stvec   | 控制 Trap 处理代码的入口地址                                 |