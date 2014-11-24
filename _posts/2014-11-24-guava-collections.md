---
layout: post
title: Guava提高的集合
categories: guava
tags: 笔记 java
---

#### 使用 FluentIterable 类

FluentIterable 类提供一组强大的接口来操作 Iterable 实例，通过一种流畅的编程方式。这种流畅的编程方式允许我们使用链式调用，使我们的代码更加可读。

##### 使用 FluentIterable.filter 方法

FluentIterable.filter 方法接受一个 Predicate 的参数。然后每一个元素都会传入 Predicate.apply 方法进行检查如果返回 true 就保留这个元素。如果没有返回 true 的元素就返回一个空的 Iterable。下面例子中我们将演示 from 和 filter 方法：

```java
@Before
public void setUp() {
    person1 = new Person("Wilma", "Flintstone", 30, "F");
    person2 = new Person("Fred", "Flintstone", 32, "M");
    person3 = new Person("Betty", "Rubble", 31, "F");
    person4 = new Person("Barney", "Rubble", 33, "M");
	personList = Lists.newArrayList(person1, person2, person3, person4);
}

@Test
public void testFilter() throws Exception {
    Iterable<Person> personsFilteredByAge = FluentIterable.from(personList).
    filter(new Predicate<Person>() {
        @Override
        public boolean apply(Person input) {
        return input.getAge() > 31;
        }
    });
    assertThat(Iterables.contains(filtered, person2), is(true));
    assertThat(Iterables.contains(filtered, person4), is(true));
    assertThat(Iterables.contains(filtered, person1), is(false));
    assertThat(Iterables.contains(filtered, person3), is(false));
}
```

在 FluentIterable.from() 方法中传入 list，然后链式调用 filter 方法传入 Predicate 参数。在断言中，我们使用 Iterables.contains 方法来检查结果。Iterables 是一个操作 Iterable 实例很使用的类。

##### 使用 FluentIterable.transform 方法

FluentIterable.transform 方法对于每一个元素有一个 Fucntion 实例的映射操作。 他创建一个与原来提供的大小相同的可迭代对象，元素由转换后的对象构成。与 filter 方法的不同在于，filter 方法可能移除任何或所有元素。这里展示 transform，依然使用上面例子当中 setUp 方法：

```java
@Test
public void testTransform() throws Exception {
    List<String> transformedPersonList = FluentIterable.from(personList).
    transform(new Function<Person, String>() {
        @Override
        public String apply(Person input) {
       		return Joiner.on('#').join(input.getLastName(),
        		input.getFirstName(), input.getAge());
        }
    }).toList();
    assertThat(transformed.get(1), is("Flintstone#Fred#32"));
}
```

我们转换 personList 当中的每一个元素成为以 # 分割连接 last name、first name、和 age 的字符串。

我们使用 FluentIterable.from，这次链式调用 transform 方法并且传入 Function 实例参数，我们依然可以链式调用 toList 方法返回`List<String>` 实例。

我们也可能调用 toSet, toMap, toSortedList, 和 toSortedSet 方法。

toMap 方法考虑了 FluentIterable 实例的元素是键和 Function 需要将值映射到这些键。 toSortedList和 toSortedSet 通过一个 Comparator 参数来指定排序方式。

#### Lists

Lists 是一个操作 List 实例的实用类。提供了最大的方便来创建一个 List 实例。

```java
List<Person> personList = Lists.newArrayList();
```

##### 使用 Lists.partition 方法

该 Lists.partition() 方法是返回大小为 n 的子列表从一个给定列表中的有趣方法。例如，假设你有4个 Person 对象并且使用下面的静态工厂方法创建 List：

```java
List<Person> personList = Lists.newArrayList(person1, person2, person3, person4);
```

我们使用Lists.partition()方法指定成2部分：

```java
List<List<Person>> subList = Lists.partition(personList,2);
```

