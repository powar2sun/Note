#	Collection Maps
![collectionMap](https://github.com/powar2sun/Note/blob/master/Language/pictures/collectionMap.png)
##	Collection
![collection](https://github.com/powar2sun/Note/blob/master/Language/pictures/collection.png)
##	Map
![map](https://github.com/powar2sun/Note/blob/master/Language/pictures/map.png)
##	Point
![points](https://github.com/powar2sun/Note/blob/master/Language/pictures/points.png)
##	Array
*	One-Dimensional Array

```
// create
Type[] var = new T[size_num];
//create and initial
T[] vars = new T[]{one,two,three};

```

*	Two-Dimensional Array

```
//create
T[][] varT = new T[size_must][size_opts];
//create and initial
T[][] varTs = new T[][]{{one,two},{single},{three,four,five}};
```

#####	Markdown 连接与脚注 语法
*	超链接

```
//内联方式
xxx[example](http://xxx.com)
//引用方式
[one][1] xxx [two][2] xxx [three][3]

[1]: url
[2]: http://xxx.com
[3]: url
```

*	脚注

```
[^jiaozhu]: http://xxx.com
```
#	Map Detail
####	HashMap
*	它根据键的hashCode值存储数据


#   HashCode() Best Practice
```java
public class SimpleHashCode{
    private static long counter=0;
    private static long id = counter++;
    private String name;
    
    @Override
    public int hashCode(){
//非零的常数值    
        int result = 17;
//为该域计算int类型的散列码c
/*
* boolean域：(f ? 1:0)
* String域：f.hashCode()
* byte,char,short,int域：(int)f
* float域：Float.floatToIntBits(f)
* double域：Double.doubleToIntBits(f)
* 引用域：递归调用该引用各个域的hashCode()
* 数组域：每个元素当成单独域处理
* */        
        if(name != null){
            result = 31*result+name.hashCode();
        }
        result=result*31+(int)id;
        return result;
    }
    @Override
    public boolean equals(Object o){
        return o instanceof SimpleHashCode && id ==((SimpleHashCode)o).id;
    }
}
```




















