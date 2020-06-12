[TOC]

## `Optional`

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