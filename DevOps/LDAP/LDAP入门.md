# LDAP 入门

**LDAP** 是一个为查询、浏览和搜索而优化的专业分布式数据库，它成树状结构组织数据，就好象Linux/Unix系统中的文件目录一样。 目录数据库和关系数据库不同，它有优异的读性能，但写性能差，并且没有事务处理、回滚等复杂功能，不适于存储修改频繁的数据。所以目录天生是用来查询的，就好象它的名字一样。 目录服务是由目录数据库和一套访问协议组成的系统。类似以下的信息适合储存在目录中：

- 企业员工和企业客户之类人员信息；
- 公用证书和安全密钥；
- 邮件地址、网址、IP等电脑信息；
- 电脑配置信息。 …

深入到运维的世界，你会发现大部分工具还活在上个世纪，产品设计完全反人类，比如`cn`, `dc`, `dn`, `ou`这样的命名方式，如果不钻研个一天两天，鬼知道它在说什么；`cn`是什么？中国的缩写？你想多了，这和中国没有任何关系。经过一系列这样疯狂的洗脑之后，你才能逐渐明白`LDAP`到底想干什么。抛弃你所有的认知，把自己当成一个什么都不懂的幼儿园孩子，然后我们从头学起`LDAP`。



## LDAP简称对应

1. o– organization（组织-公司）
2. ou – organization unit（组织单元-部门）
3. c - countryName（国家）
4. dc - domainComponent（域名）
5. sn – suer name（真实名称）
6. cn - common name（常用名称）



## 目录树概念

1. **目录树**：在一个目录服务系统中，整个目录信息集可以表示为一个目录信息树，树中的每个节点是一个条目(Entry)。

2. **条目**：每个条目就是一条记录，每个条目有自己的唯一可区别的名称（DN）。

   > LDAP目录的条目（entry）由属性（attribute）的一个聚集组成，并由一个唯一性的名字引用，即**专有名称**（**distinguished name**，DN）。例如，DN能取这样的值：“ou=people,dc=wikipedia,dc=org”。

3. **对象类**：与某个实体类型对应的一组属性，对象类是可以继承的，这样父类的必须属性也会被继承下来（ObjectClass）。
4. **属性**：描述条目的某个方面的信息，一个属性由一个属性类型和一个或多个属性值组成，属性有必须属性和非必须属性(Attribute)。



#### 条目 Entry

条目，也叫记录项，是LDAP中最基本的颗粒，就像字典中的词条，或者是数据库中的记录。通常对LDAP的添加、删除、更改、检索都是以条目为基本对象的。

`dn`：每一个条目都有一个唯一的标识名（distinguished Name ，DN），如 dn：”cn=baby,ou=marketing,ou=people,dc=mydomain,dc=org” 。通过DN的层次型语法结构，可以方便地表示出条目在LDAP树中的位置，通常用于检索。

`rdn`：一般指dn逗号最左边的部分，如cn=baby。它与RootDN不同，RootDN通常与RootPW同时出现，特指管理LDAP中信息的最高权限用户。

`Base DN`：LDAP目录树的最顶部就是根，也就是所谓的“Base DN”，如”dc=mydomain,dc=org”。





#### 属性 Attribute

每个条目都可以有很多属性（Attribute），比如常见的人都有姓名、地址、电话等属性。每个属性都有名称及对应的值，属性值可以有单个、多个，比如你有多个邮箱。

属性不是随便定义的，需要符合一定的规则，而这个规则可以通过schema制定。比如，如果一个entry没有包含在 inetorgperson 这个 schema 中的 `objectClass: inetOrgPerson` ,那么就不能为它指定employeeNumber属性，因为employeeNumber是在inetOrgPerson中定义的。

LDAP为人员组织机构中常见的对象都设计了属性(比如commonName，surname)。下面有一些常用的别名：

| 属性            | 语法             | 描述             | 值(举例)             |
| :-------------- | :--------------- | :--------------- | :------------------- |
| cn              | Directory String | 姓名             | sean                 |
| sn              | Directory String | 姓               | Chow                 |
| ou              | Directory String | 单位（部门）名称 | IT_SECTION           |
| o               | Directory String | 组织（公司）名称 | example              |
| telephoneNumber | Telephone Number | 电话号码         | 110                  |
| objectClass     |                  | 内置属性         | organizationalPerson |



#### 对象类 ObjectClass

对象类是属性的集合，LDAP预想了很多人员组织机构中常见的对象，并将其封装成对象类。比如人员（`person`）含有姓（`sn`）、名（`cn`）、电话(`telephoneNumber`)、密码(`userPassword`)等属性，单位职工(`organizationalPerson`)是人员(`person`)的继承类，除了上述属性之外还含有职务（`title`）、邮政编码（`postalCode`）、通信地址(`postalAddress`)等属性。

