##	类的种类
```java
/**
 *  Interface and AbstractClass
 *    类，只能继承一个类，但可实现多个接口。接口，可继承多个接口
 *    接口定义函数，抽象类提供默认实现
 *    自定义函数继承抽象类，实现多个接口
 *
 *  访问控制符(可见域)
 *          public      protected       default     private
 *    同类    yes           yes            yes         yes
 *    同包    yes           yes            yes         no
 *    子类    yes           yes            no          no
 *    异包    yes           no             no          no
 */
public class OutClass {
    private static String staticVar;
    private int normalAge;

    public static void staticMethod(){
        System.out.println("OutClass.staticMethod");
    }

    public void normalMethod(){
        System.out.println("OutClass.normalMethod");
    }

    /**
     * 静态内部类
     */
    static class StaticInnerClazz{
//        静态内部类，可访问其外部类的静态属性
        private String staticInnerVar = staticVar;
        public static String staticInnerVar2 = staticVar;
        /*
        静态内部类，不可访问其外部类非静态属性
        private int staticInnerAge = normalAge;
        */
        public void StaticInnerClazzMethod(){
//            静态内部类，可访问其外部类的静态方法
            staticMethod();
            /*
            静态内部类，不可访问非静态方法
            normalMethod();
            */
        }
    }
    /**
     * 成员内部类
     */
    class InnerClazz{
//        成员内部类，可访问其外部类静态属性
        private String innerVar = staticVar;
//        成员内部类，可访问其外部类非静态属性
        private int innerAge = normalAge;

        public void innerClazzMethod(){
//            成员内部类，可访问其外部类静态方法
            staticMethod();
//            成员内部类，可访问其外部类非静态方法
            normalMethod();
        }
    }

    public void methodTest() {
        /**
         * 局部内部类
         *  皆可访问其作用域内的，静态非静态的属性和方法
         */
        class LocalClazz{
            private String localVar=staticVar;
            private int localAge = normalAge;

            public void localMethod(){
                staticMethod();
                normalMethod();
            }
        }
//        静态内部类，可在该类外直接使用
        String ss = StaticInnerClazz.staticInnerVar2;
//        成员内部类，需要先new对象才可使用
        InnerClazz innerClazz = new InnerClazz();
        innerClazz.innerClazzMethod();
    }

    /**
     * 匿名内部类
     */
    private ACAnonymous aca = new ACAnonymous() {
        @Override
        void acaMethod() {
            System.out.println("抽象类的匿名内部类");
        }
    };

    private IAnonymous ia = new IAnonymous() {
        @Override
        public void iaMethod() {
            System.out.println("接口的匿名内部类");
        }
    };

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("使用匿名内部类的同时，又使用了接口匿名类");
            }
        }).start();
    }
}

abstract class ACAnonymous{
    abstract void acaMethod();
}

interface IAnonymous{
    void iaMethod();
}
```

##	类的初始化
```java
public class Grandpa {
    String normalVar = "hello,grandpa";
    public Grandpa() {
        System.out.println("Grandpa.Grandpa");
    }

    static {
        System.out.println("Grandpa.static initializer");
    }

    {
        System.out.println("Grandpa.instance initializer");
    }
}

public class Parent extends Grandpa{
    static String staticVar = "hello,parent";
    static{
        System.out.println("Parent.static initializer");
    }

    {
        System.out.println("Parent.instance initializer");
    }

    public Parent() {
        System.out.println("Parent.Parent");
    }
}

public class Child extends Parent {
    static {
        System.out.println("Child.static initializer");
    }

    {
        System.out.println("Child.instance initializer");
    }

    public Child() {
        System.out.println("Child.Child");
    }
}

public class MainApp {
    public static void main(String[] args) {
        /**
         * 创建一个子类对象时
         *   先从最高父类开始逐层调用静态初始化代码块
         *   再从最高父类开始逐层调用普通代码块和构造器
         */
        Child child = new Child();
        System.out.println("==================");
        /**
         * 子类访问其父类的静态属性
         *   从最高父类开始逐层调用父类的静态初始化
         *   子类本身的静态初始化并不调用
         * 子类访问自己的静态属性
         *   从最高父类开始逐层调用父类的静态初始化
         *   子类本身的静态初始化也会调用
         * 注意：
         *   静态初始化只会调用一次，再次使用时不会重复调用
         *   只调用静态属性并不会引起调用构造器和普通代码块
         */
        System.out.println(Child.staticVar);
    }
}
```
