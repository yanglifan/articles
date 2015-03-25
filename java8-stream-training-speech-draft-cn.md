# 什么是 Java 8 Stream
这里的 stream 并不是值得 InputStream、OutputStream，而是指的流式处理。所以 stream 这个流可以理解为流水线加工处理。

进一步讲，Java 8 引入 Stream 这个特性，在结合 Java 8 的 Lambda 表达式，使得 Java 语言包含了函数式编程的特性。

# 为什么需要 Java 8 Stream

我们先来对比一下 SQL 和 Java 传统方式在处理数据时候的差别：

首先是 SQL。我们假设有一个表 `students`。如果我们要查询所有男生的数量，我们可以用下面的语句：

```sql
select sum(id) from students where gender = 'MALE'
```

如果用 Java 8 之前的版本处理一个 `Student` 的集合

```java
int sum = 0;
for (Student s : students) {
    if (t.getGender() == MALE) {
        sum++;
    }
}
return sum;
```

但是在 Java 8 里，我们可以这样写：

```java
students.stream().filter(s -> s.getGender() == MALE).count();
```

对比一下，是不是 Java 8 写出来的查询语句和 SQL 一样简洁。当然这里还有 Lambda 表达式的功劳，我们这里暂不介绍。

# Stream API 中的方法介绍
## Intermediate methods
* `Stream<T> filter(Predicate<? super T> predicate)`
* `<R> Stream<R> map(Function<? super T,? extends R> mapper)`
* `<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)`
* `Stream<T> distinct()`
* `Stream<T> limit(long maxSize)`
* `Stream<T> sorted()`

## Terminate methods
* `long count()`
* `void forEach(Consumer<? super T> action)`
* `Optional<T> findAny()`
* `<R,A> R collect(Collector<? super T,A,R> collector)`
* `T reduce(T identity, BinaryOperator<T> accumulator)`

## 实例

# 并发流