通过对象类可以方便的定义条目类型。每个条目可以直接继承多个对象类，这样就继承了各种属性。如果2个对象类中有相同的属性，则条目继承后只会保留1个属性。对象类同时也规定了哪些属性是基本信息，必须含有(Must 活Required，必要属性)：哪些属性是扩展信息，可以含有（May或Optional，可选属性）。

对象类有三种类型：**结构类型（Structural）**、**抽象类型(Abstract)** 和 **辅助类型（Auxiliary）**。

- 结构类型是最基本的类型，它规定了对象实体的基本属性，每个条目属于且仅属于一个结构型对象类。
- 抽象类型可以是结构类型或其他抽象类型父类，它将对象属性中共性的部分组织在一起，称为其他类的模板，条目不能直接集成抽象型对象类。
- 辅助类型规定了对象实体的扩展属性。每个条目至少有一个结构性对象类。

下面是 `inetOrgPerson` 对象类的在 `schema` 中的定义，可以清楚的看到它的父类SUB和可选属性MAY、必要属性MUST(继承自 `organizationalPerson` )，关于各属性的语法则在schema中的attributetype定义。

```
# inetOrgPerson
# The inetOrgPerson represents people who are associated with an
# organization in some way.  It is a structural class and is derived
# from the organizationalPerson which is defined in X.521 [X521].
objectclass     ( 2.16.840.1.113730.3.2.2
  NAME 'inetOrgPerson'
      DESC 'RFC2798: Internet Organizational Person'
  SUP organizationalPerson
  STRUCTURAL
      MAY (
              audio $ businessCategory $ carLicense $ departmentNumber $
              displayName $ employeeNumber $ employeeType $ givenName $
              homePhone $ homePostalAddress $ initials $ jpegPhoto $
              labeledURI $ mail $ manager $ mobile $ o $ pager $
              photo $ roomNumber $ secretary $ uid $ userCertificate $
              x500uniqueIdentifier $ preferredLanguage $
              userSMIMECertificate $ userPKCS12 )
      )
```



#### Schema

对象类（`ObjectClass`）、属性类型（`AttributeType`）、语法（`Syntax`）分别约定了条目、属性、值，他们之间的关系如下图所示。 所以这些构成了模式(Schema)——对象类的集合。 条目数据在导入时通常需要接受模式检查，它确保了目录中所有的条目数据结构都是一致的。





## 示例

一个 Knox 自带的 LDAP 的配置文件如下：

```
version: 1

# Please replace with site specific values
dn: dc=hadoop,dc=apache,dc=org
objectclass: organization
objectclass: dcObject
o: Hadoop
dc: hadoop

# Entry for a sample people container
# Please replace with site specific values
dn: ou=people,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass:organizationalUnit
ou: people

# Entry for a sample end user
# Please replace with site specific values
dn: uid=guest,ou=people,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass:person
objectclass:organizationalPerson
objectclass:inetOrgPerson
cn: Guest
sn: User
uid: guest
userPassword:123456

# entry for sample user admin
dn: uid=admin,ou=people,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass:person
objectclass:organizationalPerson
objectclass:inetOrgPerson
cn: Admin
sn: Admin
uid: admin
userPassword:123456

# entry for sample user sam
dn: uid=sam,ou=people,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass:person
objectclass:organizationalPerson
objectclass:inetOrgPerson
cn: sam
sn: sam
uid: sam
userPassword:123456

# entry for sample user tom
dn: uid=tom,ou=people,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass:person
objectclass:organizationalPerson
objectclass:inetOrgPerson
cn: tom
sn: tom
uid: tom
userPassword:123456

# create FIRST Level groups branch
dn: ou=groups,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass:organizationalUnit
ou: groups
description: generic groups branch

# create the analyst group under groups
dn: cn=analyst,ou=groups,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass: groupofnames
cn: analyst
description:analyst  group
member: uid=sam,ou=people,dc=hadoop,dc=apache,dc=org
member: uid=tom,ou=people,dc=hadoop,dc=apache,dc=org


# create the scientist group under groups
dn: cn=scientist,ou=groups,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass: groupofnames
cn: scientist
description: scientist group
member: uid=sam,ou=people,dc=hadoop,dc=apache,dc=org

# create the admin group under groups
dn: cn=admin,ou=groups,dc=hadoop,dc=apache,dc=org
objectclass:top
objectclass: groupofnames
cn: admin
description: admin group
member: uid=admin,ou=people,dc=hadoop,dc=apache,dc=org
```

























