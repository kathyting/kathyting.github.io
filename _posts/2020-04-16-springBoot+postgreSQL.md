---
layout:     post
title:     SpringBoot+postgreSQL
subtitle:   SpringBoot+postgreSQL
date:       2020-04-16
author:     KT
header-img: img/post-bg-springboot.jpg
catalog: true
tags:
    - Web
    - SpringBoot
    - postgreSQL
    - 开源框架
---
# 前言

>先安装postgreSQL。此处采用从EDB下载。下载链接：https://www.enterprisedb.com/downloads/postgres-postgresql-downloads
按照步骤安装即可。
此处使用EDB安装时我选择了(PostgreSQL Server, pgAdmin4, Command Line Tools)这三项，在安装完后，打开pgAdmin,点击左侧的Servers>PostgreSQL12,输入密码（自己指定），点击OK即可。
打开一个terminal, 输入psql postgres后，有可能会让输入密码，如果遇到输入刚刚指定的密码都出现无法验证通过的情况时，可以采取如下方法：
step1:进入/Library/PostgreSQL/12/data
step2:vi pg_hba.conf
step3:将有md5的地方都改成trust
step4:重启postgresql服务  sudo service postgresql restart

另外，如果是macOS Catalina的话，默认的是Zsh作为default shell. 此时如果想进入/Library/PostgreSQL/12/data去修改pg_hba.conf会发现无法cd 到data, 会出现cd: permission denied: data 的提示。解决办法是输入sudo su postgres,然后它会让你输入passsword.这个password就是你电脑的密码。进入后会看到%符号变成了$.这是输入psql postgres回车，输入之前你在pgAdmin4中指定的那个密码，就会进入到数据库了。

当看到
psql (12.2, server 11.7)
Type "help" for help.

postgres=#
这几行字的时候，恭喜你，已经成功进入到postgreSQL了。现在我们要做的是建立一个名为students的database.
postgres=# CREATE DATABASE students;

其他的一些命令有：
\q  exit database
\l(此处是小写的L) //list all the databases you have
\dl(此处是小写的L)  //list tables
\du //list users on the database
postgres -V  //check postgres version
CREATE ROLE [user] WITH LOGIN PASSWORD [password] ; //create role user with its password
ALTER ROLE [user] CREATEDB;  //give database creation permission to user
psql postgres  //login postgres database

# SpringBoot+PostgresSQL+JPA
Step1: Create springboot project in STS
File-> New project -> Srping Starter Project


Step 2:Create project structer.
Create 3 packages under src/main/java/com.example.demo
package com.example.demo.controller;
package com.example.demo.model;
package com.example.demo.repository;


step3: Configure postgreSQL in the project
open src/main/resources,edit application.properties
### application.properties
spring.datasource.url=jdbc:postgresql://127.0.0.1:5432/students
spring.datasource.username=postgres
spring.datasource.password=admin
spring.jpa.show-sql=true 

#Hibernate Properties
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
#Hibernate ddl auto
spring.jpa.hibernate.ddl-auto=update 

其中studens是我们建的数据库名，password处的admin是之前在pgAdmin中指定的密码。
step4:  Define entity-Student.java
### Student.java
package com.example.demo.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity 
@Table(name = "students") 
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
  private long id;
    @Column(name="first_name")
  private String firstName;
    
    @Column(name="last_name")
  private String lastName;
    @Column(name="email_add")
  private String email;
  
public Student() {
    super();
    // TODO Auto-generated constructor stub
}
public Student(String firstName, String lastName, String email) {
    super();
    this.firstName = firstName;
    this.lastName = lastName;
    this.email = email;
}
public long getId() {
    return id;
}
public void setId(long id) {
    this.id = id;
}
public String getFirstName() {
    return firstName;
}
public void setFirstName(String firstName) {
    this.firstName = firstName;
}
public String getLastName() {
    return lastName;
}
public void setLastName(String lastName) {
    this.lastName = lastName;
}
public String getEmail() {
    return email;
}
public void setEmail(String email) {
    this.email = email;
}
  
}


step5: Create Spring Data Repository-StudentRepository.java
### StudentRepository.java
package com.example.demo.repository;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import com.example.demo.model.Student;
@Repository
public interface StudentRepository extends JpaRepository<Student,Long>{

}

step6:Create Spring Rest Controller- StudentController.java
### StudentController.java
@RestController
@RequestMapping("/myapi/")
public class StudentController {
    @Autowired
    private StudentRepository stuRepository;
    
   //get students
    @GetMapping("students")
    private List<Student> getAllStudent(){
        return this.stuRepository.findAll();
    }
    // get student id
    #GetMapping("/students/{id}")
    public ResponseEntity<Student> getStudentById(@PathVariable(value="id") Long studentId)
    throws Exception{
        Student student = stuRepository.findById(studentId)
                .orElseThrow(()->new Exception("Student not found "));
        return ResponseEntity.ok().body(student);
    }
    //delete student by id
    //update student 
}






