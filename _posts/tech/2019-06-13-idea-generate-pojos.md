---
layout: post
category: Tech
title: 使用 IntelliJ IDEA 快速生成项目代码
tags: [java,notes]
excerpt: 通过Idea的"Generate POJOs.groovy"的快速生成项目模版代码，可极大的提高开发效率
---

{% include JB/setup %}

前几年写了篇文章：[Java Web开发时快速生成模版代码](/tech/2017/11/01/Java-web-generate-code)，是通过"git patch"文件方式来生成项目代码，虽然比复制粘贴的方式方便不少，但是还有比较繁琐，最近发现"IntelliJ IDEA (Ultimate Edition)"可根据数据库快速生成"POJO"代码，如下图所示:

![alt](https://d3nmt5vlzunoa1.cloudfront.net/datagrip/files/2018/02/ContextMenu.png)

![alt](https://d3nmt5vlzunoa1.cloudfront.net/datagrip/files/2018/02/ScriptItself.png)

既然"POJO"代码可以生成，那"Controller", "Service"之类的代码也应该可以通过类似的方式生成。

我们可以修改`Generate POJOs.groovy`文件，改成我们需要的模版即可。

本例以我写的 [chaos-server](https://github.com/cofcool/chaos-server) 框架为基础。

示例项目为 "Spring Boot" 项目, 数据库方面使用 "JPA", 目录结构为

```
├── controller
├── repository
│   └── entity
└── service
    └── impl
```

示例代码:

```groovy
import com.intellij.database.model.DasTable
import com.intellij.database.model.DasColumn
import com.intellij.database.model.ObjectKind
import com.intellij.database.util.Case
import com.intellij.database.util.DasUtil


/*
 * Available context bindings:
 *   SELECTION   Iterable<DasObject>
 *   PROJECT     project
 *   FILES       files helper
 */
basePackageName = "net.cofcool.test.server"
packageName = basePackageName + ".repository.entity"
repositoryPackageName = basePackageName + ".repository"
servicePackageName = basePackageName + ".service"
serviceImplPackageName = basePackageName + ".service.impl"
ControllerPackageName = basePackageName + ".controller"

allTables=new HashMap<>()

schemeName = "test"

typeMapping = [
        (~/(?i)int/)                      : "Integer",
        (~/(?i)long/)                     : "Long",
        (~/(?i)number/)                   : "String",
        (~/(?i)float|double|decimal|real/): "Double",
        (~/(?i)datetime|timestamp/)       : "java.sql.Timestamp",
        (~/(?i)date/)                     : "java.sql.Date",
        (~/(?i)time/)                     : "java.sql.Time",
        (~/(?i)/)                         : "String"
]

// 生成 entity
FILES.chooseDirectoryAndSave("Choose entity directory", "Choose where to store generated files") { dir ->
    SELECTION.filter { it instanceof DasTable && it.getKind() == ObjectKind.TABLE }.each {
        generate(it, dir)
    }
}

// 生成 repository
FILES.chooseDirectoryAndSave("Choose repository directory", "Choose where to store generated files") { dir ->
  allTables.each { className, idType ->
    new File(dir, className + "Repository.java").withPrintWriter { out ->
      generateRepository(out, className, idType)
    }
  }
}

// 生成 service
FILES.chooseDirectoryAndSave("Choose service directory", "Choose where to store generated files") { dir ->
  allTables.each { className, idType ->
    new File(dir, className + "Service.java").withPrintWriter { out ->
      generateService(out, className, idType)
    }
  }
}

// 生成 service 实现
FILES.chooseDirectoryAndSave("Choose service impl directory", "Choose where to store generated files") { dir ->
  allTables.each { className, idType ->
    new File(dir, className + "ServiceImpl.java").withPrintWriter { out ->
      generateServiceImpl(out, className, idType)
    }
  }
}

// 生成 controller
FILES.chooseDirectoryAndSave("Choose  controller directory", "Choose where to store generated files") { dir ->
    allTables.each { className, idType ->
        new File(dir, className + "Controller.java").withPrintWriter { out ->
            generateController(out, className, idType)
        }
    }
}


def generate(table, dir) {
    className = javaName(table.getName(), true)
    def fields = calcFields(table, className)
    allTables.put(className, table)
    new File(dir, className + ".java").withPrintWriter { out ->
        generate(out, className, fields,table)
    }
}

def generate(out, className, fields, table) {
    out.println "package $packageName ;"
    out.println ""
    out.println '''
import java.io.Serializable;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;
'''
    out.println ""
    out.println ""
    out.println ""
    generateComment(out, table.getComment())
    out.println ""
    out.println "@Table(name =\"" + table.getName() + "\", schema = \""  + schemeName +  "\")"
    out.println "@Entity"
    out.println "@Getter"
    out.println "@Setter"
    out.println "@NoArgsConstructor"
    out.println "@ToString"
    out.println "public class $className  implements Serializable {"
    out.println ""
    out.println ""
    out.println genSerialID()
    out.println ""
    fields.each() {
        out.println "    /**"
        out.println "     * " + it.comment
        out.println "     */"

        if (it.annos.size() >0)
        {
            it.annos.each() {
                out.println "    ${it}"
            }
        }
        out.println "    private ${it.type} ${it.name};"
        out.println ""
    }
    out.println ""

    out.println "}"
}

def generateRepository(out, className, idType) {
    out.println "package $repositoryPackageName ;"
    out.println ""
    out.println "import net.cofcool.test.server.repository.entity.$className;"
    out.println "import org.springframework.data.jpa.repository.JpaRepository;"
    out.println "import org.springframework.data.jpa.repository.JpaSpecificationExecutor;"
    out.println ""
    out.println ""
    generateComment(out)
    out.println "public interface " + className + "Repository extends JpaRepository<$className, $idType>, JpaSpecificationExecutor<$className> {"
    out.println ""
    out.println "}"
}

private void generateComment(out, comment = null) {
    out.println "/**"
    out.println " * " + comment
    out.println " *"
    out.println " * <p>Date: " + new java.util.Date().toString() + "</p>"
    out.println " */"
}

// controller 使用的注释是为了生成 YApi 文档，并不是标准的 Java Doc 格式
def generateController(out, className, table) {
    def lit = toLowerCaseFirstOne(className)
    out.println "package $ControllerPackageName;"
    out.println ""
    out.println '''
import javax.annotation.Resource;
import net.cofcool.chaos.server.common.core.Message;
import net.cofcool.chaos.server.common.core.Page;
import net.cofcool.chaos.server.common.core.Result.ResultState;
import net.cofcool.chaos.server.common.util.ValidationGroups.Delete;
import net.cofcool.chaos.server.common.util.ValidationGroups.Insert;
import net.cofcool.chaos.server.common.util.ValidationGroups.Select;
import net.cofcool.chaos.server.common.util.ValidationGroups.Update;
import net.cofcool.chaos.server.core.annotation.Api;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
'''
    out.println "import net.cofcool.test.server.repository.entity.$className;"
    out.println "import $servicePackageName" + "." + className + "Service;"
    out.println ""
    out.println ""
    out.println ""
    out.println "/**"
    out.println " * @description: " + table.getComment()
    out.println " * @menu " + table.getComment()
    out.println " *"
    out.println " * <p>Date: " + new java.util.Date().toString() + "</p>"
    out.println " */"
    out.println "@Api"
    out.println "@RestController"
    out.println "@RequestMapping(value = \"/$lit\", method = RequestMethod.POST)"
    out.println "public class " + className + "Controller {"
    out.println ""
    out.println "    @Resource"
    out.println "    private " + className + "Service " + lit + "Service;"
    out.println ""
    out.println ""
    out.println "    /**"
    out.println "     * @description: " + table.getComment() + "分页查询"
    out.println "     * @param: [Page<" + className + ">]"
    out.println "     * @menu " + table.getComment()
    out.println "     * @return: Message"
    out.println "     * @data: "+ new java.util.Date().toString()
    out.println "     */"
    out.println "    @PostMapping(\"/query\")"
    out.println "    public Message query(@RequestBody Page<$className> page) {"
    out.println "        return " + lit + "Service.query(page, page.getCondition(" + className + ".class)).result();"
    out.println "    }"
    out.println ""
    out.println "    /**"
    out.println "     * @description: " + table.getComment() + "添加"
    out.println "     * @param: [" + className + "]"
    out.println "     * @menu " + table.getComment()
    out.println "     * @return: Message<$className>"
    out.println "     */"
    out.println "    @PostMapping(\"/add\")"
    out.println "    public Message<$className> add(@RequestBody @Validated(Insert.class) $className entity) {"
    out.println "        return " + lit + "Service.add(entity).result();"
    out.println "    }"
    out.println ""
    out.println "    /**"
    out.println "     * @description: " + table.getComment() + "修改"
    out.println "     * @param: [" + className + "]"
    out.println "     * @menu " + table.getComment()
    out.println "     * @return: Message<$className>"
    out.println "     * @data: "+ new java.util.Date().toString()
    out.println "     */"
    out.println "    @PostMapping(\"/update\")"
    out.println "    public Message update(@RequestBody @Validated(Update.class) $className entity) {"
    out.println "        return " + lit + "Service.update(entity).result();"
    out.println "    }"
    out.println ""
    out.println "    /**"
    out.println "     * @description: " + table.getComment() + "详情"
    out.println "     * @param: [" + className + "]"
    out.println "     * @menu " + table.getComment()
    out.println "     * @return: Message<$className>"
    out.println "     * @data: "+ new java.util.Date().toString()
    out.println "     */"
    out.println "    @PostMapping(\"/detail\")"
    out.println "    public Message detail(@RequestBody @Validated(Select.class) $className entity) {"
    out.println "        return " + lit + "Service.queryById(entity).result();"
    out.println "    }"
    out.println ""
    out.println "    /**"
    out.println "     * @description: " + table.getComment() + "全部查询"
    out.println "     * @param: [" + className + "]"
    out.println "     * @menu " + table.getComment()
    out.println "     * @return: Message<List<$className>>"
    out.println "     * @data: "+ new java.util.Date().toString()
    out.println "     */"
    out.println "    @PostMapping(\"/queryAll\")"
    out.println "    public Message queryAll(@RequestBody $className entity) {"
    out.println "        return " + lit + "Service.queryAll(entity).result();"
    out.println "    }"
    out.println ""
    out.println "    /**"
    out.println "     * @description: " + table.getComment() + "删除"
    out.println "     * @param: [" + className + "]"
    out.println "     * @menu " + table.getComment()
    out.println "     * @return: Message<$className>"
    out.println "     * @data: "+ new java.util.Date().toString()
    out.println "     */"
    out.println "    @PostMapping(\"/delete\")"
    out.println "    public ResultState delete(@RequestBody @Validated(Delete.class) $className entity) {"
    out.println "        return " + lit + "Service.delete(entity);"
    out.println "    }"
    out.println ""
    out.println ""
    out.println "}"
}

def toLowerCaseFirstOne(s){
    if(Character.isLowerCase(s.charAt(0)))
        return s;
    else
        return (new StringBuilder()).append(Character.toLowerCase(s.charAt(0))).append(s.substring(1)).toString();
}

def generateService(out, className, idType) {
    out.println "package $servicePackageName;"
    out.println ""
    out.println "import net.cofcool.test.server.repository.entity.$className;"
    out.println "import net.cofcool.test.server.api.BaseService;"
    out.println ""
    out.println ""
    out.println ""
    generateComment(out)
    out.println "public interface " + className + "Service extends BaseService<$className> {"
    out.println ""
    out.println "}"
}

def generateServiceImpl(out, className, idType) {
    out.println "package $serviceImplPackageName;"
    out.println ""
    out.println '''
import net.cofcool.chaos.server.common.core.Page;
import net.cofcool.chaos.server.data.jpa.support.Paging;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.stereotype.Service;
import net.cofcool.test.server.api.BaseServiceImpl;
'''
    out.println "import $packageName" + "." + className + ";"
    out.println "import $repositoryPackageName" + "." + className + "Repository;"
    out.println "import $servicePackageName" + "." + className + "Service;"
    out.println ""
    out.println ""
    out.println ""
    generateComment(out)
    out.println "@Service"
    out.println "public class " + className + "ServiceImpl extends BaseServiceImpl<$className, $idType, $className" + "Repository>  implements " + className + "Service {"
    out.println "    @Override"
    out.println "    protected Object queryWithSp(Specification<$className> sp, Page<$className> condition, $className entity) {"
    out.println "        return getJpaRepository().findAll(sp, Paging.getPageable(condition));"
    out.println "    }"
    out.println ""
    out.println "    @Override"
    out.println "    protected $idType getEntityId($className entity) {"
    out.println "        return entity.getId();"
    out.println "    }"
    out.println ""
    out.println "}"
}

def calcFields(table, javaName) {
    DasUtil.getColumns(table).reduce([]) { fields, col ->
        def spec = Case.LOWER.apply(col.getDataType().getSpecification())
        def typeStr = typeMapping.find { p, t -> p.matcher(spec).find() }.value
        def comm = [
                name : columnName(col.getName()),
                type : typeStr,
                annos: ["@Column(name = \"" + col.getName() + "\" )"],
                comment: col.getComment()
        ]
        if (table.getColumnAttrs(col).contains(DasColumn.Attribute.PRIMARY_KEY)) {
            comm.annos += ["@Id"]
            comm.annos += ["@GeneratedValue(strategy = GenerationType.IDENTITY)"]
        }
        fields += [comm]
    }
}


def javaName(str, capitalize) {
    def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
            .collect { Case.LOWER.apply(it).capitalize() }
            .subList(1, 2) // 去除表前缀, 例如 user_
            .join("")
            .replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
    capitalize || s.length() == 1 ? s : Case.LOWER.apply(s[0]) + s[1..-1]
}

def columnName(str) {
    def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
            .collect { Case.LOWER.apply(it).capitalize() }
            .join("")
            .replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
    s.length() == 1? s : Case.LOWER.apply(s[0]) + s[1..-1]
}

static String genSerialID()
{
    return "    private static final long serialVersionUID =  " + Math.abs(new Random().nextLong())+"L;"
}
```