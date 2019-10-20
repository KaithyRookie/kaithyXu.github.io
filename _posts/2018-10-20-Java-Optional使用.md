---
layout:     post
title:      JDK8 Optional操作学习
subtitle:   JDK8 Optional操作学习
date:       2018-10-20
author:     KaithyXu
header-img: img/Optional.jpg
catalog: true
tags:
    - Java
---
## JDK8 Optional操作学习


### 介绍

Optional是JDK8中提供用于包含未知对象的工具类，即可以利用Optional包装对象来避免繁琐的空指针检查，以及NullPointException的处理，在Optional中，用value代表所包裹的对象。


### API

 

| 方法名称 | 返回类型 | 参数 | 功能 |
| --- | --- | --- | --- |
| static empty() | Optional<T> | 无 | 生成一个空的Optional对象 |
| static of| Optional | T value | 返回一个包裹给定对象的Optional对象 |
| static ofNullable | Optional | T value | 若参数对象不为null，这返回包裹该参数对象的Optional对象，否则返回一个空的Optional对象 |
| equals | boolean | Object | 仅当参数类型为Optional 且 参数中的Optional包裹的值与参数中的值相等时返回为true其余都返回false |
| filtrer | Optional<T> | Predicate<? super T > predicate | 过滤器，过滤出满足predicate函数中定义的判断条件的value，并将此value用Optional包裹起来，若value都不满足，这返回一个空的Optional对象 |
| flatMap | <U>Optional<U> | Function<? super T,Optional<U>> mapper | 根据提供的映射函数，将Optional中的value值应用与mapper方法，并将返回结果包装为一个Optional对象，若mapper返回结果为null，这返回一个空的Optional对象 |
| get | T | 无 | 返回Optional中的value，若Optional为空，则抛出NoSuchElementException |
| hsahCode | int | 无 | 返回value的hashCode值，若value为null，则返回0 |
| ifPresent | void | Consumer<? super T> consumer | 若Optional中value不为空，则让value执行参数提供的consumer函数，否则不采取任何行动 |
| ifPresentOrElse | void | Consumer<? super T> consumer, Runnable emptyAcrion | 若Optional中value不为空，则让value执行参数提供的consumer函数，否则执行参数提供的emptyAction中的run方法 |
| isPresent | boolean | 无 | 判断Optional中value是否为空，为空则返回false |
| map | <U>Optional<U> | Function<? super T, extends U> mapper | 若Optional中value不为空，则将value应用与提供的mapper方法，并且若返回结果不为空，则返回包裹该结果的Optional对象 |
| orElse | T | T other | 若Optional中的value不为空，则返回该value，否则返回参数提供的other |
| orElseGet | T | Supplier<? extend T > other | 若Optional中的value不为空，则返回value，否则返回函数other的执行结果 |
| orElseThrow | <X extends Throwable> T | Supplier<? extends X> exceptionSupplier | 若Optional中的value为空，则执行参数提供的函数，并抛出参数函数中指定的异常信息 |


### Optiona使用
1. Optiona对象的获取一共有三种方式，都是静态方法，同时Optional源码中将构造函数设置为私有，以及提供了一个静态常量 EMPTY来表示值为空的Optional对象。of 与 ofNullable的区别在于当给定的参数为null时，of会抛出NullPointException，而ofNullable则会返回一个空的Optional对象，对于希望避免处理NullPointException的建议还是使用ofNullable。
2. get是返回当前Optional对象中的value对象，一般需要配合isPresent() 或 isEmpty()先进行判断在做处理，否则还需要专门处理NoSuchElementException异常。
3. ifPresent值仅提供了当Optional中value不为null时的处理方法，而ifPresentOrElse则提供了两种处理方法，value不为空则执行comsumer方法，否则执行Runnable的emptyAction方法.

```
Person p1 = null;
        Optional<Person> pop2 = Optional.ofNullable(p1);
        pop2.ifPresent(person -> {System.out.println(person);});

        pop2.ifPresentOrElse(person -> {System.out.println(person);},() ->{System.out.println("the value is empty!");});
```

4. filter用于判断，若Optional中的value满足参数提供的predicate函数，这返回该Optional对象,否则返回value为空的Optional对象

```
Person person1 = new Person("kaithy",23);
        Person person2 = new Person("lucy",24);
        Person person3 = new Person("lili",25);

        List<Person> persons = new ArrayList<>();
        persons.add(person1);
        persons.add(person2);
        persons.add(person3);

        Optional<List<Person>> optional = Optional.ofNullable(persons);

        Optional<List<Person>> fop = optional.filter(people -> {return people.contains(person1);});
        //判断是否为同一个对象，结果显示为true
        System.out.println(fop == optional);
        System.out.println(fop.equals(optional));
```

5. map 与 flatMap 功能都是将Optional中的value执行参数提供的mapper函数，将value映射为一个新的值，并将新值用Optional包裹，若value为null，则会返回EMPTY。区别在于，若mapper函数返回的结果为null，map会返回一个EMPTY，但是flatMap则会抛出NullPointException。
（PS：(1).映射的后的值会用一个新的Optional对象包裹，而非更新原有的value值;
(2).所有value为null的Optional对象都是相同的，即Optional中提供的EMPTY这一静态常量）;

```
Person person1 = new Person("kaithy",23);
        Person person2 = new Person("lucy",24);

        Optional<Person> optional = Optional.ofNullable(person1);

        Optional<Person> mapOp = optional.map(person -> {
            return null;
        });

        System.out.println(mapOp.isPresent());
        
        Optional<Person> mapOp2 = optional.map(person -> {return null;});
        
        System.out.println(mapOp == mapOp2);
        
        Optional<Person> mapOp3 = optional.map(person -> {return person;});
        //mapper返回值相同，判断是否是同一个Optional对象
        System.out.println(optional == mapOp3);

        //会抛出NullPointException
        Optional<Person> flatMapOp = optional.flatMap(person -> {
            return null;
        });
```

6. or 中提供的参数用于当Optional中的value为null时，生成一个特定的Optional对象的方法，当value不为空，这直接返回当前的Optional对象，否则就用supplier函数生成一个Optional对象，若supplier为null或返回结果为null，会抛出NullPointException。

```
Person person1 = new Person("kaithy",23);
        Person person2 = new Person("lucy",24);

        Optional<Person> optional = Optional.ofNullable(person1);

        Optional<Person> mapOp = optional.map(person -> {
            return null;
        });

        mapOp = mapOp.or(()->{
            return Optional.ofNullable(person2);
        });

        System.out.println(mapOp.isPresent());

```

7. orElse、orElseGet、orElseThrow分别代表了一种获取当前Optional中value值的方式，若value为null，orElse返回参数中的值，orElseGet返回参数提供的函数的执行结果，orElseThrow这抛出异常，默认NullPointException，也可以在参数中指定要抛出的异常类型

```
Person person1 = new Person("kaithy",23);
        Person person2 = new Person("lucy",24);

        Optional<Person> optional = Optional.ofNullable(person1);

        Optional<Person> mapOp = optional.map(person -> {
            return null;
        });

        Person p1 = mapOp.orElse(person2);

        System.out.println(p1);

        Person p2 = mapOp.orElseGet(() ->{
            return new Person("laila",25);
        });

        System.out.println(p2);

        try {
            Person p3 = mapOp.orElseThrow(() ->{
                return new RuntimeException("empty value");
            });
            System.out.println(p3);
        }catch (RuntimeException e) {
            System.out.println(e.getMessage());
        }

```