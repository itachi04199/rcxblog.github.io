---
layout: post
title: 使用Guava进行函数编程
categories: guava
tags: 笔记 java
---

#### 使用 Function 接口

函数式编程强调使用函数，以实现其目标与不断变化的状态。这与大多数开发者熟悉的改变状态的编程方式形成对比。Function 接口让我们在 java 代码当中引入函数式编程成为可能。

Function 接口当中只有2个方法：

```java
public interface Function<F,T> {
	T apply(F input);
	boolean equals(Object object);
}
```

我们不会具体的使用 equals 方法来判断 A对象与 B对象是否相等，只会调用 apply 方法来比较 A对象与 B对象是否相等。

apply 方法接受一个参数并且返回一个对象。一个好的功能实现应该没有副作用，这意味着当一个对象传入到 apply 方法当中后应该是保持不变的。

下面是一个接受 Date 对象并且返回 Date 格式化后字符串的例子：

```java
public class DateFormatFunction implements Function<Date,String> {
    @Override
    public String apply(Date input) {
    	SimpleDateFormat dateFormat = new SimpleDateFormat("dd/mm/yyyy");
    	return dateFormat.format(input);
    }
}
```

在这个例子当中，我们可以清楚看到 Date 对象正在通过 SimpleDateFormat 类转换成我们期望格式的字符串。虽然这个例子可能过于简单，但是它演示了 Function 接口的作用，转换一个对象并且隐藏了实现的细节。通过这个例子我们可以使用实现了 Function 接口的类，我们也可以使用匿名类来实现。看看下面的例子：

```java
Function<Date,String> function = new Function<Date, String>() {
    @Override
    public String apply( Date input) {
    	return new SimpleDateFormat("dd/mm/yyyy").format(input);
    }
};
```

这2个例子没什么不同。一个是简单的实现了 Function 接口，另一个是匿名类。实现 Function 接口的类有一个优点，你可以使用依赖注入来传递一个 Function 对象到一个协作的类，使得代码高内聚。

##### 使用 Function 接口的参考

这可能是一个很好的时间来讨论在你的代码当中引入 Function 接口和使用匿名类。在 java 当前的状态中，我们没有其他的语言有的闭包特性。当 JAVA8 发布后这将会改变，现在 java 的回答就是使用匿名类。当你使用匿名类来充当闭包的时候，语法是相当的繁重并且如果使用太频繁，会使你的代码很难跟踪和维护。事实上，我们对上面的例子进行分析下，上面的服务知识为了演示 Functions 是如何使用，我们不能获取到更多的好处。例如，下面的更典型一些来实现相同目标的功能。

```java
public String formatDate(Date input) {
	return new SimpleDateFormat("dd/mm/yyyy").format(input);
}
```

现在比较一下这两个例子，我们会发现后面一个例子更加容易读懂。当我们使用 Function 取决于你想在什么地方进行转换。

如果你有一个类里面包含一个 Date 实例变量和一个返回日期转换成期望字符串的方法，你可能更好的执行后面的例子。

然而，如果你有一个 Date 对象的集合并且想获取这些 Date 的字符串形式的 list，使用 Function 接口可能是一个不错的方法。

##### 使用 Functions 类

Functions 类包含一些实用的方法来操作 Fucntion 接口的实例。在本节当中，我们将会讲解这些方法当中的两个方法可以帮助我们使 Fucntion 接口更加有效。

##### 使用 Functions.forMap 方法

forMap 方法接受一个 `Map<K,V>` 的参数并且返回一个 `Function<K,V>` 实例，执行 apply 方法会在 map 当中进行查找。例如，考虑下面的类代表美国的一个州。

```java
public class State {
    private String name;
    private String code;
    private Set<City> mainCities = new HashSet<City>();
}
```

现在你有一个名为 stateMap 的 `Map<String, State>` ,他的 key 就是州名的缩写。现在我们可以创建一个通过州代码来查找的函数，你只需要下面几步：