subList 将包含[[person1,person2],[person3,person4]]。partition 方法返回与传入参数大小相同的连续子表，除了最后一个子表外，它可能更小些。

例如，如果传入参数是3，我们将得到下面的子表[[person1,person2,person3],[person4]].

#### Sets

Sets 是一个操作 Set 实例的实用类。有静态方法来创建 HashSets 、 LinkedHashSets 和 TreeSets。我们可以使用Sets类中的方法创建排序的Set，或者操作Set实例中有哪些相同的元素和不同的元素。同时有一个过滤器的方法，该功能已被覆盖，并在此将不再重复。

##### 使用Sets.difference方法

Sets.difference 方法接受2个 Set 实例作为参数，并且返回一个 SetView 实例，里面元素是在第一个 Set 当中但是不在第二个Set当中的元素。SetView 是 Sets 的一个静态抽象内部类并且代表一个不可变的 Set 实例。任何在第二个 Set 中但不在第一个 Set 中的元素不被包含。例如，下面的 SetView 中只会有一个元素“1”：

```java
Set<String> s1 = Sets.newHashSet("1","2","3");
Set<String> s2 = Sets.newHashSet("2","3","4");
Sets.difference(s1,s2);
```

如果调换这个参数传入的顺序，SetView 里面的元素就是“4”。

##### 使用 Sets.symmetricDifference 方法

Sets.symmetricDifference 方法返回不在两个 set 中同时存在的元素，返回的结果也是不可变。

```java
Set<String> s1 = Sets.newHashSet("1","2","3");
Set<String> s2 = Sets.newHashSet("2","3","4");
Sets.SetView setView = Sets.symmetricDifference(s1, s2);
//返回结果是 1, 4
```

##### 使用 Sets.intersection 方法

Sets.intersection 也同样返回不可变的 SetView 实例，里面的元素是在两个 Set 当中都存在的元素。看下面的例子：

```java
@Test
public void testIntersection(){
    Set<String> s1 = Sets.newHashSet("1","2","3");
    Set<String> s2 = Sets.newHashSet("3","2","4");
    Sets.SetView<String> sv = Sets.intersection(s1,s2);
    assertThat(sv.size()==2 && sv.contains("2") && sv.contains("3"),is(true));
}
```

##### 使用 Sets.union 方法

Sets.union 方法接受两个 Set 实例并且返回一个 SetView 实例包含这2个 Set 当中的所有元素。让我们看下面的例子：

```java
@Test
public void testUnion(){
    Set<String> s1 = Sets.newHashSet("1","2","3");
    Set<String> s2 = Sets.newHashSet("3","2","4");
    Sets.SetView<String> sv = Sets.union(s1,s2);
    assertThat(sv.size()==4 && sv.contains("2") && sv.contains("3") && sv.contains("4")
    	&& sv.contains("1"),is(true));
}
```

#### Maps

Maps 使创建 map 更加简单。下面例子的用集合来创建 map，通常用于一些缓存或者快速查找。下面例子，假设我们有一个 Book 的 List，我们将要把他们存入 map 当中以 Book 的 ISBN 为 key。首先一个可能将 List 转换成 Map 的方法如下：

```java
List<Book> books = someService.getBooks();
Map<String,Book> bookMap = new HashMap<String,Book>()
for(Book book : books){
	bookMap.put(book.getIsbn(),book);
}
```

前面的代码很简单，但是我们可以做的更好。

##### 使用Maps.uniqueIndex方法

Maps.uniqueIndex 方法接受一个 iterable 或 iterator 类型的参数，和一个 Function 类型的参数。iterable 或 iterator 的元素将存入 map，Function 用于传入一个元素返回这个元素的 key 值。所以我们从写上面的例子，会如下：

```java
List<Book> books = someService.getBooks();
Map<String,Book> bookMap = Maps.uniqueIndex(books.iterator(),new
    Function<Book, String>(){
    @Override
    public String apply( Book input) {
        return input.getIsbn();
    }
});
```

