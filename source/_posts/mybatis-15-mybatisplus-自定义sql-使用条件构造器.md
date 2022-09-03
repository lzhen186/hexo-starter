---
title: MyBatis 15.Mybatisplus 自定义sql 使用条件构造器
date: 2022-09-03T15:35:20.082Z
---
# Mybatisplus 自定义sql 使用条件构造器

## 两种方式

### 注解方式

``` sql
动态查找：
@Select("select ${ew.SqlSelect} from ${tableName} ${ew.customSqlSegment}")
List<File> listFileByCondition(@Param("tableName") String tableName, @Param("ew") Wrapper wrapper);

ew.SqlSelect：所需要查找的字段

tableName：使用的是那张表

ew.customSqlSegment：条件
用法：allFileMapper.listFileByCondition(tableName,Wrappers.query().select("*").in("relation_uuid", uuids));
结果： select * from tablName where relation_uuid in ()


动态修改：
@Update("update ${tableName} set ${ew.sqlSet} ${ew.customSqlSegment}")
int updateByCondition(@Param("tableName") String tableName, @Param("ew") Wrapper wrapper);

ew.sqlSet：修改的字段

tableName：使用的是那张表

ew.customSqlSegment：条件

用法：
mapper.updateByCondition(tableName, Wrappers.update().set("state", "初始状态").in("id", ids));
结果： update tableName set state = '初始状态' where id in ()
```

### xml方式

``` sql
查找：
<select id="listFileByCondition" resultType="com.example.entity.File">
	SELECT ${ew.SqlSelect} FROM ${tableName} ${ew.customSqlSegment}
</select>


修改：
<update id="listFileByCondition" >
	update ${tableName} ${ew.SqlSelect}  ${ew.customSqlSegment}
</update>
```

### 查找带分页

``` sql
xml用法：
Page<File> selectPage(Page page, @Param("tableName") String tableName, @Param("ew") Wrapper wrapper);

<select id="selectPage" resultType="com.example.entity.File">
        select * from ${tableName} ${ew.customSqlSegment}
</select>

注解分页：
/**
     * 
     * @param rowBounds 分页对象 直接传入page即可
     * @param wrapper 条件构造器
     * @return
     */
    List<User> selectUserWrapper(RowBounds rowBounds, @Param("ew") Wrapper<User> wrapper);
```

`UserMapper.xml`加入对应的xml节点：

```xml
    <!-- 条件构造器形式 -->
    <select id="selectUserWrapper" resultType="user">
        SELECT
        <include refid="Base_Column_List" />
        FROM USER
        <where>
            ${ew.sqlSegment}
        </where>
    </select>
```

测试类：

```java
@Test
    public void testCustomSql() {
        User user = new User();
        user.setCode("703");
        user.setName("okong-condition");
        user.insert();
        
        EntityWrapper<User> qryWrapper = new EntityWrapper<>();
        qryWrapper.eq(User.CODE, user.getCode());
        
        Page<User> pageUser = new Page<>();
        pageUser.setCurrent(1);
        pageUser.setSize(10);
        
        List<User> userlist = userDao.selectUserWrapper(pageUser, qryWrapper);
        System.out.println(userlist.get(0));
        log.info("自定义sql结束");
    }
```

## 自定义SQL语句

> 在一些需要多表关联时，条件构造器和通用CURD都无法满足时，还可以自行手写sql语句进行扩展。**注意：这都是`mybatis`的用法。**

**以下两种方式都是改造`UserDao`接口。**

### 注解形式

```java
@Select("SELECT * FROM USER WHERE CODE = #{userCode}")
    List<User> selectUserCustomParamsByAnno(@Param("userCode")String userCode);
```

### xml形式

```java
List<User> selectUserCustomParamsByXml(@Param("userCode")String userCode);
```

同时，`UserMapper.xml`新增一个节点：

```xml
    <!-- 由于设置了别名：typeAliasesPackage=cn.lqdev.learning.mybatisplus.samples.biz.entity，所以resultType可以不写全路径了。 -->
    <select id="selectUserCustomParamsByXml" resultType="user">
        SELECT 
        <include refid="Base_Column_List"/> 
        FROM USER 
       WHERE CODE = #{userCode}
    </select>
```