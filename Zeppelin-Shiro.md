# 配置Zeppelin使用Apache Shiro进行鉴权（多用户登录）

本人使用的Ambari搭建的Hadoop集群，Zeppelin也是Ambari管理的集群中的一部分。

Ambari安装集群方法见本人的另外一篇文章[Ambari安装Hadoop集群教程](http://www.kevinsui.com/posts/ambari_installation/)

Zeppelin有`ini`、`Active Directory`、`LDAP`、`PAM`、`ZeppelinHub`、`Apache Shiro Realms（JndiLdapRealm、JdbcRealm or create our own）`六种鉴权方式
具体介绍见[Apache Shiro authentication for Apache Zeppelin](https://zeppelin.apache.org/docs/0.8.0-SNAPSHOT/security/shiroauthentication.html#configure-realm-optional)

*ZeppelinHub 经初步测试，貌似因ZeppelinHub官网域名更换，导致无法使用*

**本文仅介绍`LADP`、`JdbcRealm`两种鉴权方式**

## ladp与jdbc对比

1. 使用ldapRealm需要安装Knox，然后启动Demo LDAP服务，用户需在配置文件配置，添加用户需重启服务。
2. 使用jdbcRealm需安装mysql数据库，**可动态添加用户**。

如需动态添加用户，推荐使用jdbcRealm。

## jdbcRealm使用教程
### 创建数据库和表
``` sql
DROP DATABASE IF EXISTS shiro;
CREATE DATABASE shiro
	CHARACTER SET utf8
	COLLATE utf8_general_ci;

DROP TABLE IF EXISTS roles_permissions;
CREATE TABLE roles_permissions (
  id BIGINT(20) NOT NULL AUTO_INCREMENT,
  role_name VARCHAR(100) DEFAULT NULL,
  permission VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (id),
  UNIQUE INDEX idx_roles_permissions (role_name, permission)
)
ENGINE = INNODB
AUTO_INCREMENT = 2
AVG_ROW_LENGTH = 16384
CHARACTER SET utf8
COLLATE utf8_general_ci;

DROP TABLE IF EXISTS user_roles;
CREATE TABLE user_roles (
  id BIGINT(20) NOT NULL AUTO_INCREMENT,
  username VARCHAR(100) DEFAULT NULL,
  role_name VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (id),
  UNIQUE INDEX idx_user_roles (username, role_name)
)
ENGINE = INNODB
AUTO_INCREMENT = 2
AVG_ROW_LENGTH = 16384
CHARACTER SET utf8
COLLATE utf8_general_ci;

DROP TABLE IF EXISTS users;
CREATE TABLE users (
  id BIGINT(20) NOT NULL AUTO_INCREMENT,
  username VARCHAR(100) DEFAULT NULL,
  password VARCHAR(100) DEFAULT NULL,
  password_salt VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (id),
  UNIQUE INDEX idx_users_username (username)
)
ENGINE = INNODB
AUTO_INCREMENT = 3
AVG_ROW_LENGTH = 8192
CHARACTER SET utf8
COLLATE utf8_general_ci;
```

### Zeppelin Shiro.ini 配置文件
``` sh
[users]
# List of users with their password allowed to access Zeppelin.
# To use a different strategy (LDAP / Database / ...) check the shiro doc at http://shiro.apache.org/configuration.html#Configuration-INISections
admin = admin, admin
user1 = user1, role1, role2
user2 = user2, role3
user3 = user3, role2

# Sample LDAP configuration, for user Authentication, currently tested for single Realm
[main]
#使用ldap或jdbc其中一个

###########################ldap start###########################
# ldapRealm = org.apache.zeppelin.realm.LdapGroupRealm
# ldapRealm.contextFactory.environment[ldap.searchBase] = dc=hadoop,dc=apache,dc=org
# ldapRealm.contextFactory.url = ldap://slave1:33389
# ldapRealm.userDnTemplate = uid={0},ou=people,dc=hadoop,dc=apache,dc=org
# ldapRealm.contextFactory.authenticationMechanism = SIMPLE

# ldapRealm.contextFactory.systemUsername = uid=sam,ou=people,dc=hadoop,dc=apache,dc=org
# ldapRealm.contextFactory.systemPassword = sam-password
###########################ldap end###########################


###########################jdbc start###########################
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
# jdbcRealm.authenticationQuery = SELECT password from users where username = ?
# jdbcRealm.userRolesQuery = select role from userroles where userID = (select id FROM user WHERE username = ?)

ds = com.mysql.jdbc.jdbc2.optional.MysqlDataSource
ds.serverName = rm-xxxxxxxxxxxxx.mysql.rds.aliyuncs.com
ds.user = user
ds.password = password
ds.databaseName = db
jdbcRealm.dataSource= $ds

passwordMatcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher
jdbcRealm.credentialsMatcher = $passwordMatcher
###########################jdbc end###########################


### A sample PAM configuration
#pamRealm=org.apache.zeppelin.realm.PamRealm
#pamRealm.service=sshd


sessionManager = org.apache.shiro.web.session.mgt.DefaultWebSessionManager
### If caching of user is required then uncomment below lines
cacheManager = org.apache.shiro.cache.MemoryConstrainedCacheManager
securityManager.cacheManager = $cacheManager

securityManager.sessionManager = $sessionManager
# 86,400,000 milliseconds = 24 hour
securityManager.sessionManager.globalSessionTimeout = 86400000
shiro.loginUrl = /api/login

[roles]
role1 = *
role2 = *
role3 = *
admin = *

[urls]
# This section is used for url-based security.
# You can secure interpreter, configuration and credential information by urls. Comment or uncomment the below urls that you want to hide.
# anon means the access is anonymous.
# authc means Form based Auth Security
# To enfore security, comment the line below and uncomment the next one
/api/version = anon
#/api/interpreter/** = authc, roles[admin]
#/api/configurations/** = authc, roles[admin]
#/api/credential/** = authc, roles[admin]
#/** = anon
/** = authc
```

### 添加数据

密码需要使用SHA-256方式加密后存入数据库

### 重启Zeppelin

KO！

## ldapRealm使用教程

1. 安装Knox，启动Demo LDAP服务

2. 修改Shiro.ini配置文件，添加上边ldap中的内容，重启Zeppelin

   其中 ldapRealm.contextFactory.url = ldap://slave1:33389的slave1改为Knox所在服务器的ip

具体教程会在[Kevin's Blog(www.kevinsui.com)](www.kevinsui.com)中更新。


## 在Zeppelin中，用户登录后，在每个notebook中设置访问权限。
