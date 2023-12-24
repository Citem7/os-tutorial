*Concepts you may want to Google beforehand: memory offsets, pointers*
>**内存偏移（Memory offsets）**
>内存偏移（Memory offsets）指的是内存地址之间的距离或位移。在计算机系统中，内存被组织成线性的地址空间，每个字节都有唯一的地址。内存偏移用于访问地址空间中的特定位置。
>
>在低级编程或系统编程中，内存偏移经常用于访问数组或结构体中的单个元素。通过将偏移量添加到基础内存地址，可以访问数据结构中的特定元素或字段。这通常使用指针算术来完成，其中偏移量乘以数据类型的大小来计算内存地址。
>
>例如，考虑C语言中的整数数组：
>
>```C
>int numbers[5] = {10, 20, 30, 40, 50};
>```
>要使用内存偏移访问数组中的第三个元素（30），可以使用指针算术：
>```C
>int* ptr = &numbers[0];  // 获取数组的基础地址
>int offset = 2;  // 第三个元素的偏移量
>int* target = ptr + offset;  // 计算内存地址
>int value = *target;  // 访问内存地址处的值
>printf("%d\n", value);  // 输出：30
>```
**Goal: Learn how the computer memory is organized**

Please open page 14 [of this document](
http://www.cs.bham.ac.uk/~exr/lectures/opsys/10_11/lectures/os-dev.pdf)<sup>1</sup>
and look at the figure with the memory layout.
![Typical lower memory layout after boot.](https://github.com/Citem7/os-tutorial/blob/master/03-bootsector-memory/Typical%20lower%20memory%20layout%20after%20boot.png)

The only goal of this lesson is to learn where the boot sector is stored
>这节课的唯一目标是学习引导扇区存储在哪里。
>
>**引导扇区** 也称为*主引导记录（Master Boot Record，MBR）*，存储在存储设备（如硬盘驱动器或固态硬盘）的第一个扇区中。它是计算机系统启动过程中的重要组成部分。
>
>在x86架构中，引导扇区位于柱面0、磁头0、扇区1（CHS 0/0/1）。该扇区大小为512字节，是磁盘的第一个扇区。它包含引导加载程序的初始代码和分区表。
>
>计算机启动时，基本输入/输出系统（BIOS）会将引导扇区加载到内存中，并将控制权转移到引导加载程序代码。引导加载程序继续引导过程，从指定的分区加载操作系统或其他引导加载程序。
>
>需要注意的是，随着新技术（如统一可扩展固件接口，UEFI）的出现，引导过程和引导相关信息的存储位置可能有所不同。但是，在传统的基于BIOS的系统中，引导扇区通常存储在存储设备的第一个扇区中
>
I could just bluntly tell you that the BIOS places it at `0x7C00` and
get it done with, but an example with wrong solutions will make things clearer.
>内存地址0x0000通常保留用于系统ROM或BIOS代码，而不是引导扇区的存储位置。

We want to print an X on screen. We will try 4 different strategies
and see which ones work and why.

**Open the file `boot_sect_memory.asm`**

First, we will define the X as data, with a label:
```nasm
the_secret:
    db "X"
```
>首先，我们将X定义为数据，并为其添加一个标签：
>```nasm
>the_secret:
>   db "X"
>```
>
>这段代码使用汇编语言（NASM）的语法来定义了一个名为`the_secret`的标签，并将一个字节大小的数据`"X"`存储在该标签下。`db`指令用于定义字节（byte）大小的数据，并使用引号将字符`X`括起来表示一个ASCII字符。
>
>这段代码的含义是将字符`X`存储在名为`the_secret`的标签下，以便后续使用该标签来引用这个数据。

Then we will try to access `the_secret` in many different ways:

1. `mov al, the_secret`
2. `mov al, [the_secret]`
3. `mov al, the_secret + 0x7C00`
4. `mov al, 2d + 0x7C00`, where `2d` is the actual position of the 'X' byte in the binary

>接下来，我们将尝试以多种不同的方式访问`the_secret`：
>1. `mov al, the_secret`
>2. `mov al, [the_secret]`
>3. `mov al, the_secret + 0x7C00`
>4. `mov al, 2d + 0x7C00`，其中`2d`是二进制文件中字符'X'的实际位置。
>
>让我们逐个解释这些指令的含义和作用：
>
>1. `mov al, the_secret`：这条指令将`the_secret`标签中的数据（即字符'X'）直接移动到寄存器`al`中。由于`the_secret`是一个标签，它在汇编代码中被解析为一个内存地址。
>
>2. `mov al, [the_secret]`：这条指令将`the_secret`标签中的数据（即字符'X'）所在内存地址处的值移动到寄存器`al`中。方括号表示间接寻址，指示CPU根据内存地址访问相应的值。
>
>3. `mov al, the_secret + 0x7C00`：这条指令将`the_secret`标签所在的内存地址加上偏移量`0x7C00`后的地址处的值移动到寄存器`al`中。这种偏移量通常用于引用位于特定内存位置的数据。
>
>4. `mov al, 2d + 0x7C00`：这条指令将偏移量`2d`加上`0x7C00`后的地址处的值移动到寄存器`al`中。这里的`2d`是字符'X'在二进制文件中的实际位置（偏移量），通过将偏移量与基地址`0x7C00`相加，可以得到该字符所在的内存地址。
>
>需要注意的是，上述指令的具体含义和效果可能取决于上下文和执行环境。这些指令在实际代码中的使用可能会有不同的目的和结果。

Take a look at the code and read the comments.

Compile and run the code. You should see a string similar to `1[2¢3X4X`, where
the bytes following 1 and 2 are just random garbage.
>编译并运行这段代码。您应该会看到类似于`1[2¢3X4X`的字符串，其中在1和2之后的字节只是随机的垃圾数据。

If you add or remove instructions, remember to compute the new offset of the X
by counting the bytes, and replace `0x2d` with the new one.
>如果您添加或删除指令，请记住通过计算字节数来计算字符“X”的新偏移量，并将`0x2d`替换为新的值。0x2d的十进制是45

Please don't continue onto the next section unless you have 100% understood
the boot sector offset and memory addressing.


The global offset
-----------------
>**全局偏移量**
>在计算机编程中，全局偏移量是指相对于程序或数据集的起始位置的固定偏移量。它用于访问存储在内存或磁盘中的特定位置的数据。全局偏移量通常在程序中定义，并用于确定变量或数据结构的位置。

Now, since offsetting `0x7c00` everywhere is very inconvenient, assemblers let
us define a "global offset" for every memory location, with the `org` command:
>现在，由于在每个地方都进行偏移`0x7c00`非常不方便，汇编器允许我们使用`org`命令为每个内存位置定义一个"全局偏移量"。

```nasm
[org 0x7c00]
```
>org 0x7c00指令用于将全局偏移量设置为0x7c00。这意味着后续的代码和数据将相对于0x7c00进行定位。通过设置全局偏移量，我们可以使用相对地址来引用代码和数据，而无需手动计算偏移量。
>
>在引导扇区中，通常使用的是绝对地址0x7c00，而不是相对地址。因此，org指令在这种情况下并不适用。
>
>以下是一个修正后的示例，展示了在引导扇区中使用绝对地址0x7c00的情况：
>
>```nasm
>start:
>    mov ax, 0x1234    ; 将0x1234存储到寄存器ax中
>    add ax, 0x5678    ; 将0x5678加到ax中
>    ; 其他指令和代码...
>
>    jmp $    ; 无限循环，防止程序继续执行
>
>times 510-($-$$) db 0    ; 填充引导扇区的剩余空间
>dw 0xAA55    ; 引导扇区的结束标志
```
>
>在这个修正后的例子中，我们没有使用org指令，而是直接在代码中使用绝对地址0x7c00。最后的两行代码用于填充引导扇区的剩余空间，并添加引导扇区的结束标志0xAA55。
>
>请注意，在引导扇区中，我们必须确保代码和数据不超过512字节（引导扇区的大小）。因此，需要小心控制代码的长度，以确保它适合于引导扇区的限制。

Go ahead and **open `boot_sect_memory_org.asm`** and you will see the canonical
way to print data with the boot sector, which is now attempt 2. Compile the code
and run it, and you will see how the `org` command affects each previous solution.
>请打开boot_sect_memory_org.asm文件，你将看到打印数据的引导扇区的典型方式，这是第二次尝试。编译代码并运行它，你将看到org指令如何影响之前的每个解决方案。

Read the comments for a full explanation of the changes with and without `org`

-----

[1] This whole tutorial is heavily inspired on that document. Please read the
root-level README for more information on that.
