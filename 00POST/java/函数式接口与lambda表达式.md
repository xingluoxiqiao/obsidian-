# 函数式接口
1. 函数式接口是指只包含一个抽象方法的接口。Java中引入了函数式接口的概念，主要是为了支持Lambda表达式的使用。Lambda表达式是一种轻量级的匿名函数，可以直接传递给函数式接口的对象。函数式接口的定义要求使用`@FunctionalInterface`注解，该注解用于强制检查接口是否符合函数式接口的标准。
2. 以下是函数式接口的特征：
	- 接口中只能有一个抽象方法（可以包含默认方法和静态方法）。
	- `@FunctionalInterface` 注解用于标识该接口是函数式接口（可选）。
# 函数式接口的实现方式
1. **使用匿名内部类：** 使用匿名内部类是最传统的方式，它允许直接实现接口的抽象方法。
```java
MyFunctionalInterface myFunction = new MyFunctionalInterface() {     
	@Override     
	public void myMethod() {         
		System.out.println("Implementation using anonymous inner class");     
	} 
};
```
2. **使用Lambda表达式：** Lambda表达式是一种更简洁的方式，特别适用于函数式接口。
```java
MyFunctionalInterface myFunction = 
	() -> System.out.println("Implementation using Lambda expression");
```
3. **使用方法引用：** 如果接口的抽象方法与现有方法的签名兼容，可以使用方法引用。方法引用提供了一种更简洁的语法。
```java
//在这里，`myObject::myMethod` 是对 `MyFunctionalInterface` 中的抽象方法的实现。
  MyClass myObject = new MyClass(); 
  MyFunctionalInterface myFunction = myObject::myMethod;
```
4. **使用构造方法引用：** 如果函数式接口代表的是一个构造函数，可以使用构造方法引用。
```java
//这里假设 `MyClass` 是一个函数式接口 `MyFunctionalInterface` 的实现类。
 MyFunctionalInterface myFunction = MyClass::new;
```
# lambda表达式
Lambda表达式是Java中引入的一种轻量级的函数式编程特性，它允许你将一个函数（或称为代码块）作为一个参数传递给方法，或者更简洁地实现函数式接口。

Lambda表达式的基本语法如下：
- `(parameters) -> expression`
- `(parameters) -> { statements; }`
其中，`parameters` 是Lambda表达式的参数列表，`expression` 或 `{ statements; }` 是Lambda表达式的主体。

以下是一些示例，演示了Lambda表达式的不同用法：
1. **不带参数的Lambda表达式：**
    `Runnable runnable = () -> System.out.println("Hello, Lambda!");` 
	`new Thread(runnable).start();`
2. **带参数的Lambda表达式：**
    `(x, y) -> x + y`
    这表示一个接受两个参数 `x` 和 `y`，并返回它们的和。
3. **带主体的Lambda表达式：**
```java
//主体可以包含多条语句，并且可以有返回值。
(x, y) -> {     
	int sum = x + y;     
	System.out.println("Sum: " + sum);     
	return sum; 
}
```
4. **遍历集合的Lambda表达式：**
```java
//这里使用了 `forEach` 方法和Lambda表达式，简化了集合的遍历操作。
    List<String> languages = Arrays.asList("Java", "Python", "JavaScript"); 
    languages.forEach(language -> System.out.println(language));`
```
Lambda表达式的引入使得代码更加简洁、易读，并支持更函数式的编程风格。在使用Lambda表达式时，通常用于实现函数式接口（只有一个抽象方法的接口）的匿名实现。