在这个例子中，我们从 List 当中获取了一个 iterator 对象，并且定义了一个 Function 实例返回每本 Book 的 ISBN 号，他们将被当中 map 中的 key。这个例子我们使用了匿名内部类，如果传入的 Function 使用依赖注入的方式，我们可以简单的改变使用 Book 的哪个属性作为 key，并且不影响其他代码。

##### 使用 Maps.asMap 方法

与 Maps.uniqueIndex 做相反的操作。Maps.asMap 方法接受 Set 对象作为 keys，并且 Function 生成每个 key 对应的 value。

有另外一个方法 Maps.toMap，接受相同的参数返回一个不可变的 ImmutableMap。这样做的意义在于，对 Maps.asMap 方法返回的 Map 所做的任何更改会修改原始 Map，从 Maps.toMap 方法返回的 Map 将不改变原始 Map。

在 Maps 类当中有些很好的方法来转换 map 的值。Maps.transformEntries 方法使用 Maps.EntryTransformer 接口来生成新的 value，基于原来的 map 当中的 key 和 value。还有一个方法 Maps.transformValues，使用 Function 接受一个 value 转换成一个新的 value。

#### Multimaps

Map 是在编程中经常被使用的大数据结构，有些时候，程序员需要一个以上的值与给定键相关联。 当我们自己实现的时候，通常 value 会是个 list 或者 set，Guava 会更简单些。

静态方法返回 map 实例，并且有 put(key,value) 这样熟悉的操作。细节是，如果给定的 key 是存在的然后创建一个 collection 并且将值添加进去。

#### ArrayListMultimap

ArrayListMulitmap 是一个使用 ArrayList 来存储多值情况的 map。创建一个 ArrayListMulitmap 实例使用下面的方法。

```java
ArrayListMultimap<String,String> multiMap = ArrayListMultimap.create();
ArrayListMutilmap<String,String> multiMap = ArrayListMultimap.create(numExcpectedKeys, numExpectedValuesPerKey);
ArrayListMulitmap<String,String> mulitMap = ArrayListMultimap.create(listMultiMap);
```

第一种方式知识简单的创建了一个 ArrayListMultimap 实例，通过默认的大小的key和ArrayList值。

第二种方式传入期望key的大小和期望ArrayList的大小。

最后一种方式简单的创建一个ArrayListMultimap使用Multimap参数。

通过例子来看看ArrayListMultimap如何使用：

```java
@Test
public void testArrayListMultiMap(){
    ArrayListMultimap<String,String> multiMap = ArrayListMultimap.create();
    multiMap.put("Foo","1");
    multiMap.put("Foo","2");
    multiMap.put("Foo","3");
    List<String> expected = Lists.newArrayList("1","2","3");
    assertEquals(multiMap.get("Foo"), expected);
}
```

我们创建一个多映射 map 并且添加3个值为同一个 key。既然我们只创建了一个多映射 map 并且使用熟悉的 put 方法进行添加 key 和 value。最后我们调用 get 方法传入 Foo 并且返回的 list 是否是我们期望的。

现在让我们考虑另外一个用法。你认为会发生什么事情我们添加超过一次相同的 key 和 value 对。考虑下面的单元测试是否可以通过？

```java
@Test
public void testArrayListMultimapSameKeyValue(){
    ArrayListMultimap<String,String> multiMap = ArrayListMultimap.create();
    multiMap.put("Bar","1");
    multiMap.put("Bar","2");
    multiMap.put("Bar","3");
    multiMap.put("Bar","3");
    multiMap.put("Bar","3");
    List<String> expected = Lists. newArrayList("1","2","3","3","3");
    assertEquals(multiMap.get("Bar"),expected);
}
```

List 并没有强迫元素是唯一的，这单元测试是通过的。

我们简单的添加另一个元素到 List 关联到给定的 key 上。现在来个小测验，考虑下面 multimap:

