# 用户数据管理

## 管理用户

~~~sql
/*查询用户*/
SELECT * FROM user

/*创建用户*/
CREATE USER '用户名'@ '主机名' IDENTIFUED BY '密码'

/*修改用户密码*/
ALTER USER '用户名'@ '主机名' IDENTIFUED WITH mysql_native_password BY '新密码'

/*删除用户*/
DROPUSER '用户名'@ '主机名'
~~~

主机名可以用%通配符，表示任意

## 权限控制

~~~sql
/*查询权限*/
SHOW GRANTS FOR '用户名'@ '主机名'

/*授予权限*/
GRANT 权限列表 ON 数据库名.表名 TO '用户名'@ '主机名'

/*撤销权限*/
REVOKE 权限列表 ON 数据库名.表名 FROM '用户名'@ '主机名'
~~~

