#   Exception Handler
Both `Error` and `RuntimeException` are unchecked exceptions.
they indicate a flaw with the program,and usually should not be caught.
##  Exception
![throwable](https://github.com/powar2sun/Note/blob/master/Language/pictures/throwable.png)

### Example
```java
public class DemoException extends Exception {
    public DemoException() {
    }

    public DemoException(String message) {
        super(message);
    }
}

public class TestException {
    /**
     * 抛出一个受检查的异常，上抛后仍须处理
     * @throws DemoException
     */
    public void method1() throws DemoException {
//        针对这些不同的Checked Exception 做出不同的处理
        throw new DemoException("Test Exception");
    }

    /**
     * 抛出一个运行时异常，上抛后可捕可抛可放
     * @param msg
     */
    public void method2(String msg) {
        if (msg == null) {
//            暗示着程序上的错误，而造成程序无法继续执行下去
            throw new NullPointerException("the message is null");
        }
    }

    /**
     * 捕获了一个受检查的异常，无须上抛进行处理
     */
    public void method3() {
        try {
            method1();
        } catch (DemoException e) {
            e.printStackTrace();
        }
    }
}

```
### NULL Handler -- Optional
A container object which may or may not contain a non-null value. 
If a value is present, isPresent() returns true and get() returns the value

```java
import java.util.Optional;

public class Demo {
    public Optional<String> say() {
        return Optional.of("hello");
    }

    public void test() {
        Optional<String> say = say();
        String s = say.get();
    }
}
```

Optional主要用作方法返回类型，其中明确需要表示“无结果”，以及使用null可能导致错误的地方。
类型为Optional的变量本身不应该为null;它应该始终指向一个可选实例
