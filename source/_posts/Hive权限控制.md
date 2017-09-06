---
title: Hive权限控制
date: 2017-02-25 12:54:59
tags: Hive
toc: true
categories: 大数据技术
---
目前hive支持简单的权限管理，默认情况下是不开启，这样所有的用户都具有相同的权限，同时也是超级管理员，也就对hive中的所有表都有查看和改动的权利。Hive本身也不提供创建用户组和用户的命令,Hive用户组和用户即Linux用户组和用户，和hadoop一样，本身不提供用户组和用户管理，只做权限控制。
<!-- more -->
## 1.启用权限控制 ##
开启启身份认证后，任何用户必须被grant privilege才能对实体进行操作。在hive-site.xml文件中进行相应的配置
```
<property>
<name>hive.security.authorization.enabled</name>
<value>true</value>
<description>enable or disable the hive clientauthorization</description>
</property>
<property>
<name>hive.security.authorization.createtable.owner.grants</name>
<value>ALL</value>
<description>the privileges automatically granted to the ownerwhenever a table gets created. An example like "select,drop" willgrant select and drop privilege to the owner of the table</description>
</property>
```
## 2.角色管理 ##
```
--创建和删除角色  
create role role_name;  
drop role role_name;  
--展示所有roles  
show roles  
--赋予角色权限  
grant select on database db_name to role role_name;    
grant select on [table] t_name to role role_name;    
--查看角色权限  
show grant role role_name on database db_name;   
show grant role role_name on [table] t_name;   
--角色赋予用户  
grant role role_name to user user_name  
--回收角色权限  
revoke select on database db_name from role role_name;  
revoke select on [table] t_name from role role_name;  
--查看某个用户所有角色  
show role grant user user_name;
```
## 3.HIVE支持的权限 ##

权限名称 | 含义
---|---
ALL | 所有权限
ALTER | 允许修改元数据（modify metadata data of object）---表信息数据
UPDATE|	允许修改物理数据（modify physical data of object）---实际数据
CREATE|允许进行Create操作
DROP|允许进行DROP操作
INDEX|允许建索引（目前还没有实现）
LOCK|当出现并发的使用允许用户进行LOCK和UNLOCK操作
SELECT|允许用户进行SELECT操作
SHOW_DATABASE|允许用户查看可用的数据库


## 4.与角色相关的表 ##
表名|含义
---|--
Db_privs|记录了User/Role在DB上的权限
Tbl_privs|记录了User/Role在table上的权限
Tbl_col_privs|记录了User/Role在table column上的权限
Roles|记录了所有创建的role
Role_map|记录了User与Role的对应关系

## 5.配置超级管理员 ##
为限制任何用户都可以进行Grant/Revoke操作，提高安全控制，需事先Hive的超级管理员。在hive-site.xml中添加hive.semantic.analyzer.hook配置，并实现自己的权限控制类HiveAdmin。
```
<property>
    <name>hive.semantic.analyzer.hook</name>
    <value>com.hive.HiveAdmin</value>
</property>
```
com.hive.HiveAdmin类代码如下：
```
package com.bigdata.hive;  
import org.apache.hadoop.hive.ql.parse.ASTNode;  
import org.apache.hadoop.hive.ql.parse.AbstractSemanticAnalyzerHook;  
import org.apache.hadoop.hive.ql.parse.HiveParser;  
import org.apache.hadoop.hive.ql.parse.HiveSemanticAnalyzerHookContext;  
import org.apache.hadoop.hive.ql.parse.SemanticException;  
import org.apache.hadoop.hive.ql.session.SessionState;  
public class  HiveAdmin extends AbstractSemanticAnalyzerHook {  
private static String[] admin = {"admin", "root","yangql"};  

@Override  
public ASTNode preAnalyze(HiveSemanticAnalyzerHookContext context,ASTNode ast) throws SemanticException {  
		switch (ast.getToken().getType()) {  
				case HiveParser.TOK_CREATEDATABASE:  
				case HiveParser.TOK_DROPDATABASE:  
				case HiveParser.TOK_CREATEROLE:  
				case HiveParser.TOK_DROPROLE:  
				case HiveParser.TOK_GRANT:  
				case HiveParser.TOK_REVOKE:  
				case HiveParser.TOK_GRANT_ROLE:  
				case HiveParser.TOK_REVOKE_ROLE:  
						    String userName = null;  
						    if (SessionState.get() != null&&SessionState.get().getAuthenticator() != null){  
						        userName=SessionState.get().getAuthenticator().getUserName();  
						    }  
						    if (!admin[0].equalsIgnoreCase(userName) && !admin[1].equalsIgnoreCase(userName)) {  
						        throw new SemanticException(userName + " can't use ADMIN options, except "   
						                            + admin[0]+","+admin[1] +".");  
						    }         
						    break;  
			  default:break;  
}  
    return ast;  
    }  
public static void main(String[] args) throws SemanticException {  
    String[] admin = {"admin", "root"};  
    String userName = "root";  
    for(String tmp: admin){  
        System.out.println(tmp);  
        if (!tmp.equalsIgnoreCase(userName)) {  
            throw new SemanticException(userName + " can't use ADMIN options, except "   
                                + admin[0]+","+admin[1] +".");  
        }         
    }  
}  

}  
```

## 6.Hive 权限管理流程总结 ##
- 创建超级管理员
- 新建linux用户组和用户，也可以在既定用户组下建用户，赋予用户hive目录权限
- 超级管理员进入hive，授权新建用户组和用户的操作权限
- 客户端可以通过新建用户名和密码连接到hive执行授权内的动作
