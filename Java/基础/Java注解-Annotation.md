---
title: Java注解-Annotation
date: 2020-06-06 11:26:45
tags:
---

## 一、什么是注解

注解（Annotation），也叫元数据，一种代码级别的说明。JDK1.5引入的特性，与类、接口、枚举是在同一个层次，它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。**注解的本质就是一个继承了 Annotation 接口的接口**，所有的注解类型都继承自Annotation。

## 二、元注解

元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解：

### @Target：

**用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

@Target：表示该注解类型的所使用的程序元素类型。当注解类型声明中没有@Target元注解，则默认为可适用所有的程序元素。如果存在指定的@Target元注解，则编译器强制实施相应的使用限制。关于程序元素(ElementType)是枚举类型，共定义8种程序元素，如下表：

| ElementType     | 含义                               |
| :-------------- | :--------------------------------- |
| ANNOTATION_TYPE | 注解类型声明                       |
| CONSTRUCTOR     | 构造方法声明                       |
| FIELD           | 字段声明（包括枚举常量）           |
| LOCAL_VARIABLE  | 局部变量声明                       |
| METHOD          | 方法声明                           |
| PACKAGE         | 包声明                             |
| PARAMETER       | 参数声明                           |
| TYPE            | 类、接口（包括注解类型）或枚举声明 |

例如，上面源码@Target的定义中有一行`@Target(ElementType.ANNOTATION_TYPE)`，意思是指当前注解的元素类型是注解类型。

```
public enum ElementType {
    /** 表示可以用来修饰类、接口、注解类型或枚举类型Class, interface (including annotation type), or enum declaration */
    TYPE,
    /** 可以用来修饰属性（包括枚举常量）Field declaration (includes enum constants) */
    FIELD,
    /** 可以用来修饰方法Method declaration */
    METHOD,
    /** 可以用来修饰参数 Formal parameter declaration */
    PARAMETER,
    /** 可以用来修饰构造器 Constructor declaration */
    CONSTRUCTOR,
    /** 可用来修饰局部变量Local variable declaration */
    LOCAL_VARIABLE,
    /** 可以用来修饰注解类型Annotation type declaration */
    ANNOTATION_TYPE,
    /** 可以用来修饰包Package declaration */
    PACKAGE,
    /**
     * 标注在类型参数上Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,
    /**
     * 用于标注任意类型(不包括class)Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```



### @Retention：

**定义了`Annotation`的生命周期**

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}
```

- 仅编译期：`RetentionPolicy.SOURCE`；

- 仅class文件：`RetentionPolicy.CLASS`；

- 运行期：`RetentionPolicy.RUNTIME`。注解信息将在运行期(JVM)也保留，因此可以通过反射机制读取注解的信息（源码、class文件和执行的时候都有注解的信息）

  **如果`@Retention`不存在，则该`Annotation`默认为`CLASS`。因为通常我们自定义的`Annotation`都是`RUNTIME`，所以，务必要加上`@Retention(RetentionPolicy.RUNTIME)`这个元注解：**

### @Documented：

**用来描述注解是否被抽取到api文档中。在生成javadoc文档的时候将该Annotation也写入到文档中。**

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```

@Documented：表示拥有该注解的元素可通过javadoc此类的工具进行文档化。该类型应用于注解那些影响客户使用带注释(comment)的元素声明的类型。如果类型声明是用Documented来注解的，这种类型的注解被作为被标注的程序成员的公共API。

例如，上面源码@Retention的定义中有一行`@Documented`，意思是指当前注解的元素会被javadoc工具进行文档化，那么在查看Java API文档时可查看当该注解元素。

### @Inherited：

**如果一个使用了`@Inherited`修饰的`annotation`类型被用于一个class，则这个`annotation`将被用于该class的子类。并且仅针对`class`的继承，对`interface`的继承无效：**

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

@Inherited：表示该注解类型被自动继承，如果用户在当前类中查询这个元注解类型并且当前类的声明中不包含这个元注解类型，那么也将自动查询当前类的父类是否存在Inherited元注解，这个动作将被重复执行知道这个标注类型被找到，或者是查询到顶层的父类。

### @Repeatable：

它表示在同一个位置重复相同的注解。在没有该注解前，一般是无法在同一个类型上使用相同的注解的。

## 三、常见注解

- @Override：用于标记该方法是复写的父类中的某个方法。属于标记注解，不需要设置属性值；只能添加在方法的前面。

- @Deprecated：说明被修饰内容已被“废弃”，不再建议用户使用。

- @SuppressWarnings：忽略警告。

- @TargetApi：版本注解。

- @SuppressLint：避免在lint检查时报错

- **Nullable**：用于标记方法参数或者返回值可以为空；

