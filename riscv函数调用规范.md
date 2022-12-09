# 函数调用规范

Caller: 调用者

Callee: 被调用者

指令长度一般（除精简指令集）为 32 bits，即 4 Bytes

```rust
fn do_something_fun() { // caller
	do_something_bored() // callee
}

fn do_something_bored() {
    // some code...
}
```

| 寄存器  | ABI 名称 | 描述                     | Saver      |
| ------- | -------- | ------------------------ | ---------- |
| x0      | zero     | 硬编码为零值             | -----      |
| x1      | ra       | Return Addr 函数返回地址 | **Caller** |
| x2      | sp       | Stack Pointer 栈顶指针   | Callee     |
| x3      | gp       | Global Pointer           | -----      |
| x4      | tp       | Thread Pointer           | -----      |
| x5-x7   | t0-t2    | Temporaries 临时寄存器   | **Caller** |
| x8      | s0/fp    | 保留寄存器 / 栈帧段指针  | Callee     |
| x9      | s1       | 保留寄存器               | Callee     |
| x10-x11 | a0-a1    | 函数入参、返回值         | **Caller** |
| x12-x17 | a2-a7    | 函数入参                 | **Caller** |
| x18-x27 | s2-s11   | 保留寄存器               | Callee     |
| x28-x31 | t3-t6    | 临时寄存器               | **Caller** |

## 跳转指令

### 无条件跳转

```assembly
jal rd, offset/label => x[rd] = pc+4; pc += sext(offset)
jalr rd, offset(rs1)
```

### 有条件跳转

```assembly
# 格式: bxx rs1，rs2，offset => if (rs1 `xx` rs2) pc += sext(offset)
# sext: sign-extend（符号扩展）表示将该立即数扩展到32位，如果是有符号数则进行符号扩展，若是无符号数则进行无符号扩展
beq   # 相等时分支
bne   # 不相等时分支
ble   # 小于时分支
bltu  # 无符号小于时分支
bge   # 大于时分支
bgeu  # 无符号大于时分支
```

### 例子

```c
void sum() {
    return; // jalr x0, 0(x5) => x0 += 4, jump to 0(x5), also use `j 0(x5)`
}
void _start() {
    sum(); // jal x5, sum => x5 += 4, jump to sum
    // ...
}
```
