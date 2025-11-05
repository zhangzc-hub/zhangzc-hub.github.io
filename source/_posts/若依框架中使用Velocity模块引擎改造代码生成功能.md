---
title: 若依框架中使用Velocity模块引擎改造代码生成功能
date: 2025-09-04 14:10:15
tags:
- 若依
categories:
- Java
---

# 若依框架中使用Velocity模块引擎改造代码生成功能

- 支持mybatis-plus
- 支持Lombok
- 支持Swagger自动添加
- 支持LocalDateTime

在若依框架中，主要生成代码的模块是zzyl-generator模块

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/image.png)

上述代码中，以`vm`为后缀的文件，其实是velocity模板引擎的对应的模板文件，如果想要改造代码生成的内容，我们必须要先学习velocity，而后再来看懂这些代码，才能支撑我们去改造这些模板文件，来生成我们想要的内容

## 1、Velocity模块引擎

### 1.1、概述

Velocity是一个基于Java的模板引擎，可以通过特定的语法获取在java对象的数据 , 填充到模板中，从而实现界面和java代码的分离 !

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_14-17-40.jpg)

常见的应用场景：

- Web应用程序 : 作为为应用程序的视图, 展示数据。
- 源代码生成  : Velocity可用于基于模板生成Java源代码。
- 自动电子邮件 : 网站注册 , 认证等的电子邮件模板。
- 网页静态化  : 基于velocity模板 , 生成静态网页。

### 1.2、快速入门

需求：根据下面html模板，完成对数据的填充

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>velocity快速入门</title>
</head>
<body>

    <h3>心怀梦想，坚持不懈，成功即在前方。加油少年！</h3>
    
</body>
</html>
```

- 要求：**加油少年，**这几个字，需要使用动态填充进来

**入门案例实现步骤**

①准备模板

把上面提供的html文档，拷贝一份到若依项目中的ruoyi-generator模块下的reources目录中

模板文件命名：`index.html.vm`

如下图

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/e44ce461-6e3a-427f-acdb-78fc53d9db12.png)

模板文件内容：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>velocity快速入门</title>
</head>
<body>

<h3>心怀梦想，坚持不懈，成功即在前方。${message}</h3>

</body>
</html>
```

**注意**：上述代码中的 加油少年  修改为了  ${message}  这是一个动态变量，方便**动态**填充数据

②编写java代码实现数据填充，并生成文件

```java
import com.ruoyi.common.constant.Constants;
import org.apache.velocity.Template;
import org.apache.velocity.VelocityContext;
import org.apache.velocity.app.Velocity;

import java.io.FileWriter;
import java.io.IOException;
import java.util.Properties;

public class VelocityDemoTest {

    public static void main(String[] args) throws IOException {

        Properties p = new Properties();
        //velocity资源加载器，告诉模板引擎到类路径下寻找资源（比如模板）
        p.setProperty("resource.loader.file.class", "org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader");
        // 定义字符集
        p.setProperty(Velocity.INPUT_ENCODING, Constants.UTF8);
        //初始化velocity引擎
        Velocity.init(p);

        //创建velocity上下文对象
        VelocityContext context = new VelocityContext();
        //数据模型，这里的key需要跟模板中的变量对应上，不然填充不了数据
        context.put("message", "加油朋友！！！");
        //获取模板
        Template template = Velocity.getTemplate("vms/index.html.vm", "UTF-8");
        //输出
        FileWriter fileWriter = new FileWriter("D:\\workspace\\index.html");
        //合并模板和数据模型
        template.merge(context,fileWriter);
        //关闭流
        fileWriter.close();

    }
}
```

最终的效果：

在指定的目录中生成index.html文件

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_14-23-28.jpg)

打开之后的效果：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_14-24-07.jpg)

### 1.3、AI辅助解读Velocity语法

大模型解读后的代码如下：

