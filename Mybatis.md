# 1. Mybatis框架

---

####  1.编写UserMapper.xml配置文件

```xml
<mapper namespace="userMapper">
    <insert id="add" parameterType="com.itheima.domain.User">
		insert into user values(#{id},#{username},#{password})
	</insert>
</mapper>
```

![image-20220324222237415](https://gitee.com/ingachin/mdimage/raw/master/image-20220324222237415.png)

#### 2.编写Mybatis配置文件  

**`sqlMapConfig.xml`**

```xml
<configuration>
    <environments default="development">
        <environment id="development">
        <transactionManagertype="JDBC"/>
        <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/><property name="url" value="jdbc:mysql:///test"/>
        <property name="username" value="root"/><property name="password" value="root"/>
        </dataSource>
        </environment>
    </environments>
<mappers> <mapper resource="com/itheima/mapper/UserMapper.xml"/> </mappers>
</configuration>
```

##### 使用typeAliases可以设置别名

```xml
<typeAliases type="全包名" aliaes="别名"></typeAliases>
```

  

#### 3.使用动态代理生成Mapper对象

Mapper 接口开发需要遵循以下规范： 

1、 Mapper.xml文件中的namespace与mapper接口的全限定名相同 

2、 Mapper接口方法名和Mapper.xml中定义的每个statement的id相同 

3、 Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql的parameterType的类型相同 

4、 Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同

```java
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
```

  

#### 4.动态sql

##### 1.`<if>`

```xml
<where>
    <if test="id!=0">
    and id=#{id}
    </if>
    
    <if test="username!=null">
    and username=#{username}
    </if>
    
</where>
```

##### 2.`<where>`

```xml
<where>
    <if test="id!=0">
    and id=#{id}
    </if>
    
    <if test="username!=null">
    and username=#{username}
    </if>
    
</where>
```



#### 5. 自定义类型转换器

##### 1.定义类型转换器  

```java
public class MyDateTypeHandler extends BaseTypeHandler<Date> {
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Date date, JdbcType type){
    	preparedStatement.setString(i,date.getTime()+"");
    }
    public Date getNullableResult(ResultSet resultSet, String s) throws SQLException {
    	return new Date(resultSet.getLong(s));
    }
    public Date getNullableResult(ResultSet resultSet, int i) throws SQLException {
    	return new Date(resultSet.getLong(i));
    }
    public Date getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
    	return callableStatement.getDate(i);
    }
}
```

##### 2.在sqlmapperconfig.xml中配置

```xml
<typeHandlers>
	<typeHandler handler="com.itheima.typeHandlers.MyDateTypeHandler"></typeHandler>
</typeHandlers>
```

  

#### 6.配置分页助手插件

```xml
<plugin interceptor="com.github.pagehelper.PageHelper">
    <!-- 指定方言 -->
    <property name="dialect" value="mysql"/>
</plugin>
```

#### 7.Mybatis多表操作

`UserMapper.xml`配置

```xml
<mapper namespace="com.itheima.mapper.UserMapper">
    <resultMap id="userMap" type="com.itheima.domain.User">
        <result column="id" property="id"></result>
        <result column="username" property="username"></result>
        <result column="password" property="password"></result>
        <result column="birthday" property="birthday"></result>
        <collection property="orderList" ofType="com.itheima.domain.Order">
            <result column="oid" property="id"></result>
            <result column="ordertime" property="ordertime"></result>
            <result column="total" property="total"></result>
        </collection>
    </resultMap>
    <select id="findAll" resultMap="userMap">
    select *,o.id oid from user u left join orders o on u.id=o.uid
    </select>
</mapper>
```



#### 8.Myabtis整合Spring



##### 1.将`SqlSession`对象创建权交给`Spring`

```xml
<!--加载jdbc.properties-->
<context:property-placeholder location="classpath:jdbc.properties"/>
<!--配置数据源-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
<!--配置MyBatis的SqlSessionFactory-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:sqlMapConfig.xml"/>
</bean>
```

  

##### 2.配置Mapper扫描,使Spring产生Mapper类

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<property name="basePackage" value="com.itheima.mapper"/>
</bean>
```

  









