# Java泛型

## 参数命名约定

按照惯例，类型参数名称被命名为单个大写字母，因此可以使用普通类或接口名称轻松区分类型参数。以下是常用类型参数名称列表 -

- **E** - Element，主要由Java Collections框架使用。
- **K** - Key，主要用来表示地图的参数类型。
- **V** - 值，主要用于表示地图的参数类型值。
- **N** - 数字，主要用于表示数字。
- **T** - Type，主要用于表示第一个泛型类型参数。
- **S** - Type，主要用于表示第二个泛型类型参数。
- **U** - Type，主要用来表示第三个泛型类型参数。
- **V** - Type，主要用于表示第四个泛型类型参数。



```java

public class GenericsTester {
   public static void main(String[] args) {
      Box<Integer, String> box = new Box<Integer, String>();
      box.add(Integer.valueOf(10),"Hello World");
      System.out.printf("Integer Value :%d\n", box.getFirst());
      System.out.printf("String Value :%s\n", box.getSecond());

      Pair<String, Integer> pair = new Pair<String, Integer>();
      pair.addKeyValue("1", Integer.valueOf(10));
      System.out.printf("(Pair)Integer Value :%d\n", pair.getValue("1"));

      CustomList<Box> list = new CustomList<Box>();
      list.addItem(box);
      System.out.printf("(CustomList)Integer Value :%d\n", list.getItem(0).getFirst());
   }
}

class Box<T, S> {
   private T t;
   private S s;

   public void add(T t, S s) {
      this.t = t;
      this.s = s;
   }

   public T getFirst() {
      return t;
   }

   public S getSecond() {
      return s;
   }
}

class Pair<K,V>{
   private Map<K,V> map = new HashMap<K,V>();

   public void addKeyValue(K key, V value) {
      map.put(key, value);
   }

   public V getValue(K key) {
      return map.get(key);
   }
}

class CustomList<E>{
   private List<E> list = new ArrayList<E>();

   public void addItem(E value) {
      list.add(value);
   }

   public E getItem(int index) {
      return list.get(index);
   }
}

// 输出结果
Integer Value :10
String Value :Hello World
(Pair)Integer Value :10
(CustomList)Integer Value :10
```



## 类型推断

类型推断表示Java编译器查看方法调用及其相应声明以检查和确定类型参数的能力。推理算法检查参数的类型，如果可用，返回指定的类型。推理算法试图找到一个可以满足所有类型参数的特定类型。

编译器生成未经检查的转换警告不使用大小写类型推断。

**句法**

```java
Box<Integer> integerBox = new Box<>();
```

- **Box** - Box是一个通用类。
- **< >** - 钻石运算符表示类型推断

使用菱形运算符，编译器确定参数的类型。这个操作符从Java SE 7版本开始是可用的。

使用您选择的任何编辑器创建以下Java程序。

```java

public class GenericsTester {
   public static void main(String[] args) {
      //type inference   
      Box<Integer> integerBox = new Box<>();
      //unchecked conversion warning
      Box<String> stringBox = new Box<String>();

      integerBox.add(new Integer(10));
      stringBox.add(new String("Hello World"));

      System.out.printf("Integer Value :%d\n", integerBox.get());
      System.out.printf("String Value :%s\n", stringBox.get());
   }
}

class Box<T> {
   private T t;

   public void add(T t) {
      this.t = t;
   }

   public T get() {
      return t;
   }   
}

// 输出
Integer Value :10
String Value :Hello World
```



## 泛型方法

你可以编写一个可以用不同类型的参数调用的泛型方法声明。根据传递给泛型方法的参数类型，编译器会适当地处理每个方法调用。以下是定义通用方法的规则 

- 所有通用方法声明都有一个类型参数部分，该部分由方括号返回类型（下一个示例中的）前面的尖括号（<和>）分隔。
- 每个类型参数部分包含一个或多个由逗号分隔的类型参数。类型参数（也称为类型变量）是指定泛型类型名称的标识符。
- 类型参数可用于声明返回类型并充当传递给泛型方法的参数类型的占位符，这些参数被称为实际类型参数。
- 泛型方法的主体声明与其他任何方法一样。请注意，类型参数只能表示引用类型，而不能表示基本类型（如int，double和char）。

以下示例说明了如何使用单个通用方法打印不同类型的数组

```java

public class GenericMethodTest {
   // generic method printArray
   public static < E > void printArray( E[] inputArray ) {
      // Display array elements
      for(E element : inputArray) {
         System.out.printf("%s ", element);
      }
      System.out.println();
   }

   public static void main(String args[]) {
      // Create arrays of Integer, Double and Character
      Integer[] intArray = { 1, 2, 3, 4, 5 };
      Double[] doubleArray = { 1.1, 2.2, 3.3, 4.4 };
      Character[] charArray = { 'H', 'E', 'L', 'L', 'O' };

      System.out.println("Array integerArray contains:");
      printArray(intArray);   // pass an Integer array

      System.out.println("\nArray doubleArray contains:");
      printArray(doubleArray);   // pass a Double array

      System.out.println("\nArray characterArray contains:");
      printArray(charArray);   // pass a Character array
   }
}

// 输出
Array integerArray contains:
1 2 3 4 5

Array doubleArray contains:
1.1 2.2 3.3 4.4

Array characterArray contains:
H E L L O
```



