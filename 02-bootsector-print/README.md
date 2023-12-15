*Concepts you may want to Google beforehand: interrupts, CPU
registers*
>**linux系统中断是怎么实现的**
>
>在Linux系统中，中断是通过硬件和内核共同实现的。当硬件设备需要处理某些事件时（比如数据传输完成或者出现错误），它会发送一个中断信号给CPU。CPU会立即停止当前正在执行的任务，保存当前的状态，然后跳转到预先定义好的中断处理程序的地址。
>
>在Linux内核中，有一个中断描述符表（IDT），它包含了所有可能的中断类型和相应的中断处理程序的地址。当一个中断被触发时，CPU会根据中断号在IDT中查找对应的中断处理程序的地址，并跳转到该地址执行相应的处理程序。
>
>在中断处理程序中，内核会根据中断类型进行相应的处理，可能会更新数据结构、唤醒等待中的进程或者执行其他必要的操作。处理完成后，CPU会恢复之前的状态，继续执行被中断的任务。

**Goal: Make our previously silent boot sector print some text**
>让我们之前无声的引导扇区打印一些文本

We will improve a bit on our infinite-loop boot sector and print
something on the screen. We will raise an interrupt for this.

On this example we are going to write each character of the "Hello"
word into the register `al` (lower part of `ax`), the bytes `0x0e`
into `ah` (the higher part of `ax`) and raise interrupt `0x10` which
is a general interrupt for video services.
>在这个例子中，我们将把“Hello”单词的每个字符写入寄存器al（ax寄存器的低位），将字节0x0e写入ah（ax寄存器的高位），然后触发0x10中断，这是用于视频服务的通用中断。

>这段话描述了一个汇编语言的例子，通过将“Hello”单词的每个字符写入寄存器al和将0x0e写入寄存器ah，然后触发0x10中断来实现一些视频服务的操作。在汇编语言中，0x10中断通常用于调用视频服务，比如在屏幕上打印字符或者进行其他与视频显示相关的操作。

>**AX寄存器**
>
>ax寄存器是x86架构中的一个通用寄存器，它是一个16位的寄存器，可以被用于存储数据或者地址。ax寄存器可以被分为两个独立的8位寄存器：ah（ax的高位）和al（ax的低位）。这种结构使得ax寄存器可以同时存储一个16位的值或者两个8位的值。
>在汇编语言中，ax寄存器通常用于进行算术运算、数据传输以及地址计算等操作。同时，由于ax寄存器是通用寄存器，它还可以用于存储其他类型的数据，比如函数参数或者返回值。

`0x0e` on `ah` tells the video interrupt that the actual function
we want to run is to 'write the contents of `al` in tty mode'.
>0x0e在ah寄存器中告诉视频中断，我们要运行的实际功能是“以tty模式写入al寄存器的内容”。

>当在汇编语言中将0x0e写入ah寄存器时，这告诉了视频中断服务程序，我们希望使用该中断来在TTY（Teletype）模式下将al寄存器中的内容写入到屏幕上。TTY模式是一种简单的文本终端模式，它允许以文本形式在屏幕上进行输入和输出。因此，通过将0x0e写入ah寄存器，我们告诉系统我们希望使用视频中断来在TTY模式下进行文本输出。
>
>这种操作通常用于在汇编语言中编写与屏幕交互的程序，比如在显示器上打印字符或者进行其他文本输出操作。通过设置适当的寄存器值并触发相应的中断，程序可以利用系统提供的视频服务来实现屏幕输出。
>
>**TTY模式**
>
>TTY（Teletype）模式是一种文本终端模式，最初用于远程通信和计算机终端。在计算机领域，TTY模式通常指的是一种简单的文本界面，用户可以在其中输入文本命令并查看文本输出。TTY模式通常不包括图形用户界面（GUI），而是以纯文本形式进行交互。
>
>在UNIX和类UNIX系统中，TTY通常指代终端设备，比如物理终端、虚拟终端或串口终端。这些终端设备可以通过TTY模式进行文本交互，用户可以在其中输入命令并查看程序输出。
>
>虽然现代计算机系统已经普遍使用了图形用户界面，但在系统维护、故障排除和一些特定的应用场景中，TTY模式仍然具有重要的作用。例如，在Linux系统中，用户可以通过虚拟终端（Virtual TTY）切换到纯文本的TTY模式，以进行系统管理和故障排除操作。
>
>总的来说，TTY模式是一种基于文本的终端模式，用于在计算机系统中进行纯文本的输入和输出。

We will set tty mode only once though in the real world we 
cannot be sure that the contents of `ah` are constant. Some other
process may run on the CPU while we are sleeping, not clean
up properly and leave garbage data on `ah`.
>在实际情况中，我们只需要设置TTY模式一次，尽管在现实世界中我们无法确保ah寄存器的内容是恒定不变的。一些其他的进程可能会在我们休眠时在CPU上运行，没有进行适当的清理，导致在ah寄存器上留下了垃圾数据。

