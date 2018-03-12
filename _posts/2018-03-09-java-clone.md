---
layout: post
title:  "浅克隆与深克隆"
subtitle: ""
date:  2018-03-09
author: "ABei"
catalog: true
tags: 
    - Java
    - Summary
---

> Java中实现克隆有两种方案：重写父类的clone()和实现Serializable接口<br>

复制一个简单的变量，比如：

```java
int num = 5;
int copyNum = num
```

不仅仅是int类型，其他的基本类型也是适用这种情况。但是如果你复制的是一个对象，那么情况就会有些复杂了。

```java
class Student {  
    private int number;  
  
    public int getNumber() {  
        return number;  
    }  
  
    public void setNumber(int number) {  
        this.number = number;  
    }  
}  

public class Test {  
      public static void main(String args[]) {  
        Student stu1 = new Student();  
        stu1.setNumber(12345);  
        Student stu2 = stu1;  
          
        System.out.println("学生1:" + stu1.getNumber());  
        System.out.println("学生2:" + stu2.getNumber());  
    }  
}  
```

- 结果：
    - 学生1:12345
    - 学生2:12345

显然这个结果符合我们的预期，但是接下来我们改一下赋值方式。这个时候，我们改变stu2实例的number字段，将结果打印出来看看：

```java
stu2.setNumber(54321);  
System.out.println("学生1:" + stu1.getNumber());  
System.out.println("学生2:" + stu2.getNumber()); 
```

- 结果：
    - 学生1:54321
    - 学生2:54321

这个结果就有点意思了。stu2改变值的同时也将stu1的值也修改了，这个结果是有点意外了。为什么会出现这个情况呢？
其实，原因出在`stu1 = stu2;`这句话上面。stu1和stu2在内存堆中都指向同一个对象。
![](/img/in-post/2018-3-09-java-clone-memory.png)

我们想要的样子是stu2修改内容之后，不会影响到stu1的内容。显然这种情况不符合我们的预期。

那么，如何才能实现我们所期望的复制对象呢？
是否记得`Object`？它有一个protected的clone方法。
在Java中所有的类都是缺省的继承自Java语言包中的Object类的，查看它的源码，发现里面有一个访问限定符为protected的方法clone()：

```java
protected native Object clone() throws CloneNotSupportedException;
```

仔细查看，你会发现它是一个native方法。那么,clone()的具体实现不在当前文件中，而是在用其他语言实现的文件中。同时，每个类都默认继承于Object类，因此它的子类都含有clone()方法，要想对一个对象进行复制，就需要覆盖clone方法。

#### 为什么需要克隆？

在了解克隆之前，请思考一个问题。为什么需要克隆的存在呢？为什么不能new一个对象来代替呢？
其实原因很简单，new出来的对象，其成员属性都是初始化状态，而实际上情况是我们需要对象是正处在某个特定状态的对象。如果是通过new来拷贝对象，那么操作将会很复杂也很繁琐。

#### 浅克隆

- 步骤：
    1. 实现`Cloneable`接口
    2. 覆盖clone()方法

```java
class Student implements Cloneable{  
    private int number;  
  
    public int getNumber() {  
        return number;  
    }  
  
    public void setNumber(int number) {  
        this.number = number;  
    }  
      
    @Override  
    public Object clone() {  
        Student stu = null;  
        try{  
            stu = (Student)super.clone();  
        }catch(CloneNotSupportedException e) {  
            e.printStackTrace();  
        }  
        return stu;  
    }  
}  
public class Test {  
    public static void main(String args[]) {  
        Student stu1 = new Student();  
        stu1.setNumber(12345);  
        Student stu2 = (Student)stu1.clone();  
          
        System.out.println("学生1:" + stu1.getNumber());  
        System.out.println("学生2:" + stu2.getNumber());  
          
        stu2.setNumber(54321);  
      
        System.out.println("学生1:" + stu1.getNumber());  
        System.out.println("学生2:" + stu2.getNumber());  
    }  
}  
```

- 结果：
    - 学生1:12345
    - 学生2:12345
    - 学生1:12345
    - 学生2:54321

