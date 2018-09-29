# Dozer

Java的Bean映射工具，用于对象转换。

常见的场景就是VO、DTO、PO的互相转换，能进行简单的字段转换处理加工。

支持XMl、注解、API的形式来使用，本文主要介绍基于XML形式的使用

## 安装使用

- 引入Maven依赖
    ```XML
    <dependency>
        <groupId>com.github.dozermapper</groupId>
        <artifactId>dozer-core</artifactId>
        <version>6.4.0</version>
    </dependency>
    ```
- 在Resource文件中引入
    1. dozerBeanMapping.xml（处理映射的主要文件，负责处理映射关系）
    2. dozer-global-configuration.xml（dozer全局配置文件）
- 在Spring配置中声明bean：DozerBeanMapper
    ```Java
    @Bean
    public DozerBeanMapper dozerBeanMapper() {
        DozerBeanMapper mapper = new DozerBeanMapper();
        ArrayList<String> mappingFileUrls = Lists.newArrayList();
        mappingFileUrls.add("dozer-mapper/dozer-global-configuration.xml");
        mappingFileUrls.add("dozer-mapper/bean-mapping.xml");
        mapper.setMappingFiles(mappingFileUrls);
        return mapper;
    }
    ```
- 之后就可以直接使用注入的形式来直接调用dozer
    ```Java
    @Autowired
    private DozerBeanMapper dozerBeanMapper;

    public void dozerTest(CampaignVO vo){
        CampaignDTO dto = dozerBeanMapper.map(vo,dto);
    }
    ```

## 范例

```XMl
 <mapping>
        <!-- 此处为配置需要转换的两个对象，不讲究顺序-->
        <class-a>com.meili.UserShopCouponVO</class-a>
        <class-b>com.meili.SendingShopCouponDTO</class-b>
        <!-- dozer默认转换同名的属性，如果不同名可在此处特别注明-->
        <!-- 如果两个字段需要进行一些加工，可以自定义converter类-->
        <field custom-converter="com.meili.xiaodian.promotion.utils.converter.DoublePriceConverter">
            <!--需要转换属性名-->
            <a>limitPrice</a>
            <b>limitPrice</b>
        </field>
        <field>
            <a>id</a>
            <b>shopId</b>
        </field>
    </mapping>
```

## 同名字段会自动进行映射，无需配置；不同名字段，默认不映射，需要配置

## 指定属性映射（常用于不同名字段）

```XML
<field>
    <a>id</a>
    <b>shopId</b>
</field>
```

## 排除某些属性

1. 使得某些属性不参与映射
    ```XMl
    <field-exclude>
        <a>fieldToExclude</a>
        <b>fieldToExclude</b>
    </field-exclude>
    ```
2. 使得空值不参与映射
    ```XML
    <mapping map-null="false">
        <class-a>com.github.dozermapper.core.vo.AnotherTestObject</class-a>
        <class-b>com.github.dozermapper.core.vo.AnotherTestObjectPrime</class-b>
        <field>
            <a>field4</a>
            <b>to.one</b>
        </field>
    </mapping>
    ```
3. 使得空字符串不参与映射
    ```XML
    <mapping map-empty-string="false">
        <class-a>com.github.dozermapper.core.vo.AnotherTestObject</class-a>
        <class-b>com.github.dozermapper.core.vo.AnotherTestObjectPrime</class-b>
        <field>
            <a>field4</a>
            <b>to.one</b>
        </field>
    </mapping>
    ```

## 继承映射：当存在继承情况时

当存在继承情况，先声明父类之间的转换，再声明子类的转换。这样就不用每次针对子类，复制父类的代码了

```XML
<!--优化前-->
<mapping>
    <class-a>com.github.dozermapper.core.vo.SubClass</class-a>
    <class-b>com.github.dozermapper.core.vo.SubClassPrime</class-b>
    <field>
        <!-- this is the same for all sub classes -->
        <a>superAttribute</a>
        <b>superAttr</b>
    </field>
    <field>
        <a>attribute2</a>
        <b>attributePrime2</b>
    </field>
</mapping>
<!--子类每次都复制父类的映射，不利于维护，还需要抄代码-->
<mapping>
    <class-a>com.github.dozermapper.core.vo.SubClass2</class-a>
    <class-b>com.github.dozermapper.core.vo.SubClassPrime2</class-b>
    <field>
        <!-- this is the same for all sub classes -->
        <a>superAttribute</a>
        <b>superAttr</b>
    </field>
    <field>
        <a>attribute2</a>
        <b>attributePrime2</b>
    </field>
</mapping>
```

