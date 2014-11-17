---
layout: post
title: Guava基本工具类
categories: guava
tags: 笔记 java
---

#### 使用 Joiner 类

将任意字符串通过分隔符进行连接到一起是大多程序员经常做的事情。他们经常使用 array，list，iterable 并且循环变量将每一个临时变量添加到 StringBuilder 当中去，并且中间添加分隔符。这些笨重的处理方式如下:

```java
public String buildString(List<String> stringList, String delimiter){
    StringBuilder builder = new StringBuilder();
    for (String s : stringList) {
        if(s != null){
        	builder.append(s).append(delimiter);
        }
    }
    builder.setLength(builder.length() – delimiter.length());
    return builder.toString();
}
```

注意要删除在最后面的分隔符。不是很难懂，但是使用 Joiner 类可以得到简单的代码模板。同样的例子使用 Joiner 类代码如下：

```java
public String buildString(List<String> stringList, String delimiter){
       return  Joiner.on(delimiter).skipNulls().join(stringList);
}
```

这样更加简明并且不会出错。如果你想将 null 值替换掉，可以使用如下方法：

```java
Joiner.on("|").useForNull("no value").join(stringList);
```

使用 Joiner 类有几点需要注意。

Joiner 类不仅仅可以处理字符串的 array、list、iterable，他还可以处理任何对象的 array、list、iterable。结果就是调用每一个元素的 toString() 方法。

因此，如果没有使用 skipNulls 或者 useForNull ，就会抛出空指针异常。

Joiner 对象一旦被创建就是不可变的，所以他们是线程安全的，可以被当作常量来看待。然后看看下面的代码片段：

```java
Joiner stringJoiner = Joiner.on("|").skipNulls();
//使用useForNull方法将会返回一个新的Joiner实例
stringJoiner.useForNull("missing");
stringJoiner.join("foo", "bar", null);
```

在上面的代码实例当中，useForNull 方法并没有起作用，null 值仍然被忽略了。

Joiner 不仅仅能返回字符串，还可以与 StringBuilder 一起使用：

```java
StringBuilder stringBuilder = new StringBuilder();
Joiner joiner = Joiner.on("|").skipNulls();
//返回的StringBuilder实例当中包含连接完成的字符串
joiner.appendTo(stringBuilder,"foo","bar","baz");
```

上面的例子，我们传入一个 StringBuilder 的参数并且返回一个 StringBuilder 实例。

Joiner 类也可以使用实现了 Appendable 接口的类。

```java
FileWriter fileWriter = new FileWriter(new File("path")):
List<Date> dateList = getDates();
Joiner joiner = Joiner.on("#").useForNulls(" ");
// 返回由字符串拼接后的FileWriter实例
joiner.appendTo(fileWriter, dateList);
```

这是一个与上一个相似的例子。我们传入一个FileWriter实例和一组数据，就会将这组数据拼接后附加到FileWriter当中并且返回。

我们可以看到，Joiner 使一些公共的操作变的非常简单。有一个特殊的方法会实现 MapJoiner 方法，MapJoiner 方法像 Joiner 一样使用分割符将每组 key 与 value 分开，同时 key 与 value 之间也有个分隔符。MapJoiner 方法的创建如下：

```java
Joiner.MapJoiner mapJoiner = Joiner.on("#").withKeyValueSeparator("=");
```

Joiner.on("#") 方法会创建一个 Joiner 的实例。

使用返回的 Joiner 实例调用 withKeyValueSeparator 方法将会返回 MapJoiner 对象。

下面是对 MapJoiner 方法的单元测试代码：

```java
@Test
public void testMapJoiner() {
    String expectedString = "Washington D.C=Redskins#New York City=Giants#Philadelphia=Eagles#Dallas=Cowboys";
    Map<String,String> testMap = Maps.newLinkedHashMap();
    testMap.put("Washington D.C", "Redskins");
    testMap.put("New York City", "Giants");
    testMap.put("Philadelphia", "Eagles");
    testMap.put("Dallas", "Cowboys");
    String returnedString = Joiner.on("#").withKeyValueSeparator("=").join(testMap);
    assertThat(returnedString,is(expectedString));
}
```

上面的单元测试开始时创建一个 key 和 value 都是字符串的 LinkedHashMap 实例，值得注意的是我们使用静态工厂方法newLinkedHashMap() 来创建，Maps 类是在 com.google.common.collect 包当中。然后使用 Joiner 类创建一个使用 key 与 value 拼接的字符串。最后我们断言他是否与我们期望返回的字符串相同。