```java
multiMap.put("Foo","1");
multiMap.put("Foo","2");
multiMap.put("Foo","3");
multiMap.put("Bar","1");
multiMap.put("Bar","2");
multiMap.put("Bar","3");
```

multiMap.size() 的结果是6，不是2。调用 size() 方法是取走每个 list 当中的值，不是 map 当中 list 的实例数。

此外，调用 values() 方法返回一个包含这6个值的集合，并不是返回一个集合包含2个 list，每个 list 包含3个元素。

这可以会使我们迷惑，我们需要记住 multimap 不是一个真正的 map。如果我们需要一个经典 map，我们需要这样做：

```java
Map<String,Collection<String>> map = multiMap.asMap();
```

调用 asMap() 方法返回一个 map，每一个 key 对应的点都是在 multimap 当中的集合。

返回的 map 是一个真实的视图，如果改变 map 会反应到 multimap 上。另外需要注意的是返回的 map 不支持像前面的 put()。

#### HashMultimap

HashMultimap 是基于 hash 表的。不像 ArrayListMultimap,插入相同的 key-value 对是不可以的。让我们看下例子：

```java
HashMultimap<String,String> multiMap = HashMultimap.create();
multiMap.put("Bar","1");
multiMap.put("Bar","2");
multiMap.put("Bar","3");
multiMap.put("Bar","3");
multiMap.put("Bar","3");
```

在这个例子中，我们插入了相同的值对应 Bar 的 key 3次，然而，我们调用 multiMap.size() 返回的结果是3，很明显只有一对被保留。

一些 multimap 中的其他实现的。

首先，有三个不变的实现：ImmutableListMultimap，ImmutableMultimap 和 ImmutableSetMultimap。

有 LinkedHashMultimap，它返回的集合与它们插入的顺序相同。

最后，我们有 TreeMultimap，可以使键和值以自然顺序或比较器指定的顺序排序s。

#### BiMap

BiMap 是可以从一个值映射出一个键的 Map。BiMap 由于保持键和值是唯一的，所以可以从值映射到键。BiMap 操作添加的时候是不同的。看下面的例子：

```java
BiMap<String,String> biMap = HashBiMap.create();
biMap.put("1","Tom");
//调用下面这句会抛出IllegalArgumentException异常
biMap.put("2","Tom");
```

上面的例子我们添加值相同的两个不同键，在传统的 map 当中是可以的，但是在 bimap 当中会抛出异常。

##### 使用 BiMap.forcePut 方法

为了添加不同键映射到相同的值上，我们需要调用 forcePut 方法。forcePut 方法的调用会安静的移除原有相同值的 key-value 的映射关系。如果，插入的是相同的 key 和 value，那么这个map不会有变化。然而，如果值是相同但是键不相同，以前的键会被丢弃。看下面的例子：

```java
@Test
public void testBiMapForcePut() throws Exception {
    BiMap<String,String> biMap = HashBiMap.create();
    biMap.put("1","Tom");
    biMap.forcePut("2","Tom");
    assertThat(biMap.containsKey("1"),is(false));
    assertThat(biMap.containsKey("2"),is(true));
}
```

执行后的结果是 map 当中有 2-Tom 的映射关系，如果在后面添加相同值的 key，以前插入的 key 就会被覆盖掉。所以使用 forcePut 方法很清楚的说明我们将会替换原有的 key。

##### 使用 BiMap.inverse 方法

看下面的例子：

```java
@Test
public void testBiMapInverse() throws Exception {
    BiMap<String,String> biMap = HashBiMap.create();
    biMap.put("1","Tom");
    biMap.put("2","Harry");
    assertThat(biMap.get("1"),is("Tom"));
    assertThat(biMap.get("2"),is("Harry"));
    BiMap<String,String> inverseMap = biMap.inverse();
    assertThat(inverseMap.get("Tom"),is("1"));
	assertThat(inverseMap.get("Harry"),is("2"));
}
```

