# RustOS riscv32架构测试报告

计54 寇明阳 2015011318

## 一、用户程序样例测试

我复现了王润基同学的测试工作，在qemu模拟器上模拟riscv32架构测试了原ucore上的用户程序，并获得了如下结果：

### 1、`waitkill`用户程序：测试通过

注意到由于线程调用，因此在程序返回后依然有输出。
输出结果：

```
>> waitkill
wait child 1.
child 2.
child 1.
kill parent ok.
>> kill child1 ok.

>>
```

### 2、`sleep`用户程序：测试通过

注意到这里的用时与x86_64架构略有不同。
输出结果：

```
>> sleep
sleep 1 x 100 slices.
sleep 2 x 100 slices.
sleep 3 x 100 slices.
sleep 4 x 100 slices.
sleep 5 x 100 slices.
sleep 6 x 100 slices.
sleep 7 x 100 slices.
sleep 8 x 100 slices.
sleep 9 x 100 slices.
sleep 10 x 100 slices.
use 1000 msecs.
sleep pass.
>>
```

### 3、`spin`用户程序：测试通过

输出结果：

```
>> spin
I am the parent. Forking the child...
I am the parent. Running the child...
I am the child. spinning ...
I am the parent.  Killing the child...
kill returns 0
wait returns 0
spin may pass.
>>
```

### 4、`sh`用户程序：测试未通过

两种架构的错误信息不同。
输出结果：

```
>> sh
user sh is running!!!$ [ERROR] unknown syscall id: 0x66, args: [0, 7000ff97, 1, 7000ff70, 25, 73]
[ERROR] Process 2 error:
TrapFrame {
    x: [
        0x0,
        0x800e1c,
        0x7000ff38,
        0x0,
        0x0,
        0x0,
        0x7000ff70,
        0x0,
        0x0,
        0x3,
        0x66,
        0x0,
        0x7000ff97,
        0x1,
        0x7000ff70,
        0x25,
        0x73,
        0x20,
        0x1f,
        0xffe,
        0xd,
        0x8,
        0xa,
        0x804000,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0
    ],
    sstatus: Sstatus {
        bits: 0x80046000
    },
    sepc: 0x800144,
    sbadaddr: 0x0,
    scause: Scause {
        bits: 0x8
    }
}
>>
```

### 5、`forktest`用户程序：测试通过，但存在一定问题

在第一次运行时，由于在`sh`程序之后调用`forktest`，导致了user panic，输出结果如下：

```
>> forktest
I am child 31
I am child 30
I am child 29
I am child 28
I am child 27
I am child 26
I am child 25
I am child 24
I am child 23
I am child 22
I am child 21
I am child 20
I am child 19
I am child 18
I am child 17
I am child 16
I am child 15
I am child 14
I am child 13
I am child 12
I am child 11
I am child 10
I am child 9
I am child 8
I am child 7
I am child 6
I am child 5
I am child 4
I am child 3
I am child 2
I am child 1
user panic at user/forktest.c:28:
    wait got too many

>> I am child 0

>> 
```

第二次重启操作系统后运行`forktest`，运行正常：

```
>> forktest
I am child 31
I am child 30
I am child 29
I am child 28
I am child 27
I am child 26
I am child 25
I am child 24
I am child 23
I am child 22
I am child 21
I am child 20
I am child 19
I am child 18
I am child 17
I am child 16
I am child 15
I am child 14
I am child 13
I am child 12
I am child 11
I am child 10
I am child 9
I am child 8
I am child 7
I am child 6
I am child 5
I am child 4
I am child 3
I am child 2
I am child 1
I am child 0
forktest pass.
>>
```

而当连续运行两次`forktest`后，系统一定会报错并崩溃，输出结果如下：

```
>> forktest
[ERROR] 

PANIC in src/lang.rs at line 22
    out of memory
```


### 6、`faultread`用户程序：测试未通过

输出结果：

