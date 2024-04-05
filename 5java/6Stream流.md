作用：结合lambda表达式，简化集合和数组的操作
可以理解为是一种流水线作业，将集合或数组中的元素放到流水线（stream流）上，进行响应的处理后（使用链式编程可以更方便的处理），进行输出或其他操作
使用步骤：获取Stream流对象，使用中间方法处理数据，使用终结方法处理数据
获取Stream流对象的方法：
单列集合：collection中的默认方法stream
双列集合：不能直接获取（可以转化为单列集合后获取）
数组：Arrays工具类中的静态方法stream
零散数据（相同数据类型）：Stream接口中的静态方法of
常见的方法：
中间方法：filter，limit，skip，distinct，concat，map
终结方法：forEach，count，collect，toArray

## 获取Stream流对象：
```
//单列集合
ArrayList<String> list=new ArrayList<>();  
Collections.addAll(list,"a","b","c","d","e");  
//创建流水线对象，后续可以进行其他操作
Stream<String> stream1 = list.stream();

//双列集合
HashMap<String,Integer> hashMap=new HashMap<>();  
hashMap.put("aaa",111);  
hashMap.put("bbb",222);  
hashMap.put("ccc",333);  
hashMap.put("ddd",444);  
hashMap.put("eee",555);
//1.通过获取键构造单列集合
Stream<String> stream2 = hashMap.keySet().stream();
//2.通过获取键值对构造单列集合
Stream<Map.Entry<String, Integer>> stream = hashMap.entrySet().stream();

//数组

//基本数据类型
int[] arr1={1,2,3,4,5,6};
//引用数据类型
String[] arr2={"a","b","c"};  
IntStream stream = Arrays.stream(arr1);  
Stream<String> stream1 = Arrays.stream(arr2);

//零散数据
Stream.of(1,2,3,4,5);
Stream.of("a","b","c");

//Stream接口中静态方法of的细节
//方法的形参是一个可变参数，可以传递一堆零散的数据。也可以传递数组
//但是数组必须是引用数据类型的，如果传递基本数据类型，是会把整个数组当做一个元素，放到Stream当中。
```
## 中间方法
![[Pasted image 20230713163230.png]]
1.中间方法返回的是新的stream流，原来的stream流只能使用一次，因此建议使用链式编程
2.修改stream流中的数据，不会影响原来集合或数组中的数据
```
//filter过滤函数
list.stream().filter(s->过滤条件(满足条件留下));

//limit截取前几个元素，可以配合filter或skip使用，在某条件下再截取前几个元素
list.stream().limit(int);

//skip跳过前几个元素，可以配合filter或skip使用，在某条件下再跳过前几个元素
list.stream().skip(int);

//distinct去重(依赖hashcode和equals方法，若传递自定义数据类型，需重写这两个方法)
list.stream().distinct();

//concat合并两个流(尽可能使两个流的数据类型保持一致)
Stream.concat(list1.stream(),list2.stream);

//map改变数据类型（但更像是提供一个对流中数据进行操作的平台）
list.stream().map(s-> Integer.parseInt(s.split( regex: "-")[1]));

```
## 终结方法
```
//forEach遍历
list.stream().forEach(s->操作)；

//count记录集合元素个数
list.stream().count();

//toArray将流中的数据收集起来存到一个数组中
//参数的作用：创建一个指定类型的数组
//返回值：装着流中所有数据的数组
list.stream().toArray(value(元素个数)->new String[value]);

//collect收集流中的数据，放到集合中（List Set Map）
List<String> newList = list.stream( )
		.filter(s ->“男”".equals(s.split( regex:"-"")[1]))
		.collect(collectors.toList()/toSet());
List<String> newList = list.stream( )
		.filter(s ->“男”".equals(s.split( regex:"-"")[1]))
		.collect(collectors.toMap(键的生成规则，值的生成规则));
  
//    参数一:  
//    Function泛型一:表示流中每一个数据的类型  
//    泛型二:表示Nap集合中键的数据类型  
//    方法apply形参:依次表示流里面的每一个数据  
//    方法体:生成键的代码  
//    返回值:己经生成的键  
//    参数二:  
//    Function泛型一:表示流中每一个数据的类型  
//    泛型二:表示Hap集合中值的数据类型  
//    方法apply形参:依次表示流里面的每一个数据  
//    方法体:生成值的代码  
//    返回值:已经生成的值
Map<String,Integer> map2 = list.stream()  
        .filter(s ->"男".equals(s.split( "-")[1]))  
        .collect(Collectors.toMap(  
	        s-> s.split("-")[1],  
	        s->Integer.parseInt(s.split("-")[2])));
	
```