```java
Function<String,State> lookup = Functions.forMap(stateMap);
//Would return State object for NewYork
lookup.apply("NY");
```

使用 Functions.forMap 方法有一个需要注意的。如果传入的 key 在 map 当中不存在会抛出 IllegalArgumentException 异常。

然而，有一个重载的 forMap 方法增加一个默认值参数，如果 key 没找到会返回传入的默认值。通过使用 Function 接口来执行 state 的查找，你可以很容易的改变这个实现。

##### 使用 Functions.compose 方法

假设你现在有一个代表city的类，代码如下：

```java
public class City {
    private String name;
    private String zipCode;
    private int population;
    public String toString() {
    	return name;
    }
}
```

考虑下面的情形：你要创建一个 Function 实例，传入一个 State 对象返回 State 类当中 mainCities 使用逗号分割的字符串。代码如下：

```java
public class StateToCityString implements Function<State,String> {
    @Override
    public String apply(State input) {
    	return Joiner.on(",").join(input.getMainCities());
    }
}
```

更进一步来说。你希望只有一个 Function 实例通过传入 State 的名称缩写来返回这 State 当中 mainCities 使用逗号分割的字符串。Guava 提高了一个很好的方法来解决这种情况，Functions.compose 方法接受两个 Function 实例作为参数，并且返回这两个Function 组合后的 Function。所以我们可以使用上面的两个 Function 来举一个例子：

```java
Function<String, State> lookup = Functions.forMap(stateMap);
Function<State, String> stateFunction = new StateToCityString();
Function<String, String> composed = Functions.compose(stateFunction ,lookup);
```

现在调用 composed.apply("NY") 方法将会返回："Albany,Buffalo,NewYorkCity"。

看下方法的调用顺序。composed 接受一个 “NY” 参数并且调用 lookup.apply() 方法，从 lookup.apply() 方法中返回的值传入了 stateFunction.apply() 方法当中并且返回执行结果。

**可以理解为第二个参数的输入参数就是 composed 方法的输入参数，第一个参数的输出就是 composed 方法的输出**。如果不使用 composed 方法，在前面的例子将如下所示：

```java
String cities = stateFunction.apply(lookup.apply("NY"));
```

#### 使用 Predicate 接口

Predicate 接口与 Function 接口的功能相似。像 Function 接口一样，Predicate 接口也有2个方法：

```java
public interface Predicate<T> {
    boolean apply(T input)
	boolean equals(Object object)
}
```

Function 接口的情况是，我们不会去详细的讲解 equals 方法。apply 方法会返回对输入断定后的结果。

在 Fucntion 接口使用的地方来转换对象，Predicate 接口则用于过滤对象。

Predicates 类的使用方针与 Functions 类一样。当一个简单方法能实现的不使用 Predicates 类。同时，Predicate 接口没有任何副作用。

##### Predicate 接口的例子

这是 Predicate 接口的一个简单的例子，我们使用上面例子当中的 City 类。我们定义一个 Predicate 来判断这个城市人口是否小于等于50W。

```java
public class PopulationPredicate implements Predicate<City> {
    @Override
    public boolean apply(City input) {
    	return input.getPopulation() <= 500000;
    }
}
```

在这个例子当中，我们简单的检查了传入的 City 对象当中的人口属性，当人口属性小于等于500000的时候会返回 true。

通常情况下，你可以将 Predicate 的实现定义成匿名类来过滤集合当中的每一个元素。Predicate 接口和 Function 接口是如此的相似，许多情况下使用 Fucntion 接口的时候同样可以使用 Predicate 接口。

##### 使用 Predicates 类

Predicates 类是包含一些实用的方法来操作 Predicate 接口的实例。Predicates 类提供了一些非常有用的方法，从布尔值条件中得到期望的值，也可以使用 “and” 和 “or” 方法来连接不同的 Predicate 实例作为条件，并且如果提供 “not” 方法一个 Predicate 实例返回的值是 false 则 “not” 方法返回 true，反之亦然。

