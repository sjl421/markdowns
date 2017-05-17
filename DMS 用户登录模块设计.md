# DMS 用户登录模块设计

## 用户登录流程

1. 用户访问 dms-front，dms-front 的filter将过滤当前请求来判断当前用户是否登录，如果没有登录，将跳转到`/dms-front/login.html`界面，输入`用户名、密码、连接串信息`，然后访问dms-front的login controller；
2. dms-front的login controller 将会将当前会话的 `用户名，密码，连接串、sessionid）` 作为request body来向`dms-server`的`login controller`发送请求，来验证当前用户的登录权限是否正确；
3. `dms-server`将在本地mysql数据库中维护一份可以登录的用户表，包括`用户名、密码（加密后）`，如果验证用户名和密码是正确的，
4. ​