```java
// Velocity模板开始，定义包名
package ${packageName}.domain;

// 循环导入所需的类库
#foreach ($import in $importList)
import ${import};
#end

// 导入其他必要的类库
import org.apache.commons.lang3.builder.ToStringBuilder;
import org.apache.commons.lang3.builder.ToStringStyle;
import com.zzyl.common.annotation.Excel;
#if($table.crud || $table.sub)
import com.zzyl.common.core.domain.BaseEntity;
#elseif($table.tree)
import com.zzyl.common.core.domain.TreeEntity;
#end

/**
 * ${functionName}对象 ${tableName}
 *
 * @author ${author}
 * @date ${datetime}
 */
#if($table.crud || $table.sub)
#set($Entity="BaseEntity")
#elseif($table.tree)
#set($Entity="TreeEntity")
#end

// 定义Java类，继承自BaseEntity或TreeEntity
public class ${ClassName} extends ${Entity}
{
    // 序列化版本ID
    private static final long serialVersionUID = 1L;

    // 循环定义类中的属性
    #foreach ($column in $columns)
    #if(!$table.isSuperColumn($column.javaField))
        // 添加属性的注释
        /** $column.columnComment */

        // 根据属性是否为列表类型，决定是否添加Excel注解
        #if($column.list)
        #set($parentheseIndex=$column.columnComment.indexOf("（"))
        #if($parentheseIndex != -1)
        #set($comment=$column.columnComment.substring(0, $parentheseIndex))
        #else
        #set($comment=$column.columnComment)
        #end
        #if($parentheseIndex != -1)
        // 如果属性名中有括号，则截取括号前的内容作为Excel注解的名字
        @Excel(name = "${comment}", readConverterExp = "$column.readConverterExp()")
        #else
        // 如果是日期类型，添加JSON格式化注解
        #elseif($column.javaType == 'Date')
        @JsonFormat(pattern = "yyyy-MM-dd")
        @Excel(name = "${comment}", width = 30, dateFormat = "yyyy-MM-dd")
        #else
        // 其他情况直接添加Excel注解
        @Excel(name = "${comment}")
        #end
        #end

        // 定义属性
        private $column.javaType $column.javaField;

    #end
    #end

    // 如果是子表，添加子表的集合属性
    #if($table.sub)
        /** $table.subTable.functionName信息 */
        private List<${subClassName}> ${subclassName}List;

    #end

    // 生成getter和setter方法
    #foreach ($column in $columns)
    #if(!$table.isSuperColumn($column.javaField))
        #if($column.javaField.length() > 2 && $column.javaField.substring(1,2).matches("[A-Z]"))
        #set($AttrName=$column.javaField)
        #else
        #set($AttrName=$column.javaField.substring(0,1).toUpperCase() + ${column.javaField.substring(1)})
        #end

        // setter方法
        public void set${AttrName}($column.javaType $column.javaField)
        {
            this.$column.javaField = $column.javaField;
        }

        // getter方法
        public $column.javaType get${AttrName}()
        {
            return $column.javaField;
        }
    #end
    #end

    // 如果是子表，生成子表集合的getter和setter方法
    #if($table.sub)
        public List<${subClassName}> get${subClassName}List()
        {
            return ${subclassName}List;
        }

        public void set${subClassName}List(List<${subClassName}> ${subclassName}List)
        {
            this.${subclassName}List = ${subclassName}List;
        }

    #end

    // 重写toString方法
    @Override
    public String toString() {
        return new ToStringBuilder(this,ToStringStyle.MULTI_LINE_STYLE)
        #foreach ($column in $columns)
        #if($column.javaField.length() > 2 && $column.javaField.substring(1,2).matches("[A-Z]"))
        #set($AttrName=$column.javaField)
        #else
        #set($AttrName=$column.javaField.substring(0,1).toUpperCase() + ${column.javaField.substring(1)})
        #end
            .append("${column.javaField}", get${AttrName}())
        #end
        #if($table.sub)
            .append("${subclassName}List", get${subClassName}List())
        #end
            .toString();
    }
}
```

- Velocity语法解释
  - `${variable}`: 表示插入变量值。
  - `#foreach` 和 `#end`: 循环结构，用于遍历列表。
  - `#if` 和 `#end`: 条件判断结构。
  - `#set`: 设置变量。
  - `#elseif`: 条件分支。
- Java类结构说明
  - 包声明: 定义类所在的包。
  - 导入语句: 导入类所需要的类库。
  - 类注释: 描述类的功能和作者信息。
  - 类定义: 定义类并继承自`BaseEntity`或`TreeEntity`。
  - 属性定义: 根据模板参数定义类的属性。
  - 属性注释: 对属性进行描述。
  - Excel注解: 标记属性以方便导出Excel。
  - getter和setter方法: 自动生成属性的访问方法。
  - 子表集合: 如果是子表，则定义子表集合。
  - 重写toString方法: 提供类实例的字符串表示形式。

## 2、代码生成流程

目前，我们已经基本熟悉了velocity的作用和一些语法，那接下来，我们就通过这些知识来去改造若依框架的代码生成部分，在改造之前，我们对于生成代码的核心部分来阅读一下

代码位置：com.zzyl.generator.service.GenTableServiceImpl.generatorCode()

```java
/**
 * 生成代码
 * @param tableName 表名
 * @param zip       压缩输出流
 */
private void generatorCode(String tableName, ZipOutputStream zip)
{
    // 查询表信息
    GenTable table = genTableMapper.selectGenTableByName(tableName);
    // 设置主子表信息
    setSubTable(table);
    // 设置主键列信息
    setPkColumn(table);

    // 初始化Velocity引擎
    VelocityInitializer.initVelocity();

    // 准备Velocity上下文   获取详细的该表的数据，并设置模板所需要的数据模型
    VelocityContext context = VelocityUtils.prepareContext(table);

    // 获取模板列表   读取resources/vm目录中的定义的模板文件
    List<String> templates = VelocityUtils.getTemplateList(table.getTplCategory(), table.getTplWebType());
    for (String template : templates)
    {
        // 渲染模板
        StringWriter sw = new StringWriter();
        Template tpl = Velocity.getTemplate(template, Constants.UTF8);
        // 合并模板和数据模型
        tpl.merge(context, sw);
        try
        {
            // 将渲染结果添加到zip文件
            zip.putNextEntry(new ZipEntry(VelocityUtils.getFileName(template, table)));
            IOUtils.write(sw.toString(), zip, Constants.UTF8);
            IOUtils.closeQuietly(sw);
            zip.flush();
            zip.closeEntry();
        }
        catch (IOException e)
        {
            // 记录日志
            log.error("渲染模板失败，表名：" + table.getTableName(), e);
        }
    }
}
```

