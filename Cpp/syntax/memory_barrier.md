# 内存栅栏

由于有了 cache 的存在，很多操作不用写到内存就可以继续执行后面的操作

为了保证某些操作时写入到内存后再执行，就引入了内存栅栏

内存栅栏的作用是保证所有栅栏之前的内存操作都要写入到内存中

内存栅栏是显示的在一些执行点上保证顺序一致性