从结果上，stu2是stu1的复制品，并且stu2的修改不会影响到stu1的值，上面这种复制就是被称作浅克隆。

#### 深克隆

现在在Student类中，添加一个Address类：

```java
class Address {
    private String addInfo;

	public String getAddInfo() {
		return addInfo;
	}

	public void setAddInfo(String addInfo) {
		this.addInfo = addInfo;
	}
}

public class Student implements Cloneable {
	private Address addr; 
	private int number;

	public Address getAddr() {
		return addr;
	}

	public void setAddr(Address addr) {
		this.addr = addr;
	}

	public int getNumber() {
		return number;
	}

	public void setNumber(int number) {
		this.number = number;
	}

	@Override
	protected Object clone() {
		Student student = null;
		try {
			student = (Student) super.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		return student;
	}
}

public class Test {
    public static void main(String[] args) {
        Address addr = new Address();
        addr.setAddInfo("杭州市");
        Student stu1 = new Student();  
        stu1.setNumber(123);  
        stu1.setAddr(addr);  
        Student stu2 = (Student)stu1.clone();  
        System.out.println("学生1:" + stu1.getNumber() + ",地址:" + stu1.getAddr().getAdd());  
        System.out.println("学生2:" + stu2.getNumber() + ",地址:" + stu2.getAddr().getAdd()); 

        addr.setAdd("西湖区");  
        System.out.println("学生1:" + stu1.getNumber() + ",地址:" + stu1.getAddr().getAdd());  
        System.out.println("学生2:" + stu2.getNumber() + ",地址:" + stu2.getAddr().getAdd()); 
    }
}
```

- 结果：
    - 学生1:123,地址:杭州市  
    - 学生2:123,地址:杭州市  
    - 学生1:123,地址:西湖区  
    - 学生2:123,地址:西湖区

为什么克隆之后，设置address的时候还会修改全部student的address？**原因是stu2克隆的address其实是stu1中address的引用，而不是值。**<br>
那么我们怎么才能只复制address的值呢？
很简单，只要Address类也实现Cloneable接口，在Student的clone方法内也添加对Address的clone即可。具体如下：

```java
class Address implements Cloneable{
	private String add;

	public String getAdd() {
		return add;
	}

	public void setAdd(String add) {
		this.add = add;
	}

	@Override
	protected Object clone(){
		Address addr = null;
		try {
			addr = (Address) super.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		return addr;
	}
	
}

public class Student implements Cloneable {
	private Address addr; 
	private int number;

	public Address getAddr() {
		return addr;
	}

	public void setAddr(Address addr) {
		this.addr = addr;
	}

	public int getNumber() {
		return number;
	}

	public void setNumber(int number) {
		this.number = number;
	}

	@Override
	protected Object clone() {
		Student student = null;
		try {
			student = (Student) super.clone();
            student.addr = (Address) addr.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		return student;
	}
	
}
public class Test {  
      
    public static void main(String args[]) {  
          
        Address addr = new Address();  
        addr.setAdd("杭州市");  
        Student stu1 = new Student();  
        stu1.setNumber(123);  
        stu1.setAddr(addr);  
          
        Student stu2 = (Student)stu1.clone();  
          
        System.out.println("学生1:" + stu1.getNumber() + ",地址:" + stu1.getAddr().getAdd());  
        System.out.println("学生2:" + stu2.getNumber() + ",地址:" + stu2.getAddr().getAdd());  
          
        addr.setAdd("西湖区");  
          
        System.out.println("学生1:" + stu1.getNumber() + ",地址:" + stu1.getAddr().getAdd());  
        System.out.println("学生2:" + stu2.getNumber() + ",地址:" + stu2.getAddr().getAdd());  
    }  
}
```

- 结果：
    - 学生1:123,地址:杭州市  
    - 学生2:123,地址:杭州市  
    - 学生1:123,地址:西湖区  
    - 学生2:123,地址:杭州市 

这个结果符合我们的预期，上面这个过程就是深克隆。

#### Serializable

在上面的例子里面，其实已经都模拟了浅克隆和深克隆了。

#### Ref
[参考地址](http://blog.csdn.net/tounaobun/article/details/8491392)<br>