## 3、Lombok集成

### 3.1、依赖导入

```java
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.22</version>
</dependency>
```

### 3.2、修改模板

我们主要修改的模板为 `domain.java.vm`，让它支持lombok

修改模板如下：

```java
package ${packageName}.domain;

#foreach ($import in $importList)
import ${import};
#end
import org.apache.commons.lang3.builder.ToStringBuilder;  ## 用不到，可删
import org.apache.commons.lang3.builder.ToStringStyle;  ## 用不到，可删
import com.zzyl.common.annotation.Excel;
#if($table.crud || $table.sub)
import com.zzyl.common.core.domain.BaseEntity;
#elseif($table.tree)
import com.zzyl.common.core.domain.TreeEntity;
#end
import lombok.Data;  ## 导入lombok的依赖

/**
 * ${functionName}对象 ${tableName}
 * 
 * @author ${author}
 * @date ${datetime}
 */
#if($table.crud || $table.sub)
#set($Entity="BaseEntity")
#elseif($table.tree)
#set($Entity="TreeEntity")
#end
@Data  ## 添加lombok的注解
public class ${ClassName} extends ${Entity}
{
    private static final long serialVersionUID = 1L;

#foreach ($column in $columns)
#if(!$table.isSuperColumn($column.javaField))
    /** $column.columnComment */
#if($column.list)
#set($parentheseIndex=$column.columnComment.indexOf("（"))
#if($parentheseIndex != -1)
#set($comment=$column.columnComment.substring(0, $parentheseIndex))
#else
#set($comment=$column.columnComment)
#end
#if($parentheseIndex != -1)
    @Excel(name = "${comment}", readConverterExp = "$column.readConverterExp()")
#elseif($column.javaType == 'Date')
    @JsonFormat(pattern = "yyyy-MM-dd")
    @Excel(name = "${comment}", width = 30, dateFormat = "yyyy-MM-dd")
#else
    @Excel(name = "${comment}")
#end
#end
    private $column.javaType $column.javaField;

#end
#end
#if($table.sub)
    /** $table.subTable.functionName信息 */
    private List<${subClassName}> ${subclassName}List;

#end

## 以下是生成属性的get/set方法  start....  删除这部分内容
#foreach ($column in $columns)
#if(!$table.isSuperColumn($column.javaField))
#if($column.javaField.length() > 2 && $column.javaField.substring(1,2).matches("[A-Z]"))
#set($AttrName=$column.javaField)
#else
#set($AttrName=$column.javaField.substring(0,1).toUpperCase() + ${column.javaField.substring(1)})
#end
    public void set${AttrName}($column.javaType $column.javaField) 
    {
        this.$column.javaField = $column.javaField;
    }

    public $column.javaType get${AttrName}() 
    {
        return $column.javaField;
    }
#end
#end

## 以下是生成属性的get/set方法  end....  删除这部分内容

## 如果是主子表，生成子表相关的get/set方法，start....  删除这部分内容
#if($table.sub)
    public List<${subClassName}> get${subClassName}List()
    {
        return ${subclassName}List;
    }

    public void set${subClassName}List(List<${subClassName}> ${subclassName}List)
    {
        this.${subclassName}List = ${subclassName}List;
    }

#end

## 如果是主子表，生成子表相关的get/set方法，end....  删除这部分内容


##以下代码生成 toString方法， start....  删除这部分内容
    @Override
    public String toString() {
        return new ToStringBuilder(this,ToStringStyle.MULTI_LINE_STYLE)
#foreach ($column in $columns)
#if($column.javaField.length() > 2 && $column.javaField.substring(1,2).matches("[A-Z]"))
#set($AttrName=$column.javaField)
#else
#set($AttrName=$column.javaField.substring(0,1).toUpperCase() + ${column.javaField.substring(1)})
#end
            .append("${column.javaField}", get${AttrName}())
#end
#if($table.sub)
            .append("${subclassName}List", get${subClassName}List())
#end
            .toString();
    }
##以下代码生成 toString方法， end....  删除这部分内容

}
```

### 3.3、生成后的效果

修改完成之后，重启项目，找到代码生成的功能，通过**代码预览**可以查看实体类的代码：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_14-59-09.jpg)

> - 正常添加了关于lombok的注解
> - 删除了setter、 getter  、toString 等方法

**测试**

