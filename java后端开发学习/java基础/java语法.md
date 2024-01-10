# java语法知识

## Serializable接口

参考[谈谈我对Serializable接口的理解](https://juejin.cn/post/7090150041024725028)
Serializable接口中没有任何方法或者字段，只是用于标识可序列化的语义。实现了Serializable接口的类可以被ObjectOutputStream转换为字节流（序列化），同时也可以通过ObjectInputStream解析为对象（反序列化）。
简单理解：**序列化就是将内存中数据存储到磁盘中的过程**。

> 1. 静态成员变量是不能被序列化的——序列化是针对对象属性的，而静态成员变量是属于类的。
> 2. 当一个父类实现序列化，子类就会自动实现序列化，不需要显式实现Serializable接口。
> 3. 当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化。