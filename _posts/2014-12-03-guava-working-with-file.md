---
layout: post
title: Guava操作文件工具类
categories: guava
tags: 笔记 java
---

读写文件是一个很核心的编程任务。java 提供了一个丰富和健壮的库来进行 I/O 操作，但是执行一些基本操作非常繁琐。Guava 提供了一些简单的工具类。

#### 复制文件

Files 提供了一些有用的方法来操作 File 对象。对于 java 开发人员来说，从一个文件复制到另外一个文件是有挑战性的。但是，让我们考虑如何使用 Guava 来实现相同的功能。

```java
File original = new File("path/to/original");
File copy = new File("path/to/copy");
Files.copy(original, copy);
```

#### 移动/从命名文件

使用 Guava 进行移动文件是非常简单的：

```java
public class GuavaMoveFileExample {
    public static void main(String[] args) {
        File original = new File("src/main/resources/copy.txt");
        File newFile = new File("src/main/resources/newFile.txt");
        try{
            Files.move(original, newFile);
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```

#### 把文件当中字符串来操作

有时候我们需要操作文件里面的字符。Files 类有一个方法可以按行来读取文件并且把每行的信息存放到 list 当中，并且返回这个 list 。

```java
@Test
public void readFileIntoListOfStringsTest() throws Exception{
    File file = new File("src/main/resources/lines.txt");
    List<String> expectedLines = Lists.newArrayList("The quickbrown","fox jumps over","the lazy dog");
    List<String> readLines = Files.readLines(file,Charsets.UTF_8);
    assertThat(expectedLines,is(readLines));
}
```

还有一个重载的 readLines 方法，接受一个 LineProcessor 实例，读取的每一行都会调用 LineProcessor.processLine 并且把这行的数据传入进去，这个方法会返回一个布尔值。当文件读到末尾或者这个 LineProcessor.processLine 返回 false 的时候将停止读取文件。

看下面的例子:

```java
public class MyLineProcessor implements LineProcessor<List<String>>
{
    private List<String> result = Lists.newArrayList();

    @Override
    public boolean processLine(String line) throws IOException
    {
        result.add(line);
        return true;
    }

    @Override
    public List<String> getResult()
    {
        return result;
    }
}

@Test
public void readLinesTest2() throws Exception
{
    LineProcessor<List<String>> my = new MyLineProcessor();
    List<String> list = Files.readLines(new File("a.txt"), Charsets.UTF_8, my);
    System.out.println(my.getResult());
}
```

#### 哈希文件

可以使用下面的方法进行哈希文件：