```XML
<!--优化后-->
<mapping>
    <class-a>com.github.dozermapper.core.vo.SuperClass</class-a>
    <class-b>com.github.dozermapper.core.vo.SuperClassPrime</class-b>
    <field>
        <a>superAttribute</a>
        <b>superAttr</b>
    </field>
</mapping>
<!--子类每次只需要负责自己的映射即可-->
<mapping>
    <class-a>com.github.dozermapper.core.vo.SubClass</class-a>
    <class-b>com.github.dozermapper.core.vo.SubClassPrime</class-b>
    <field>
        <a>attribute</a>
        <b>attributePrime</b>
    </field>
</mapping>
<mapping>
    <class-a>com.github.dozermapper.core.vo.SubClass2</class-a>
    <class-b>com.github.dozermapper.core.vo.SubClassPrime2</class-b>
    <field>
        <a>attribute2</a>
        <b>attributePrime2</b>
    </field>
</mapping>
```

## 单向映射

当有特殊需求，只允许A->B的单项映射时（粒度可以为整个类或者某些属性），只需要配置“one-way“属性

```XML
<!--类级别的单向映射-->
<mapping type="one-way">
    <class-a>com.github.dozermapper.core.vo.TestObjectFoo</class-a>
    <class-b>com.github.dozermapper.core.vo.TestObjectFooPrime</class-b>
    <field>
        <a>oneFoo</a>
        <b>oneFooPrime</b>
    </field>
</mapping>

<mapping>
    <class-a>com.github.dozermapper.core.vo.TestObjectFoo2</class-a>
    <class-b>com.github.dozermapper.core.vo.TestObjectFooPrime2</class-b>
    <!--字段级别的单向映射-->
    <field type="one-way">
        <a>oneFoo2</a>
        <b>oneFooPrime2</b>
    </field>
    <!--当字段A->B映射，将被排除；B->A就会加入映射-->
    <field-exclude type="one-way">
        <a>fieldToExclude</a>
        <b>fieldToExclude</b>
    </field-exclude>
</mapping>
```

## 范型映射

由于范型在运行时会有范型擦除的操作，所以范型进行映射的时候需要指定范型类型（常见如List）

```XML
<field>
    <a>aList</a>
    <b>bList</b>
    <a-hint>B1,B2</a-hint>
    <b-hint>BPrime1,BPrime2</b-hint>
</field>
```

## 底层属性映射：当存在类嵌套的情况

以个人信息类为例

```Java
@Data
public class PersonVO{
    private Integer age;
    private String name;
    //籍贯信息，其中有比较多的复杂信息，如身份证，出生地，父母身份证号等
    private Home home;
}
@Data 
public class Home{
    private String personId;
    private List<String> parentIds;
}
@Data
public class PersonDTO{
    private Integer age;
    private String name;
    private Integer personId;
    private List<Integer> parentIds;
}
```

```XML
<mapping>
    <class-a>com.github.PersonVO</class-a>
    <class-b>com.github.PersonDTO</class-b>
    <!--字段名（类）.字段名-->
    <field>
        <a>home.personId</a>
        <b>personId</b>
    </field>
    <field>
        <!-- java.util.List to java.util.List -->
        <!-- 范型映射，需要指定范型的属性-->
        <a>home.parentIds</a>
        <b>parentIds</b>
        <a-hint>java.lang.String</a-hint>
        <b-hint>java.lang.Integer</b-hint>
    </field>
</mapping>
```

## 特定属性的自定义转化

当某些属性需要进行一些操作以后才能转换，如id的加解密。这个时候就可以自定义一个Converter

```Java
//配置自定义Converter类
import com.github.dozermapper.core.DozerConverter;

public class NewDozerConverter extends DozerConverter<String, Boolean> {

    public NewDozerConverter() {
        super(String.class, Boolean.class);
    }

    @Override
    public Boolean convertTo(String source, Boolean destination) {
        if ("yes".equals(source)) {
            return Boolean.TRUE;
        } else if ("no".equals(source)) {
            return Boolean.FALSE;
        }
        throw new IllegalStateException("Unknown value!");
    }

    @Override
    public String convertFrom(Boolean source, String destination) {
        if (Boolean.TRUE.equals(source)) {
            return "yes";
        } else if (Boolean.FALSE.equals(source)) {
            return "no";
        }
        throw new IllegalStateException("Unknown value!");
    }
}
```

```XML
<mapping>
    <class-a>com.github.dozermapper.core.vo.BeanA</class-a>
    <class-b>com.github.dozermapper.core.vo.BeanB</class-b>
    <!--使用自定义的转换工具-->
    <field custom-converter="com.github.dozermapper.core.converters.NewDozerConverter">
        <a>amount</a>
        <b>amount</b>
    </field>
</mapping>
```

## 其他的小特性

1. 支持给mapping映射设置map-id

    ```XML
    <mapping map-id="caseA">
        <class-a>com.github.dozermapper.core.vo.context.ContextMapping</class-a>
        <class-b>com.github.dozermapper.core.vo.context.ContextMappingPrime</class-b>
        <field-exclude>
            <a>loanNo</a>
            <b>loanNo</b>
        </field-exclude>
    </mapping>
    ```

    ```java
    ContextMappingPrime cmpA = mapper.map(cm, ContextMappingPrime.class, "caseA");
    ```

2. 支持指定数组的index映射

    ```XML
    <field>
        <a>offSpringName</a>
        <b>pets[1].offSpring[2].petName</b>
    </field>
    ```
