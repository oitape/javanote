- Lambda 表达式执行主要是依靠 invokedynamic 的 JVM 指令来实现
- stream方法（简单方法忽略）
    - sorted：排序功能，允许自定义排序
    - peek：可以再peek里做没有返回值的事情
    ```java
     list.stream()
     .map(StudentDTO::getCode) 
     .peek(s -> log.info("当前循环的是{}",s)) .collect(Collectors.toList());
    ```
    - limit：限制输出的个数
    - findFirst：匹配到第一个满足条件的就返回
    ```java
    Long id = students.stream() 
    .filter(s->StringUtils.equals(s.getName(),"小美"))
    .findFirst() 
    .orElse(new StudentDTO()) //防止空指针 区别于 .get()方法
    .getId();
    ```
    - groupingBy：能够根据字段进行分组，toMap是吧List的数据格式转化成Map格式
    ```java
    // 学生根据名字进行分类
    Map<String, List<StudentDTO>> map = students
    .stream()    
    .collect(Collectors.groupingBy(StudentDTO::getName));
    ```
- 