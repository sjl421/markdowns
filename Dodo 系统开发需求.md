# Dodo 系统开发需求

1. 通过受益部门信息获取部门权签人

   ```
   def get_department_authentication(self):
   ```

2. 判断当前申请人是不是一线技术岗位人员

   ```python
   def is_front_line_staff(self):
   ```

3. 获取当前人所对应的一些信息

   ```python
   # 获取当前申请人所有的个人项目编码
   def get_self_project():

   #获取当前申请人所有的公共项目编码
   def get_public_project(staff):
       
   # 获取当前申请人的所在部门：
   def get_department(staff):
       
   # 获取当前项目的的项目经理：
   def get_project_manager(project_code):
       
   #当前申请人的工号：
   ```

4. many2one：

   eg： 请假单是一个模型（class），

   请假单中有一个申请人这个字段，这个字段就是many2one字段，对应到hr.employee中的员工

   其中一个请假单是数据库中的一条记录，	

   ​