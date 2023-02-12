# MyBatis

#### foreach 循环插入数据

```java
# controller
@PostMapping("/addStudent")
public Integer addStudent(@RequestBody List<Student> student) {
    return userService.addStudent(student);
}
# service
public Integer addStudent(List<Student> student) {
  return userDao.addStudent(student);
}
# dao
Integer addStudent(List<Student> student);
# mapper 
<insert id="addStudent">
    insert into student(name,age)
    values
  //注意，这里collection就是要循环的对象 item就是每一个循环的对象  注意下面要使用student.
    <foreach collection="list" item="student" index="index" separator=",">
        (#{student.name},#{student.age})
    </foreach>
</insert>
```

#### foreach 批量更新数据

```java
<foreach>是可以执行多条修改语句的，只是需要数据库的连接url上设置一下，加上 &allowMultiQueries=true
# controller
  @PostMapping("/updateName")
public Integer updateStudent(@RequestBody List<Student> student) {
    return userService.updateStudent(student);
}
# service
public Integer updateStudent(List<Student> student) {
    return userDao.updateStudent(student);
}
# dao
Integer updateStudent(List<Student> student);
# mapper
<update id="updateStudent" parameterType="java.util.List">
    <foreach collection="list" item="student" separator=";" open="" close="">
        update student
        <set>
            name = #{student.name}
        </set>
        where id = #{student.id}
    </foreach>
</update>
```

```xml
association用于一对一和多对一，collection用于一对多。
多对一
多个学生对应一个老师
如果对于学生这边，就是一个多对一的现象，即从学生这边关联一个老师
<mapper namespace="com.zx.dao.UserDao">
    <select id="getStudents" resultMap="StudentTeacher">
        select *
        from student
    </select>
    <resultMap id="StudentTeacher" type="com.zx.entity.Student">
        <!--association关联属性  property属性名 javaType属性类型 column在多的一方的表中的列名-->
        <association property="teacher" javaType="com.zx.entity.Teacher" column="tid" select="getTeacher"/>
    </resultMap>
    <!--
       这里传递过来的id，只有一个属性的时候，下面可以写任何值
       association中column多参数配置：
           column="{key=value,key=value}"  key是传给下个sql的取值名称，value是片段一中sql查询的字段名。
       -->
    <select id="getTeacher" resultType="com.zx.entity.Teacher">
        select *
        from teacher
        where id = #{id}
    </select>
</mapper>
```



#### mybatis的一级缓存和二级缓存

```markdown
一级缓存  默认开启 是sqlsession级别的缓存
二级缓存  手动开启 是mapper级别的缓存 或者可以说是namespace级别的缓存
```

