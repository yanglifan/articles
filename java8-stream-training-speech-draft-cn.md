# ʲô�� Java 8 Stream
����� stream ������ֵ�� InputStream��OutputStream������ָ����ʽ�������� stream ������������Ϊ��ˮ�߼ӹ�����
 
��һ������Java 8 ���� Stream ������ԣ��ڽ�� Java 8 �� Lambda ���ʽ��ʹ�� Java ���԰����˺���ʽ��̵����ԡ�
 
# Ϊʲô��Ҫ Java 8 Stream
 
���������Ա�һ�� SQL �� Java ��ͳ��ʽ�ڴ�������ʱ��Ĳ��
 
������ SQL�����Ǽ�����һ���� `students`���������Ҫ��ѯ�������������������ǿ������������䣺
 
```sql
select sum(id) from students where gender = 'MALE'
```
 
����� Java 8 ֮ǰ�İ汾����һ�� `Student` �ļ���
 
```java
int sum = 0;
for (Student s : students) {
    if (t.getGender() == MALE) {
        sum++;
    }
}
return sum;
```
 
������ Java 8 ����ǿ�������д��
 
```java
students.stream().filter(s -> s.getGender() == MALE).count();
```
 
�Ա�һ�£��ǲ��� Java 8 д�����Ĳ�ѯ���� SQL һ����ࡣ��Ȼ���ﻹ�� Lambda ���ʽ�Ĺ��ͣ����������ݲ����ܡ�
 
# Stream API �еķ�������
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
 
## ʵ��
 
# ������

# �ο�
* [Lambdas and Streams in Java 8 Libraries](http://www.drdobbs.com/jvm/lambdas-and-streams-in-java-8-libraries/240166818)
* [Lesson: Aggregate Operations](http://docs.oracle.com/javase/tutorial/collections/streams/index.html)
* [Java 8 �е� Streams API ���](http://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/)