可以把生成后的代码，拷贝到项目中，如果护理项目能够正常访问和操作，就算修改成功了，后期再次生成的代码，全部都支持lombok

## 4、Mybatis-Plus集成

我们改造主要参考的是官方推荐的项目：https://gitee.com/tellsea/ruoyi-vue-plus（目前已集成mybatis-plus）

### 4.1、添加依赖

在父工程中的pom文件中添加mybatis-plus的依赖，如下：

```xml
<properties>
    <mybatis-plus-spring-boot.version>3.5.2</mybatis-plus-spring-boot.version>
</properties>

<!-- 依赖声明 -->
<dependencyManagement>
    <dependencies>
        <!-- mybatis-plus 增强CRUD -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus-spring-boot.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-annotation</artifactId>
            <version>${mybatis-plus-spring-boot.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

在zzyl-common模块中新增mybatis-plus的依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-annotation</artifactId>
</dependency>
```

### 4.2、核心配置

appliation.yml文件中新增MyBatisPlus配置，同时**删除Mybatis相关的配置**

```yaml
# MyBatisPlus配置
mybatis-plus:
  # 搜索指定包别名
  typeAliasesPackage: com.zzyl.**.domain
  # 配置mapper的扫描，找到所有的mapper.xml映射文件
  mapperLocations: classpath*:mapper/**/*Mapper.xml
  # 全局配置
  global-config:
    db-config:
      id-type: auto   #id生成策略为自增
  configuration: 
    map-underscore-to-camel-case: true    #字段与属性，自动转换为驼峰命名
```

新增核心配置类，**删除MyBatisConfig**

```Java
package com.zzyl.framework.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.BlockAttackInnerInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.OptimisticLockerInnerInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

/**
 * Mybatis Plus 配置
 *
 * @author ruoyi
 */
@EnableTransactionManagement(proxyTargetClass = true)
@Configuration
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 分页插件
        interceptor.addInnerInterceptor(paginationInnerInterceptor());
        // 乐观锁插件
        interceptor.addInnerInterceptor(optimisticLockerInnerInterceptor());
        // 阻断插件
        interceptor.addInnerInterceptor(blockAttackInnerInterceptor());
        return interceptor;
    }

    /**
     * 分页插件，自动识别数据库类型 https://baomidou.com/guide/interceptor-pagination.html
     */
    public PaginationInnerInterceptor paginationInnerInterceptor() {
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
        // 设置数据库类型为mysql
        paginationInnerInterceptor.setDbType(DbType.MYSQL);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        paginationInnerInterceptor.setMaxLimit(-1L);
        return paginationInnerInterceptor;
    }

    /**
     * 乐观锁插件 https://baomidou.com/guide/interceptor-optimistic-locker.html
     */
    public OptimisticLockerInnerInterceptor optimisticLockerInnerInterceptor() {
        return new OptimisticLockerInnerInterceptor();
    }

    /**
     * 如果是对全表的删除或更新操作，就会终止该操作 https://baomidou.com/guide/interceptor-block-attack.html
     */
    public BlockAttackInnerInterceptor blockAttackInnerInterceptor() {
        return new BlockAttackInnerInterceptor();
    }

}
```

### 4.3、代码生成模板

在目前的模板文件中，我们需要修改的模板共有3个，分别是：

- mapper.java.vm   继承BaseMapper 
- service.java.vm   继承IService<T> 
- serviceImpl.java.vm    继承ServiceImpl<XxxMapper,T>  常见方法的使用（单表的增删改查）

> - domain.java.vm  无需修改，类名与表名一致，会自动映射，主键已经在yaml文件中配置
> - controller.java.vm 无需修改，保留原有的接口方法和命名

**mapper.java.vm**