```java
public class HashFileExample {
    public static void main(String[] args) throws IOException {
    File file = new File("src/main/resources/sampleTextFileOne.txt");
    HashCode hashCode = Files.hash(file, Hashing.md5());
    System.out.println(hashCode);
	}
}
````

#### 写文件

当我们操作输入/输出流的时候，我们需要一下几步：

1. 打开输入/输出流
2. 读取或者输出字节到流当中
3. 当做完操作后在 finally 块当中关闭流

我们不得不一次次的重复，很容易出错。Files 类提供便利的方法写文件或者从文件读取内容到字节数组中。

#### 写或者追加到文件

看下面的例子:

```java
@Test
public void appendingWritingToFileTest() throws IOException {
    File file = new File("src/test/resources/quote.txt");
    file.deleteOnExit();
    String hamletQuoteStart = "To be, or not to be";
    Files.write(hamletQuoteStart,file, Charsets.UTF_8);
    assertThat(Files.toString(file,Charsets.UTF_8),is(hamletQuoteStart));
    String hamletQuoteEnd = ",that is the question";
    Files.append(hamletQuoteEnd,file,Charsets.UTF_8);
    assertThat(Files.toString(file, Charsets.UTF_8),is(hamletQuoteStart + hamletQuoteEnd));
    String overwrite = "Overwriting the file";
    Files.write(overwrite, file, Charsets.UTF_8);
    assertThat(Files.toString(file, Charsets.UTF_8),is(overwrite));
}
```

使用 Files.write 方法往文件中写入数据，使用 Files.append 方法往文件中追加数据。

#### InputSupplier 和 OutputSupplier

Guava 提供了 InputSupplier 和 OutputSupplier 接口来作为使用 InputStreams/Readers 和 OutputStreams/Writers 的支持工具。

#### Sources 和 Sinks

Guava I/O 有 Sources 和 Sinks 的概念分别对应读和写文件。Sources 和 Sinks 不是 streams, readers, writers，但是提供的功能与上面的类似。Sources 和 Sinks 有两种使用方式：

- 我们可以从提供者检索底层流。每次提供者返回一个流，都是一个新的对象。调用者检索基本流后负责关闭流。
- 有基本的便利方法用于执行基本操作，像读取一个流或者写一流。

有两种类型的 Sources: ByteSource 和 CharsSource。

有两种类型的 Sinks: ByteSink 和 CharSink。

Source 和 Sink 类分别提供相同的功能。他们的不同的方法操作的是 characters 或者 bytes。Files 类提供了一些方法供 ByteSink 和 CharSink 来操作文件。我们也可以使用 Files 的静态工厂方法创建 ByteSource, ByteSink, CharSource, 和 CharSink ,ByteStreams, CharStreams 实例。

#### ByteSource

ByteSource 类代表可读的字节源。我们可以期望从文件中获基本的字节源，但是我们也可以从一个字节数组中获取。

我们可以使用 Files 类的静态方法从一个文件中创建 ByteSource。

```java
@Test
public void createByteSourceFromFileTest() throws Exception {
    File f1 = new File("src/main/resources/sample.pdf");
    ByteSource byteSource = Files.asByteSource(f1);
    byte[] readBytes = byteSource.read();
    assertThat(readBytes,is(Files.toByteArray(f1)));
}
```

#### ByteSink

ByteSink 类代表一个可写的字节源。我们可以把字节写入文件或者写到字节数组中。看下面例子：

```java
@Test
public void testCreateFileByteSink() throws Exception {
    File dest = new File("src/test/resources/byteSink.pdf");
    dest.deleteOnExit();
    ByteSink byteSink = Files.asByteSink(dest);
    File file = new File("src/main/resources/sample.pdf");
    byteSink.write(Files.toByteArray(file));
    assertThat(Files.toByteArray(dest),is(Files.
    toByteArray(file)));
}
```

#### 从 ByteSource 复制到 ByteSink

现在我们将要一起使用 ByteSource 和 ByteSink 类，将复制字节从 ByteSource 实例到 ByteSink 实例当中。可以很明显的看出这，这里有些概念。首先，我们处理抽象的 ByteSource 和 ByteSink 实例。我们不需要知道根本的源。其次，源的开闭不需要我们处理。

```java
@Test
public void copyToByteSinkTest() throws Exception {
    File dest = new File("src/test/resources/sampleCompany.pdf");
    dest.deleteOnExit();
    File source = new File("src/main/resources/sample.pdf");
    ByteSource byteSource = Files.asByteSource(source);
    ByteSink byteSink = Files.asByteSink(dest);
    byteSource.copyTo(byteSink);
    assertThat(Files.toByteArray(dest),
    is(Files.toByteArray(source)));
}
```

#### ByteStreams 和 CharStreams

ByteStreams 是处理 InputStream 和 OutputStream 实用的工具类。CharStreams 是处理 Reader 和 Writer 实用的工具类。

#### 限制 InputStreams 的大小

ByteSteams.limit 方法接收一个 InputStream 方法和一个 long 值并且返回一个包装的 InputStream 并且只读给定长度的字节。看下面例子：

```java
@Test
public void limitByteStreamTest() throws Exception {
    File binaryFile = new File("src/main/resources/sample.pdf");
    BufferedInputStream inputStream = new BufferedInputStream(new FileInputStream(binaryFile));
    InputStream limitedInputStream = ByteStreams.limit(inputStream,10);
    assertThat(limitedInputStream.available(),is(10));
    assertThat(inputStream.available(),is(218882));
}
```

#### 连接 CharStreams

CharStreams.join 接受多个 InputSupplier 实例，并且连接他们相当于一个 InputSupplier 实例，并且写出他们的内容到一个 OutputSupplier 实例中。

```java
@Test
public void joinTest() throws Exception {
    File f1 = new File("src/main/resources/sampleTextFileOne.txt");
    File f2 = new File("src/main/resources/sampleTextFileTwo.txt");
    File f3 = new File("src/main/resources/lines.txt");
    File joinedOutput = new File("src/test/resources/joined.txt");
    joinedOutput.deleteOnExit();
    List<InputSupplier<InputStreamReader>> inputSuppliers() = getInputSuppliers()(f1,f2,f3);
    InputSupplier<Reader> joinedSupplier = CharStreams.join(inputSuppliers());
    OutputSupplier<OutputStreamWriter> outputSupplier = Files.newWriterSupplier(joinedOutput, Charsets.UTF_8);
    String expectedOutputString = joinFiles(f1,f2,f3);
    CharStreams.copy(joinedSupplier,outputSupplier);
    String joinedOutputString = joinFiles(joinedOutput);
    assertThat(joinedOutputString,is(expectedOutputString));
}