>这段话似乎在讨论在编程中需要考虑到并发执行的情况。在多任务操作系统中，多个进程可能会同时运行在CPU上，它们之间的执行顺序是不确定的。因此，即使我们设置了TTY模式，但在程序执行期间，ah寄存器的内容也可能被其他进程修改，导致出现意外的行为。
>
>这种情况下，程序员需要考虑如何在程序中处理这种并发情况，比如在关键部分使用锁来保护关键资源，或者在设置TTY模式之前清理ah寄存器的内容，以确保它包含正确的数值。这样可以避免其他进程的干扰，确保程序的正确执行。

For this example we don't need to take care of that since we are
the only thing running on the CPU.
>对于这个例子，我们不需要担心这一点，因为我们是唯一在CPU上运行的东西。

Our new boot sector looks like this:
```nasm
mov ah, 0x0e ; tty mode
mov al, 'H'
int 0x10
mov al, 'e'
int 0x10
mov al, 'l'
int 0x10
int 0x10 ; 'l' is still on al, remember?
mov al, 'o'
int 0x10

jmp $ ; jump to current address = infinite loop

; padding and magic number
times 510 - ($-$$) db 0
dw 0xaa55 
```
>这段代码是一个简单的汇编代码示例，它使用了中断0x10来在屏幕上打印单词"Hello"。让我来逐行解释一下：
>
>1. `mov ah, 0x0e`：将0x0e（表示TTY模式）移入ah寄存器。
>2. `mov al, 'H'`：将字符'H'移入al寄存器。
>3. `int 0x10`：触发中断0x10，将al寄存器中的内容以TTY模式打印到屏幕上。
>4. `mov al, 'e'`：将字符'e'移入al寄存器。
>5. `int 0x10`：再次触发中断0x10，将al寄存器中的内容以TTY模式打印到屏幕上。
>6. `mov al, 'l'`：将字符'l'移入al寄存器。
>7. `int 0x10`：再次触发中断0x10，将al寄存器中的内容以TTY模式打印到屏幕上。
>8. `int 0x10`：再次触发中断0x10，此时'al'寄存器中仍然存储着字符'l'，因此'l'会再次被打印到屏幕上。
>9. `mov al, 'o'`：将字符'o'移入al寄存器。
>10. `int 0x10`：最后一次触发中断0x10，将al寄存器中的内容以TTY模式打印到屏幕上。
>11. `jmp $`：跳转到当前地址，形成一个无限循环。
>
>最后的部分是填充和引导扇区的魔数。
>
>这段代码的作用是在屏幕上打印单词"Hello"，然后进入一个无限循环。
>
>在这段代码中，使用了`times 510 - ($-$$) db 0`来填充剩余的空间，确保整个引导扇区的大小达到了512字节。接着使用`dw 0xaa55`来添加引导扇区的结束标记，即引导扇区的魔数`0xaa55`。这个魔数是一个标识，用于告诉计算机这是一个有效的引导扇区。
>
>在引导扇区的最后，通常会添加这个特定的魔数，以确保计算机能够正确识别引导扇区，并且在引导时能够执行其中的引导加载程序。
>
>**魔数**
>在计算机领域，"魔数"通常指的是一种特定的标识，用于识别文件格式、数据结构或者特定类型的数据。这个"魔数"通常是一个固定的数值或者字符串，位于文件或数据的特定位置，用于表明该文件或数据的类型或格式。
>
>在引导扇区中，"魔数"通常是指一个特定的16位数值，例如0xaa55。这个魔数位于引导扇区的最后两个字节，用于标识这个扇区是一个有效的引导扇区。当计算机引导时，系统会检查引导扇区的最后两个字节是否等于0xaa55，如果相符，计算机会将这个扇区加载到内存并执行其中的引导加载程序。
>
>因此，"魔数"在计算机中扮演着识别和验证数据格式的重要角色，它能够确保计算机能够正确地识别和处理特定类型的数据。

You can examine the binary data with `xxd file.bin`
>可以使用命令`xxd file.bin`来查看文件中的二进制数据。当你运行`xxd file.bin`时，它会显示`file.bin`中二进制数据的十六进制表示，同时在右侧显示ASCII表示。这对于检查二进制文件的内容非常有用，比如可执行程序、磁盘镜像或其他二进制数据。

Anyway, you know the drill:

`nasm -fbin boot_sect_hello.asm -o boot_sect_hello.bin`

`qemu boot_sect_hello.bin`

Your boot sector will say 'Hello' and hang on an infinite loop
