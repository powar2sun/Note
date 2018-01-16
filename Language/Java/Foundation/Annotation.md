#   Annotation 注解
![annotation1](https://github.com/powar2sun/Note/blob/master/Language/pictures/annotation1.png)
![annotation2](https://github.com/powar2sun/Note/blob/master/Language/pictures/annotation2.png)
##  Meta Annotation 元注解
![metaAnnotation](https://github.com/powar2sun/Note/blob/master/Language/pictures/metaAnnotation.png)
##  自定义注解
```java
import java.lang.annotation.*;

@Target(ElementType.FIELD)
/**
 * 保留策略选定 运行时
 *  利于一些框架借助反射API进行处理
 */
@Retention(RetentionPolicy.RUNTIME)
@Documented
/**
 * 自定义注解
 *  @interface
 *      指明一个注解，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节
 *  每个method
 *      指明一个配置参数
 *      访问修饰符只能是：public 或 default(默认)
 *      返回指-配置参数的类型
 *      方法名-配置参数的名称
 *      若只有一个配置项，最好把名称设为 value
 *  配置参数支持的类型
 *      基本数据类型
 *      String
 *      Class
 *      enum
 *      Annotation
 *      以上类型的数组[]
 *  default
 *      指明，若无参数值所采用的默认值
 */
public @interface DemoAnnotation {
    String name() default "";
    int age() default -1;
    public enum SomeHobby{MUSIC,VIDEO,MOVIE}
    SomeHobby hobby() default SomeHobby.MUSIC;
}
```
