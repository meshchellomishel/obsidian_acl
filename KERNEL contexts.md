
**Types of contexts**

- process context
- irq context

But irq context may be different:

- softirq
- hardirq

[stack overflow](https://stackoverflow.com/posts/47331677/timeline)	

The purpose of this switch for **hardware interrupts** is that the **hardware** wants to pass some data to the **program** (GPIO toggled, new chars on UART arrived, etc). It's easy to understand, since when you're writing your program, all you need to do is to implement the handlers - whenever HW needs your action, the handler is called.

The purpose of this switch for **software interrupts** is that the **program** wants to pass some data to the **hardware**. More specifically, it wants to access some resources which cannot be accessed from current context. It applies to a common design, where we don't want high level applications to mess with the hardware, e.g. write directly to flash or change USB control registers, so we block it from upper layers and delegate this task to the OS core (like linux kernel). The core receives requests via software interrupt from upper layers, performs some HW-related operations and returns responses. Software interrupts are also be used to trigger OS maintenance tasks - e.g. context switch when currently executed task blocks on mutex.

***

#### Maybe useful


1. [More about interrupts](https://www.oreilly.com/library/view/understanding-the-linux/0596005652/ch04s06.html)
2. [About schedule](https://linux-kernel-labs.github.io/refs/pull/189/merge/labs/deferred_work.html)
3. [About spin lock](https://stackoverflow.com/questions/50637489/spin-lock-irqsave-in-interrupt-context)
4. [Book about kernel programming](https://www.oreilly.com/library/view/linux-kernel-programming/9781789953435/?_gl=1*zko5j0*_ga*MTA3NjI2MDQyOC4xNzIwNTE2MjU2*_ga_092EL089CH*MTcyMDUzMzkwNS4yLjEuMTcyMDUzMzk0OS4xNi4wLjA.)
5. [Another book](http://www.doc-developpement-durable.org/file/Projets-informatiques/cours-&-manuels-informatiques/Linux/Linux%20Kernel%20Development,%203rd%20Edition.pdf)
6. 