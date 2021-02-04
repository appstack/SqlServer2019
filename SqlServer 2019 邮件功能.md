### SqlServer 2019 发邮件配置

#### 创建配置文件

```plsql
EXEC sp_configure 'show advanced options',1
RECONFIGURE WITH override
go

EXEC sp_configure 'database mail xps',1
RECONFIGURE WITH override
go

--创建邮件帐户信息
EXEC msdb.dbo.Sysmail_add_account_sp
  @account_name ='Fk_Admin',-- 邮件帐户名称   
  @email_address ='fanzy@intmes.com',-- 发件人邮件地址    
  @display_name ='访客预约系统',-- 发件人姓名 
  @MAILSERVER_NAME = 'smtp.exmail.qq.com',-- 邮件服务器地址
  @PORT =25,-- 邮件服务器端口 
  @USERNAME = 'fanzy@intmes.com',-- 用户名 
  @PASSWORD = '****************' -- 密码  
GO

--数据库配置文件
EXEC msdb.dbo.Sysmail_add_profile_sp
  @profile_name = 'SQLServer_2019',-- 配置名称 
  @description = '访客预约中心配置文件' -- 配置描述
go

--用户和邮件配置文件相关联
EXEC msdb.dbo.Sysmail_add_profileaccount_sp
  @profile_name = 'SQLServer_2019',-- 配置名称
  @account_name = 'Fk_Admin',-- 邮件帐户名称    
  @sequence_number = 1 -- account 在 profile 中顺序（默认是1）
go 


--配置查看
select * from msdb.dbo.sysmail_account;
select * from msdb.dbo.sysmail_profile;
--delete from msdb.dbo.sysmail_account
--delete from msdb.dbo.sysmail_profile
```

#### 发邮件

```plsql
DECLARE @emailcontent NVARCHAR(2000) = '这是从 FZY 上的数据库邮件发出的测试电子邮件。'; --邮件内容
DECLARE @emailsubject NVARCHAR(200) ='03数据库邮件测试'; --邮件主题
DECLARE @emailadress NVARCHAR(100); --邮件发送地址
DECLARE @error INT; --错误数
DECLARE @logfield NVARCHAR(max); --日志字段内容
DECLARE @num INT; --数据行数     

EXEC msdb.dbo.Sp_send_dbmail
  @profile_name ='SQLServer_2019',--配置文件名称
  @recipients='1001@qq.com',--收件email地址
  @copy_recipients = '1001@qq.com;1002@qq.com',--抄送email地址
  @subject= @emailsubject,--邮件主题
  @body=@emailcontent,--邮件正文内容
  @body_format='html' --邮件内容格式


```

#### 发送结果

```sql
select * from msdb.dbo.sysmail_allitems; --全部消息
select * from msdb.dbo.sysmail_faileditems; --失败状态的消息
select * from msdb.dbo.sysmail_unsentitems; --看未发送的消息
select * from msdb.dbo.sysmail_sentitems; --查看已发送的消息

select * from msdb.dbo.sysmail_event_log; --邮件服务器日记

```