同样也有 Predicates.compose 方法，但是他接受一个 Predicate 实例和一个 Function 实例，并且返回 Predicate 执行后的值把 Function 执行后的值当中它的参数。让我们看些例子，我们会更好的理解如何在代码中使用 Predicates 类。先看个特殊的例子，假设我们有下面两个类的实例。

```java
public class TemperateClimatePredicate implements Predicate<City> {
    @Override
    public boolean apply(City input) {
    	return input.getClimate().equals(Climate.TEMPERATE);
    }
}

public class LowRainfallPredicate implements Predicate<City> {
    @Override
    public boolean apply(City input) {
    	return input.getAverageRainfall() < 45.7;
    }
}
```

值得重申，虽然不是必需的，我们通常会定义 Predicate 实例作为匿名类，但为清楚起见，我们将使用具体类。

##### 使用 Predicates.and 方法

Predicates.and 方法接受多个 Predicate 对象并且返回一个 Predicate 对象，因此调用返回的 Predicate 对象的 apply 方法当所有 Predicate 对象的 apply 方法都返回 true 的时候会返回 true。

如果其中一个 Predicate 对象返回 false，其他的 Predicate 对象的执行就会停止。

例如，假如我们只允许城市人口小于500,000并且年降雨量小于45.7英寸的。

```java
Predicate smallAndDry = Predicates.and(smallPopulationPredicate, lowRainFallPredicate);
//下面是Predicates.and方法的签名：
Predicates.and(Iterable<Predicate<T>> predicates);
Predicates.and(Predicate<T> ...predicates);
```

##### 使用 Predicates.or 方法

Predicates.or 方法接受多个 Predicate 对象并且返回一个 Predicate 对象,如果当中有一个 Predicate 对象的 apply 方法返回 true 则总方法就返回 true。

如果有一个 Predicate 实例返回 true，就不会继续执行。

例如，假设我们想包含城市人口小于等于500,000或者有温带气候的城市。

```java
Predicate smallTemperate = Predicates.or(smallPopulationPredicate, temperateClimatePredicate);
//下面是Predicates.or方法的签名：
Predicates.or(Iterable<Predicate<T>> predicates);
Predicates.or(Predicate<T> ...predicates);
```

##### 使用 Predicates.not 方法

Predicates.not 接受一个 Predicate 实例并且执行这个 Predicate 实例的逻辑否的功能。加入我们想获得人口大于500,000的城市。使用这个方法可以代替重写一个 Predicate 实例：

```java
Predicate largeCityPredicate = Predicate.not(smallPopulationPredicate);
```

##### 使用 Predicates.compose 方法

Predicates.compose 接受一个 Function 实例和一个 Predicate 实例作为参数，并且从 Fucntion 实例当中返回的对象传入到 Predicate 对象当中并且进行评估。在下面的例子当中，我们创建一个新的 Predicate 对象：

```java
public class SouthwestOrMidwestRegionPredicate implements Predicate<State> {
    @Override
    public boolean apply(State input) {
    	return input.getRegion().equals(Region.MIDWEST) ||
    			input.getRegion().equals(Region.SOUTHWEST);
    }
}
```

下一步，我们将使用原来的 Function 实例 lookup 来创建一个 State 对象，并且使用上面例子的 Predicate 实例来评估下这个 State 是否在 MIDWEST 或者 SOUTHWEST：

```java
Predicate<String> predicate = Predicates.compose(southwestOrMidwestRegionPredicate, lookup);
```

#### 使用Supplier接口

Supplier 接口只有一个方法如下：

```java
public interface Supplier<T> {
	T get();
}
```

get 方法返回泛型 T 的实例。Supplier 接口可以帮助我们时间几个典型的创建模式。

当 get 方法被调用，我们可以返回相同的实例或者每次调用都返回新的实例。 Supplier 也可以让你灵活选择是否当 get 方法调用的时候才创建实例。