```Java
package ${packageName}.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.apache.ibatis.annotations.Mapper;
import java.util.List;
import ${packageName}.domain.${ClassName};
#if($table.sub)
import ${packageName}.domain.${subClassName};
#end

/**
 * ${functionName}Mapper接口
 * 
 * @author ${author}
 * @date ${datetime}
 */
@Mapper
public interface ${ClassName}Mapper extends BaseMapper<${ClassName}>
{
    /**
     * 查询${functionName}
     * 
     * @param ${pkColumn.javaField} ${functionName}主键
     * @return ${functionName}
     */
    public ${ClassName} select${ClassName}By${pkColumn.capJavaField}(${pkColumn.javaType} ${pkColumn.javaField});

    /**
     * 查询${functionName}列表
     * 
     * @param ${className} ${functionName}
     * @return ${functionName}集合
     */
    public List<${ClassName}> select${ClassName}List(${ClassName} ${className});

    /**
     * 新增${functionName}
     * 
     * @param ${className} ${functionName}
     * @return 结果
     */
    public int insert${ClassName}(${ClassName} ${className});

    /**
     * 修改${functionName}
     * 
     * @param ${className} ${functionName}
     * @return 结果
     */
    public int update${ClassName}(${ClassName} ${className});

    /**
     * 删除${functionName}
     * 
     * @param ${pkColumn.javaField} ${functionName}主键
     * @return 结果
     */
    public int delete${ClassName}By${pkColumn.capJavaField}(${pkColumn.javaType} ${pkColumn.javaField});

    /**
     * 批量删除${functionName}
     * 
     * @param ${pkColumn.javaField}s 需要删除的数据主键集合
     * @return 结果
     */
    public int delete${ClassName}By${pkColumn.capJavaField}s(${pkColumn.javaType}[] ${pkColumn.javaField}s);
#if($table.sub)

    /**
     * 批量删除${subTable.functionName}
     * 
     * @param ${pkColumn.javaField}s 需要删除的数据主键集合
     * @return 结果
     */
    public int delete${subClassName}By${subTableFkClassName}s(${pkColumn.javaType}[] ${pkColumn.javaField}s);
    
    /**
     * 批量新增${subTable.functionName}
     * 
     * @param ${subclassName}List ${subTable.functionName}列表
     * @return 结果
     */
    public int batch${subClassName}(List<${subClassName}> ${subclassName}List);
    

    /**
     * 通过${functionName}主键删除${subTable.functionName}信息
     * 
     * @param ${pkColumn.javaField} ${functionName}ID
     * @return 结果
     */
    public int delete${subClassName}By${subTableFkClassName}(${pkColumn.javaType} ${pkColumn.javaField});
#end
}
```

**service.java.vm**

```Java
package ${packageName}.service;

import com.baomidou.mybatisplus.extension.service.IService;
import java.util.List;
import ${packageName}.domain.${ClassName};

/**
 * ${functionName}Service接口
 * 
 * @author ${author}
 * @date ${datetime}
 */
public interface I${ClassName}Service extends IService<${ClassName}>
{
    /**
     * 查询${functionName}
     * 
     * @param ${pkColumn.javaField} ${functionName}主键
     * @return ${functionName}
     */
    public ${ClassName} select${ClassName}By${pkColumn.capJavaField}(${pkColumn.javaType} ${pkColumn.javaField});

    /**
     * 查询${functionName}列表
     * 
     * @param ${className} ${functionName}
     * @return ${functionName}集合
     */
    public List<${ClassName}> select${ClassName}List(${ClassName} ${className});

    /**
     * 新增${functionName}
     * 
     * @param ${className} ${functionName}
     * @return 结果
     */
    public int insert${ClassName}(${ClassName} ${className});

    /**
     * 修改${functionName}
     * 
     * @param ${className} ${functionName}
     * @return 结果
     */
    public int update${ClassName}(${ClassName} ${className});

    /**
     * 批量删除${functionName}
     * 
     * @param ${pkColumn.javaField}s 需要删除的${functionName}主键集合
     * @return 结果
     */
    public int delete${ClassName}By${pkColumn.capJavaField}s(${pkColumn.javaType}[] ${pkColumn.javaField}s);

    /**
     * 删除${functionName}信息
     * 
     * @param ${pkColumn.javaField} ${functionName}主键
     * @return 结果
     */
    public int delete${ClassName}By${pkColumn.capJavaField}(${pkColumn.javaType} ${pkColumn.javaField});
}
```

**serviceImpl.java.vm**

