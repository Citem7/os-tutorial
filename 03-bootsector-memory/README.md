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
![Typical lower memory layout after boot.](D:\github\OS_tutorial\03\Typical lower memory layout after boot.png)
The only goal of this lesson is to learn where the boot sector is stored

I could just bluntly tell you that the BIOS places it at `0x7C00` and
get it done with, but an example with wrong solutions will make things clearer.

We want to print an X on screen. We will try 4 different strategies
and see which ones work and why.

**Open the file `boot_sect_memory.asm`**

First, we will define the X as data, with a label:
```nasm
the_secret:
    db "X"
```

Then we will try to access `the_secret` in many different ways:

1. `mov al, the_secret`
2. `mov al, [the_secret]`
3. `mov al, the_secret + 0x7C00`
4. `mov al, 2d + 0x7C00`, where `2d` is the actual position of the 'X' byte in the binary

Take a look at the code and read the comments.

Compile and run the code. You should see a string similar to `1[2¢3X4X`, where
the bytes following 1 and 2 are just random garbage.

If you add or remove instructions, remember to compute the new offset of the X
by counting the bytes, and replace `0x2d` with the new one.

Please don't continue onto the next section unless you have 100% understood
the boot sector offset and memory addressing.


The global offset
-----------------

Now, since offsetting `0x7c00` everywhere is very inconvenient, assemblers let
us define a "global offset" for every memory location, with the `org` command:

```nasm
[org 0x7c00]
```

Go ahead and **open `boot_sect_memory_org.asm`** and you will see the canonical
way to print data with the boot sector, which is now attempt 2. Compile the code
and run it, and you will see how the `org` command affects each previous solution.

Read the comments for a full explanation of the changes with and without `org`

-----

[1] This whole tutorial is heavily inspired on that document. Please read the
root-level README for more information on that.