```
>> faultread
[ERROR] Process 2 error:
TrapFrame {
    x: [
        0x0,
        0x8002e0,
        0x7000ffe8,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x7000fffc,
        0x0,
        0x7000fffc,
        0x1,
        0x0,
        0x0,
        0x800a14,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0
    ],
    sstatus: Sstatus {
        bits: 0x80046000
    },
    sepc: 0x80099c,
    sbadaddr: 0x0,
    scause: Scause {
        bits: 0xd
    }
}
>>
```

### 7、`forktree`用户程序：测试未通过，并导致系统崩溃

开始输出了一部分正常的测试信息，在等待很长时间后输出了溢出信息，系统崩溃。

输出结果：

```
>> forktree
forktree process will sleep 400 ticks
0002: I am ''
>> 0004: I am '1'
0005: I am '11'
0007: I am '111'
0009: I am '1111'
0008: I am '1110'
0006: I am '110'
000b: I am '1101'
000a: I am '1100'
0003: I am '0'
000d: I am '01'
000f: I am '011'
0011: I am '0111'
0010: I am '0110'
000e: I am '010'
0013: I am '0101'
0012: I am '0100'
000c: I am '00'
0015: I am '001'
0017: I am '0011'
0016: I am '0010'
0014: I am '000'
0019: I am '0001'
0018: I am '0000'
0002: I am '10'
001b: I am '101'
001d: I am '1011'
001c: I am '1010'
001a: I am '100'
001f: I am '1001'
001e: I am '1000'
[ERROR] 

PANIC in /home/kmy/OperatingSystem/RustOS/crate/process/src/scheduler.rs at line 136
    attempt to add with overflow

```

### 8、`divzero`用户程序：测试通过

输出结果：

```
>> divzero 
value is -1.
user panic at user/divzero.c:9:
    FAIL: T.T

>> 
```

### 9、`yield`用户程序：测试通过

输出结果：

```
>> yield
Hello, I am process 2.
Back in process 2, iteration 0.
Back in process 2, iteration 1.
Back in process 2, iteration 2.
Back in process 2, iteration 3.
Back in process 2, iteration 4.
All done in process 2.
yield pass.
>>
```

### 10、`faultreadkernel`用户程序：测试未通过

输出结果：

```
>> faultreadkernel
[ERROR] Process 2 error:
TrapFrame {
    x: [
        0x0,
        0x800340,
        0x7000ffe8,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x7000fffc,
        0x0,
        0x7000fffc,
        0x1,
        0x0,
        0x0,
        0xfac00000,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0
    ],
    sstatus: Sstatus {
        bits: 0x80046000
    },
    sepc: 0x800a00,
    sbadaddr: 0xfac00000,
    scause: Scause {
        bits: 0xd
    }
}
>>
```

### 11、`exit`用户程序：测试通过

输出结果：

```
>> exit
I am the parent. Forking the child...
I am parent, fork a child pid 3
I am the parent, waiting now..
I am the child.
waitpid 3 ok.
exit pass.
>>
```

### 12、`softint`用户程序：测试通过，但无意义，因为是x86_86架构下的用户程序

测试无输出，因为是x86_86架构下的用户程序，这样的测试没有意义。
输出结果：

```
>> softint
>>
```

### 13、`badsegment`用户程序：测试通过

输出结果：

```
>> badsegment
user panic at user/badsegment.c:9:
    FAIL: T.T

>>
```

### 14、`hello`用户程序：测试通过

输出结果：

```
>> hello
Hello world!!.
I am process 2.
hello pass.
>> 
```

### 15、`ls`用户程序：测试通过，但无意义，因为没有文件系统

无输出，因为没有文件系统，这样的测试没有意义。
输出结果：

```
>> ls
>>
```

### 16、`priority`用户程序：测试通过

输出结果：

```
>> priority
priority process will sleep 400 ticks
main: fork ok,now need to wait pids.
child pid 7, acc 4000, time 12898
child pid 6, acc 4000, time 12898
child pid 5, acc 4000, time 12898
child pid 4, acc 4000, time 12898
child pid 3, acc 4000, time 12898
main: pid 3, acc 4000, time 12898
main: pid 4, acc 4000, time 12898
main: pid 5, acc 4000, time 12899
main: pid 6, acc 4000, time 12899
main: pid 7, acc 4000, time 12899
main: wait pids over
stride sched correct result: 1 1 1 1 1
>>
```