## 多类型参数

通用类可以有多个类型参数。以下示例将展示上述概念。

使用您选择的任何编辑器创建以下Java程序。

```java

public class GenericsTester {
   public static void main(String[] args) {
      Box<Integer, String> box = new Box<Integer, String>();
      box.add(Integer.valueOf(10),"Hello World");
      System.out.printf("Integer Value :%d\n", box.getFirst());
      System.out.printf("String Value :%s\n", box.getSecond());

      Box<String, String> box1 = new Box<String, String>();
      box1.add("Message","Hello World");
      System.out.printf("String Value :%s\n", box1.getFirst());
      System.out.printf("String Value :%s\n", box1.getSecond());
   }
}

class Box<T, S> {
   private T t;
   private S s;

   public void add(T t, S s) {
      this.t = t;
      this.s = s;
   }

   public T getFirst() {
      return t;
   }

   public S getSecond() {
      return s;
   }
}

// 输出
Integer Value :10
String Value :Hello World
String Value :Message
String Value :Hello World
```



## 参数化类型

泛型类可以具有参数化类型，其中可以用参数化类型替换类型参数。以下示例将展示上述概念。

```java

public class GenericsTester {
   public static void main(String[] args) {
      Box<Integer, List<String>> box
         = new Box<Integer, List<String>>();

      List<String> messages = new ArrayList<String>();

      messages.add("Hi");
      messages.add("Hello");
      messages.add("Bye");      

      box.add(Integer.valueOf(10),messages);
      System.out.printf("Integer Value :%d\n", box.getFirst());
      System.out.printf("String Value :%s\n", box.getSecond());


   }
}

class Box<T, S> {
   private T t;
   private S s;

   public void add(T t, S s) {
      this.t = t;
      this.s = s;
   }

   public T getFirst() {
      return t;
   }

   public S getSecond() {
      return s;
   }
}
// 
Integer Value :10
String Value :[Hi, Hello, Bye]
```



## 原始类型

原始类型是泛型类或接口的一个对象，如果它的类型参数在创建时被传递了npt。以下示例将展示上述概念。

```java

public class GenericsTester {
   public static void main(String[] args) {
      Box<Integer> box = new Box<Integer>();

      box.set(Integer.valueOf(10));
      System.out.printf("Integer Value :%d\n", box.getData());


      Box rawBox = new Box();

      //No warning
      rawBox = box;
      System.out.printf("Integer Value :%d\n", rawBox.getData());

      //Warning for unchecked invocation to set(T)
      rawBox.set(Integer.valueOf(10));
      System.out.printf("Integer Value :%d\n", rawBox.getData());

      //Warning for unchecked conversion
      box = rawBox;
      System.out.printf("Integer Value :%d\n", box.getData());
   }
}

class Box<T> {
   private T t;

   public void set(T t) {
      this.t = t;
   }

   public T getData() {
      return t;
   }
}

// 
Integer Value :10
Integer Value :10
Integer Value :10
Integer Value :10
```



## 有界类型参数

有时您可能需要限制允许传递给类型参数的类型。例如，对数字进行操作的方法可能只想接受Number或其子类的实例。这是有界类型参数的用途。

要声明有界的类型参数，请列出类型参数的名称，其后跟随extends关键字，后跟其上限。





下面的例子说明了如何在一般意义上使用扩展来表示“扩展”（如在类中）或“实现”（如在接口中）。此示例是返回三个Comparable对象中最大的一个的泛型方法

```java

public class MaximumTest {
   // determines the largest of three Comparable objects

   public static <T extends Comparable<T>> T maximum(T x, T y, T z) {
      T max = x;   // assume x is initially the largest

      if(y.compareTo(max) > 0) {
         max = y;   // y is the largest so far
      }

      if(z.compareTo(max) > 0) {
         max = z;   // z is the largest now                 
      }
      return max;   // returns the largest object   
   }

   public static void main(String args[]) {
      System.out.printf("Max of %d, %d and %d is %d\n\n",
         3, 4, 5, maximum( 3, 4, 5 ));

      System.out.printf("Max of %.1f,%.1f and %.1f is %.1f\n\n",
         6.6, 8.8, 7.7, maximum( 6.6, 8.8, 7.7 ));

      System.out.printf("Max of %s, %s and %s is %s\n","pear",
         "apple", "orange", maximum("pear", "apple", "orange"));
   }
}

// 
Max of 3, 4 and 5 is 5

Max of 6.6,8.8 and 7.7 is 8.8

Max of pear, apple and orange is pear
```



