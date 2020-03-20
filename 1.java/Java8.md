## 一、`Java8` 的三个编程概念

* **流处理**
  * 从输入流中一个一个读取数据项，然后以同样的方式将数据项写入输出流。
* **用行为参数化把代码传递给方法**
  * 即函数作为第一公民，可以作为值来传递
* **并行与共享可变数据**

## 二、流简介

​		`Stream API `和 `Collection API`的行为差不多，但`Collection API`主要为了**访问和存储数据**，而`Stream API`主要用于描述对**数据的计算**。

​		经典的`Java`程序只能利用单核进行计算，流提供了多核处理数据的能力。但前提是传递给`Stream API`的方法不会**互动**（即有可变的共享对象）时，才能多核工作。

## 三、Lambda

**Lambda**表达式由 **参数列表** 、**箭头** 和 **主体** 组成：

![微信截图_20200204105243.png](https://upload-images.jianshu.io/upload_images/6974508-2871cac6251a11de?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四、函数式接口

> 函数式接口指只定义一个抽象方法的接口

​		注：哪怕有再多默认方法，只要接口中只定义了**一个抽象方法**，它仍然是函数式接口。

​	Lambda允许你直接以内联的形式为函数式接口的抽象方法提供实现，并把其作为函数式接口的实例。

### `FunctionalInterface`注解

​	`@FunctionalInterface`用于表示该接口为函数式接口。如果它不是函数式接口的话，编译器将返回一个提示原因的错误。

​		注：`@FunctionalInterface`不是必需的，但最好为函数式接口都标注`@FunctionalInterface`.

### 函数描述符

​		函数式接口的**抽象方法的基本签名** 本质上就是 `Lambda`表达式的签名。`Java8`将这种抽象方法叫作**函数描述符**。

​		`Runnable`接口的`run`方法即不接受任何参数也不返回，其函数描述符为：`() -> void`。 该函数描述符代表了函数列表为空且返回void的函数。

​		`Scala`、`Kotlin`等语言在其类型系统中提供 显式的类型注释 来描述函数的类型（即函数类型）

|    函数接口     |  函数描述符  | 基本类型特化                                                 |
| :-------------: | :----------: | ------------------------------------------------------------ |
| `Predicate<T>`  | T -> boolean | `IntPredicate`  `LongPredicate`, `DoublePredicate`           |
|  `Consumer<T>`  |  T -> void   | `IntConsumer`, `LongConsumer`, `DoubleConsumer`              |
| `Function<T,R>` |    T -> R    | `IntFunction`, `IntToDoubleFunction`, `IntToLongFunction`, `LongFunction`, `LongToDoubleFunction`, `LongToIntFunction`, `DoubleFunction`, `ToIntFunction`, `ToDoubleFunction`, `ToLongFunction` |
|  `Supplier<T>`  |   () -> T    | `BooleanSupplier`, `IntSupplier`, `LongSupplier`, `DoubleSupplier` |

## 五、方法引用

> 方法引用可以把现有方法像`Lambda`一样传递。

方法引用主要分三类：

* 指向**静态方法**的方法引用。（例如 `Integer` 的 `parseInt`方法，写作`Integer::parseInt`）
* 指向**任意类型实例方法**的方法引用.（例如`String`的`length`,写作String::length）
  * 适用于对象作为`Lambda`表达式的一个参数。
* 指向现存对象或表达式实例方法的方法引用
  * 适用于调用现存外部对象的方法。
  * 适用于内部的私有方法。

注：构造函数、数组构造函数以及父类调用的方法引用形式比较特殊：

>  利用 **类名** 和 关键字 **new** 来生成构造方法的方法引用。

* 对于默认构造函数，可以使用`Supplier`签名。

  ```java
  Supplier<Apple> c1 = Apple::new;
  //等价于：
  Supplier<Apple> c1 = () -> new Apple();
  ```

* 对于存在参数的构造方法，可根据参数情况寻找适合的函数式接口的签名。

  ```java
  Function<Integer,Apple> c2 = Apple::new;
  //等价于：
  Function<Integer,Apple> c2 = (weight) -> new Apple(weight);
  ```

## 六、流

> 从支持数据处理操作的源生成的元素序列 —— 流
>
> 流允许以声明性方式处理数据集合。还可以透明地并行处理，无须写任何多线程代码。

注：

* 流只遍历一次。遍历完后，流被消费了，需要重新从原始数据源那里再次获取一个新的流进行遍历。
* 只有触发终端操作，中间操作才会被执行。
  * ​	中间操作一般都可以合并起来，在终端操作中一次性全部处理。

### 筛选

* `filter`方法：接受一个**谓词**（一个返回`boolean`的函数）作为参数，并返回一个包括所有符合谓词的元素的流。

  ```java
  //输出结果:[1, 3, 0]
  List<Integer> numbers = Arrays.asList(1,3,8,6,0,7,5,6);
  numbers.stream()
      //筛选只小于4的元素
      .filter(i -> i < 4)
      .collect(Collectors.toList());
  ```

* `distinct`方法：依据流所生成元素的 `hashCode` 和 `equals` 方法，返回一个元素各异的流。（即返回一个没有重复元素的流）

  ```java
  //输出结果为：[2,4]
  List<Integer> numbers = Arrays.asList(1,2,1,3,3,2,4);
  numbers.stream()
      .filter(i -> i % 2 == 0)
      //一共存在3个元素符合filter筛选，而这其中存在重复的2。distinct()只会返回2和4
      .distinct()
      .collect(Collectors.toList());
  ```

### 流的切片

* `takeWhile `方法：在第一个 **不符合** 要求的元素时停止处理。

```java
//输出结果为：[1, 2, 3, 3]
//在初始列表中的数据已排序的情况下：
List<Integer> numbers = Arrays.asList(1,2,3,3,4,4,5,6);
numbers.stream()
    //当发现第一个 i < 4 为 false 的元素时，则停止处理
    .takeWhile(i -> i < 4)
    .collect(Collectors.toList());
```

* `dropWhile`方法：在第一个 **符合** 要求的元素时停止处理，并返回所有剩余的元素。

```java
//输出结果：[4, 4, 5, 6]
//在初始列表中的数据已排序(由高到低)的情况下：
List<Integer> numbers = Arrays.asList(1,2,3,3,4,4,5,6);
numbers.stream()
    //当发现第一个i < 4 为 true 的元素时，则停止处理,并返回所有剩余的元素。
    .dropWhile(i -> i < 4)
    .collect(Collectors.toList());
```

* `limit`方法：返回一个不超过给定长度的流。
  * 如果流是有序的（如：源是`List`），则按顺序返回前 `n` 个元素。
  * 如果流是无序的（如：源是`set`），则不会以任意顺序排序。
  * 对于无限流，可以使用`limit`将其变成有限流。

```java
//输出结果:[1, 3]
List<Integer> numbers = Arrays.asList(1,3,8,6,0,7,5,6);
numbers.stream()
    //筛选只小于4的元素
    .filter(i -> i < 4)
    //只返回前两个值
    .limit(2)
    .collect(Collectors.toList());
```

* `shkip`方法：返回一个扔掉前 `n` 个元素的流。
  * 如果流中元素不足 `n` 个，则返回一个空流。

```java
//输出结果:[3, 0]
List<Integer> numbers = Arrays.asList(1,3,8,6,0,7,5,6);
numbers.stream()
    //筛选只小于4的元素
    .filter(i -> i < 4)
    //跳过第一个值
    .skip(2)
    .collect(Collectors.toList());
```

### 映射

* `map`方法：将流中的每一个元素映射成一个新的元素。

```java
//输出结果:[6, 2, 4, 1]
List<String> languages = Arrays.asList("Kotlin","Go","Java","C");
languages.stream()
    //将 字符串 转为 int 
    .map(String::length)
    .collect(Collectors.toList());
```

* `flatMap`方法：把 一个流 中的 **每一个值** 转换成 **另一个流**，然后把 **所有流** 连接起来成一个流。
  * 简单说就是：把流中的 **元素（如：列表，数组）**化为新的流，或把流中的 **元素** 结合 **外部的列表 (数组) ** 化为新的流，再把新的流的元素整合到一个流中。

```java
//输出结果：[K, o, t, l, i, n, G, J, a, v, C]
List<String> languages = Arrays.asList("Kotlin","Go","Java","C");
languages.stream()
    .map(str -> str.split(""))
    //Arrays::stream 将 str.split("") 返回的字符数组转换为流,再由 flatMap 统一将这些流合并成一个流.最终：Stream<String[]> 转换为 Stream<String>，
    //flatMap 本质也是对流的元素进行转换（map也是对流的元素进行转换）。将流的元素转换为新的流，再将其整合进一个流中。
    .flatMap(Arrays::stream) // 等价于：flatMap(strArray -> Arrays.stream(strArray))
    .distinct()
    .collect(Collectors.toList());
```

#### 练习：

1、返回所有对数

给定列表`[ 1,2,3 ] `和 列表`[ 3, 4 ]`,返回` [ (1,3) , (1,4) , (2,3) , (2,4) , (3,3) , (3,4) ]`

```java
//输出结果：[ (1,3) , (1,4) , (2,3) , (2,4) , (3,3) , (3,4) ]
List<Integer> numbers1 = Arrays.asList(1,2,3);
List<Integer> numbers2 = Arrays.asList(3,4);
List<int[]> pairs = 
    numbers1.stream()
    	//将其扁平化为一个流
        .flatMap(i ->
                numbers2.stream()
                 	//将其转换为一个数组，并返回这个流
                    .map(j -> new int[]{i,j})
        ).collect(Collectors.toList());
```

### 查找与匹配

* `anyMatch`方法：检查流中是否**至少有一个**元素匹配给定的谓词。

```java
//输出结果:true
List<Integer> numbers = Arrays.asList(1,2,3,5,6,8);
numbers.stream().anyMatch(i -> i > 3);
```

* `allMatch`检查谓词是否匹配所有元素。

* `allMatch`方法：检查流中全部元素都**匹配**给定的谓词。

```java
//输出结果:true
List<Integer> numbers = Arrays.asList(1,2,3,5,6,8);
numbers.stream().allMatch(i -> i < 10);
```

* `noneMatch`方法：检查流中全部元素都**不匹配**给定的谓词。( 与`allMatch`相对 )

```java
//输出结果:true
List<Integer> numbers = Arrays.asList(1,2,3,5,6,8);
numbers.stream().noneMatch(i -> i > 10);
```

* `findAny`方法：返回当前流中的任意元素。

```java
List<Apple> inventory = Arrays.asList(
    new Apple(80,"green"),
    new Apple(155, "green"),
    new Apple(120, "red"));
Optional<Apple> apple = inventory.stream()
    .filter(a -> a.getColor().equals("green"))
    .findAny();
```

* `findFirst`方法：返回当前流中的第一个元素。

```java
List<Apple> inventory = Arrays.asList(
    new Apple(80,"green"),
    new Apple(155, "green"),
    new Apple(120, "red"));
Optional<Apple> apple = inventory.stream()
    .filter(a -> a.getColor().equals("green"))
    .findFirst();
```

注：1、`anyMatch`、`allMatch` 和 `noneMatch` 都属于终端操作。

​		2、`anyMatch`、`allMatch` 、 `noneMatch` 、 `findFirst` 和 `findAny` 不用处理整，只要找到一个元素，就可以得到结果了。

​		3、`findAny` 和 `findFirst` 同时存在的原因是 **并行** 。`findAny`在并行流中限制较少。

### 归约

> 将流中所有元素反复结合起来，从而得到一个值的**查询**，可以被归类为**归约操作**。(用函数式编程语言的术语来说，这称为**折叠**)

**reduce**方法：接收的`Lambda`将列表中的所有元素进行处理并归约成一个新值。

**有初始值**：

接收一个**初始值** 和 一个`BinaryOperator<T>`将两个元素结合起来产生一个新值。

```java
T reduce(T identity, BinaryOperator<T> accumulator);
```

**无初始值**：

一个`BinaryOperator<T>`将两个元素结合起来产生一个新值。

```java
Optional<T> reduce(BinaryOperator<T> accumulator);
```

* 求和

```java
//输出值：36
List<Integer> numbers = Arrays.asList(1,3,8,6,0,7,5,6);
//使用带初始值的reduce方法
int sum = numbers.stream()
    .reduce(0,Integer::sum);//等价于 reduce(0,(a,b) -> a + b)
//或使用无初始值的reduce方法
Optional<Integer> sumOptional = numbers.stream().reduce(Integer::sum);
```

* 最大值

```java
Optional<Integer> maxOptional = numbers.stream().reduce(Integer::max);
```

* 最小值

```java
Optional<Integer> minOptional = numbers.stream().reduce(Integer::min);
```

### 数值流

​		原先的归约求和代码中，`Integet::sum`暗含装箱和拆箱的成本。`Stream API`提供了**原始类型流特化**，专门支持处理数值流的方法。`Java8`引入原始类型特化接口解决数值流拆箱与装箱的问题：`IntStream`、`DoubleStream` 和 `LongStream`，分别将流中的元素特化为 `int`、 `long` 和 `double`。

* 映射到数值流

`mapToInt`、`mapToDouble` 和 `mapToLong`用于将流转换为特化流：

```java
//输出值：36
List<Integer> numbers = Arrays.asList(1,3,8,6,0,7,5,6);
int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();
```

* 转换回对象流

当需要把原始流转换成一般流时(如：把 `int` 装箱回 `Integer` )，可以使用 `boxed`。

```java
List<Integer> numbers = Arrays.asList(1,3,8,6,0,7,5,6);
//使用 IntStrean 特化流
IntStream intStream = numbers.stream()
    .mapToInt(Integer::intValue);
Stream<Integer> stream = intStream.boxed();
```

* 默认值`OptionalInt`

Optional也相应的提供原始类型特化版本：`OptionalInt`、`OptionalLong` 和 `OptionalDouble`。

```java
List<Integer> numbers = Arrays.asList(1,3,8,6,0,7,5,6);
//使用 OptionalInt 特化Optional
OptionalInt maxNumber = numbers.stream()
    .mapToInt(Integer::intValue)
    .max();
```

#### 数值范围

`IntStream`和 `LongStream`提供产生生成数值范围的静态方法：`range`和`rangeClosed`。

`range`方法生成半闭区间（左闭右开），`rangeClosed`方法生成闭区间。

```java
IntStream.range(1,100)
    .filter(n -> n % 2 == 0)
    .count();
```

### 构建流

* 由值创建流

静态方法 `Stream.of` 接受任意数量的参数，显式创建一个流。

```java
//显式创建字符串流
Stream<String> strStream =Stream.of("Java","Kotlin","Go");
```

静态方法`Stream.empty`创建一个空流。

```java
Stream<String> strStream =Stream.empty();
```

* 由数组创建流

静态方法`Arrays.stream`将数组创建为一个流。

```java
int[] numbers = {2,3,5,6,7};
int sum = Arrays.stream(numbers).sum();
```

* 由文件生成流

`java.nio.file.Files`中很多静态方法会返回一个流，以便利用`Stream API`处理文件等`I/O`操作。

如：`Files.lines`返回一个由指定文件中的各行构成的字符串流：

```java
long uniqueWords = 0;
//流会自动关闭，不需要额外try-finally操作
try(Stream<String> lines = 
    Files.lines(Paths.get("data.text"), Charset.defaultCharset())){
    //统计有多少不重复的单词。
	uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
        .distinct()
        .count();
}catch (IOException e){}
```

* 由函数生成流：创建无限流

`Stream API`提供了两个静态方法来从函数生成流：`Stream.iterate()` 和 `String.generate()`

不同于从集合创建的流，这两个静态方法创建的流没有固定大小，称为**无限流**。

**迭代：**

`iterate`方法接收一个接受**一个初始值**作为**流的第一个元素**。再接收**一个Lambda**依次应用在每一个产生的新值上。

```java
Stream.iterate(0,n -> n + 2)
    .limit(10)
    .forEach(System.out::println);
```

`Java 9`对`iterate`方法进行增加，接受多一个谓词作为判断迭代调用何时终止。(谓词作为第二参数传入)

```java
IntStream.iterate(0,n -> n < 100,n -> n + 2)
    .forEach(System.out::println);
```

当然，也可以使用`takeWhile`对流执行短路操作（`takeWhile`函数`Java9`开始支持）：

```java
IntStream.iterate(0,n -> n + 2)
    .takeWhile(n -> n < 100)
    .forEach(System.out::println);
```

**生成：**

`generate`接受一个`Supplier<T>`类型的`Lambda`提供新值。

```java
Stream.generate(Math::random)
    .limit(5)
    .forEach(System.out::println);
```

## 七、用流收集数据

流支持两种类型的操作：**中间操作** 和 **末端操作**。

* 中间操作可以相互链接起来，将一个流转换为另一个流。中间操作不会消耗流，目的是建立一个流水线。
* 末端操作会消耗流，以产生一个最终结果。

### 归约和汇总

* `Collectors` 工厂类提供了很多**归约**的静态工厂方法。

  * `Collectors.counting()` 用于统计总和。

  ```java
  //求总和
  long count = menu.stream().collect(Collections.counting());
  ```

  * `Collectors.maxBy` 和 `Collectors.minBy` 用来计算流中的最大值和最小值。

  ```java
  //求最大值
  Optional<Dish> mostCalorieDish = 
      menu.stream().collect(
      	Comparator.maxBy(
              Comparator.comparingInt(Dish::getCalories)
          )
  	);
  ```

* 同时`Collectors` 类专门为**汇总**提供了一些工厂方法。

  * `Collectors.summingInt`、`Collectors.summingLong` 和 `Collectors.summingDouble` 分别用于对 `int`、`long` 和 `double`进行求和。

  ```java
  int sumValue = menu.stream().collect(summingInt(Dish::getCalories));
  ```

  * `Collectors.averagingInt`、`Collectors.averagingLong` 和 `Collectors.averagingDouble` 分别用于对 `int`、`long` 和 `double`进行求平均值。

  ```java
  double avgValue = menu.stream().stream().collect(averagingInt(Dish::getCalories));
  ```

* `Collectors.joining` 工厂方法会对流中每一个对象应用 `toString` 方法得到所有字符串连接成一个字符串。

```java
String nameStr = menu.stream().map(Dish::getName).collect(joining());
```

### 分组

`Collections` 的 `groupingBy()` 方法会把流中的元素分成不同的组。

![微信截图_20200224172850.png](https://upload-images.jianshu.io/upload_images/6974508-70c385d5b236d755.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 操作分组的元素

* 过滤

如果在 `groupingBy()` 之前，使用 `filter()` 对流进行过滤操作，可能会造成键的丢失。

例如：

​	存在以下Map: `{ FISH = [ prawns, salmon], OTHER = [french fries, rice ], MEAT = [pork , beef, chicken] }`

但如果在使用`filter()` 后，再 `groupingBy()` 可能对某些键在结果映射中完全消失：

​	 `{ OTHER = [french fries, rice ], MEAT = [pork , beef, chicken] }`

为此，`Collectors` 类提供了 `filtering()` 静态工厂方法，它**接受一个谓词对每一个分组中的元素执行过滤操作**。最后不符合谓词条件的键将得到空的列表：

`{ FISH = [], OTHER = [french fries, rice ], MEAT = [pork , beef, chicken] }`

```java
Map<Dish.Type,List<Dish>> caloricDishesByType = menu.stream()
    .collect( groupingBy(Dish::getType),
            filtering(dish -> dish.getCalories() > 500,toList()))
```

**注**：

​	使用重载的 `groupingBy()` 方法 和 `filtering()`方法 ：先分组再过滤；

​	先使用 `filter()`，再使用 `groupingBy()` 方法：先过滤再分组。



* 映射

`Collectors` 提供 `mapping` 静态工厂方法，接受一个映射函数和另外一个 `Collectors` 函数作为参数。映射函数将分组中的元素进行转换，作为参数的 `Collectors` 函数会收集对每个元素执行该映射函数的结果。

```java
Map<Dish.Type,List<String>> dishNamesByType = menu.stream().collect(
    groupingBy(
        Dish::getType,
        mapping(
            //将元素转换为其名字
            Dish::getName,
            //用于收集该组进行完映射的元素
            Collectors.toList()
        )
    )
)
```

`Collectors` 工具类也提供了 `flatMapping`,跟 `flatMap` 类似的功能。

#### 多级分组

同时`Collectors` 工具类也提供了可以嵌套分组的`groupingBy()`,用于进行多级分组

注：

​	可以理解为在进行完第一次分组后，再对每一组元素进行再次分组。

​	`groupingBy(f)`( f 是分类函数 ) 实际上是`groupingBy(f，toList())`的简便写法。

```java
Map<Dish.Type,Map<CaloricLevel,List<Dish>>> dishesByTypeCaloricLevel = 
    menu.stream().collect(
    	groupingBy(
            Dish::getType,
            groupingBy(dish -> {
                if(dish.getCalories() <= 400)
                    return CaloricLevel.DIET;
                else if(dish.getCalories() <= 700)
                    return CaloricLevel.NORMAL;
                else 
                    return CaloricLevel.FAT;
            })
        )
	);
```

![微信截图_20200226150753.png](https://upload-images.jianshu.io/upload_images/6974508-4d235c221ff7477f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**按子组收集数据**

`groupingBy()`的第二个收集器可以是任何类型。例如可以使用 `counting()` 收集器作为它的第二个参数，统计分组的数量：

```java
Map<Dish.TYPE,Long> typesCount = menu.stream().collect(
	groupingBy(Dish::getType,counting())
);
```

得到以下的map: 	`{ MEAT = 3 , FISH = 2 , OTHER = 4 }`

```java
Map<Dish.Type,Dish> mostCaloricByType = 
    menu.stream().collect(
		groupingBy(Dish::getTpye,
                   collectingAndThen(
                       	//maxBy返回的是Optional类型对象
                   		maxBy(comparingInt(Dish::getCalories)),
                       //当找到最大值后，会执行get操作。
                       	Optional::get
                   )
		)
	);
```

如果 `menu` 中没有某一类型的`Dish`,该类型不会对应一个 `Optional.empty()` 值，而且根本不会在`Map`的键中。所以转换函数`Optional::get`的操作是安全的。

#### 分区

`Collectors` 工具类提供 `partitionedMenu()` 静态工厂函数来实现分区，分区是分组的**特殊情况**。由谓词作为分类函数，这意味着得到的分组 `Map` 的键类型是 `Boolean` ,最多分为 `true` 和 `false` 两组。

```java
//将得到以下结果：
Map<Boolean,List<Dish>> partitionedMenu = 
    menu.stream().collect(
    	//分区函数
    	partitioningBy(
            //分期的标准
            Dish::isVegetarian
        )
	)
```

同时`partitionedMenu()`也和`groupingBy()`类似，可以进行二级分区。

### 收集器接口

```java
public interface Collector<T, A, R> {
	//创建一个空的累加器
    Supplier<A> supplier();
	//将元素添加到结果容器
    BiConsumer<A, T> accumulator();
	//合并两个结果(定义了对流的各个子部分进行并行处理时，各个子部分归约所得的累加器如何合并)
    BinaryOperator<A> combiner();
	//对结果容器应用最终转换
    Function<A, R> finisher();
	//定义收集器的行为
    Set<Characteristics> characteristics();
}
```

泛型的定义如下：

​	T 表示流中要手机的项目的泛型。

​	A 表示累加器的类型。（累加器是收集过程中用于累积部分结果的对象）

​	R 表示收集操作得到的对象的类型。

#### 以`ToListCollector`为例

```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {
    public ToListCollector() {}
	
    //创建ArrayList对象作为累加器
    public Supplier<List<T>> supplier() {
        return ArrayList::new;
    }
	
    //利用add函数将流中的元素添加到列表中
    public BiConsumer<List<T>, T> accumulator() {
        return List::add;
    }
	
    //两个累加器（即两个ArrayList对象）进行相加
    public BinaryOperator<List<T>> combiner() {
        return (list, list2) -> {
            list.addAll(list2);
            return list;
        };
    }
	
    //累加器进行最终的转换
    public Function<List<T>, List<T>> finisher() {
        //Function.identity()表示给什么返回什么，也就是不进行转换
        //恒等
        return Function.identity();
    }

    //定义收集器的行为
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(Characteristics.IDENTITY_FINISH, Characteristics.CONCURRENT));
    }
}
```

Characteristics的三个枚举：

* UNORDERED —— 归约结果不受流中项目的遍历和累积顺序的影响。
* CONCURRENT—— accumulator 函数可以从多个线程同时调用，且该收集器可以并行归约流。（仅仅只是数据源无序时才会并行处理）
* IDENTITY_FINISH—— 表明完成器方法返回的函数是一个恒等函数，可以跳过。累加器对象会直接用作归约过程的最终结果。这也意味着，将累加器A不加检查的转换为结果R是安全的。

#### 进行自定义收集，而不去实现 `Collector`

对于 `IDENTITY_FINISH` 的收集操作，`Stream`重载的 `collect` 方法接受三个函数——`supplier`、`accumulator` 和 `combiner`。该 `collect` 方法创建的收集器的 `Characteristics` 永远是`Characteristics.IDENTITY_FINISH` 和  `Characteristics.CONCURRENT`

```java
List<Dish> dishes = menu.stream().collect(
    //创建累加容器
	ArrayList::new,
    //将流元素添加到累加容器中
    List::add,
    //合并累加容器
    List::addAll
);
```

## 八、并行数据处理与性能

* 对顺序流调用 `parallel()` 方法并不意味着流本身有任何实际的变化，它仅仅在**内部设置了一个boolean标志**，表示你想让调用`parallel()`之后的所有操作都并行执行。对并行流调用 `sequential` 方法就可以把它变成顺序流。
* 并行流默认的线程数量等于你处理器的核数。



**使用并行流时，考虑以下因素**：

* 留意自动装箱和拆箱。（应尽量将其转为原始类型流）
* 对于较小数据量，无需使用并行流。
* 考虑流背后的数据结构是否容易分解。
* 部分操作本身在并行流上的性能比顺序流差。如：`limit` 和 `findFirst`
* 考虑合并 步骤的代价是大是小。
* 考虑操作流水线的总操作成本。当单个元素通过流水线的成本较高时，使用并行流比较好。

**流的数据源和可分解性：**

| 源                | 可分解性 |
| ----------------- | -------- |
| `ArrayList`       | 差       |
| `LinkedList`      | 差       |
| `IntStream.range` | 极佳     |
| `Stream.iterate`  | 差       |
| `HashSet`         | 好       |
| `TreeSet`         | 好       |

## 九、`Collection API`的增强功能

`Arrays.asList()` 创建一个固定大小的列表，列表的元素可以更新，但不可以增加或删除。

**Java 9** 引入以下工厂方法：

* `List.of` ——创建一个只读列表，不可`set` 、`add`等操作。

* `Set.of` —— 创建一个只读的`Set`集合。

* `Map.of` —— 接受的列表中，以键值交替的方式创建`map`的元素。、

  * 当创建Map的键值对过多时，可以使用`map.ofEntries()` 和 `Map.entry()`创建map.

  ```java
  import static java.util.Map.entry;
  Map<String,Integer> ageOfFriends = Map.ofEntries(
  	entry("Raphael",30),
      entry("Olivia",25),
      entry("Thibaut",26)
  );
  ```

### 重载与变参

在`Java API`中，`List.of`包含多个重载版本：

```java
static <E> List<E> of(E e1);
static <E> List<E> of(E e1, E e2);
```

而不提供变参版本是因为需要额外的分配一个数组，这个数组被封装于列表中。使用变参版本的方法，就要负担分配数组、初始化以及最后进行垃圾回收的开销。（如果元素数量超过10个，实际调用的还是变参方法。）、

### 使用 `List` 、 `Set` 和 `Map`

* `removeIf()` —— 移除集合中匹配指定谓词的元素。（该方法由`Collection`接口提供默认方法，`List` 和 `Set` 都可用）

  * ```java
    //Collection.java
    //Predicate(谓词)的函数描述符是：(T) -> boolean
    default boolean removeIf(Predicate<? super E> filter)
    ```

  * 当使用for-each遍历列表，进行移除操作时，会导致`ConcurrentModificationException`.因为遍历使用的迭代器对象和集合对象的状态同步。我们只能显示调用迭代器对象（`Iterator`对象）的`remove`方法。因此`Java8`提供`removeIf`方法，安全简便的删除符合谓词的元素。

* `replaceAll()` ——  使用一个函数替换`List`或 `Map` 中的元素。（该方法由`List`接口提供默认方法）

  * ```java
    //List.java
    //UnaryOperator的函数描述符是：(T) -> T
    default void replaceAll(UnaryOperator<E> operator)
    ```

  * 该函数只是在列表内部进行同类型的转换，并没有创建新的列表。也就是说初始为`List<String>`，函数执行完还是`List<String>`.

  * ```java
    //Map.java
    // BiFunction的函数描述符是：(K,V) -> V
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function)
    ```

* `sort()` —— 对列表自身进行排序。（该方法由`List`接口提供默认方法）

  * ```java
    //List.java
    //Comparator的函数描述符是：(T,T) -> boolean
    default void sort(Comparator<? super E> c)
    ```

* `forEach()` —— List 和 Set，甚至是Map在`Java8`中都支持`forEach`方法。而遍历提供的便捷，特别是Map的遍历。

  * ```java
    //Iterable.java
    //Consumer(消费者)的函数描述符是：(T) -> void
    default void forEach(Consumer<? super T> action)
    //Map.java
    //BiConsumer(二元消费者)的函数描述符是：(T,U) -> void
    default void forEach(BiConsumer<? super K, ? super V> action)
    ```

* `Entry.comparingByValue()` 和 `Entry.comparingByKey()` —— 对Map的值或键进行排序。

* `Map.compute` —— 使用指定的键计算新的值，并将其存储到Map中，并返回新值。。(指定一个`key`,再提供一个`BiFunction`，依据**key**和**旧值**，计算新值。如果新值为`null`,则不会加入到`Map`中并将旧值移除。)

  * ```java
    //BiFunction的函数描述符是：(K,V) -> V
    default V compute(K key,BiFunction<? super K, ? super V, ? extends V> remappingFunction) 
    ```

* `Map.computeIfAbsent` —— 如果指定的键没有对应的值（没有该键或者该键对应的值是空）,使用该键计算新的值，并添加到`Map`中（如果新值为`null`,则不会加入到`Map`中并将旧值移除。），并返回新值。

  * ```java
    //Function的函数描述符是：(K) -> V
    default V computeIfAbsent(K key,Function<? super K, ? extends V> mappingFunction)
    ```

  * 该方法对于值需要初始化时有用。比如向`Map<K,List<V>>`添加一个元素( 初始化对应的`ArrayList`，并返回该值)：

    ```java
    map.computeIfAbsent("daqi", name -> new ArrayList<>())
        .add("Java8")
    ```

* `Map.computeIfPresent` —— 如果指定的键在Map中存在，依据该键和旧值计算该键的新值，并将其添加到Map中。（如果新值为`null`,则不会加入到`Map`中，并将旧值移除。）

  * ```java
    //BiFunction的函数描述符是：(K,V) -> V
    default V computeIfPresent(K key,BiFunction<? super K, ? super V, ? extends V> remappingFunction) 
    ```

* `Map.remove` —— 重载版本的`remove`可以删除`Map`中某个键对应某个特定值的映射对。（即 `Key` 和 `Value`都匹对上，才从`Map`中移除）

  * ```java
    default boolean remove(Object key, Object value) 
    ```

* `Map.replace` —— 重载版本的`replace`可以仅在原有键对应某个特定的值时才进行替换。（即 `Key` 和 `Value`都匹对上，才从`Map`中替换）

  * ```java
    default V replace(K key, V value)
    ```

* `Map.merge` ——  如果指定的键在Map中存在，依据该键和旧值计算该键的新值，并将其添加到Map中； 如果指定的键在Map中不存在，依据指定的value作为Key的值，并将其添加到Map中。

  * ```java
    //BiFunction的函数描述符：(V,V) -> V
    default V merge(K key, V value,BiFunction<? super V, ? super V, ? extends V> remappingFunction)
    ```

  * 该函数可用于`Map`的合并，或用于将`Collector`转换成`Map`.

    ```java
    Map<String,Integer> language1 = new HashMap<>();
    language1.put("Java",8);
    language1.put("Kotlin",1);
    Map<String,Integer> language2 = new HashMap<>();
    language2.put("Java",11);
    language2.put("Go",1);
    
    //合并Map
    language1.forEach((key,value) -> {
        //Map的value可null,merge函数不允许value为null
        if (value != null) 
            language2.merge(key,value,Integer::sum);
    });
    ```

  * ```java
    static class Score{
        private int score;
        private int studentId;
        private String studentName;
    
        public Score(int studentId, String studentName,int score) {
            this.score = score;
            this.studentId = studentId;
            this.studentName = studentName;
        }
        //get和set方法
    }
    
    //Collector转换为Map（用途：统计）
    List<Score> languageList = new ArrayList<>();
    languageList.add(new Score(1,"Java",80));
    languageList.add(new Score(2,"Kotlin",90));
    languageList.add(new Score(2,"Java",85));
    languageList.add(new Score(1,"Kotlin",70));
    Map<String,Integer> language3 = new HashMap<>();
    //Collectors.toMap(Function<? super T, ? extends K>,Function<? super T, ? extends U>,BinaryOperator<U>)内部也是通过Map.merge()实现的。
    languageList.stream().collect(Collectors.toMap(Score::getStudentId,Score::getScore,Integer::sum));
    ```

## 十、 重构

### 改善代码可读性

* 用`lambda`表达式取代匿名类。

  * 匿名类和`lambda`表达式中的 `this` 和 `super` 的含义不同。在匿名类中，`this` 代表的是类自身；在`lambda`表达式中，`this` 代表的是包含类。

  * 匿名类可屏蔽包含类的变量，而lambda表达式不能（导致编译报错）。

    ```java
    int a = 10;
    //lambda表达式
    Runnable r1 = () -> {
        //idea爆红，提示：该变量已在作用域中被定义。
        int a = 1;
    };
    //匿名类
    Runnable r2 = new Runnable() {
        @Override
        public void run() {
            //编译正常
            int a = 2;
        }
    };
    ```

  * 匿名内部类的类型是在初始化时确定的，lambda的类型取决于它的上下文。当出现两个或以上方法参数的函数描述符与`lambda`的函数描述符匹配时，需要显示的类型转换来解决。

    ```java
    interface daqiRunnable{
    	public void action();
    }
    //无论Runnable，还是daqiRunnable，其函数描述符为() -> void
    public static void doSomething(Runnable r){}
    public static void doSomething(daqiRunnable r){}
    
    public static void main(String[] args) {
        //显示类型转换
        doSomething((daqiRunnable) () -> {});
    }
    ```

* 用**方法引用** 重构`lambda`表达式，提高代码的可读性。

  * 将较复杂的Lambda逻辑封装在方法中，使用方法引用替代该Lambda。

  * 尽量使用静态辅助方法。比如：`comparing` 和 `maxBy`

    ```java
    list.sort((a1,a2) -> a1.getWeight().compareTo(a2.getWeight()));
    //替换成：
    list.sort(Comparator.comparing(Apple::getWeight));
    ```

  * 很多通用的**归约操作**，都可以借助`Collectors`的辅助方法 + 方法引用替代。

    ```java
    list.stream()
        .map(Dish::getCalories)
        .reduce(0,(c1,c2) -> c1 + c2);
    //替换成Collectors的辅助方法
    list.stream()
        .collect(summingInt(Dish::getCalories));
    ```

* 用`Stream API`重构命令式的数据处理

## 十一、`Optional`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Optional<T>`类是一个容器类，代表一个值存在不存在。`Optional<T>`用于避免和 `null` 检查相关的 `bug`。

### 创建`Optional`

* `Optional.empty()` —— 创建一个空的Optional对象

   ```java
    Optional<String> optStr = Optional.empty();
   ```

* `Optional.of()` —— 依据非空值创建一个`Optional`对象。如果试图传入一个`null`值，会马上抛出一个`NullPointerException`。

   ```java
    Optional<String> optStr = Optional.of(str);
   ```

* `Optional.ofNullable()` —— 创建一个允许为`null`值的`Optional`对象。

   ```java
    Optional<String> optStr = Optional.ofNullable(str);
   ```

### `map` 和 `flatMap`

* `map`操作 ——  当`Optional`的值不为空时，将`Optional`的值转换为对应的值，并将其封装成`Optional`对象返回。如果原本的`Optional`对象的值为空，则返回一个**空的** `Optional`对象。

   ```java
    public <U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent()) {
            return empty();
        } else {
            //拿到新值后，再封装成Optional类型
            return Optional.ofNullable(mapper.apply(value));
        }
    }
   ```

* `flatMap` 操作 —— 当`Optional`的值不为空时，将`Optional`的值转换为对应的值，新的值必须为`Optional`类型，并将其直接返回。如果原本的`Optional`对象的值为空，则返回一个**空的**`Optional`对象。

   ```java
    public <U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent()) {
            return empty();
        } else {
            @SuppressWarnings("unchecked")
            //拿到新值后强转Optional类型，不进行封装
            Optional<U> r = (Optional<U>) mapper.apply(value);
            return Objects.requireNonNull(r);
        }
    }
   ```

**注：**

 **`Optional` 的 `map` 和 `flatMap`如何选择？**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`map`会将转换好的值进行一次`Optional`包装；`flatMap`会确保转换好的值为`Optional`对象，然后直接返回。**使用 `map` 还是 `flatMap` 取决于 转换好的值 是否是`Optional`对象**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果 **转换好的值** 不是`Optional`对象，使用`map`,对其进行再次包装，以便执行进一步`Optional`的操作。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果  **转换好的值** 是 `Optional` 对象，使用`flatMap`，直接返回。

### Optional的其他行为

* `isPresent`方法：`Optional` 包含值的时候返回 `true` ,否则返回 `false`。
* `ifPresent`方法：当值存在时，使用该值执行给定的代码块，否则什么都不做。
    * 与Kotlin的安全调用运算符相似，只有值不为空时，具体方法才会被调用。
* `get`方法: 如果值存在则将其返回，否则抛出一个`NoSuchElement Exception` 异常。
* `orElse`方法：如果值存在则将其返回，否则返回一个**默认值**。 
* `filter`方法：如果值存在，并且满足提供的谓词，就返回自身；否则返回一个空的`Optional`对象。
* `orElseGet`方法：如果有值则将其返回，否则返回一个由指定的`Supplier`接口生成的值。(`Supplier`方法只有在`Optional`不为空时才执行调用)
* `orElseThrow`方法：如果有值则将其返回，否则返回一个由指定的`Supplier`接口生成的异常。
* `orElseThrow`重载方法(无参)：如果有值则将其返回，否则直接抛出`NoSuchElementException` 。**(Java 10)**
* `or`方法：如果值存在则将其返回，否则返回由指定的`Supplier`接口生成的另一个 `Optional` 对象。**(Java 9)**
* `ifPresentOrElse`方法：如果值存在，则使用该值作为参数，执行指定的Consumer接口；如果该值不存在，则执行给定的Runnable，处理值为空的情况。**(Java 9)**

### `Optional`与序列化

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于值可能缺失（即可能为null）的属性，可以将其使用`Optional`包裹，明确表示该属性的值可缺失，类似`kotlin`的可空类型，强制需要进行空检查。
```java
public class Person{
    //Car可能为null,使用Optional对其进行封装
    private Optional<Car> car;

    public Optional<Car> getCar() {return car; }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但由于`Optional`的设计初衷仅仅是要支持能返回`Optional`对象的语法，因此它没实现 `Serializable` 接口。如果使用某些要求序列化的库或框架，在域模型中使用 `Optional`，有可能引发程序故障。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果一定要实现序列化的域模型，替代方案是：提供一个能访问声明为`Optional`、变量值可能缺失的接口：

```java
public class Person{
    //虽然知道car可空，但不提倡一开始就定义Optional<Car>类型。
    private Car car;
    public Optional<Car> getCarAsOptional(){
        return Optional.ofNullable(car);
    }
}
```

### Optional 与 流

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **JAVA 9** 引入了 `Optional` 的 `stream()` 方法，该方法可以把一个含值的 `Optional` 对象转换成由该值构成的`Stream` 对象，或者把一个空的 `Optional` 对象转换成等价的空`Stream`。该方法为处理由 `Optional` 构成的 `Stream` 提供极大的便利。
```java
//Optional#stream源码
public Stream<T> stream() {
    if (!isPresent()) {
        return Stream.empty();
    } else {
        return Stream.of(value);
    }
}
```
借助`Optional`,可以安全的对流元素进行转换、刷选的操作:
```java
//依据前面序列化的要求，提供返回Optional封装属性的方法。
class Person{
    private Car car;
    public Optional<Car> getCarAsOptional() { return Optional.ofNullable(car); }
}

class Car{
    private Insurance insurance;
    public Optional<Insurance> getInsuranceAsOptional() { return Optional.ofNullable(insurance); }
}

class Insurance{
    private String name;
    public String getName() { return name; }
}

//接收一个Person列表
public Set<String> getCarInsuranceNames(List<Person> persons){
    return persons.stream()
        //将流转换为Stream<Optional<Car>>
        .map(Person::getCarAsOptional)
        //将流转换为Stream<Optional<Insurance>>
        .map(optCar -> optCar.flatMap(Car::getInsuranceAsOptional))
        //将流转换为Stream<Optional<String>>
        .map(optIns -> optIns.map(Insurance::getName))
        //该流转换成Stream<String>
        //如果使用Java8的，也可以参考Optional#stream进行处理
        .flatMap(Optional::stream)
        //将结果收集成Set
        .collect(Collectors.toSet());
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 在 **JDK 1.8** 的环境下，可模仿`Optional#stream() `写一个将`Optional`对象转换为`Stream`对象的静态方法，然后使用方法引用将替代`Optional::stream`即可。这里可以使用`OptionalUtility::stream` 替代 `Optional::stream`。
```java
//OptionalUtility.java
public static <T> Stream<T> stream(Optional<T> optional) {
    if (!optional.isPresent()) {
        return Stream.empty();
    } else {
        return Stream.of(optional.get());
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于早已定义好，**不提供**一个能访问声明为`Optional`、变量值可能缺失的接口的域模型，可以手动将流的元素转换为`Optional`对象:
```java
class Person{
    private Car car;
    public Car getCar() {return car;}
}

 class Car{
    private Insurance insurance;
    public Insurance getInsurance() { return insurance;}
 }

class Insurance{
    private String name;
    public String getName() { return name; }
}
persons.stream()
    //对元素使用Optional包装
    .map(Optional::ofNullable)
    .map(optPer -> optPer.map(Person::getCar))
    .map(optCar -> optCar.map(Car::getInsurance))
    .map(optIns -> optIns.map(Insurance::getName))
    //如果使用Java8的，也可以参考Optional#stream进行处理
    //拆除Optional包装
    .flatMap(Optional::stream)
    .collect(Collectors.toSet());
```

### 异常与Optional

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于某些原因，函数无法返回某个值，除了返回`null`外，`Java API` 比较常见的替代做法是抛出一个异常(最典型的例子就是`Interger.parseInt()` )。我们可以使用空的 `Optional` 对象，对遭遇无法转换的String进行建模时返回的非法值进行建模，从而不需要再封装try/catch ：

```java
//OptionalUtility.java
public static Optional<Integer> stringToInt(String s){
    try{
        return Optional.ofNullable(Integer.parseInt(s));
    }catch (Exception e){
        return Optional.empty();
    }
}

//
HashMap<String,String> map = new HashMap<>();
map.put("Java","8");
int version = Optional.ofNullable(map.get("Java"))
    //使用OptionalUtility#stringToInt进行转换。
    .flatMap(OptionalUtility::stringToInt)
    .orElse(0);
```

### 以不解包的方式组合两个Optional对象

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当需要操作两个`Optional`对象进行运算并返回包含结果的`Optional`对象时，或许你会想到以下实现方法：

```java
public Insurance findInsurance(Person person,Car car){
    //经过运算得到正确的Insurance
    //此处模拟返回一个Insurance对象
    return insurance;
}

public Optional<Insurance> nullSafeFindInsurance(Optional<Person> person,Optional<Car> car){
    //判断两个Optional的值都存在
    if (person.isPresent() && car.isPresent()){
        //只有两个Optional的值都存在，才取出进行运算。
        return Optional.ofNullable(findInsurance(person.get(),car.get()));
    }else {
        //否则返回一个空Optional对象。
        return Optional.empty();
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但这样的代码跟我们**手动判空**区别不大，但其实可以结合 `flatMap` 和 `map`，不解包的方式下实现：

```java
public Optional<Insurance> nullSafeFindInsurance(Optional<Person> person,Optional<Car> car){
    //如果person的值不存在，则直接返回一个空Optional对象。
    //至于为什么用flatMap，因为car.map返回的是一个Optional对象
    return person.flatMap(p ->
		//如果map的值不存在，则直接返回一个空Optional对象。
		car.map(c ->
			//执行到这一步，说明两个Optional的值都存在，利用这两个值进行运算。
			//运算出的Insurance值，map函数会对其使用Optional进行封装。
			findInsurance(p,c))
	);
}
```

### 简化if-else

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当需要对某一个变量深度获取值时，往往会伴随多次判空，使用 `Optional` 能优化 `if-else` 结构：

```java
public String getInsuranceByPerson(Person person){
    return Optional.ofNullable(person)
            .map(Person::getCar)
            .map(Car::getInsurance)
            .map(Insurance::getName)
            .orElse("daqi");
}

```

### 总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `Optional `类有时候并没有让代码变得更简洁，他的作用更多的是把 类型 转换为 对应的"可空类型" ，强制进行空检查（与Kotlin定义可空类型相似）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 总体来说，`Optional`可确保 流 进行`map`、`filter`操作时的空安全，以及对某个变量进行深度取值时简化`if-else`流程和确保空安全。当然也可以把普通的空检查转换为 `Optional` 后，再进行操作。



## 十二、日期和时间API

### LocalDate 和 LocalTime

​		`LocalDate`类的实例是一个不可变对象，只提供了简单的日期，并不含当天的时间信息，同时不附带任何和时区相关的信息。	`LocalTime`

* 使用静态工厂方法 `now` 从系统时钟中获取当前的日期：

```java
LocalDate localDate = LocalDate.now();
//获取年
int year = localDate.getYear();
//获取月
int month = localDate.getMonthValue();
//获取日
int day = localDate.getDayOfMonth();
```

* 使用静态工厂方法 `of` 创建一个 `LocalDate` 实例:

```java
LocalDate date = LocalDate.of(2020,3,20);
```

* 使用静态工厂方法 `now` 从系统时钟中获取当前的时间:

```java
LocalTime localTime = LocalTime.now();
//获取小时
int hour = localTime.getHour();
//获取分钟
int minute = localTime.getMinute();
//获取秒
int second = localTime.getSecond();
```

* 使用静态工厂方法 `of` 创建一个 `LocalTime` 实例:

```java
//创建具有时分的LocalTime对象
LocalTime localTime = LocalTime.of(10,30);
//创建具有时分秒的LocalTime对象
LocalTime localTime = LocalTime.of(10,30,30);
```

### 合并日期和时间

​		复合类`LocalDateTime`，是` LocalDate` 和 `LocalTime `的合体,同时表示日期和时间，但不带有时区信息。可以像 `LocalDate` 和 `LocalTime`一样，利用静态工厂方法` now `和 静态工厂方法 `of `创建。也可以通过合并`LocalDate` 对象 和 `LocalTime`对象创建。

```java
//now()静态工厂方法创建当前时间的LocalDateTime对象
LocalDateTime localDateTime = LocalDateTime.now();
//of()静态工厂方法创建指定时间的LocalDateTime对象(2020-3-20 15:30:30)
LocalDateTime localDateTime = LocalDateTime.of(2020,3,20,15,30,30);
//合并LocalDate对象和LocalTime对象创建LocalDateTime对象
LocalDateTime localDateTime = LocalDateTime.of(localDate,localTime);
```

通过 `LocalDate#atTime`方法，向`LocalDate`对象传递`LocalTime`对象，创建一个`LoaclTimeDate`对象：

```java
//对LocalDate对象 传递 LocalTime对象
LocalDateTime localDateTime = localDate.atTime(localTime);
```

通过`LocalTime#atDate`方法，向`LocalTime`对象 传递 `LocalDate`对象，创建一个`LoaclTimeDate`对象：

```java
//对LocalTime对象 传递 LocalDate对象
LocalDateTime localDateTime = localTime.atDate(localDate);
```

`LoaclTimeDate`对象可以借助`toLocalDate()`方法 和 `toLocalTime()`方法分别提取 `LocalDate` 对象 和 `LocalTime`对象：

```java
LocalDate localDate = localDateTime.toLocalDate();
LocalTime localTime = localDateTime.toLocalTime();
```

### 机器的日期和时间格式

​		从计算机的角度来看，建模时间最自然的格式是表示一个持续的时间段上某个点的单一大整型数。Instant类对时间建模的方式，基本上以Unix元年的时间开始所经历的秒数进行计算。

`Instant`可支持静态工厂方法`now `,获取当前时刻的时间戳。

```java
Instant instant = Instant.now();
```

### 定义 Duration 或 Period

​		`LocalDate`类 、`LocalTime`类、`Instant`类 和 ` LocalDateTime`类都是实现了 `Temporal`接口。可以创建 两个`LocalTime`对象、 两个 ` LocalDateTime`对象，或者两个 `Instant`对象的之间的`Duration `。(由于`LocalDateTime` 和 Instant 是为不同目的设计的，两者不能混用。如果试图在这两类对象之间创建`Duration`,会触发一个`DateTimeException`异常。)

```java
Duration duration = Duration.between(localDateTime1,localDateTime2);
Duration duration = Duration.between(localTime1,localTime2);
Duration duration = Duration.between(instant1,instant2);
```

​		由于 `Duration` 类的静态主要用于**以秒和纳秒衡量时间的长短**，所以不能仅向 `between`方法中传递一个`LocalDate`对象做参数。当需要计量两个`LocalDate`之间的时长 时，可以使用`Period#between()`:

```java
Period period = Period.between(LocalDate.of(2020,3,20),
               LocalDate.of(2020,3,21));
```

​		除了以两个Temporal对象的差值方式定义`Duration` 对象 或 `Period`对象时长外，还可以使用`Duration` 类 或 `Period`类都提供工厂方法进行创建创建。

```java
Duration threeMinutes = Duration.ofMinutes(3);
Period tenDays = Period.ofDays(10);
```

### 操纵日期

​		想修改一个`LocalDate`对象最直接的方法是使用`withAttribute`方法。`withAttribute`方法会创建一个对象的副本，并按照需要修改对应的属性。（返回一个修改了属性的`LoacalDate`对象，不修改原来的对象）

```java
LocalDate date1 = LocalDate.of(2020,1,21);
LocalDate date2 = date1.withYear(2021);
LocalDate date3 = date1.withDayOfMonth(25);
LocalDate date4 = date1.with(ChronoField.MONTH_OF_YEAR,2);
```

​		`Temporal` 对象使用 `get` 和 `with` 方法，对读取和修改进行区分开。如果 Temporal 对象不支持请求访问的字段，就会抛出一个 `UnsupportedTemporalTypeException` 异常。

* Temporal#minus 创建一个副本，通过将当前Temporal对象的值减去一定的时长创建该副本

  ```java
  //2020-01-21
  LocalDate date1 = LocalDate.of(2020,1,21);
  //2014-01-21
  LocalDate date4 = date1.minus(6, ChronoUnit.YEARS);
  ```

* Temporal#plus 创建一个副本，通过将当前Temporal对象的值增加上一定的时长创建该副本

  ```java
  //2020-01-21
  LocalDate date1 = LocalDate.of(2020,1,21);
  //2026-01-21
  LocalDate date4 = date1.plus(6, ChronoUnit.YEARS);
  ```

以声明的方式操作`LocalDate`对象：

```java
// 2020-01-21
LocalDate date1 = LocalDate.of(2020,1,21);
// 2017-01-31
//可链式调用
LocalDate date2 = date1.plusDays(10).minusYears(3);
```

### 解析和格式化日期

* 创建新的格式器(`DateTimeFormatter`)最简单的方法是通过它的静态工厂方法以及常量。（比如定义好的`DateTimeFormatter.ISO_LOCAL_DATE`）

  ```java
  LocalDate date1 = LocalDate.of(2020,1,21);
  String s1 = date1.format(DateTimeFormatter.ISO_LOCAL_DATE);
  ```

* 可以使用工厂方法`parse()`打到重创该日期对象的目的：

  ```java
  LocalDate date1 = LocalDate.parse("2020-03-20",DateTimeFormatter.ISO_LOCAL_DATE);
  ```

* 可以使用工厂方法`DateTimeFormatter.ofPattern()` 自定义模式格式化模式:

  ```java
  LocalDateTime dateTime = LocalDateTime.now();
  DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
  String s1 = dateTime.format(formatter);
  ```