#### 使用 Splitter 类

程序员另一个经常处理的问题是对字符串以特定分隔符进行分割并且获取一个字符串数组。
如果你需要读取文本文件，你总是会做这样的事情。但是 String.split 方法不够完美，看下面的例子：

```java
String testString = "Monday,Tuesday, ,Thursday,Friday,,";
String[] parts = testString.split(",");
//parts is [Monday, Tuesday, , Thursday,Friday]
```

可以看到， String.split 方法省略了最后的2个空串。在有些时候，这个做法是你需要的，但是这些事情是应该由程序员来决定是否省略。

Splitter 类可以帮助我们实现与 Joiner 类相反的功能。

Splitter 可以使用单个字符、固定字符串、正则表达式串、正则表达式对象或者 CharMatcher 对象（另一个 Guava 的类）来分割字符串。

可以给定具体分割符来创建 Splitter 对象然后使用。一旦拥有了 Splitter 实例后就可以调用 split 方法，并且会返回包含分割后字符串的迭代器对象。

```java
Splitter.on('|').split("foo|bar|baz");
Splitter splitter = Splitter.on("\\d+");
```

在上面的例子当中，我们看到一个 Splitter 实例使用了'|'字符分割，另外一个实例使用了正则表达式进行分割。

Splitter 有一个可以处理前面空格和后面空格的方法是 trimResults() 。

```java
//使用|分割字符串，并且去掉被分割的字符串两边的空白。
Splitter splitter = Splitter.on('|').trimResults();
```

与 Joiner 类一样 Splitter 类同样是一个不可变的类，所以在使用的时候应该使用调用 trimResults() 方法后返回的 Splitter 实例。

```java
Splitter splitter = Splitter.on('|');
splitter.trimResults();

Iterable<String> parts = splitter.split("1|2|3|||");
```

Splitter 类，像 Joiner 与 MapJoiner 一样也有 MapSplitter 类。

MapSplitter 类可以将字符串转换成 Map 实例返回，并且元素的顺序与字符串给定的顺序相同。使用下面方法构造一个 MapSplitter 实例：

```java
//MapSplitter is defined as an inner class of Splitter
Splitter.MapSplitter mapSplitter = Splitter.on("#").withKeyValueSeparator("=");
```

可以看到 MapSplitter 的创建方式与 MapJoiner 一样。首先给 Splitter 指定一个分隔符，然后指定 MapSplitter 对象 key 与 value 的分隔符。下面是一个关于 MapSplitter 的例子，实现的是与 MapJoiner 相反地功能。

```java
@Test
public void testSplitter() {
    String startString = "Washington D.C=Redskins#New York City=Giants#Philadelphia=Eagles#Dallas=Cowboys";
    Map<String,String> testMap = Maps.newLinkedHashMap();
    testMap.put("Washington D.C", "Redskins");
    testMap.put("New York City", "Giants");
    testMap.put("Philadelphia", "Eagles");
    testMap.put("Dallas", "Cowboys");
    Splitter.MapSplitter mapSplitter = Splitter.on("#").withKeyValueSeparator("=");
    Map<String,String> splitMap = mapSplitter.split(startSring);
    assertThat(testMap,is(splitMap));
}
```

#### 使用 Guava 操作字符串

Guava 提供给我们非常好用的类来操作字符串。这些类如下：

- CharMatcher
- Charsets
- Strings

现在我们看一看如何在代码中使用它们。

在第一个例子当中，这个单元测试将会展示使用 ASCII 类方法来确定一个字符是否是小写。第二个例子将展示将小写字符串转换成大写。

##### 使用 Charsets 类

在 java 当中，有6个标准字符集在每一个 java 平台都会被支持。这与经常运行下面代码是相关的：

```java
byte[] bytes = someString.getBytes();
```

但是有一个问题关于上面的代码。没有指定你想返回字节的字符集，你将会获得系统使用运行时默认的字符集返回的字节，这可能会产生问题。有可能默认字符集不是你想要的。所以最佳的做法是像下面这样：

```java
try{
	bytes = "foobarbaz".getBytes("UTF-8");
}catch (UnsupportedEncodingException e){
	//This really can't happen UTF-8 must be supported
}
```

但是仍然有两个问题在上面的代码当中：

UTF-8 在 java 平台一定会被支持，所以 UnsupportedEncodingException 一定不会被抛出，但是如果字符串的定义拼写错误会导致抛出异常。

Charsets 类可以帮助我们，Charsets 类提供了 static final 的六个 java 平台支持的字符集。使用 Charsets 类我们可以使上面的例子更简单些：