inverse 方法会反转原来map当中的映射关系，转换为 value-key 的一个 map。虽然我们只说明了 HashBiMap 的这个方法，我们也可以使用 EnumBiMap, EnumHashBiMap 和 ImmutableBiMap 的 inverse 方法。

#### Table

Map 是非常强大的集合在普通的编程当中。但是有些时候一个单一的 map 是不够的，我们还需要 map 映射到 map。虽然非常有用，但是在 java 当中创建和使用是非常笨重的。幸运的，guava 提供了 table 的集合。一个 table 是一个集合包含两组 key，一个是行一个是列的，这些键映射到一个值上。我们不是很明确的像 map 映射到 map 的调用，然而 table 可以给我们提高很使用的功能。

有几种创建 table 的方式，用 HashBasedTable 举例，创建一个类似于 `Map<R, Map<C, V>>` 数据结构的 table。

```java
HashBasedTable<Integer,Integer,String> table = HashBasedTable.create();
//创建5行5列的table
HashBasedTable<Integer,Integer,String> table = HashBasedTable.create(5,5);
//从已存在的table来创建个table
HashBasedTable<Integer,Integer,String> table = HashBasedTable.create(anotherTable);
```

##### Table 的操作

下面是一个操作 table 的例子：

```java
HashBasedTable<Integer,Integer,String> table = HashBasedTable.create();
table.put(1,1,"Rook");
table.put(1,2,"Knight");
table.put(1,3,"Bishop");
boolean contains11 = table.contains(1,1);
boolean containColumn2 = table.containsColumn(2);
boolean containsRow1 = table.containsRow(1);
boolan containsRook = table.containsValue("Rook");
table.remove(1,3);
table.get(3,4);
```

上面的例子正是我们希望看到的 map，但考虑到简洁的方式我们可以直接去访问值而不是使用传统的 map 结构来访问。

##### Table 视图

table 提供了一些方法来获取不同的视图：

```java
Map<Integer,String> columnMap = table.column(1);
Map<Integer,String> rowMap = table.row(2);
```

column 方法返回一个 map 里面的映射关系是给定列的键值映射关系。row 方法返回相反的，它返回给定行的所有 column-value 的映射关系的 map。返回的 columnMap 和 rowMap 的修改会反应到原始的 table 上。还有其他的实现表我们应该讨论简要如下:

- ArrayTable 是实现是一个二纬数组
- ImmutableTable 创建后就不能更新，row，key，和 value的添加使用 ImmutableTable.Builder。
- TreeBasedTable 的 row 和 column 是可排序的。

#### Range

Range 类允许我们创建一个区间，并且可以与 Comparable 类型一起工作。Range 对象可以定义是否包含端点，即是否为开闭区间。看下面例子：

```java
Range<Integer> numberRange = Range.closed(1,10);
//下面的两行返回true
numberRange.contains(10);
numberRange.contains(1);

Range<Integer> numberRange = Range.open(1,10);
//下面的两行返回false
numberRange.contains(10);
numberRange.contains(1);
```

我们可以以下面这些方法来创建 Range 对象。openClosed, closedOpen, greaterThan, atLeast, lessThan, and atMost。什么的这些方法都是静态工厂方法，返回我们需要的 Range 对象。

##### Ranges 与任意的 comparable 对象

因为 Range 对象可以与 实现了 comparable 接口的任何对象一起工作，它使我们更简单的过滤这些对象来获得我们需要的区间。看下面的例子：

```java
public class Person implements Comparable<Person> {
    private String firstName;
    private String lastName;
    private int age;
    private String sex;
    @Override
    public int compareTo(Person o) {
    	return ComparisonChain.start().
        compare(this.firstName,o.getFirstName()).
        compare(this.lastName,o.getLastName()).
        compare(this.age,o.getAge()).
        compare(this.sex,o.getSex()).result();
    }
}
```

我们想创建一个 Range 实例里面包含 Person 对象，并且年龄是35到50之间的。但是你看 compareTo 方法，我们有一个问题。

