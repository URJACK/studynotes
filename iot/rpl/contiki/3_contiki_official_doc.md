# Contiki

Contiki是为了Wireless Sensor Network设计的一个嵌入式系统，侧重于网络。（虽然Contiki也可以单独运行，但是这并不是它设计的初衷。）

| 层级               | 使用协议                |
| ------------------ | ----------------------- |
| Application        | CoAP                    |
| Transport          | UDP                     |
| Network            | IpV6、RPL               |
| Adaptation         | 6LoWPAN                 |
| MAC                | CSMA、link-layer bursts |
| Radio Duty Cycling | ContikiMAC              |
| Physical           | 802.15.4                |

其中Contiki是一个事件驱动型操作系统，有一个事件驱动的核心。

主要的进程模式是protothread，这个进程模式，使得contiki这个操作系统只使用一个堆栈，节约了本不富裕的嵌入式设备上的内存。

但是这也造成了，在进行进程切换时，系统不会去保存当前进程的堆栈，及寄存器信息，所以contiki也警告说，在protothread模式中编程时，local variable要小心使用。<u>所以一般都是定义 static 和global variable。</u>防止切换时，变量信息丢失。

## 官方文档Multitasking and scheduling

Contiki-NG采用协作式多任务处理样式的**原始Contiki事件驱动模型**。 抢占式多线程支持已被删除，这使得Contiki-NG严格非抢占式。

### 进程与事件Processes and Events

Contiki-NG中的应用程序通常是使用Process抽象编写的。 进程建立在称为Protothreads的轻量级线程库的顶部。

在典型的执行方案中，进程（或原型线程）将空闲，直到接收到事件为止。 在接收到事件时，该进程将通过执行相应的工作块来消耗它，然后它将暂停自己的执行，直到接收到下一个事件为止。 一些示例事件是：

- 计时器到期，
- 接收传入的网络数据包
- 通过串行线路接收消息
- 用户操作，例如按下按钮
- 传感器驱动器指示有新读数可用
- 用户定义的事件。

由于Contiki-NG调度程序是协作的，因此它将永远不会在正在运行的进程（或**原型线程**）与另一个进程之间进行上下文切换。 因此，为了使事情正常运行，每个正在运行的**原型线程**都必须在完成任务后<u>自愿将控制权交还给调度程序</u>。 因此，对于开发人员而言，重要的是要确保进程不会过多地控制控件，并且将长时间的操作拆分为多个进程调度，以使此类操作可以在它们最后一次停止时恢复。

### Contiki-NG调度程序和事件调度

Contiki-NG模型使用两种类型的事件：**异步**和**同步**。

- **异步事件**被放入队列中，并以循环方式分派给接收进程。 异步事件可以发布到特定进程，也可以广播。 在后一种情况下，内核会将事件分派给所有进程。 无法指定进程将接收广播事件的顺序。
- **同步事件**使接收过程立即得到安排。 通常，这将导致发布过程的执行停止，直到接收过程使用完该事件为止，此时恢复发布过程。 同步事件无法广播。

Contiki-NG还支持**轮询机制**。 轮询是分派给单个接收进程（无法广播）的<u>特定类型的高优先级异步事件</u>。

内核将在异步事件分派之前以及之间调度所有轮询的进程。 如果是广播事件，则在将事件分发到两个连续的进程之间也将进行轮询。

### 中断和上下文Interrupts and contexts

Contiki-NG多任务模型的非抢先，合作性质意味着，进程永远不会被调度程序意外中断。 但是，进程的执行可能（并且经常会）被硬件中断处理程序意外中断。 中断处理程序本身可以被更高优先级的中断处理程序抢占。

考虑到这一点，可以根据在其中执行上下文的上下文来对Contiki-NG代码进行分类：

- 代码始终在中断上下文之外运行； 我们将其称为“主”（或“主线程”）上下文
- （总是或有时）在中断上下文中运行的代码。

始终在主上下文中运行的代码的实现要简单得多，例如不需要重新输入功能。

### 访问共享资源Access to Shared Resources

同步访问共享资源在很大程度上取决于代码的哪些部分访问它们。

当仅从主线程上下文中访问共享资源时，开发人员无需依赖任何同步原语，因为调度程序不会意外中断对资源的读/写操作。