```java
byte[] bytes2 = "foobarbaz".getBytes(Charsets.UTF_8);
```

值得注意的是在 JAVA7 当中，StandardCharsets 也提供了同样的功能。现在我们看看 Strings 类。

##### 使用 Strings 类

Strings 类提供一些便利实用的方法处理字符串。你是否写过像下面的代码：

```java
StringBuilder builder = new StringBuilder("foo");
char c = 'x';
for(int i=0; i<3; i++){
	builder.append(c);
}
return builder.toString();
```

上面例子当中的6行代码我们可以使用一行代码来替换。

```java
Strings.padEnd("foo",6,'x');
```

第二个参数是很重要的，6指定返回的字符串最小长度为6，而不是指定 `'x'` 字符在字符串后面追加多少次。

如果提供的字符串长度已经大于了6，则不会进行追加。

同样也有一个相类似的 padStart 方法可以在给定字符串的前面追加字符到指定的长度。

在 Strings 类当中有三个非常有用的方法来处理空值的：

- nullToEmpty：这个方法接受一个字符串参数，如果传入的参数不是 null 值或者长度大于0则原样返回，否则返回空串("")；
- emptyToNull：这个方法类似于 nullToEmpty ，它将返回 null 值如果传入的参数是空串或者 null 。
- isNullOrEmpty：这个方法会检查传入参数是否为 null 和长度，如果是 null 和长度为0就返回 true。

##### 使用 CharMatcher 类

CharMatcher 提供了在一定字符串中匹配是否存在特定字符串的功能。在 CharMatcher 类当中的方法也可以让格式化更加简单。

例如，你可以将多行的字符串格式化成一行，并且换行符将会以空格来代替。

```java
CharMatcher.BREAKING_WHITESPACE.replaceFrom(stringWithLinebreaks,' ');
```

还有一个版本 replaceFrom 的，需要一个 CharSequence 的值作为第2个参数值，而不是一个单一的字符。

移除多个空格和 tab 以单个空格来代替，代码如下：

```java
@Test
public void testRemoveWhiteSpace(){
	String tabsAndSpaces = "String with spaces and         tabs";
	String expected = "String with spaces and tabs";
	String scrubbed = CharMatcher.WHITESPACE.collapseFrom(tabsAndSpaces,' ');
	assertThat(scrubbed,is(expected));
}
```

在上面的测试代码中，我们把所有多个空格和 tab 都替换成了一个空格，所有都在一行上。

上面例子在某些情况下可行，但是如果字符串在开头就有空格返回的字符串前面依然会包含空格，这时可以使用trimAndCollapseFrom方法：

```java
@Test
public void testTrimRemoveWhiteSpace(){
    String tabsAndSpaces = "   String with spaces and       tabs";
    String expected = "String with spaces and tabs";
    String scrubbed = CharMatcher.WHITESPACE. trimAndCollapseFrom(tabsAndSpaces,' ');
    assertThat(scrubbed,is(expected));
}
```

在这个测试当中，我们再一次将把多个空格和 tab 替换成一个空格也在一行上。

下面的例子是保留所匹配的字符的例子：

```java
@Test
public void retainFromTest()
{
    String lettersAndNumbers = "foo989yxbar234";
    String expected = "989234";
    String actual = CharMatcher.JAVA_DIGIT.retainFrom(lettersAndNumbers);
    assertEquals(expected, actual);
}
```

在这个例子当中我们找到"foo989yxbar234"字符串当中所有的数字并且保留下来。

往下继续之前，我们应该看看最后一个 CharMatcher 类中的强大功能。可以联合多个 CharMatcher 类实例创建一个新的 CharMatcher 类实例。

假设你需要创一个匹配数字或空格的 CharMatcher 类实例：

```java
CharMatcher cm = CharMatcher.JAVA_DIGIT.or(CharMatcher.WHITESPACE);
```

#### 使用Preconditions类

Preconditions 类是用来验证我们的代码状态的静态方法的集合。 Preconditions 非常重要，因为他们保证我们的期望执行成功的代码得到满足。 如果条件与我们期望的不同，我们会及时反馈问题。和以前一样，使用前提条件的重要性是确保我们代码的行为，并在调试中很有用。

你可以写你自己的先决条件，像下面这样：

```java
if(someObj == null){
    throw new IllegalArgumentException(" someObj must not be null");
}
```

使用 Preconditions 当中的方法（需要静态导入），检查一个空值更简单。

```java
checkNotNull(someObj,"someObj must not be null");
```

