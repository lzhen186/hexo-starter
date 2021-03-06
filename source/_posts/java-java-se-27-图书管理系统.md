---
title: Java Java SE 27.图书管理系统
date: 2022-07-04T12:15:54.542Z
tags: [javase]
---
# 1.图书管理系统项目演示

项目目录：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143223.png)

主界面和选择：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143122.png)



查看书籍：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143437.png)

添加书籍：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143557.png)

删除书籍：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143724.png)

修改书籍：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143923.png)

退出：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614144151.png)

**图书管理系统分析:**
1.定义Book类
2.完成主界面和选择
3.完成查询所有图书
4.完成添加图书
5.完成删除图书
6.完成修改图书
7.使用Debug追踪调试

# 2.图书管理系统之标准Book类

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614144236.png)

我们发现每一本书都有书名和价格,定义一个Book类表示书籍

```java
public class Book {
    private String name;
    private double price;

    public Book() {
    }

    public Book(String name, double price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }
}
```

# 3.图书管理系统之主界面和选择的实现

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143122.png)

主界面的内容其实就是通过打印语句打印出来的.但是要注意因为每个操作过后都会重新回到主界面,所以使用`while(true)`死循环的方式.

```java
public class BookManager {
    public static void main(String[] args) {
        while (true) {
            //这是学生管理系统的主界面
            System.out.println("--------欢迎来到学生管理系统--------");
            System.out.println("1.查看所有书籍");
            System.out.println("2.添加书");
            System.out.println("3.删除书");
            System.out.println("4.修改书");
            System.out.println("5.退出");
            System.out.println("请输入你的选择：");

            //创建键盘录入对象
            Scanner sc = new Scanner(System.in);
            int num = sc.nextInt();
            switch (num) {
                case 1:
                    // 查看所有书籍
                    break;
                case 2:
                    // 添加书籍
                    break;
                case 3:
                    // 删除书
                    break;
                case 4:
                    // 修改书
                    break;
                case 5:
                    // 退出
                    break;
                default:
                    System.out.println("输入错误,请重新输入");
                    break;
            }
        }
    }
}
```

# 4.图书管理系统之查询所有图书

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143437.png)

```java
public class BookManger {
    public static void main(String[] args) {
        Map<String, ArrayList<Book>> map = new HashMap<>();
        // 创建集合对象，用于存储学生数据
        ArrayList<Book> it = new ArrayList<Book>();
        it.add(new Book("Java入门到精通", 99));
        it.add(new Book("PHP入门到精通", 9.9));
        map.put("it书籍", it);
        ArrayList<Book> mz = new ArrayList<Book>();
        mz.add(new Book("西游记", 19));
        mz.add(new Book("水浒传", 29));
        map.put("名著", mz);

        while (true){
            //这是图书管理系统的主界面
            System.out.println("--------欢迎来到图书管理系统--------");
            System.out.println("1.查看所有书籍");
            System.out.println("2.添加书");
            System.out.println("3.删除书");
            System.out.println("4.修改书");
            System.out.println("5.退出");
            System.out.println("请输入你的选择：");

            //创建键盘录入对象
            Scanner sc = new Scanner(System.in);
            int num = sc.nextInt();
            switch (num){
                case 1:
                    //查看所有书籍
                    FindAllBook.find(map);
                    break;
                case 2:
                    //添加书籍
                    AddBook.add(map);
                    break;
                case 3:
                    //删除书籍
                    DeleteBook.delete(map);
                    break;
                case 4:
                    //修改书籍
                    EditBook.edit(map);
                    break;
                case 5:
                    // 退出
                    System.out.println("谢谢你的使用");
                    // JVM退出
                    System.exit(0); 
                    break;
                default:
                    System.out.println("输入错误,请重新输入");
                    break;
            }
        }
    }
}
```

```java
public class FindAllBook {
    public static void find(Map<String, ArrayList<Book>>map){
        System.out.println("类型\t\t书名\t价格");
        Set<Map.Entry<String, ArrayList<Book>>> entries = map.entrySet();
        for (Map.Entry<String, ArrayList<Book>> entry : entries) {
            String key = entry.getKey();
            System.out.println(key);

            ArrayList<Book> value = entry.getValue();
            for (Book book : value) {
                System.out.println("\t\t" + book.getName() + "\t" + book.getPrice());
            }
        }
    }
}
```

# 5.图书管理系统之添加图书

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143557.png)

```java
public class AddBook {
    public static void add(Map<String, ArrayList<Book>>map){
        // 创建键盘录入对象
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入要添加书籍的类型:");
        String type = sc.next();
        System.out.println("请输入要添加的书名:");
        String name = sc.next();
        System.out.println("请输入要添加书的价格:");
        double price = sc.nextDouble();
        Book book = new Book(name, price);

        // 拿到书籍列表
        ArrayList<Book> books = map.get(type);
        if (books == null) {
            // 如果书籍列表不存在创建一个书籍列表
            books = new ArrayList<>();
            map.put(type, books);
        }

        // 将书添加到集合中
        books.add(book);
        System.out.println("添加" + name + "成功");
    }
}
```

# 6.图书管理系统之删除图书

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143724.png)

```java
public class DeleteBook {
    public static void delete(Map<String, ArrayList<Book>>map){
        // 创建键盘录入对象
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入要删除书籍的类型:");
        String type = sc.next();
        System.out.println("请输入要删除的书名:");
        String name = sc.next();

        // 拿到书籍列表:用Map集合的
        ArrayList<Book> books = map.get(type);
        if (books == null) {
            System.out.println("您删除的书籍类型不存在");
            return;
        }

        for (int i = 0; i < books.size(); i++) {
            Book book = books.get(i);
            if (book.getName().equals(name)) {
                // 找到这本书,删除这本书
                books.remove(i);
                System.out.println("删除" + name + "书籍成功");
                return; // 删除书籍后结束方法
            }
        }
        System.out.println("没有找到" + name + "书籍");
    }
}
```

# 7.图书管理系统之修改图书

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200614143923.png)

```java
public class EditBook {
    public static void edit(Map<String, ArrayList<Book>>map){
        // 创建键盘录入对象
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入要修改书籍的类型:");
        String type = sc.next();
        System.out.println("请输入要修改的书名:");
        String oldName = sc.next();

        // 拿到书籍列表
        ArrayList<Book> books = map.get(type);
        if (books == null) {
            System.out.println("您修改的书籍类型不存在");
            return;
        }


        for (int i = 0; i < books.size(); i++) {
            Book book = books.get(i);
            if (book.getName().equals(oldName)) {
                // 找到这本书,修改这本书
                System.out.println("请输入新的书名:");
                String newName = sc.next();
                System.out.println("请输入新的价格:");
                double price = sc.nextDouble();
                book.setName(newName);
                book.setPrice(price);
                System.out.println("修改成功");
                return; // 修改书籍后结束方法
            }
        }
        System.out.println("没有找到" + oldName + "书籍");
    }
}
```

