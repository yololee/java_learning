# Java异常面试题

### Java异常架构

```java
- Throwable
    - Error(错误)
    	- Virtual MachineError（虚拟机运行错误）
    	- NoClassDefFoundError（类定义错误）
    	- OutOfMemoryError(内存不足错误)
    	- StackOverflowError(栈溢出错误)
    - Exception(异常)
    	- 运行时异常
    	- 编译时异常
```

### Error 和 Exception 区别是什么？

- Error：是程序正常运行中，不大可能出现的错误，通常为虚拟机相关错误，如系统崩溃，内存不足，堆栈溢出等。编译器不会对这类错误进行检测，应用程序也不应对这类错误进行捕获，一旦这类错误发生，通常应用程序会被终止，仅靠应用程序本身是无法恢复的
- Exception：是程序正常运行中，可以预料的意外情况，通常遇到这种异常，应对其进行处理，使应用程序可以继续正常运行。

### 运行时异常和一般异常(受检异常)有什么区别

- 运行时异常：包括 RuntimeException 类及其子类，表示 JVM 在运行期间可能出现的异常。Java 编译器不会检查运行时异常
- 受检异常：是Exception 中除 RuntimeException 及其子类之外的异常。Java 编译器会检查受检异常
- 区别：是否强制要求调用者必须处理此异常，如果强制要求调用者必须进行处理，那么就使用受检异常，否则就选择非受检异常(RuntimeException)。一般来讲，如果没有特殊的要求，我们建议使用RuntimeException异常。

### JVM 是如何处理异常的？

在一个方法中如果发生异常，这个方法会创建一个异常对象，并转交给 JVM，该异常对象包含异常名称，异常描述以及异常发生时应用程序的状态。创建异常对象并转交给 JVM 的过程称为抛出异常。可能有一系列的方法调用，最终才进入抛出异常的方法，这一系列方法调用的有序列表叫做调用栈。

JVM 会顺着调用栈去查找看是否有可以处理异常的代码，如果有，则调用异常处理代码。如果 JVM 没有找到可以处理该异常的代码块，JVM 就会将该异常转交给默认的异常处理器（默认处理器为 JVM 的一部分），默认异常处理器打印出异常信息并终止应用程序。

### throw 和 throws 的区别是什么？

- throw 关键字用在方法内部，只能用于抛出一种异常，用来抛出方法或代码块中的异常
- throws 关键字用在方法声明上，可以抛出多个异常，用来标识该方法可能抛出的异常列表。调用该方法的方法必须包含可处理异常的代码，否则也要在方法声明中用 throws 关键字声明相应的异常。

### try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

会执行，在 return 前执行。

### 常见的 RuntimeException 有哪些？

- ClassCastException(类转换异常)
- IndexOutOfBoundsException(数组越界)
- NullPointerException(空指针)
- ArrayStoreException(数据存储异常，操作数组时类型不一致)
- 还有IO操作的BufferOverflowException异常

