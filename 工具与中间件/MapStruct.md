# 为何要使用MapStruct
在一个成熟的工程中，尤其是现在的分布式系统中，应用与应用之间，还有单独的应用细分模块之后，DO 一般不会让外部依赖，这时候需要在提供对外接口的模块里放 DTO 用于对象传输，也即是 DO 对象对内，DTO对象对外，DTO 可以根据业务需要变更，并不需要映射 DO 的全部属性。
有一种通用的映射工具可以帮助我们解决在建立DO、DTO对象映射关系时遇到的嵌套、类型转换等问题。这就是MapStruct。在大多数情况下，和其他的对象映射框架对比，MapStruct的性能最高。
原理类似于lombok，MapStruct都是在编译期进行实现，而且基于*Getter、Setter*，**由于没有使用反射所以一般不存在运行时性能问题。**

# Spring Boot 集成 MapStruct
###### 在 Spring Boot 的 pom.xml 下引入 MapStruct 的 maven 依赖坐标:
```
         <dependencies>
            <dependency>
                 <groupId>org.mapstruct</groupId>
                 <artifactId>mapstruct</artifactId>
                 <version>${mapstruct.version}</version>
                 <scope>compile</scope>
             </dependency>
             <dependency>
                 <groupId>org.mapstruct</groupId>
                 <artifactId>mapstruct-processor</artifactId>
                 <version>${mapstruct.version}</version>
                 <scope>compile</scope>
             </dependency>
                   <!-- other dependencies -->
         </dependencies>
```
# MapStruct的使用
### 转换源到目标的映射
```
//声明一个映射接口用@org.mapstruct.Mapper （不要跟mybatis注解混淆）标记，说明这是一个实体类型转换接口
@Mapper
 public interface CarMapping {
     /**
      * 用来调用实例 实际开发中可使用注入Spring  不写
      */
     CarMapping CAR_MAPPING = Mappers.getMapper(CarMapping.class);
 
 
     /**
      *  源类型 目标类型 成员变量相同类型 相同变量名 不用写{@link org.mapstruct.Mapping}来映射
      *
      * @param car the car
      * @return the car dto
      */
     @Mapping(target = "type", source = "carType.type")
     @Mapping(target = "seatCount", source = "numberOfSeats")
     CarDTO carToCarDTO(Car car);
 
 }
```
- source 代表转换的源
- target 代表转换的目标

在进行转换时以成员变量的参数名为依据，如果有嵌套比如 Car 里面有个 CarType 类型的成员变量 carType，其 type 属性 来映射 CarDTO 中的 type 字符串，我们使用 type.type 来获取属性值。如果有多层以此类推。MapStruct 最终调用的是 setter 和 getter 方法，而非反射。这也是其性能比较好的原因之一。
若源对象属性与目标对象属性名字一致，会自动映射对应属性，不一样的需要指定，也可以用 format 转成自己想要的类型，也支持表达式的方式。如果某个属性你不想映射，可以加个 ignore=true。
再补充一个基本映射的例子。下面是一个 Mapper 接口 PersonConverter，其中有两个方法，一个是单实体映射，另一个是List映射。
这里定义两个 DO 对象 Person 和 User，其中 user 是 Person 的一个属性 ，一个 DTO 对象 PersonDTO.
```
@NoArgsConstructor
@AllArgsConstructor
@Data
public class Person {
    private Long id;
    private String name;
    private String email;
    private Date birthday;
    private User user;
}

@NoArgsConstructor
@AllArgsConstructor
@Data
public class User {
    private Integer age;
}

@NoArgsConstructor
@AllArgsConstructor
@Data
public class PersonDTO {
    private Long id;
    private String name;
    /**
     * 对应 Person.user.age
     */
    private Integer age;
    private String email;
    /**
     * 与 DO 里面的字段名称(birthDay)不一致
     */
    private Date birth;
    /**
     * 对 DO 里面的字段(birthDay)进行拓展,dateFormat 的形式
     */
    private String birthDateFormat;
    /**
     * 对 DO 里面的字段(birthDay)进行拓展,expression 的形式
     */
    private String birthExpressionFormat;

}
```
对应的Mapper接口：
```
@Mapper
public interface PersonConverter {
    PersonConverter INSTANCE = Mappers.getMapper(PersonConverter.class);
    @Mappings({
        @Mapping(source = "birthday", target = "birth"),
        @Mapping(source = "birthday", target = "birthDateFormat", dateFormat = "yyyy-MM-dd HH:mm:ss"),
        @Mapping(target = "birthExpressionFormat", expression = "java(org.apache.commons.lang3.time.DateFormatUtils.format(person.getBirthday(),\"yyyy-MM-dd HH:mm:ss\"))"),
        @Mapping(source = "user.age", target = "age"),
        @Mapping(target = "email", ignore = true)
    })
    PersonDTO domain2dto(Person person);

    List<PersonDTO> domain2dto(List<Person> people);
}
```