并且 Supplier 是个接口，单元测试也会更简单，相对于其他方法创建的对象，如静态工厂方法。

总之，供应 Supplier 接口的强大之处在于它抽象的复杂性和对象如何需要创建的细节，让开发人员以最好的方式创建一个对象。让我们看看如何使用 Supplier 接口。

下面代码是Supplier接口的例子：

```java
public class ComposedPredicateSupplier implements Supplier<Predicate<String>> {
    @Override
    public Predicate<String> get() {
        City city = new City("Austin,TX","12345",250000, Climate.SUB_ TROPICAL, 45.3);
        State state = new State("Texas","TX", Sets.newHashSet(city), Region.SOUTHWEST); 
        City city1 = new City("New York,NY","12345",2000000,Climate.TEMPERATE, 48.7); 
        State state1 = new State("New York","NY",Sets.newHashSet(city1), Region.NORTHEAST);
        Map<String,State> stateMap = Maps.newHashMap();
        stateMap.put(state.getCode(),state);
        stateMap.put(state1.getCode(),state1);
        Function<String,State> mf = Functions.forMap(stateMap);
        return Predicates.compose(new RegionPredicate(), mf);
    }
}
```

在这个例子当中，我们看到使用 Functions.forMap 关键一个 Function 实例可以通过 State 的缩写来查找 State，和使用 Predicate 实例来评估在那些地方是否有这个 State。

然后将 Function 实例和 Predicate 实例作为参数传入到 Predicates.compose 方法当中并且返回期望的 Predicate 实例。

我们使用了两个静态方法，Maps.newHashMap() 和 Sets.newHashSet()。现在我们每次调用都会返回新的实例。我们也可以把创建 Predicate 实例的工作放到 ComposedPredicateSupplier 的构造方法中来进行，当每次调用get的时候返回相同的实例。接着往下看，Guava 提供了更简单的选择。

##### 使用 Suppliers 类

正如我们所期望的 Guava，有一个 Suppliers 类的静态方法来操作 Supplier 实例。在前面的例子，每次调用 get 方法都会返回一个新的实例。如果我们想改变我们的实现并且每次返回相同的实例，Suppliers 给我们一些可选项。

##### 使用 Suppliers.memoize 方法

Suppliers.memoize 方法返回一个包装了委托实现的 Supplier 实例。当第一调用 get 被执行，将这个委托的 Supplier 对象实例传递进去，他返回被包装后的 Supplier 实例。包装后的 Supplier 实例会缓存调用返回的结果。后面的调用 get 方法会返回缓存的实例。

我们可以这样使用 Suppliers.memoize 方法：

```java
Supplier<Predicate<String>> wrapped = Suppliers.memoize(composedPredicateSupplier);
```

只增加一行代码我们就可以返回相同的实例。

##### 使用 Suppliers.memoizeWithExpiration 方法

Suppliers.memoizeWithExpiration 方法与 memoize 方法工作相同，只不过缓存的对象超过了时间就会抛出异常，在给定的时间当中缓存并且返回Supplier包装对象。注意的是改实例不是物理缓存。

```java
Supplier<Predicate<String>> wrapped = Suppliers.memoizeWithExpiration(composedPredicateSupplier,10L,TimeUnit.MINUTES);
```

这里我们包装了 Supplier 并且设置了超时时间为10分钟。对于 ComposedPredicateSupplier 来说没什么不同，但是 Supplier 返回的对象可能不同，可能从数据库当中恢复，例如 memoizeWithExpiration 方法会非常有用。

通过依赖注入来使用 Supplier 接口是强有力的组合。然而，如果你使用Guice（google的依赖注入框架），它包含了 `Provider<T>` 接口提供了跟 `Supplier<T>` 接口相同的功能。当然，如果你想利用缓存这个特性你可以使用 Supplier 接口。

---EOF---

