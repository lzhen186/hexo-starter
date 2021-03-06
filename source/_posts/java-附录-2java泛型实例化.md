---
title: Java 附录 2Java泛型实例化
date: 2022-07-06T01:52:31.268Z
tags: [javase]
---
# 实例化具有无参数构造函数的泛型对象

```java
//newInstance() method need constructor without parameter
//Class<T> come form Class.class
public <T> T getNewObject(Class<T> cls) {
    T t=null;
    try {
        t = cls.newInstance();
    } catch (InstantiationException|IllegalAccessException e) {
        e.printStackTrace();
    }
    return t;
}
```

调用

```java
String i =getNewObject(String.class);
```

这种方法需要泛型类具有一个无参数构造函数

# 实例化没有无参数构造函数的泛型对象

```java
//newInstance() method need constructor with parameter
public <T> T getNewObject(Constructor<T> cls, double d) {
    T t = null;
    try {
        t = cls.newInstance(d);
    } catch (InstantiationException | IllegalAccessException
             | IllegalArgumentException | InvocationTargetException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }

    return t;
}
```

调用

```java
con = Float.class.getConstructor(double.class);
Float k =getNewObject(con,10.0);
```

这种方法先确定使用泛型类的哪一个构造函数，再通过该构造函数newInstance实例出来。

通用的实例泛型对象（无需区别是否有无参数构造函数）

# 通过反射动态创建泛型实例

```java
public class BasePresenter<V extends BaseView,M extends BaseModel>{

    private M mModel;

    public void attach(){

          //1、返回表示此 Class 所表示的实体（类、接口、基本类型或 void）的直接超类的 Type
        Type genType = getClass().getGenericSuperclass();
        //2、泛型参数
        Type[] types = ((ParameterizedType) genType).getActualTypeArguments();
        //3、因为BasePresenter 有两个泛型 数组有两个
        try {
            //
            mModel= (M) ((Class)types[1]).newInstance();
            //这里需要强转得到的是实体类类路径
//            如果types[1].getClass().newInstance();并不行，得到的是泛型类型
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    public M getModel(){

        return mModel;
    }

}
```

## getSuperclass和 getGenericSuperclass的区别

- getSuperclass返回直接继承的父类不包括泛型参数
- getGenericSuperclass返回直接继承的父类包含泛型参数

## getInterfaces 和 getGenericInterface 的区别

- getInterfaces 返回直接实现的接口（不显示泛型参数）
- getGenericInterface 返回直接实现的接口（显示泛型参数）

## 封装成工具类

```java
public class ReflectionUtil {

    /**
     * 通过type获取className
     */

    public static String getClassName(Type type){
        if(type==null){

            return "";
        }
        String className = type.toString();

        if (className.startsWith("class")){
            className=className.substring("class".length());
        }
        return className;
    }

    /**
     * 获取子类在父类传入的泛型Class类型
     * 获取泛型对象的泛型化参数
     * @param o
     * @return
     */
    public static Type getParameterizedTypes(Object o){
        Type superclass = o.getClass().getGenericSuperclass();
        if(!ParameterizedType.class.isAssignableFrom(superclass.getClass())) {
            return null;
        }
        Type[] types = ((ParameterizedType) superclass).getActualTypeArguments();
        return types[0];
    }

    /**
     * 获取实现类的泛型参数
     * @param o
     * @return
     */

    public static Type getInterfaceTypes(Object o){
        Type[] genericInterfaces = o.getClass().getGenericInterfaces();

        return genericInterfaces[0];
    }

    /**
     *检查对象是否存在默认构造函数
     */

    public static boolean hasDefaultConstructor(Class<?> clazz) throws SecurityException {
        Class<?>[] empty = {};
        try {
            clazz.getConstructor(empty);
        } catch (NoSuchMethodException e) {
            return false;
        }
        return true;
    }
    /**
     * 通过Type创建对象
     */
    public static Object newInstance(Type type)
            throws ClassNotFoundException, InstantiationException, IllegalAccessException {
        Class<?> clazz = getClass(type);
        if (clazz==null) {
            return null;
        }
        return clazz.newInstance();
    }


    /**
     * 通过Type获取对象class
     * @param type
     * @return
     * @throws ClassNotFoundException
     */

    public static Class<?> getClass(Type type)
            throws ClassNotFoundException {
        String className = getClassName(type);
        if (className==null || className.isEmpty()) {
            return null;
        }
        return Class.forName(className);
    }


}
```