### 设置转换默认值和常量
当目标值是 null 时我们可以设置其默认值，注意这些都是基本类型以及对应的boxing 类型。
```
@Mapping(target = "stringProperty", source = "stringProp", defaultValue = "undefined")
```
需要注意的是常量不能对源进行引用（不能指定 source 属性），下面是正确的操作:
```
@Mapping(target = "stringConstant", constant = "Constant Value")
```
### 多对一
MapStruct 可以将几种类型的对象映射为另外一种类型，比如将多个 DO 对象转换为 DTO
例子：
- 两个 DO 对象 Item 和 Sku，一个 DTO 对象 SkuDTO
```
@NoArgsConstructor
@AllArgsConstructor
@Data
public class Item {
    private Long id;
    private String title;
}

@NoArgsConstructor
@AllArgsConstructor
@Data
public class Sku {
    private Long id;
    private String code;
    private Integer price;
}

@NoArgsConstructor
@AllArgsConstructor
@Data
public class SkuDTO {
    private Long skuId;
    private String skuCode;
    private Integer skuPrice;
    private Long itemId;
    private String itemName;
}
```
- 创建 ItemConverter（映射）接口，MapStruct 就会自动实现该接口
```
@Mapper
public interface ItemConverter {
    ItemConverter INSTANCE = Mappers.getMapper(ItemConverter.class);

    @Mappings({
            @Mapping(source = "sku.id",target = "skuId"),
            @Mapping(source = "sku.code",target = "skuCode"),
            @Mapping(source = "sku.price",target = "skuPrice"),
            @Mapping(source = "item.id",target = "itemId"),
            @Mapping(source = "item.title",target = "itemName")
    })
    SkuDTO domain2dto(Item item, Sku sku);
}
```
# Mapper 编译
当你的应用编译后。你会在编译后的目录比如 maven是 `target\generated-sources\annotations` 下的子目录发现生成了一个实现类。
# 进阶操作
### 格式化操作
处理数字格式化的操作，遵循java.text.DecimalFormat的规范:
```
@Mapping(source = "price", numberFormat = "$#.00")
```
将一个日期集合映射到日期字符串集合的格式化操作:
```
 @IterableMapping(dateFormat = "dd.MM.yyyy")
 List<String> stringListToDateList(List<Date> dates);
```
### 使用Java表达式
使用`LocalDateTime` 作为当前的时间值注入 `addTime` 属性中。
首先在`@org.mapstruct.Mapper `的` imports `属性中导入` LocalDateTime`，该属性是数组意味着你可以根据需要导入更多的处理类：
```
 @Mapper(imports = {LocalDateTime.class})
```
接下来只需要在对应的方法上添加注解`@org.mapstruct.Mapping` ，其属性`expression` 接收一个` java()` 包括的表达式：
- 无入参版本：
```
@Mapping(target = "addTime", expression = "java(LocalDateTime.now())")
```
- 携带入参的版本 
将 `Car` 的出厂日期字符串`manufactureDateStr `注入到 `CarDTO `的 `LocalDateTime` 类型属性`addTime` 中去
```
@Mapping(target = "addTime", expression = "java(LocalDateTime.parse(car.manufactureDateStr))")
 CarDTO carToCarDTO(Car car);
```
# MapStruct 转换 Mapper 注入Spring IoC 容器
如果使用要把Mapper 注入Spring IoC 容器我们只需要这么声明，不用再构建一个单例，就可以像其他 spring bean一样对CarMapping 进行引用了：
```
@Mapper(componentModel = "spring")
public interface UserConvertor {

    @Mappings({
            @Mapping(target = "name",source = "name"),
            @Mapping(target = "score",source = "score")
    })
    UserVO convert2VO(User user);
    List<UserVO> convert2ListVO(List<User> list);
}
```
# MapStruct 注解的关键词
> @Mapper 只有在接口加上这个注解， MapStruct 才会去实现该接口
    @Mapper 里有个 componentModel 属性，主要是指定实现类的类型，一般用到两个
    default：默认，可以通过 Mappers.getMapper(Class) 方式获取实例对象
    spring：在接口的实现类上自动添加注解 @Component，可通过 @Autowired 方式注入
@Mapping：属性映射，若源对象属性与目标对象名字一致，会自动映射对应属性
    source：源属性
    target：目标属性
    dateFormat：String 到 Date 日期之间相互转换，通过 SimpleDateFormat，该值为 SimpleDateFormat              的日期格式
    ignore: 忽略这个字段
@Mappings：配置多个@Mapping
@MappingTarget 用于更新已有对象
@InheritConfiguration 用于继承配置
# 参考文章
[推荐一个 Java 实体映射工具 MapStruct](https://blog.csdn.net/zhige_me/article/details/80699784)

[Spring Boot 2 实战：集成 MapStruct 类型转换神器](https://segmentfault.com/a/1190000020663215)
[官方文档](https://mapstruct.org/documentation/stable/reference/html/)