```Java
package ${packageName}.service.impl;

import java.util.List;
#foreach ($column in $columns)
#if($column.javaField == 'createTime' || $column.javaField == 'updateTime')
import com.zzyl.common.utils.DateUtils;
#break
#end
#end
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
#if($table.sub)
import java.util.ArrayList;
import com.zzyl.common.utils.StringUtils;
import org.springframework.transaction.annotation.Transactional;
import ${packageName}.domain.${subClassName};
#end
import ${packageName}.mapper.${ClassName}Mapper;
import ${packageName}.domain.${ClassName};
import ${packageName}.service.I${ClassName}Service;

import ${packageName}.mapper.${ClassName}Mapper;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import java.util.Arrays;

/**
 * ${functionName}Service业务层处理
 * 
 * @author ${author}
 * @date ${datetime}
 */
@Service
public class ${ClassName}ServiceImpl extends ServiceImpl<${ClassName}Mapper, ${ClassName}> implements I${ClassName}Service
{
    @Autowired
    private ${ClassName}Mapper ${className}Mapper;

    /**
     * 查询${functionName}
     * 
     * @param ${pkColumn.javaField} ${functionName}主键
     * @return ${functionName}
     */
    @Override
    public ${ClassName} select${ClassName}By${pkColumn.capJavaField}(${pkColumn.javaType} ${pkColumn.javaField})
    {
        ##return ${className}Mapper.select${ClassName}By${pkColumn.capJavaField}(${pkColumn.javaField});
        return getById(${pkColumn.javaField});
    }

    /**
     * 查询${functionName}列表
     * 
     * @param ${className} ${functionName}
     * @return ${functionName}
     */
    @Override
    public List<${ClassName}> select${ClassName}List(${ClassName} ${className})
    {
        return ${className}Mapper.select${ClassName}List(${className});
    }

    /**
     * 新增${functionName}
     * 
     * @param ${className} ${functionName}
     * @return 结果
     */
#if($table.sub)
    @Transactional
#end
    @Override
    public int insert${ClassName}(${ClassName} ${className})
    {
#foreach ($column in $columns)   删除这些 新增的时候，给创建时间赋值
#if($column.javaField == 'createTime')
        ${className}.setCreateTime(DateUtils.getNowDate());
#end
#end
#if($table.sub)
        int rows = ${className}Mapper.insert${ClassName}(${className});
        insert${subClassName}(${className});
        return rows;
#else
        ##return ${className}Mapper.insert${ClassName}(${className});
        return save(${className}) == true? 1 : 0;
#end
    }

    /**
     * 修改${functionName}
     * 
     * @param ${className} ${functionName}
     * @return 结果
     */
#if($table.sub)
    @Transactional
#end
    @Override
    public int update${ClassName}(${ClassName} ${className})
    {
#foreach ($column in $columns)  删除这些，修改的时候给修改时间赋值
#if($column.javaField == 'updateTime')
        ${className}.setUpdateTime(DateUtils.getNowDate());
#end
#end
#if($table.sub)
        ${className}Mapper.delete${subClassName}By${subTableFkClassName}(${className}.get${pkColumn.capJavaField}());
        insert${subClassName}(${className});
#end
##        return ${className}Mapper.update${ClassName}(${className});
        return updateById(${className}) == true ? 1 : 0;
    }

    /**
     * 批量删除${functionName}
     * 
     * @param ${pkColumn.javaField}s 需要删除的${functionName}主键
     * @return 结果
     */
#if($table.sub)
    @Transactional
#end
    @Override
    public int delete${ClassName}By${pkColumn.capJavaField}s(${pkColumn.javaType}[] ${pkColumn.javaField}s)
    {
#if($table.sub)
        ${className}Mapper.delete${subClassName}By${subTableFkClassName}s(${pkColumn.javaField}s);
#end
##        return ${className}Mapper.delete${ClassName}By${pkColumn.capJavaField}s(${pkColumn.javaField}s);
        return removeByIds(Arrays.asList(${pkColumn.javaField}s)) == true ? 1 : 0;
    }

    /**
     * 删除${functionName}信息
     * 
     * @param ${pkColumn.javaField} ${functionName}主键
     * @return 结果
     */
#if($table.sub)
    @Transactional
#end
    @Override
    public int delete${ClassName}By${pkColumn.capJavaField}(${pkColumn.javaType} ${pkColumn.javaField})
    {
#if($table.sub)
        ${className}Mapper.delete${subClassName}By${subTableFkClassName}(${pkColumn.javaField});
#end
##        return ${className}Mapper.delete${ClassName}By${pkColumn.capJavaField}(${pkColumn.javaField});
        return removeById(${pkColumn.javaField}) == true ? 1 : 0;
    }
#if($table.sub)

    /**
     * 新增${subTable.functionName}信息
     * 
     * @param ${className} ${functionName}对象
     */
    public void insert${subClassName}(${ClassName} ${className})
    {
        List<${subClassName}> ${subclassName}List = ${className}.get${subClassName}List();
        ${pkColumn.javaType} ${pkColumn.javaField} = ${className}.get${pkColumn.capJavaField}();
        if (StringUtils.isNotNull(${subclassName}List))
        {
            List<${subClassName}> list = new ArrayList<${subClassName}>();
            for (${subClassName} ${subclassName} : ${subclassName}List)
            {
                ${subclassName}.set${subTableFkClassName}(${pkColumn.javaField});
                list.add(${subclassName});
            }
            if (list.size() > 0)
            {
                ${className}Mapper.batch${subClassName}(list);
            }
        }
    }
#end
}
```

**其他必要修改**

集成MP之后，项目中的BaseEntity类中的字段有些会受影响，需要添加如下注解

由于这几个字段，并不会跟数据库中的表字段进行映射，必须要添加@TableField(exist = false)表示，表示该字段不存在于数据库表中

```Java
/**
 * Entity基类
 * 
 * @author ruoyi
 */
public class BaseEntity implements Serializable
{
    private static final long serialVersionUID = 1L;

    /** 搜索值 */
    @JsonIgnore
    @TableField(exist = false)
    private String searchValue;

    /** 请求参数 */
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    @TableField(exist = false)
    private Map<String, Object> params;

}
```

### 4.4、字段自动填充

MyBatis-Plus 提供的字段自动填充功能是一种非常实用的特性，它能够在插入或更新数据库记录时自动填充一些公共字段，如创建时间（createTime）、更新时间（updateTime）、创建人（createBy）、更新人（updateBy）等。这一功能极大地简化了开发过程，减少了重复的代码编写，提高了开发效率。

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_15-06-29.jpg)

官网链接：https://mybatis.plus/guide/auto-fill-metainfo.html（可能打不开，官网本身的问题）

