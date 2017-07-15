### ThreadLocal是什么？
线程局部变量(ThreadLocal)其实的功用非常简单，就是为每一个使用该变量的线程都提供一个变量值的副本，是Java中一种较为特殊的线程绑定机制，是每一个线程都可以独立地改变自己的副本，而不会和其它线程的副本冲突

从线程的角度看，每个线程都保持一个对其线程局部变量副本的隐式引用，只要线程是活动的并且 ThreadLocal 实例是可访问的；在线程消失之后，其线程局部实例的所有副本都会被垃圾回收（除非存在对这些副本的其他引用）。
 
通过ThreadLocal存取的数据，总是与当前线程相关，也就是说，JVM 为每个运行的线程，绑定了私有的本地实例存取空间，从而为多线程环境常出现的并发访问问题提供了一种隔离机制。
 
概括起来说，**对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，而ThreadLocal采用了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。**

### 案例
经典案例，也是常用的ThreadLocal使用方式：

    /** 
     * 数据库连接管理类 
     * @author 爽 
     * 
     */  
    public class ConnectionManager {  
      
        /** 线程内共享Connection，ThreadLocal通常是全局的，支持泛型 */  
        private static ThreadLocal<Connection> threadLocal = new ThreadLocal<Connection>();  
          
        public static Connection getCurrConnection() {  
            // 获取当前线程内共享的Connection  
            Connection conn = threadLocal.get();  
            try {  
                // 判断连接是否可用  
                if(conn == null || conn.isClosed()) {  
                    // 创建新的Connection赋值给conn(略)  
                    // 保存Connection  
                    threadLocal.set(conn);  
                }  
            } catch (SQLException e) {  
                // 异常处理  
            }  
            return conn;  
        }  
          
        /** 
         * 关闭当前数据库连接 
         */  
        public static void close() {  
            // 获取当前线程内共享的Connection  
            Connection conn = threadLocal.get();  
            try {  
                // 判断是否已经关闭  
                if(conn != null && !conn.isClosed()) {  
                    // 关闭资源  
                    conn.close();  
                    // 移除Connection  
                    threadLocal.remove();  
                    conn = null;  
                }  
            } catch (SQLException e) {  
                // 异常处理  
            }  
        }  
    }
    
这样处理的好处：

统一管理Connection；
不需要显示传参Connection，代码更优雅；
降低耦合性。

### 原理

ThreadLocal是如何做到为每一个线程维护变量的副本的呢？**其实实现的思路很简单，在ThreadLocal类中有一个Map，用于存储每一个线程的变量的副本**。

### 总结

 - ThreadLocal使用场合主要解决多线程中数据数据因并发产生不一致问题。ThreadLocal为每个线程的中并发访问的数据提供一个副本，通过访问副本来运行业务，这样的结果是耗费了内存，单大大减少了线程同步所带来性能消耗，也减少了线程并发控制的复杂度。

 

 - ThreadLocal不能使用原子类型，只能使用Object类型。ThreadLocal的使用比synchronized要简单得多。

 

 - ThreadLocal和Synchonized都用于解决多线程并发访问。但是ThreadLocal与synchronized有本质的区别。synchronized是利用锁的机制，使变量或代码块在某一时该只能被一个线程访问。而ThreadLocal为每一个线程都提供了变量的副本，使得每个线程在某一时间访问到的并不是同一个对象，这样就隔离了多个线程对数据的数据共享。而Synchronized却正好相反，它用于在多个线程间通信时能够获得数据共享。**Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。**
 当然ThreadLocal并不能替代synchronized,**它们处理不同的问题域**。Synchronized用于实现同步机制，比ThreadLocal更加复杂。

- 可以避免参数传递的访问方式


### ThreadLocal使用的一般步骤


1. 在多线程的类（如ThreadDemo类）中，创建一个ThreadLocal对象threadXxx，用来保存线程间需要隔离处理的对象xxx。
2. 在ThreadDemo类中，创建一个获取要隔离访问的数据的方法getXxx()，在方法中判断，若ThreadLocal对象为null时候，应该new()一个隔离访问类型的对象，并强制转换为要应用的类型。
3. 在ThreadDemo类的run()方法中，通过getXxx()方法获取要操作的数据，这样可以保证每个线程对应一个数据对象，在任何时刻都操作的是这个对象。