接下来，我们将要展示使用先决条件的例子：

```java
public class PreconditionExample {
    private String label;
    private int[] values = new int[5];
    private int currentIndex;

    public PreconditionExample(String label) {
    	//返回label如果不为空
    	this.label = checkNotNull(label,"Label can''t be null");
    }

    public void updateCurrentIndexValue(int index, int valueToSet) {
    	//检查索引是否有效
    	this.currentIndex = checkElementIndex(index, values.length, "Index out of bounds for values");
   	 	//检查参数值
    	checkArgument(valueToSet <= 100,"Value can't be more than 100"); 
    	values[this.currentIndex] = valueToSet;
    }

    public void doOperation(){
    	checkState(validateObjectState(),"Can't perform operation");
    }

    private boolean validateObjectState(){
    	return this.label.equalsIgnoreCase("open") && values[this.currentIndex] == 10;
    }
}
```

下面是对上面例子当中四个方法的摘要信息：

- checkNotNull(T object, Object message)：这个方法如果 object 不为 null 直接返回，如果为 null 会抛出空指针异常。
- checkElementIndex (int index, int size, Object message)：在这方法当中，index 是你将要访问的元素下标，size 是这个要访问的 array，list 或者字符串的长度。然后校验是否有效，如果无效抛出 IndexOutOfBoundsException。
- checkArgument (Boolean expression, Object message)：这方法传入布尔表达式。 这个布尔表达式如果为 true 则继续执行，否则抛出 IllegalArgumentException。
- checkState (Boolean expression, Object message)：这方法传入一个布尔表达式涉及对象的状态，而不是参数。 这个布尔表达式如果为 true 则继续执行，否则抛出 IllegalArgumentException。

#### Object工具

在这个章节当中我们将介绍帮助检查 null 值和创建 toString 和 hashCode 的方法的实用方法。我们接着去看看一个有用的类，它实现 Comparable 接口。

当我们要调试的时候，toString 方法是必须重写的，重写它的乏味无趣的。然而，Objects 类可以使用 toStringHelper 方法让重写更简单。看下面的例子：

```java
public class Book {
    private Person author;
    private String title;
    private String publisher;
    private String isbn;
	private double price;

    public String toString() {
    return Objects.toStringHelper(this).omitNullValues().add("title", title).
	add("author", author).add("publisher", publisher).add("price",price).add("isbn", isbn).toString();
	}
}
```

首先我们传入一个 Book 对象来创建一个 Objects.ToStringHelper 实例。

第二步，我们调用 omitNullValues 来排除任何 null 值的属性。

调用 add 方法来添加每一个属性的标签和属性。

##### 检查null值

firstNonNull 方法接受2个参数并且返回第一个参数如果它不为 null。

```java
String value = Objects.firstNonNull(someString, "default value");
```

firstNonNull 方法使用时候如果你不确定传入的值是否为 null 你可以提供一个默认值给它。需要注意：如果传入的2个参数都是 null，会抛出空指针异常。

##### 创建 hash codes

为类写 hashCode 方法是基本的但是乏味无趣。Objects 类可以帮助我们使用 hashCode 方法更加简单。考虑下 Book 类有4个属性：title, author, publisher, 和 isbn. 下面的代码将展示使用 Object.hashCode 方法返回 hashCode 值。

```java
public int hashCode() {
	return Objects.hashCode(title, author, publisher, isbn);
}
```

##### 实现CompareTo方法

再次使用 Book 类，下面是典型的实现 compareTo方法。

```java
public int compareTo(Book o) {
    int result = this.title.compareTo(o.getTitle());
    if (result != 0) {
    	return result;
    }

    result = this.author.compareTo(o.getAuthor());
    if (result != 0) {
        return result;
    }

    result = this.publisher.compareTo(o.getPublisher());
    if(result !=0 ) {
    	return result;
    }

    return this.isbn.compareTo(o.getIsbn());
}
```

现在让我们看一看使用 ComparisonChain 类来实现 compareTo 方法：

```java
public int compareTo(Book o) {
	return ComparisonChain.start().compare(this.title, o.getTitle()).compare(this.author, o.getAuthor()) 
.compare(this.publisher, o.getPublisher()).compare(this.isbn, o.getIsbn()).compare(this.price, o.getPrice()).result();
}
```

第二个例子显得更紧凑和更好阅读。而且，ComparisonChain 类会在第一个比较当中如果返回非零时停止比较，只有一种情况返回0，那就是所有的比较返回的都是0。

---EOF---