当可以从中断上下文内部以及外部访问共享资源时，事情就变得更加复杂。 在这种情况下，开发人员需要确保资源状态保持一致。 为此，Contiki-NG提供了一些基本的同步原语（例如互斥体和关键部分），这些原语在[synchprimitives](https://github.com/contiki-ng/contiki-ng/wiki/Documentation:-Synchronization-primitives)以及[在线API文档](https://contiki-ng.readthedocs.io/en/master/)中都有详细记录。

### 编写中断处理程序Writing interrupt handlers

考虑到以上所有内容，在开发可能在中断上下文中运行的代码时，需要格外小心。 中断处理程序开发人员需要记住，以下Contiki-NG系统和库函数在中断上下文中运行不安全：

- 发布事件：并且在中断上下文中调用是不安全的。 如果中断处理程序需要对进程进行调度，则应使用轮询机制，而不要调用:

  ```
  process_post() process_post_synch() process_poll()
  ```

- 某些数据结构操作库在中断上下文中不安全使用。 这包括：

  - The main linked list library (),`list.[ch]`
  - The queue library (),`queue.h`
  - The stack library (),`stack.h`
  - The circular linked list library (),`circular-list.[ch]`
  - The doubly-linked list library (),`dbl-list.[ch]`
  - The circular, doubly-linked list library ().`dbl-circ-list.[ch]`

- 计划计时器：事件计时器()，回调计时器()和trick流计时器()库依赖于列表操作。 因此，在中断上下文中操作事件和回调计时器是不安全的

  ```
  etimer ctimer trickle-timer
  ```

- 邻居表操作：nbr.[hc]中的所有功能。

- 看门狗定时器永远不应该在中断上下文中刷新：从中断内部调用可能会导致设备永远不会从固件崩溃中恢复的情况，因为其WDT在中断中被刷新得足够频繁了.watchdog_periodic()

## 官方文档Processes and events

Contiki-NG中的应用程序通常是使用Process抽象编写的。 进程建立在称为[Protothreads](https://dl.acm.org/doi/10.1145/1182807.1182811)的轻量级线程库的顶部。

### 进程定义Process Definition

首先在源文件的顶部声明一个进程。 <u>宏带有两个参数：一个是用于**标识进程的变量**，另一个是**进程的名称**。</u> 在第二行，我们告诉Contiki-NG，该过程应在系统启动后立即自动启动。 <u>可以在此处指定多个进程，以逗号分隔</u>。 如果该行不包含现有过程，则必须使用该功能手动启动该过程。

```c
#include "contiki.h" /* Main include file for OS-specific modules. */
#include <stdio.h> /* For printf. */

PROCESS(test_proc, "Test process");
AUTOSTART_PROCESSES(&test_proc);
```

一个基础的进程可以按照下列方式来实现：

```c
PROCESS_THREAD(test_proc, ev, data)
{
  PROCESS_BEGIN();

  printf("Hello, world!\n");

  PROCESS_END();
}
```

宏首先取调用中指定的进程的标识符。和参数包含传入事件的值，以及指向事件参数对象的可选指针。标志着进程执行开始的地方。在大多数情况下，<u>除了变量定义外</u>，程序员应<u>避免将代码放在正文的这条语句之上</u>。

```
PROCESS_THREAD() PROCESS() ev data PROCESS_BEGIN() PROCESS_THREAD()
```

对于我们的基本进程实现来说，我们不需要关注任何事件处理，因为我们只是在执行一个单一的语句，然后通过让它达到调用隐式退出进程。

```
PROCESS_END()
```

### 事件与安排Events and Scheduling

Contiki-NG建立在基于事件的执行模型上，在该模型中，进程通常在告诉调度程序它们正在等待事件之前执行大量工作，从而暂停执行。 此类事件可能是<u>计时器到期</u>，<u>传入的网络数据包</u>或<u>正在传递的串行线路消息</u>。

<u>协同安排进程，这意味着每个进程负责将控制权自愿交还给操作系统</u>，而不必执行太长时间。 因此，应用程序开发人员必须确保将长时间运行的操作拆分为多个进程调度，以允许此类操作在上一次停止的位置恢复。

#### 等待事件Waiting for events

通过在和调用所隔开的区域进行调用，可以将控制权交给调度器，只有在事件发生时才恢复执行。进程在调用后必须满足这个条件，才能继续执行。 如果条件不满足，进程就会将控制权交还给OS，直到新的事件发生。

```c
PROCESS_THREAD(test, ev, data)
{
  /* An event-timer variable. Note that this variable must be static
     in order to preserve the value across yielding. */
  static struct etimer et;

  PROCESS_BEGIN();

  etimer_set(&et, CLOCK_SECOND); /* Trigger a timer after 1 second. */
  while(1) {
    PROCESS_WAIT_EVENT_UNTIL(etimer_expired(&et));
    etimer_reset(&et);
  }

  PROCESS_END();
}
```

#### 暂停与让出Pausing and yielding

要自愿释放调度程序的控制权，可以调用，如下例所示。 然后，调度程序将传递所有排队的事件，然后立即调度已暂停的进程。

```c
PROCESS_THREAD(test_proc, ev, data)
{
  PROCESS_BEGIN();

  while(have_operations_to_do()) {
    do_some_operations();
    PROCESS_PAUSE();
  }

  PROCESS_END();
}
```

相反，它将把控制权交还给调度器，而不期望在不久之后再次被调度。相反，它将等待一个传入的事件，类似于 ，但没有必要的条件参数。外部进程或模块可以通过调用.net来轮询一个已经让步的进程。要轮询一个用变量声明的进程，可以调用 。被轮询的进程将被立即调度，并将事件传递给它。

#### 停止进程Stopping processes

一个过程可以通过三种方式停止：

- 通过允许语句被执行并隐式退出过程. <u>PROCESS_END()</u>
- 通过调用主体显式退出该过程。<u>PROCESS_EXIT()</u>  <u>PROCESS_THREAD</u>
- 另一个进程通过调用<u>process_exit()</u>杀死该进程。

停止进程后，可以通过调用从头开始重新启动。<u>process_start()</u>

#### 系统定义事件System-defined Events

Contiki-NG使用基于系统定义的事件进行常见操作，如下所示。

| Event                         | ID   | Description                                                  |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| PROCESS_EVENT_NONE            | 0x80 | No event.                                                    |
| PROCESS_EVENT_INIT            | 0x81 | Delivered to a process when it is started.在启动时交付给进程。 |
| PROCESS_EVENT_POLL            | 0x82 | Delivered to a process being polled.交付给正在轮询的进程。   |
| PROCESS_EVENT_EXIT            | 0x83 | Delivered to an exiting a process.交付给正在退出的进程。     |
| PROCESS_EVENT_SERVICE_REMOVED | 0x84 | Unused.                                                      |
| PROCESS_EVENT_CONTINUE        | 0x85 | Delivered to a paused process when resuming execution.恢复执行时交付给已暂停的进程。 |
| PROCESS_EVENT_MSG             | 0x86 | Delivered to a process upon a sensor event.在发生传感器事件时交付给进程。 |
| PROCESS_EVENT_EXITED          | 0x87 | Delivered to all processes about an exited process.交付给有关已存在进程的所有进程。 |
| PROCESS_EVENT_TIMER           | 0x88 | Delivered to a process when one of its timers expired.当其计时器之一到期时，将其交付给进程。 |
| PROCESS_EVENT_COM             | 0x89 | Unused.                                                      |
| PROCESS_EVENT_MAX             | 0x8a | The maximum number of the system-defined events.系统定义事件的最大数量。 |

#### 用户定义事件User-defined Events

还可以指定系统定义的事件未涵盖的新事件。 应该使用适当的范围定义和声明事件变量，以便事件的所有预期用户都可以访问它。 基本情况是仅在单个模块中使用它，然后可以将其定义为模块的顶部。

```c
static process_event_t my_app_event;
[...]
my_app_event = process_alloc_event();
```

请注意，不支持取消分配事件。

## 官方文档Synchronization primitives

Contiki-NG代码可以根据正在执行的上下文进行分类：

- 代码始终在中断上下文之外运行； 我们将其称为“主”（或“主线程”）上下文，
- （总是或有时）在中断上下文中运行的代码。

当仅从主线程上下文中访问共享资源时，开发人员无需依赖任何同步原语，因为调度程序不会意外中断对资源的读/写操作。

当可以从中断上下文内部以及外部访问共享资源时，事情就变得更加复杂。 在这种情况下，开发人员需要确保资源状态保持一致。 为此，Contiki-NG提供了此处记录的一些基本同步原语。

[doc：multitasking](https://github.com/contiki-ng/contiki-ng/wiki/Documentation:-Multitasking-and-scheduling)中有关Contiki-NG的多任务模型，流程调度程序和上下文的更多详细信息。

### 全局中断操纵Global Interrupt Manipulation

Contiki-NG提供了一个API，该API允许启用/禁用/恢复全局中断。 应尽可能避免禁用全局中断，并且在任何情况下都应仅在非常短的时间内执行。

要禁用全局中断，请使用以下功能：

```c
int_master_status_t int_master_read_and_disable(void);
```

该函数的返回值指示全局中断被禁用之前的状态。 此返回值的语义取决于平台，并且不应使用其值以跨平台的方式解释全局中断的状态。 此返回值应存储在变量中，然后使用该函数将其用于将全局中断恢复到其先前状态：

```c
void int_master_status_set(int_master_status_t status);
```

### 数据存储屏障Data Memory Barriers

Contiki-NG提供了用于插入编译器/ CPU内存屏障（memory_barrier()）的API，然后可将其用于实现关键部分。 实现内存屏障的插入是CPU代码开发人员的责任。

### 关键部分Critical sections

Contiki-NG提供了独立于平台/ CPU的功能，用于输入/退出关键代码部分。 这些功能依赖于上面的全局中断操作API的实现。

### 互斥体Mutexes

Contiki-NG具有与平台无关的API，用于操作互斥锁。

互斥体可以使用特定于CPU的同步指令来实现，例如，对所有受支持的基于Cortex-M3 / M4的CPU都可以完成。 如果未提供特定于CPU的实现，则该库将退回到默认实现，该默认实现在关键部分内操作互斥锁，因此依赖于在短时间内禁用全局中断。

### 原子比较和交换Atomic compare-and-swap

Contiki-NG提供了一个API，用于对8位变量进行原子比较和交换（CAS）操作。 可以使用与互斥锁非常类似的方式，使用特定于CPU的同步指令来实现8位原子CAS。 如果未提供特定于CPU的实现，则该库将退回到默认实现，该默认实现在关键部分内执行CAS操作，因此依赖于在短时间内禁用全局中断。

## 官方文档Timers

Contiki-NG提供了一组计时器库，供应用程序和OS本身使用。 计时器库包含用于检查时间段是否已过去，在计划的时间从低功耗模式唤醒系统以及实时任务计划的功能。

所有计时器都建立在时钟模块上，负责基本系统时间：

- `timer`: a simple timer, without built-in notification (caller must check if expired). Safe from interrupt.

  一个简单的计时器，没有内置通知（呼叫者必须检查是否已过期）。 不受干扰。

- `stimer`: same as `timer`, but in seconds and with significantly longer wrapping period. Safe from interrupt.

  与`timer`相同，但以秒为单位，并且包装时间明显更长。 不受干扰。

- `etimer`: schedules events to Contiki-NG processes. Unsafe from interrupt.

  将事件安排到Contiki-NG流程中。 不安全的中断。

- `ctimer`: schedules calls to a callback function. Unsafe from interrupt.

  安排对回调函数的调用。 不安全的中断。

- `rtimer`: real-time task scheduling, with execution from ISR. Safe from interrupt.

  实时任务调度，由ISR执行。 不受干扰。

### 时钟模块The Clock Module

时钟模块提供处理系统时间的功能。 下表中显示了时钟模块的API。

| Function                        | Purpose                                                      |
| ------------------------------- | ------------------------------------------------------------ |
| `clock_time_t clock_time()`     | Get the system time in `clock_time_t` ticks.以Clock_time_t滴答表示获取系统时间。 |
| `unsigned long clock_seconds()` | Get the system time in seconds.以秒为单位获取系统时间。      |
| `void clock_wait(int delay)`    | Delay the CPU for a number of clock ticks.将CPU延迟一定数量的时钟滴答。 |
| `void clock_init(void)`         | Initialize the clock module.初始化时钟模块。                 |
| `CLOCK_SECOND`                  | The number of ticks per second.每秒的滴答数。                |

系统时间被指定为与平台有关的类型clock_time_t，在大多数平台中，这是一个有限的无符号值，当它变大时会回绕。 引导时系统时间从零开始。 另外，clock_wait()会在指定的时钟滴答数内阻止CPU。

### ★计时器库The Timer Library

Contiki-NG计时器库提供了用于设置，重置和重新启动计时器以及检查计时器是否已过期的功能。 计时器被声明为结构计时器，对计时器的所有访问均由指向已声明计时器的指针进行。 下表显示了Contiki-NG计时器库的API。

| Function                                                 | Purpose                                                      |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| `void timer_set(struct timer *t, clock_time_t interval)` | 启动计时器。Start the timer.                                 |
| `void timer_reset(struct timer *t)`                      | 从上一个到期时间重新启动计时器。Restart the timer from the previous expire time. |
| `void timer_restart(struct timer *t)`                    | 从当前时间重新启动计时器。Restart the timer from current time. |
| `int timer_expired(struct timer *t)`                     | 检查计时器是否到期。Check if the timer has expired.          |
| `clock_time_t timer_remaining(struct timer *t)`          | 获取时间，直到计时器到期。Get the time until the timer expires. |

始终通过<u>调用timer_set()来初始化计时器</u>，该计时器将计时器设置为从当前时间起终止指定的延迟，并存储时间间隔。 所有其他功能也会在此延迟上运行。 下面的示例说明如何使用计时器来检测中断中的超时。

```c
#include "sys/timer.h"

static struct timer rxtimer;

void init(void) {
  timer_set(&rxtimer, CLOCK_SECOND / 2);
}

interrupt(UART1RX_VECTOR)
uart1_rx_interrupt(void)
{
  if(timer_expired(&rxtimer)) {
    /* Timeout */
    ...
  }
  timer_restart(&rxtimer);
  ...
}
```

### The Stimer Library

Contiki-NG stimer库提供类似于计时器库的计时器机制，但是使用以秒为单位的时间值，从而允许更长的到期时间。 stimer库在clock模块中使用clock_seconds()来获取当前系统时间（以秒为单位）。

stimer库的API如下所示。 它类似于计时器库，但是区别在于，时间指定为秒而不是时钟滴答。

| Function                                                    | Purpose                                                      |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| `void stimer_set(struct stimer *t, unsigned long interval)` | 启动计时器。Start the timer.                                 |
| `void stimer_reset(struct stimer *t)`                       | 从上一个到期时间重新启动stimer。Restart the stimer from the previous expire time. |
| `void stimer_restart(struct stimer *t)`                     | 从当前时间重新启动stimer。Restart the stimer from current time. |
| `int stimer_expired(struct stimer *t)`                      | 检查stimer是否已过期。Check if the stimer has expired.       |
| `unsigned long stimer_remaining(struct stimer *t)`          | 获取计时器到期之前的时间。Get the time until the timer expires. |

stimer库可以安全地从中断中使用。

### The Etimer Library

Contiki-NG etimer库提供了一种生成定时事件的计时器机制。 事件计时器将在事件计时器到期时将事件`PROCESS_EVENT_TIMER`发布到设置计时器的进程中。 etimer库在clock模块中使用clock_time来获取当前系统时间。

事件计时器被声明为`struct etimer`，对事件计时器的所有访问均由指向已声明的事件计时器的指针进行。

下表显示了etimer库的API。

| Function                                                   | Purpose                                                      |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| `void etimer_set(struct etimer *t, clock_time_t interval)` | 启动计时器。Start the timer.                                 |
| `void etimer_reset(struct etimer *t)`                      | 从上一个到期时间重新启动计时器。Restart the timer from the previous expire time. |
| `void etimer_restart(struct etimer *t)`                    | 从当前时间重新启动计时器。Restart the timer from current time. |
| `void etimer_stop(struct etimer *t)`                       | 停止计时器。Stop the timer.                                  |
| `int etimer_expired(struct etimer *t)`                     | 检查计时器是否已过期。Check if the timer has expired.        |
| `int etimer_pending()`                                     | 检查是否有任何未过期的事件计时器。Check if there are any non-expired event timers. |
| `clock_time_t etimer_next_expiration_time()`               | 获取下一个事件计时器的到期时间。Get the next event timer expiration time. |
| `void etimer_request_poll()`                               | 通知etimer库，系统时钟已更改。Inform the etimer library that the system clock has changed. |

请注意，计时器事件已发送到用于安排事件计时器的Contiki-NG进程。 如果应从回调函数或其他进程安排事件计时器，则可以使用`PROCESS_CONTEXT_BEGIN()`和`PROCESS_CONTEXT_END()`临时更改进程上下文。

以下示例显示如何使用etimer安排进程每秒运行一次。

```c
#include "sys/etimer.h"

PROCESS_THREAD(example_process, ev, data)
{
  static struct etimer et;
  PROCESS_BEGIN();

  /* Delay 1 second */
  etimer_set(&et, CLOCK_SECOND);

  while(1) {
    PROCESS_WAIT_EVENT_UNTIL(etimer_expired(&et));
    /* Reset the etimer to trig again in 1 second */
    etimer_reset(&et);
    ...
  }
  PROCESS_END();
}
```

### The Ctimer Library

The Contiki-NG ctimer library provides a timer mechanism that calls a specified function when a callback timer expires. The ctimer library use `clock_time()` in the clock module to get the current system time.

The API for the ctimer library is shown below.

| Function                                                     | Purpose                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `void ctimer_set(struct ctimer *t, clock_time_t interval, void (*callback)(void *), void *ptr)` | 启动计时器。Start the timer.                                 |
| `void ctimer_reset(struct ctimer *t)`                        | 从上一个到期时间重新启动计时器。Restart the timer from the previous expire time. |
| `void ctimer_restart(struct ctimer *t)`                      | 从当前时间重新启动计时器。Restart the timer from current time. |
| `void ctimer_stop(struct ctimer *t)`                         | 停止计时器。Stop the timer.                                  |
| `int ctimer_expired(struct ctimer *t)`                       | 检查计时器是否已过期。Check if the timer has expired.        |

该API与etimer库相似，主要区别在于`ctimer_set()`采用回调函数指针和数据指针作为参数。 当ctimer过期时，它将使用数据指针作为参数来调用回调函数。

下面的示例演示如何使用ctimer每秒安排一次对函数的回调。

```c
#include "sys/ctimer.h"
static struct ctimer timer;

static void callback(void *ptr)
{
  ctimer_reset(&timer);
  ...
}

void init(void)
{
  ctimer_set(&timer, CLOCK_SECOND, callback, NULL);
}
```

请注意，尽管回调计时器正在调用指定的回调函数，但回调的进程上下文设置为用于调度ctimer的进程。 除非您确定如何安排回调计时器，否则不要在回调中假设任何特定的进程上下文。

### The Rtimer Library

Contiki-NG rtimer库提供了实时任务的调度和执行。 rtimer库使用其自己的时钟模块进行调度，以实现更高的时钟分辨率。 宏`RTIMER_NOW()`用于获取当前系统时间（以滴答为单位），而`RTIMER_SECOND`指定每秒的滴答数。

与Contiki-NG中的其他计时器库不同，实时任务会抢占正常执行，以使任务立即执行。 这为实时任务中的操作设置了一些限制，因为大多数功能都不处理抢占。 诸如`process_poll()`之类的中断安全功能始终可以在实时任务中安全使用，但是任何可能与正常执行冲突的功能都必须进行同步。

**Contiki-NG当前仅支持一个活动的rtimer**。 除其他外，这意味着，如果您使用具有自己的rtimer的系统功能（例如TSCH堆栈），则将无法在应用程序级别使用rtimer。

下表显示了rtimer库的API。

| Function                                                     | Purpose                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `void rtimer_set(struct rtimer *task, timer_clock_t time, rtimer_clock_t duration, rtimer_callback_t func, void *ptr)` | 设置一个实时任务。Setup a real-time task.                    |
| `RTIMER_NOW()`                                               | 获取当前时间。Get the current time.                          |
| `RTIMER_CLOCK_LT(t0,t1)`                                     | 检查时间t0是否小于时间t1。Check if the time `t0` is less than the time `t1`. |
| `RTIMER_SECOND`                                              | 每秒的滴答数。The number of ticks per second.                |
| `void rtimer_init(void)`                                     | 初始化rtimer库。Initialize the rtimer library.               |
| `void rtimer_run_next(void)`                                 | 由rtimer调度程序调用以运行下一个实时任务。Called by the rtimer scheduler to run next real-time task. |

实时任务始终通过调用函数`rtimer_set()`来初始化，该函数设置延迟（时间）回调函数指针和数据指针。 持续时间字段当前未使用。 以下示例显示了如何将实时任务设置为每秒执行四次。

```c
#include "sys/rtimer.h"

static struct rtimer task;

static void callback(struct rtimer *t, void *ptr, int status)
{
  if(rtimer_reschedule(&task, RTIMER_SECOND / 4, DURATION) != RTIMER_OK) {
    /* Failed to reschedule timer. Recover by rescheduling from current time. */
    rtimer_schedule(&task, RTIMER_SECOND / 4, DURATION);
  }
  ...
}

void init(void)
{
  rtimer_setup(&task, RTIMER_HARD, callback, NULL);
  rtimer_schedule(&task, RTIMER_SECOND / 4, DURATION);
}
```

## 官方文档Memory management

Contiki-NG支持静态和动态内存分配。 在嵌入式系统中，传统上，内存分配仅限于静态大小，因为静态内存没有泄漏和碎片。 但是，当运行时内存需求发生变化时，静态内存将很麻烦。 此类更改可能发生在跟踪连接的Web服务器或支持动态编程语言的虚拟机中。 当限制为静态内存时，程序员必须猜测资源的最大使用量，并过度分配内存块以防止内存耗尽。 为了缓解此类问题，除了静态内存外，我们还提供了两种不同类型的内存分配器：半动态MEMB模块和动态HeapMem模块。

### MEMB: Memory Blocks内存块

在`os / lib / memb.h`中声明的MEMB库提供了一组内存块管理功能。 内存块被分配为大小恒定的对象数组，并放置在静态内存中。 API如下所示：

| Function                                     | Purpose                                                      |
| -------------------------------------------- | ------------------------------------------------------------ |
| `MEMB(name, structure, num)`                 | 声明一个内存块Declare a memory block.                        |
| `void memb_init(struct memb *m)`             | 初始化一个内存块Initialize a memory block.                   |
| `void *memb_alloc(struct memb *m)`           | 分配一个内存块Allocate a memory block.                       |
| `char memb_free(struct memb *m, void *ptr)`  | 释放一个内存块Free a memory block.                           |
| `int memb_inmemb(struct memb *m, void *ptr)` | 检查地址是否在存储块中。Check if an address is in a memory block. |

`MEMB()`宏声明一个内存块，其类型为`struct memb`。 由于该块已放入静态内存中，因此通常将其放置在使用内存块的C源文件的顶部。 `name`标识存储块，以后用作其他存储块功能的参数。`structure` parameter指定内存块的C类型，num表示内存块可容纳的对象数量。 `struct memb`的定义如下：

```c
struct memb {
  unsigned short size;
  unsigned short num;
  char *count;
  void *mem;
};
```

`MEMB`宏的扩展产生了三个定义静态内存的语句。 一个语句存储该内存块可以容纳的对象数量。 由于数量存储在`unsigned short`类型的变量中，因此存储块最多可以容纳`USHRT_MAX`对象。 第二条语句分配由`structure`参数引用的类型为`num`的结构数组。

一旦使用`MEMB()`声明了内存块，就必须通过调用`memb_init()`对其进行初始化。 此函数采用`struct memb`的参数来标识内存块。

初始化`struct memb`之后，我们准备使用`memb_alloc()`开始从中分配对象。 通过相同的`struct memb`分配的所有对象都具有相同的大小，这由`MEMB()`的`structure`参数的大小确定。 如果操作成功，则`memb_alloc()`返回指向已分配对象的指针；如果内存块没有可用对象，则返回NULL。

`memb_free()`取消分配先前使用`memb_alloc()`分配的对象。 需要两个参数来释放对象：`m`指向内存块，而`ptr`指向内存块中的对象。

可以检查任何指针以确定其是否在存储块的数据区域内。 如果`ptr`在内存块`m`内，则`memb_inmemb()`返回1；如果指向未知内存，则返回0。

我们展示了如何使用MEMB模块的示例。 `open_connection()`函数为`socket`标识的每个新连接分配一个新的`struct connection`变量。 关闭连接后，我们将释放内存块用于`struct connection`变量。

```c
#include "contiki.h"
#include "lib/memb.h"

struct connection {
  int socket;
};
MEMB(connections, struct connection, 16);

struct connection* open_connection(int socket)
{
  struct connection *conn;

  conn = memb_alloc(&connections);
  if(conn == NULL) {
    return NULL;
  }
  conn->socket = socket;
  return conn;
}

void close_connection(struct connection *conn)
{
  memb_free(&connections, conn);
}
```

### Heap Memory (HeapMem)堆内存

标准C库提供了一组用于分配和释放堆内存空间中的内存的函数。 对于不同的编译器工具链，目前尚不清楚默认堆内存模块在资源受限的执行环境中的性能如何。 在某些malloc实现中，大小不同的对象上的分配和释放模式可能更成问题。 因此，Contiki-NG包括一个堆存储器模块，该模块已在各种硬件平台和不同的应用程序上使用。 HeapMem模块的API与标准C的API类似。为避免名称冲突，HeapMem中的函数名称为`heapmem_alloc()`，`heapmem_realloc()`和`heapmem_free()`而不是`malloc()`，`realloc()`和`free ()`。 该API如下表所示。

| Function                                        | Purpose                                                      |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `void *heapmem_alloc(size_t size)`              | 分配未初始化的内存。Allocate uninitialized memory.           |
| `void *heapmem_realloc(void *ptr, size_t size)` | 更改分配的对象的大小。Change the size of an allocated object. |
| `void heapmem_free(void *ptr)`                  | 释放内存。Free memory.                                       |

本节中列出的所有函数都在C头文件`os / lib / heapmem.h`中声明。 `heapmem_alloc()`函数在堆上分配大小的内存字节。 如果成功分配了内存，`heapmem_alloc()`将返回一个指向它的指针。 如果没有足够的连续可用内存，`heapmem_alloc()`将返回`NULL`。

`heamem_realloc()`函数以新的`size`重新分配先前分配的块`ptr`。 如果新块较小，则将旧块中数据的`size`字节复制到新块中。 如果新块更大，则复制完整的旧块，而新块的其余部分包含未指定的数据。 一旦分配了新块并填充了其内容，便会释放旧块。 如果无法分配该块，`heapmem_realloc()`将返回NULL。 如果重新分配成功，`heapmem_realloc()`将返回一个指向新块的指针。

`heapmem_free()`释放先前通过`heapmem_alloc()`或`heapmem_realloc()`分配的块。 参数`ptr`必须指向已分配块的开始。

## 官方文档Packet buffers

该页面面向协议开发人员，描述了Contiki-NG中使用的不同类型的缓冲区。 重点是6LoWPAN堆栈，但是有关Packetbuf和Queuebuf的所有信息也适用于NullNet。

### Uip buffer

在网络层及更高层，数据包有效负载存储在uip_buf中。

为了发送数据，上层协议或应用程序应写入此缓冲区，然后触发传输。 uIP堆栈首先添加所需的IPv6标头和可能的扩展标头。 然后6LoWPAN将压缩标头，并在需要时对数据报进行分段。 每个片段将依次传递到MAC层进行传输。

在接收时，将执行相反的过程。 MAC层将调用6LoWPAN，后者将解压缩标头并将数据报重组为uip_buf，然后再调用uIP堆栈。 然后可以通过uIP和上层访问该数据报以进行处理。

uip_buf的访问规则：

- 仅来自6LoWPAN，uIP或更高版本，而不来自之下的任何层
- 仅在中断上下文之外

### Packetbuf

6LoWPAN会将链路层数据包直接构建到全局`packetbuf`中。 除了有效载荷外，`packetbuf` 还携带许多属性/元数据（请参见`packetbuf.h`）。 数据包准备好后，6LoWPAN会将其传递到MAC层进行传输。 同样，MAC层在接收数据包时，会通过`packetbuf`将数据包传递到6LoWPAN。`packetbuf`API的详细信息位于[doxygen:packetbuf](https://contiki-ng.readthedocs.io/en/latest/_api/group__packetbuf.html)。

允许规则 for `packetbuf`:

- 仅来自6LoWPAN或以下，但不来自之上的任何层
- 仅在中断上下文之外

### Queuebuf

`queuebuf`模块提供了一种同时管理多个数据包的方法。 每个`queuebuf`实例的内容基本上与全局`packetbuf`中的内容相同。 需要`queuebuf`的模块负责维护指向它们的指针，没有像`packetbuf`那样指向`queuebuf`的全局指针。

例如，当将IPv6数据报分段为多个数据包时，6LoWPAN使用`queuebuf`。 MAC层CSMA和TSCH也将`queuebuf`用于其发送队列。

`queuebuf`的访问规则：

- 仅来自6LoWPAN或以下，但不来自之上的任何层 
- 在中断上下文之外，或从中断上下文中，如果`queuebuf`实例受到锁的保护（如在TSCH中）。

启用后，Contiki-NG将自动初始化queuebuf模块。 根据您平台的配置，可以通过以下设置来禁用queuebuf模块：

```c
#define QUEUEBUF_CONF_ENABLED 0
```

请记住，某些Contiki-NG功能严格需要queuebuf模块（例如CSMA，TSCH，6LoWPAN碎片支持），因此，仅在确实需要任何此功能时，才应尝试禁用它。