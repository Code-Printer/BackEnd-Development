# Mybatis的常用标签  
mybatis的xml中的常用标签：
\<select>、\<insert>、\<update>、\<delete>、\<resultMap>、\<sql>、\<include>、\<selectKey>、\<trim>、\<where>、\<set>、\<foreach>、\<if>、\<choose>、\<when>、\<otherwise>、\<bind>

### select标签  
```xml
    <select id="queryUserByIdAndUsername" resultType="User">
        select username from t_user where id = #{id} and username = #{username}
    </select>
```
### insert  
```xml
<insert id="insertUser" parameterType="User">
	insert into t_user(username,password,gender,birth)
	values (username=#{username},password=#{password},gender=#{gender},birth=#{birth})
</insert>
```
### delete
```xml
<delete id="deleteById" parameterType="int">
	delet from t_user
	where id = #{id}
</delete>
```
### resultMap等装返回结果  
```xml
<resultMap id="rm" type="User">
	<id column="id" property="id"></id>
    <!--column值对应的是数据库的列名，property值对应得是类的属性名-->
	<result column="password" property="password"></result>
	<result column="username" property="username"></result>
	<result column="gender" property="gender"></result>
	<!--association一对一映射:在一个类属性中嵌套映射自定义的另一个属性类Address：aid address_info的映射规则-->
	<association property="address" javaType="Address">
		<id colum="aid" property="aid"></id>
		<result column="address_info" property="address_info"></result>
	</association>
	<!--描述集合的映射Order-->
	<collection property="order" ofType="Order">
		<id colum="oid" property="oid"></id>
		<result column="product" property="product"></result>
	</collection>
</resultMap>
<select id="queryUser" resultMap="rm">
	select u.id,u.username,u.password,u.gender,a.aid,a.address_info,o.oid,o.product
	from t_user u join t_address a
	on u.id = a.uid
	join t_order o
	on u.id = o.uid
	where id=#{id}
</select>
```
### sql和include
sql通过include标签引入到其他标签内容体中  
```xml
<!--通用的sql语句-->
<sql id="hhh">select * from t_user</sql>
<!--通过include引入select标签中-->
<select id="queryUserByid" resultType="User">
	<include refid="hhh"/>
	where id=#{id}
</select>
```
## 动态sql语句标签  
### if和where  
```xml
	<sql id="hhh">select *from t_user</sql>
    <select id="queryUserByUnameAndPwd resultType="User">
		<include refid="hhh"/>
		<where>
			<if test="username!=null">
				username = #{username}
			</if>
			<if test="password!=null">
				and password = #{password}
			</if>
		</where>
    </select>
```
### if和set
```xml
<update id="updateUser" parameterType="User">
	update t_user
	<set>
	<if test="password!=null">
	password = #{password},
	</if>
	<if test="username!=null">
	username=#{username}
	</if>
	where id = #{id}
</update>
```
### trim（调整sql语句）
```xml
 <select id="queryUserById2" resultType="User">
        <include refid="user_fired"/>
        <!--prefix=“where" 前缀为where，
        prefixOverrides="or|and" where子句中如果以or或者and开头，会被覆盖-->

        <trim prefix="where" prefixOverrides="or|and">
            <if test="username!=null">
                username = #{username}
            </if>
            <if test="gender!=null">
                and gender = #{gender}
            </if>
        </trim>
    </select>
    <update id="updateUserById" parameterType="User">
        update t_user
        <!--prefix=“set" 前缀为set，
        suffixOverrides=","自动将最后一个逗号删除-->

        <trim prefix="set" suffixOverrides=",">
            <if test="username!=null">
                username = #{username},
            </if>
            <if test="gender!=null">
                gender = #{gender},
            </if>
        </trim>
        where id = #{id}
    </update>
```
### foreach  
```xml
<delete id="deleteUsersById" parameterType="java.util.List">
	delete from t_user where id in
	<foreach collection="list" open="(" close=")" separator="," item="i">
	#{i}
	</foreach>
</delete>

    <insert id="insertUsers" parameterType="java.util.List">
        insert into t_user values
        <foreach collection="list" open="" close="" separator="," item="u">
            (null,#{u.username},#{u.password},#{u.gender},#{u.registTime})
        </foreach>
    </insert>
```
### choose（从前到后只取一个满足条件的）和when和otherwise(都不满则取该项)  
```xml
    <select id="findUser" resultType="User">
        select *from t_user where id = #{id}
        <choose>
            <when test="username!=null">
                AND username like #{username}
            </when>
            <when test="password!=null">
                AND password = #{password}
            </when>
            <otherwise>
                AND gender = 1
            </otherwise>
        </choose>
    </select>
```
### bind(给某个参数赋默认值)  
```xml
<!-- 本来的模糊查询方式 -->
<!--     <select id="listProduct" resultType="Product"> -->
<!--     select * from   product_  where name like concat('%',#{0},'%') -->
<!--     </select> -->
             
        <select id="listProduct" resultType="Product">
            <bind name="likename" value="'%' + name + '%'" />
            select * from   product_  where name like #{likename}
        </select>
</mapper>
``` 