这个方法比较了对象的所有属性，为了解决这个问题，我们会利用 Range 对象实现了 Predicate 接口的事实。

我们可以使用 Predicates.compose 方法创建一个新的 Predicate，首先定义 Range 实例：

```java
Range<Integer> ageRange = Range.closed(35,50);
```

然后创建一个 Function 他允许传入一个 Person 并且返回他的年龄：

```java
Function<Person,Integer> ageFunction = new Function<Person,Integer>() {
    @Override
    public Integer apply(Person person) {
    return person.getAge();
    }
};
```

最后我们创建组合的 Predicate 对象：

```java
Predicate<Person> predicate =
Predicates.compose(ageRange,ageFunction);
```

现在我们只是简单的创建了过滤年龄的 Predicate 对象。我们也可以使用组合来创建任何的 Range 对象或者 Comparable 对象。

#### 不可变集合

不可变集合有几个优点：

- 他们是线程安全的
- 他们提供保护避免未知用户访问你的代码

##### 创建不可变集合实例

所有的 Guava 不可变集合都有一个静态 Builder 类来创建实例。看下面的例子：

```java
MultiMap<Integer,String> map = new
ImmutableListMultimap.Builder<Integer,String>()
.put(1,"Foo")
.putAll(2,"Foo","Bar","Baz").putAll(4,"Huey","Duey","Luey")
.put(3,"Single").build();
```

#### Ordering

Ordering 类提供给我们不同的排序功能。Ordering 是一个抽象类。它实现了 Comparator 接口，并且有一个抽象的 compare 方法。

##### 创建 Ordering 实例

有两种方式创建 Ordering 实例：

- 创建一个实例并且实现 compare 方法
- 使用静态方法 Ordering.from 创建

##### 逆向排序

我们有如下排序器：

```java
public class CityByPopluation implements Comparator<City> {
    @Override
    public int compare(City city1, City city2) {
    return Ints.compare(city1.getPopulation(),city2.
    	getPopulation());
    }
}
```

使用上面的排序器可以对集合进行排序，并且根据从小到大的排序，如果我们想从大到小的排序可以如下：

```java
Ordering.from(cityByPopluation).reverse();
```

##### 处理 null 情况

当我们排序的时候我们需要考虑如何处理 null 值。是将 null 放在最前面还是最后面， Ordering 使这个选择更简单：

```java
Ordering.from(comparator).nullsFirst();
Ordering.from(comparator).nullsLast();
```

##### 二次排序

当我们进行排序的时候，我们需要考虑两个对象相等的情况，我们定义一个二次排序的标准。首先我们定义一个如下的排序器：

```java
public class CityByRainfall implements Comparator<City> {
    @Override
    public int compare(City city1, City city2) {
    return Doubles.compare(city1.getAverageRainfall(),city2.
    getAverageRainfal
    l());
    }
}
```

我们可以使用下面的方法来获取一个二次排序对象:

```java
Ordering.from(cityByPopulation).compound(cityByRainfall);
```

看下面的单元测试就可以知道这个排序规则：

```java
@Test
public void testSecondarySort(){
    City city1 = cityBuilder.population(100000).
    averageRainfall(55.0).build();
    City city2 = cityBuilder.population(100000).
    averageRainfall(45.0).build();
    City city3 = cityBuilder.population(100000).
    averageRainfall(33.8).build();
    List<City> cities = Lists.newArrayList(city1,city2,city3);
    Ordering<City> secondaryOrdering = Ordering.
    from(cityByPopulation).compound(cityByRainfall);
    Collections.sort(cities,secondaryOrdering);
    assertThat(cities.get(0),is(city3));
}
```

##### 检索最大最小值

Ordering 允许我们简单的检索集合当中的最大最小值。

```java
Ordering<City> ordering = Ordering.from(cityByPopluation);
List<City> topFive = ordering.greatestOf(cityList,5);
List<City> bottomThree = ordering.leastOf(cityList,3);
```

---EOF---

