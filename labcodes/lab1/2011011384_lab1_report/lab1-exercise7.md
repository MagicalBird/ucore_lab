## 练习7：用户态和内核态的切换

### 7.1 trapframe

通过修改trapframe，实现内核态和用户态的trapframe互相改变。

唯一不同是内核态发生trap时esp和ss不压栈。具体的实现方式如下:

```C
static void
lab1_switch_to_user(void) {
    asm volatile (
        "subl $8, %%esp\n"
        "int %0\n"
        "movl %%ebp, %%esp"
        :
        :"i" (T_SWITCH_TOU)
    );
}
```

```C
static void
lab1_switch_to_kernel(void) {
    asm volatile (
        "int %0\n"
        "movl %%ebp, %%esp"
        :
        :"i" (T_SWITCH_TOK)
    );
}
```

### 7.2 键盘

使用`cons_getc()`函数读入，需要删去读取后打印的字符。