private String joinFiles(File ...files) throws IOException {
    StringBuilder builder = new StringBuilder();
    for (File file : files) {
   		builder.append(Files.toString(file,Charsets.UTF_8));
    }
	return builder.toString();
}

private List<InputSupplier<InputStreamReader>> getInputSuppliers()(File ...files){
    List<InputSupplier<InputStreamReader>> list = Lists.newArrayList();
    for (File file : files) {
    	list.add(Files.newReaderSupplier(file,Charsets.UTF_8));
    }
    return list;
}
```

#### Closer

Closer 类是用于确保所有注册可关闭的对象在调用 Closer.close 之后都是关闭的。看下面的例子：

```java
public class CloserExample {
    public static void main(String[] args) throws IOException {
        Closer closer = Closer.create();
        try {
            File destination = new File("src/main/resources/copy.txt");
            destination.deleteOnExit();
            BufferedReader reader = new BufferedReader(new
            	FileReader("src/main/resources/sampleTextFileOne.txt"));
            BufferedWriter writer = new BufferedWriter(new FileWriter(destination));
            closer.register(reader);
            closer.register(writer);
            String line;
            while((line = reader.readLine())!=null){
            	writer.write(line);
            }
        } catch (Throwable t) {
        	throw closer.rethrow(t);
        } finally {
        	closer.close();
        }
    }
}
```

#### BaseEncoding

当我们处理二进制数据时候，我们有时候需要转换字节数据到字符数据。当然我们也需要从字符解码到字节。BaseEncoding 是一个抽象类里面包含工厂方法来创建不同编码方式的实例。看下面的例子：

```java
@Test
public void encodeDecodeTest() throws Exception {
    File file = new File("src/main/resources/sample.pdf");
    byte[] bytes = Files.toByteArray(file);
    BaseEncoding baseEncoding = BaseEncoding.base64();
    String encoded = baseEncoding.encode(bytes);
    assertThat(Pattern.matches("[A-Za-z0-9+/=]+",encoded),is(true));
	assertThat(baseEncoding.decode(encoded),is(bytes));
}
```

然后在看下面的例子：

```java
@Test
public void encodeByteSinkTest() throws Exception{
    File file = new File("src/main/resources/sample.pdf");
    File encodedFile = new File("src/main/resources/encoded.txt");
    encodedFile.deleteOnExit();
    CharSink charSink = Files.asCharSink(encodedFile, Charsets.UTF_8);
    BaseEncoding baseEncoding = BaseEncoding.base64();
    ByteSink byteSink = baseEncoding.encodingSink(charSink);
    ByteSource byteSource = Files.asByteSource(file);
    byteSource.copyTo(byteSink);
    String encodedBytes = baseEncoding.encode(byteSource.read());
    assertThat(encodedBytes,is(Files.toString(encodedFile,Charsets.UTF_8)));
}
```

---EOF---