在MybatisPlus中通过两步可以实现这个功能：

1、在实体类中，使用`@TableField`注解，来标明哪些字段是需要自动填充的，并且需要指定填充策略

```Java
package com.zzyl.common.core.domain;

import java.io.Serializable;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.TableField;
import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonInclude;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

/**
 * Entity基类
 * 
 * @author ruoyi
 */
@ApiModel("Entity基类")
public class BaseEntity implements Serializable
{
    private static final long serialVersionUID = 1L;

    /** 搜索值 */
    @JsonIgnore
    @TableField(exist = false)
    private String searchValue;

    /** 创建者 */
    @ApiModelProperty(value = "创建者")
    @TableField(fill = FieldFill.INSERT)
    private String createBy;

    /** 创建时间 */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @ApiModelProperty(value = "创建时间")
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;

    /** 更新者 */
    @ApiModelProperty(value = "更新者")
    @TableField(fill = FieldFill.UPDATE)
    private String updateBy;

    /** 更新时间 */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @ApiModelProperty(value = "更新时间")
    @TableField(fill = FieldFill.UPDATE)
    private Date updateTime;

    /** 备注 */
    @ApiModelProperty(value = "备注")
    private String remark;

    /** 请求参数 */
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    @ApiModelProperty(value = "请求参数")
    @TableField(exist = false)
    private Map<String, Object> params;
    
}
```

2、在zzyl-framework模块中新增MyMetaObjectHandler 来处理自动自动填充

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_15-07-54.jpg)

详细的代码如下：

```Java
package com.zzyl.framework.interceptor;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import com.zzyl.common.core.domain.model.LoginUser;
import com.zzyl.common.utils.SecurityUtils;
import org.apache.commons.lang3.ObjectUtils;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.util.Date;

@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", Date.class, new Date());
        this.strictInsertFill(metaObject, "createBy", String.class, loadUserId()+"");
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", Date.class, new Date());
        this.strictUpdateFill(metaObject, "updateBy", String.class, loadUserId()+"");
    }

    /**
     * 获取当前登录人的ID
     * @return
     */
    public static Long loadUserId(){
        //获取当前登录人的id
        LoginUser loginUser = SecurityUtils.getLoginUser();
        if(ObjectUtils.isNotEmpty(loginUser)){
            return loginUser.getUserId();
        }
        return 1L;
    }
}
```

## 5、Swagger集成

在我们定义的所有controller中，为了更好的体现接口文档，我们都需要添加swagger注解才可以。这些注解我们也可以通过代码的模板来生成，节省我们的开发工作

我们主要的工作就是来修改**controller.java.vm**文件即可，具体的修改的内容如下：

```Java
package ${packageName}.controller;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import java.util.List;
import javax.servlet.http.HttpServletResponse;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import com.zzyl.common.annotation.Log;
import com.zzyl.common.core.controller.BaseController;
import com.zzyl.common.core.domain.AjaxResult;
import com.zzyl.common.enums.BusinessType;
import ${packageName}.domain.${ClassName};
import ${packageName}.service.I${ClassName}Service;
import com.zzyl.common.utils.poi.ExcelUtil;
#if($table.crud || $table.sub)
import com.zzyl.common.core.page.TableDataInfo;
#elseif($table.tree)
#end

/**
 * ${functionName}Controller
 * 
 * @author ${author}
 * @date ${datetime}
 */
@RestController
@RequestMapping("/${moduleName}/${businessName}")
@Api(tags =  "${functionName}相关接口")
public class ${ClassName}Controller extends BaseController
{
    @Autowired
    private I${ClassName}Service ${className}Service;

    /**
     * 查询${functionName}列表
     */
    @PreAuthorize("@ss.hasPermi('${permissionPrefix}:list')")
    @GetMapping("/list")
    @ApiOperation("查询${functionName}列表")
#if($table.crud || $table.sub)
    public TableDataInfo list(${ClassName} ${className})
    {
        startPage();
        List<${ClassName}> list = ${className}Service.select${ClassName}List(${className});
        return getDataTable(list);
    }
#elseif($table.tree)
    public AjaxResult list(${ClassName} ${className})
    {
        List<${ClassName}> list = ${className}Service.select${ClassName}List(${className});
        return success(list);
    }
#end

    /**
     * 导出${functionName}列表
     */
    @PreAuthorize("@ss.hasPermi('${permissionPrefix}:export')")
    @Log(title = "${functionName}", businessType = BusinessType.EXPORT)
    @PostMapping("/export")
    @ApiOperation("导出${functionName}列表")
    public void export(HttpServletResponse response, ${ClassName} ${className})
    {
        List<${ClassName}> list = ${className}Service.select${ClassName}List(${className});
        ExcelUtil<${ClassName}> util = new ExcelUtil<${ClassName}>(${ClassName}.class);
        util.exportExcel(response, list, "${functionName}数据");
    }

    /**
     * 获取${functionName}详细信息
     */
    @PreAuthorize("@ss.hasPermi('${permissionPrefix}:query')")
    @GetMapping(value = "/{${pkColumn.javaField}}")
    @ApiOperation("获取${functionName}详细信息")
    public AjaxResult getInfo(@ApiParam(value = "${functionName}ID", required = true)
                                  @PathVariable("${pkColumn.javaField}") ${pkColumn.javaType} ${pkColumn.javaField})
    {
        return success(${className}Service.select${ClassName}By${pkColumn.capJavaField}(${pkColumn.javaField}));
    }

    /**
     * 新增${functionName}
     */
    @PreAuthorize("@ss.hasPermi('${permissionPrefix}:add')")
    @Log(title = "${functionName}", businessType = BusinessType.INSERT)
    @PostMapping
    @ApiOperation("新增${functionName}")
    public AjaxResult add(@ApiParam(value = "${functionName}实体", required = true) @RequestBody ${ClassName} ${className})
    {
        return toAjax(${className}Service.insert${ClassName}(${className}));
    }

    /**
     * 修改${functionName}
     */
    @PreAuthorize("@ss.hasPermi('${permissionPrefix}:edit')")
    @Log(title = "${functionName}", businessType = BusinessType.UPDATE)
    @PutMapping
    @ApiOperation("修改${functionName}")
    public AjaxResult edit(@ApiParam(value = "${functionName}实体", required = true)  @RequestBody ${ClassName} ${className})
    {
        return toAjax(${className}Service.update${ClassName}(${className}));
    }

    /**
     * 删除${functionName}
     */
    @PreAuthorize("@ss.hasPermi('${permissionPrefix}:remove')")
    @Log(title = "${functionName}", businessType = BusinessType.DELETE)
    @DeleteMapping("/{${pkColumn.javaField}s}")
    @ApiOperation("删除${functionName}")
    public AjaxResult remove(@PathVariable ${pkColumn.javaType}[] ${pkColumn.javaField}s)
    {
        return toAjax(${className}Service.delete${ClassName}By${pkColumn.capJavaField}s(${pkColumn.javaField}s));
    }
}
```