- **@NonNull:**用于标记方法参数或者返回值不能为空，如果为空编译器会报警告；

- @IdRes：Android中一系列**资源类型注解**。

- 使用整型常量代替枚举类型

  ```
  public class IceCreamFlavourManager {
      private int flavour;
  
      public static final int VANILLA = 0;
      public static final int CHOCOLATE = 1;
      public static final int STRAWBERRY = 2;
  
      @IntDef({VANILLA, CHOCOLATE, STRAWBERRY})
      public @interface Flavour {
      }
  
      @Flavour
      public int getFlavour() {
          return flavour;
      }
  
      public void setFlavour(@Flavour int flavour) {
          this.flavour = flavour;
      }
  }
  ```

- @UiThread :等线程注解

- @Size、@IntRange、@FloatRange等值约束注解。

- 权限注解

- @CheckResult：**返回值注解**。

## 四、自定义注解

### 注解定义

```
public @interface 注解名 {定义体}


@Documented
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
	String value() default "";
}
```



### 使用注解

#### 编译时期使用注解

编译时注解指的是@Retention的值为CLASS的注解。对于这类注解的解析，我们只需做好以下两件事儿：

- 自定义一个派生自 AbstractProcessor的“注解处理类”；
- 重写process 函数。

实际上，javac中包含的注解处理器在编译时会自动查找所有继承自 AbstractProcessor 的类，然后调用它们的 process 方法。因此我们只要做好上面两件事，编译器就会主动去解析我们的编译时注解。现在，我们把上面定义的TestAnnotation的Retention改为CLASS，我们就可以按照以下代码来解析它：

```
@SupportedAnnotationTypes("com.zhyen.com.TestAnnotation")
public class TestAnnotationClass extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        HashMap<String, String> map = new HashMap<>();
        for (TypeElement element : annotations) {
            for (Element e : roundEnv.getElementsAnnotatedWith(element)) {
                TestAnnotation annotation=e.getAnnotation(TestAnnotation.class);
                map.put(e.getEnclosingElement().toString(),annotation.value());
            }
        }
        return false;
    }
}
```

SupportedAnnotationTypes注解指出了MyProcessor向要解析的注解的完整名字（全限定名称）。process+函数的annotations参数表示待处理的注解集，通过env我们可以得到被特定注解所修饰的程序元素。process函数的返回值表示annotations中的注解是否被这个Processor接受。&oq=%40SupportedAnnotationTypes注解指出了MyProcessor向要解析的注解的完整名字（全限定名称）。process+函数的annotations参数表示待处理的注解集，通过env我们可以得到被特定注解所修饰的程序元素。process函数的返回值表示annotations中的注解是否被这个Processor接受

#### 运行时期使用直接

Class 类中提供了以下一些方法用于反射注解：

- getAnnotation：返回指定的注解

- isAnnotationPresent：判定当前元素是否被指定注解修饰

- getAnnotations：返回所有的注解

- getDeclaredAnnotation：返回本元素的指定注解

- getDeclaredAnnotations：返回本元素的所有注解，不包含父类继承而来的

**方法、字段中相关反射注解的方法基本是类似的**

## 五、小案例一个

注解类：

```
@Documented
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
    String value() default "null";

    public String name() default " ";

    int index() default 1;
}

```

使用类：

```
public class TestAnnotationClass {

    @TestAnnotation(value = "汪汪汪", name = "dog", index = 100)
    public String dog;
    @TestAnnotation(value = "喵喵喵", name = "cat", index = 200)
    public String cat;

    @TestAnnotation(value = "爱爆炸", index = 10, name = "三星")
    public void phone() {

    }

    @TestAnnotation(value = "细", index =20, name = "ARM")
    public void CPU() {

    }
}
```

获取注解:

```
Class<TestAnnotationClass> clazz = TestAnnotationClass.class;
        Field[] fields = clazz.getFields();
        for (Field field : fields) {
            TestAnnotation annotation = field.getAnnotation(TestAnnotation.class);
            String name = annotation.name();
            int index = annotation.index();
            String value = annotation.value();
            Log.d(TAG, "name = " + name + " ,index = " + index + ", value = " + value);
        }

        TestAnnotation annotation = method.getAnnotation(TestAnnotation.class);
            String name = annotation.name();
            int index = annotation.index();
            String value = annotation.value();
            Log.d(TAG, "name = " + name + " ,index = " + index + ", value = " + value);
```

结果:

```
name = cat ,index = 200, value = 喵喵喵
name = dog ,index = 100, value = 汪汪汪
方法名: CPU 
name = ARM ,index = 20, value = 细
equals
方法名: getClass
方法名: hashCode
方法名: notify
方法名: phone
name = 三星 ,index = 10, value = 爱爆炸
...
```

