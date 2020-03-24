## 日期和时间API

### LocalDate 和 LocalTime

		`LocalDate`类的实例是一个不可变对象，只提供了简单的日期，并不含当天的时间信息，同时不附带任何和时区相关的信息。	`LocalTime`

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

		复合类`LocalDateTime`，是` LocalDate` 和 `LocalTime `的合体,同时表示日期和时间，但不带有时区信息。可以像 `LocalDate` 和 `LocalTime`一样，利用静态工厂方法` now `和 静态工厂方法 `of `创建。也可以通过合并`LocalDate` 对象 和 `LocalTime`对象创建。

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

		从计算机的角度来看，建模时间最自然的格式是表示一个持续的时间段上某个点的单一大整型数。Instant类对时间建模的方式，基本上以Unix元年的时间开始所经历的秒数进行计算。

`Instant`可支持静态工厂方法`now `,获取当前时刻的时间戳。

```java
Instant instant = Instant.now();
```

### 定义 Duration 或 Period

		`LocalDate`类 、`LocalTime`类、`Instant`类 和 ` LocalDateTime`类都是实现了 `Temporal`接口。可以创建 两个`LocalTime`对象、 两个 ` LocalDateTime`对象，或者两个 `Instant`对象的之间的`Duration `。(由于`LocalDateTime` 和 Instant 是为不同目的设计的，两者不能混用。如果试图在这两类对象之间创建`Duration`,会触发一个`DateTimeException`异常。)

```java
Duration duration = Duration.between(localDateTime1,localDateTime2);
Duration duration = Duration.between(localTime1,localTime2);
Duration duration = Duration.between(instant1,instant2);
```

		由于 `Duration` 类的静态主要用于**以秒和纳秒衡量时间的长短**，所以不能仅向 `between`方法中传递一个`LocalDate`对象做参数。当需要计量两个`LocalDate`之间的时长 时，可以使用`Period#between()`:

```java
Period period = Period.between(LocalDate.of(2020,3,20),
               LocalDate.of(2020,3,21));
```

		除了以两个Temporal对象的差值方式定义`Duration` 对象 或 `Period`对象时长外，还可以使用`Duration` 类 或 `Period`类都提供工厂方法进行创建创建。

```java
Duration threeMinutes = Duration.ofMinutes(3);
Period tenDays = Period.ofDays(10);
```

### 操纵日期

		想修改一个`LocalDate`对象最直接的方法是使用`withAttribute`方法。`withAttribute`方法会创建一个对象的副本，并按照需要修改对应的属性。（返回一个修改了属性的`LoacalDate`对象，不修改原来的对象）

```java
LocalDate date1 = LocalDate.of(2020,1,21);
LocalDate date2 = date1.withYear(2021);
LocalDate date3 = date1.withDayOfMonth(25);
LocalDate date4 = date1.with(ChronoField.MONTH_OF_YEAR,2);
```

		`Temporal` 对象使用 `get` 和 `with` 方法，对读取和修改进行区分开。如果 Temporal 对象不支持请求访问的字段，就会抛出一个 `UnsupportedTemporalTypeException` 异常。

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