## 多边界

一个类型参数可以有多个边界。



```java
public static <T extends Number & Comparable<T>> T maximum(T x, T y, T z)
```

- **最大值** - 最大值是一种通用方法。
- **T** - 传递给泛型方法的泛型类型参数。 它可以采用任何对象。



T是传递给泛型类Box的类型参数，应该是Number类的子类型，并且必须包含Comparable接口。如果一个类作为绑定传递，它应该在接口之前先传递，否则会发生编译时错误。



```java

public class GenericsTester {
   public static void main(String[] args) {
      System.out.printf("Max of %d, %d and %d is %d\n\n",
         3, 4, 5, maximum( 3, 4, 5 ));

      System.out.printf("Max of %.1f,%.1f and %.1f is %.1f\n\n",
         6.6, 8.8, 7.7, maximum( 6.6, 8.8, 7.7 ));
   }

   public static <T extends Number
      & Comparable<T>> T maximum(T x, T y, T z) {
      T max = x;      
      if(y.compareTo(max) > 0) {
         max = y;   
      }

      if(z.compareTo(max) > 0) {
         max = z;                    
      }
      return max;      
   }
}
//
Max of 3, 4 and 5 is 5

Max of 6.6,8.8 and 7.7 is 8.8

```



## 通用列表

Java在List接口中提供了通用支持。

```java
List<T> list = new ArrayList<T>();
```

- **列表** - List接口的对象。
- **T** - 列表声明期间传递的泛型类型参数。



T是传递给通用接口List及其实现类ArrayList的类型参数。

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class GenericsTester {
   public static void main(String[] args) {

      List<Integer> integerList = new ArrayList<Integer>();

      integerList.add(Integer.valueOf(10));
      integerList.add(Integer.valueOf(11));

      List<String> stringList = new ArrayList<String>();

      stringList.add("Hello World");
      stringList.add("Hi World");


      System.out.printf("Integer Value :%d\n", integerList.get(0));
      System.out.printf("String Value :%s\n", stringList.get(0));

      for(Integer data: integerList) {
         System.out.printf("Integer Value :%d\n", data);
      }

      Iterator<String> stringIterator = stringList.iterator();

      while(stringIterator.hasNext()) {
         System.out.printf("String Value :%s\n", stringIterator.next());
      }
   }  
}

// 
Integer Value :10
String Value :Hello World
Integer Value :10
Integer Value :11
String Value :Hello World
String Value :Hi World
```



## 通用集

Java在Set接口中提供了通用支持。

```java
Set<T> set = new HashSet<T>();
```

- **Set** - Set Interface的对象。
- **T** - 集合声明期间传递的泛型类型参数。

T是传递给通用接口Set及其实现类HashSet的类型参数。

```java
public class GenericsTester {
   public static void main(String[] args) {

      Set<Integer> integerSet = new HashSet<Integer>();

      integerSet.add(Integer.valueOf(10));
      integerSet.add(Integer.valueOf(11));

      Set<String> stringSet = new HashSet<String>();

      stringSet.add("Hello World");
      stringSet.add("Hi World");


      for(Integer data: integerSet) {
         System.out.printf("Integer Value :%d\n", data);
      }

      Iterator<String> stringIterator = stringSet.iterator();

      while(stringIterator.hasNext()) {
         System.out.printf("String Value :%s\n", stringIterator.next());
      }
   }  
}

// 
Integer Value :10
Integer Value :11
String Value :Hello World
String Value :Hi World
```



## 通用映射

Java在Map接口中提供了通用支持。

```java
Map<K, V> map = new HashMap<K,V>();
```



```java
public class GenericsTester {
   public static void main(String[] args) {

      Map<Integer,Integer> integerMap
         = new HashMap<Integer,Integer>();

      integerMap.put(1, 10);
      integerMap.put(2, 11);

      Map<String,String> stringMap = new HashMap<String,String>();

      stringMap.put("1", "Hello World");
      stringMap.put("2","Hi World");


      System.out.printf("Integer Value :%d\n", integerMap.get(1));
      System.out.printf("String Value :%s\n", stringMap.get("1"));

      // iterate keys.
      Iterator<Integer> integerIterator   = integerMap.keySet().iterator();

      while(integerIterator.hasNext()) {
         System.out.printf("Integer Value :%d\n", integerIterator.next());
      }

      // iterate values.
      Iterator<String> stringIterator   = stringMap.values().iterator();

      while(stringIterator.hasNext()) {
         System.out.printf("String Value :%s\n", stringIterator.next());
      }
   }  
}

// 
Integer Value :10
String Value :Hello World
Integer Value :1
Integer Value :2
String Value :Hello World
String Value :Hi World
```