### 17、`badarg`用户程序：测试未通过，并导致系统崩溃

调用该用户程序可直接导致操作系统崩溃。
输出结果：

```
>> badarg
[ERROR] 

PANIC in /home/kmy/.rustup/toolchains/nightly-2018-09-18-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/src/libcore/option.rs at line 345
    called `Option::unwrap()` on a `None` value

```

### 18、`testbss`用户程序：测试未通过，并导致系统崩溃

调用该用户程序可直接导致操作系统崩溃。
输出结果：

```
>> testbss
[ERROR] 

PANIC in /home/kmy/.rustup/toolchains/nightly-2018-09-18-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/src/libcore/option.rs at line 1000
    failed to allocate frame

```

### 19、`pgdir`用户程序：测试未通过

输出结果：

```
>> pgdir
I am 2, print pgdir.
[ERROR] unknown syscall id: 0x1f, args: [a, ffff6ad9, 2, 0, 15, ffffffff]
[ERROR] Process 2 error:
TrapFrame {
    x: [
        0x0,
        0x8009d4,
        0x7000ff88,
        0x0,
        0x0,
        0x0,
        0x800164,
        0x0,
        0x0,
        0x7000fffc,
        0x1f,
        0xa,
        0xffff6ad9,
        0x2,
        0x0,
        0x15,
        0xffffffff,
        0x20,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0,
        0x0
    ],
    sstatus: Sstatus {
        bits: 0x80046000
    },
    sepc: 0x8000e4,
    sbadaddr: 0x0,
    scause: Scause {
        bits: 0x8
    }
}
>>
```

### 20、`matrix`用户程序：测试通过

输出结果：

```
>> matrix
fork ok.
pid 23 is running (17900 times)!.
pid 23 done!.
pid 22 is running (33400 times)!.
pid 21 is running (3500 times)!.
pid 21 done!.
pid 20 is running (15400 times)!.
pid 20 done!.
pid 19 is running (37100 times)!.
pid 18 is running (5900 times)!.
pid 18 done!.
pid 17 is running (26600 times)!.
pid 16 is running (3500 times)!.
pid 16 done!.
pid 15 is running (26600 times)!.
pid 14 is running (5900 times)!.
pid 14 done!.
pid 13 is running (41000 times)!.
pid 12 is running (15400 times)!.
pid 12 done!.
pid 11 is running (3500 times)!.
pid 11 done!.
pid 10 is running (41000 times)!.
pid 9 is running (23500 times)!.
pid 9 done!.
pid 8 is running (13100 times)!.
pid 8 done!.
pid 7 is running (5900 times)!.
pid 7 done!.
pid 6 is running (2600 times)!.
pid 6 done!.
pid 5 is running (1400 times)!.
pid 5 done!.
pid 4 is running (1100 times)!.
pid 4 done!.
pid 3 is running (1100 times)!.
pid 3 done!.
pid 22 done!.
pid 19 done!.
pid 17 done!.
pid 15 done!.
pid 13 done!.
pid 10 done!.
matrix pass.
>> 
```

### 21、`sleepkill`用户程序：测试未通过，并导致系统崩溃

调用该用户程序可直接导致操作系统崩溃。
输出结果：

```
>> sleepkill
[ERROR] 

PANIC in /home/kmy/OperatingSystem/RustOS/crate/process/src/event_hub.rs at line 55
    attempt to add with overflow

>>
```

## 二、其他问题

我注意到，在我的运行环境下，当RustOS运行时间达到一定时间后，系统会崩溃，并输出错误信息。在x86_64架构上也有相同的问题。

```
>> [ERROR] 

PANIC in /home/kmy/OperatingSystem/RustOS/crate/process/src/scheduler.rs at line 136
    attempt to add with overflow

```