重启服务之后，我们可以预览生成后的效果：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_15-18-37.jpg)

## 6、由Date类型改为LocalDateTime

若依目前提供的代码中，如果遇到日期类型则使用的是Date，由于目前使用比较流行的是LocalDateTime类型，我们也可以把它改为该类型进行使用

> **知识扩展**
>
> Date类型与LocalDateTime的区别？
>
> 1. 不可变性与线程安全性 
>    1. Date一旦创建Date对象，可以通过setTime(long time)进行修改，多线程下存在线程安全问题
>    2. LocalDateTime是不可变的，线程安全
> 2. 时间精度
>    1. Date可以表示到毫秒的时间值
>    2. LocalDateTime可以表示到纳秒的时间值
> 3. 易用性：LocalDateTime的API设计的更现代化，易于使用

在目前代码生成中，其中字段详细这一栏中的Java类型并不支持LocalDateTime类型，所以我们需要在前端也要修改其内容，让它支持LocalDateTime

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_15-20-12.jpg)

打开前端项目，找到src/views/tool/gen/editTable.vue文件，在这个文件添加中以上一行代码，如下：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_15-20-23.jpg)

当前端选择了LocalDateTime类型之后，在后端的代码生成中，需要导入对应的包才可以，后端的代码也需要修改

我们打开VelocityUtils类，这里面有处理导入包的逻辑，找到getImportList方法，在其中添加对应的包导入

```
/**
 * 根据列类型获取导入包
 * 
 * @param genTable 业务表对象
 * @return 返回需要导入的包列表
 */
public static HashSet<String> getImportList(GenTable genTable)
{
    List<GenTableColumn> columns = genTable.getColumns();
    GenTable subGenTable = genTable.getSubTable();
    HashSet<String> importList = new HashSet<String>();
    if (StringUtils.isNotNull(subGenTable))
    {
        importList.add("java.util.List");
    }
    for (GenTableColumn column : columns)
    {
        if (!column.isSuperColumn() && GenConstants.TYPE_DATE.equals(column.getJavaType()))
        {
            importList.add("java.util.Date");
            importList.add("com.fasterxml.jackson.annotation.JsonFormat");
        }
        else if (!column.isSuperColumn() && GenConstants.TYPE_BIGDECIMAL.equals(column.getJavaType()))
        {
            importList.add("java.math.BigDecimal");
        }
        
        //加入下面这些功能
        // 如果字段类型在前端选择的是LocalDateTime，则导入对应的包
        if (!column.isSuperColumn() && "LocalDateTime".equals(column.getJavaType()))
        {
            importList.add("java.time.LocalDateTime");
            //导入这个是为了格式化日期
            importList.add("com.fasterxml.jackson.annotation.JsonFormat");
        }
        
        
        
    }
    return importList;
}
```

修改模块文件domain.java.vm文件，修改的内容如下：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-11-04_15-22-43.jpg)
