

### 分页显示

#### 1. top方式

```plsql
DECLARE @pagesize INT =10 --页码大小
DECLARE @pageindex INT =3 --当前页

SELECT TOP(@pagesize) *
FROM   SYSES_UserID
WHERE  UserID NOT IN (SELECT TOP(@pagesize*(@pageindex-1)) UserID
                      FROM   SYSES_UserID) ;
```

#### 2. `ROW_NUMBER`方式

```plsql
DECLARE @pagesize INT =10 --页码大小
DECLARE @pageindex INT =3 --当前页
SELECT *
FROM   (SELECT Row_number()
                 OVER(
                   ORDER BY userid) rowid,
               *
        FROM   SYSES_UserID) temp
WHERE  rowid BETWEEN ( @pageindex - 1 ) * @pagesize + 1 AND @pageindex * @pagesize; 

```




### 游标

#### 基本游标

```plsql
--创建游标（scroll：滚动游标，没有scroll，只进）
declare mycur cursor scroll
for select UserID from SYSES_UserID
--打开游标
open mycur
--提前某行数据
fetch first from mycur
fetch last from mycur
fetch absolute 2 from mycur --提取第二行
fetch relative 2 from mycur --当前行下移两行
fetch next from mycur --下移一行
fetch prior from mycur --上移一行

--提取游标数据存入变量
declare @acc varchar(10)
fetch last from mycur into @acc
select * from SYSES_UserID where UserID = @acc
--关闭游标
close mycur
--删除游标
deallocate mycur

```

#### 遍历游标*	

> 难点：缺少数据一行数据

```plsql
--创建游标（scroll：滚动游标，没有scroll，只进）
declare mycur cursor scroll
for select UserID from SYSES_UserID
--打开游标
open mycur
--提取游标数据存入变量
declare @acc varchar(10)
--遍历游标
fetch absolute 1 from mycur into @acc
--@@FETCH_STATUS：0提取成功，-1失败，-2不存在
while @@FETCH_STATUS = 0 
	begin
		print '提取成功：'+@acc
		fetch next from mycur
	end
--关闭游标
close mycur
--删除游标
deallocate mycur

```

#### 利用游标进行数据编辑

```plsql
--创建游标（scroll：滚动游标，没有scroll，只进）
declare mycur cursor scroll
for select UserID from SYSES_UserID
--打开游标
open mycur
--提取游标数据存入变量
declare @acc varchar(88)
--遍历游标
fetch absolute 3 from mycur
update SYSES_UserID set GroupID = 'CB' WHERE CURRENT OF mycur
fetch absolute 4 from mycur into @acc
print @acc
delete from SYSES_UserID WHERE CURRENT OF mycur
--关闭游标
close mycur
--删除游标
deallocate mycur

```

#### 多列游标

```plsql
--创建游标（scroll：滚动游标，没有scroll，只进）
declare mycur cursor scroll
for select UserID,UserName,PassWord from SYSES_UserID
--打开游标
open mycur
--提取游标数据存入变量
declare @UserID varchar(22)
declare @UserName varchar(22)
declare @PassWord varchar(33)
--遍历游标
fetch absolute 1 from mycur into @UserID,@UserName,@PassWord--提取第一行
while @@FETCH_STATUS =0
	begin
		print '提取成功！'+@UserID
		fetch next from mycur into @UserID,@UserName,@PassWord
	end

--关闭游标
close mycur
--删除游标
deallocate mycur

```

### 函数

#### 1. 表值函数

##### 1. 标准写法

```plsql
USE [DEVELOP]

GO

-- ================================================
-- Template generated from Template Explorer using:
-- Create Multi-Statement Function (New Menu).SQL
--
-- Use the Specify Values for Template Parameters 
-- command (Ctrl-Shift-M) to fill in the parameter 
-- values below.
--
-- This block of comments will not be included in
-- the definition of the function.
-- ================================================
SET ANSI_NULLS ON

GO

SET QUOTED_IDENTIFIER ON

GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
alter FUNCTION F_TEST_TABLE (@USERID NVARCHAR(8))
RETURNS
@TEST_TABLE TABLE
(
USERID VARCHAR(9),
USERNAME VARCHAR(22)
)
AS
BEGIN
INSERT INTO @TEST_TABLE
	SELECT UserID,UserName FROM SYSES_UserID where UserID = @USERID;	
	RETURN ;
END
GO 

--用法
select * from F_TEST_TABLE('Fun');
```

##### 2. 简化写法

函数体内只能有RETURN+SQL查询结果

```plsql
CREATE FUNCTION F_test_table (@USERID NVARCHAR(8))
RETURNS TABLE
AS
    RETURN
      SELECT UserID,
             UserName
      FROM   SYSES_UserID
      WHERE  UserID = @USERID;

GO 

--用法
select * from F_TEST_TABLE('Fun');
```



#### 2. 标量值函数

##### 无参数

```plsql
DROP FUNCTION aaa_test;

CREATE FUNCTION Aaa_test()
returns VARCHAR(max)
AS
  BEGIN
      DECLARE @userid VARCHAR(max)

      SELECT @userid = (SELECT 'fzy')

      RETURN @userid
  END

SELECT dbo.Aaa_test(); --fzy
```

##### 带参数

```plsql

DROP FUNCTION Aaa_test;

CREATE FUNCTION Aaa_test(@userid VARCHAR(max))
returns VARCHAR(30)
AS
  BEGIN
      DECLARE @names VARCHAR(max)
      SELECT @names = UserName
      FROM   SYSES_UserID
      WHERE  UserID = @userid
      RETURN @names
  END

SELECT dbo.Aaa_test('fzy'); --樊正玉

```



### 存储过程

> https://www.cnblogs.com/atlj/p/11184952.html



#### 有参数

```plsql
USE [DEVELOP]

GO

/****** Object:  StoredProcedure [dbo].[Fk_Reservation_P]    Script Date: 2021/1/6 18:30:22 ******/
SET ANSI_NULLS ON

GO

SET QUOTED_IDENTIFIER ON

GO

-- =============================================
-- Author:		ID200
-- Create date: 2021-01-06 18:13
-- Description:	访客预约提交申请
-- Usage："EXEC [dbo].[Fk_Reservation_P] 'FKNO',null,null,'','','','36.6','','','','','','','','','','','','','','',''; "
-- =============================================
ALTER PROCEDURE [dbo].[Fk_reservation_p] @FkNo           NCHAR(12),
                                         @IDCard         NCHAR(18),
                                         @UName          NCHAR(8),
                                         @TelNumber      NCHAR(12),
                                         @Mail           NCHAR(18),
                                         @VisitorUnit    NCHAR(10),
                                         @Temperature    NUMERIC(3, 1),
                                         @SourceAddress  NCHAR(16),
                                         @VisitStartTime NCHAR(14),
                                         @EndTimeVisit   NCHAR(14),
                                         @SqTime         NCHAR(14),
                                         @SqTheme        NCHAR(20),
                                         @SqContent      NCHAR(50),
                                         @DoorNumber     NCHAR(12),
                                         @PeopleNums     NCHAR(2),
                                         @ReceptionUnit  NCHAR(20),
                                         @Reception      NCHAR(8),
                                         @HealthCode     TEXT,
                                         @Other1         NCHAR(10),
                                         @Other2         NCHAR(10),
                                         @VisitorType    NCHAR(1),
                                         @Platform       NCHAR(3)
AS
  BEGIN
      DECLARE @IDTest VARCHAR(14);

      SET NOCOUNT ON;

      BEGIN TRY---------------------开始捕捉异常
          EXEC [dbo].[Xp_getno]
            'FKNO',
            @IDTest OUTPUT;

          SELECT @FkNo = @IDTest;

          --测试数据
          --SET @IDCard = '420303198012120101';
          SET @UName = '五条人';
          SET @SqTime =Replace(Replace(Replace(CONVERT(VARCHAR, Getdate(), 120), '-', ''), ' ', ''), ':', '');

          BEGIN TRAN

          INSERT INTO Fk_Reservation
                      (FkNo,
                       IDCard,
                       UName,
                       TelNumber,
                       Mail,
                       VisitorUnit,
                       Temperature,
                       SourceAddress,
                       VisitStartTime,
                       EndTimeVisit,
                       SqTime,
                       SqTheme,
                       SqContent,
                       DoorNumber,
                       PeopleNums,
                       ReceptionUnit,
                       Reception,
                       HealthCode,
                       Other1,
                       Other2,
                       VisitorType,
                       Platform)
          VALUES      (@FkNo,
                       @IDCard,
                       @UName,
                       @TelNumber,
                       @Mail,
                       @VisitorUnit,
                       @Temperature,
                       @SourceAddress,
                       @VisitStartTime,
                       @EndTimeVisit,
                       @SqTime,
                       @SqTheme,
                       @SqContent,
                       @DoorNumber,
                       @PeopleNums,
                       @ReceptionUnit,
                       @Reception,
                       @HealthCode,
                       @Other1,
                       @Other2,
                       @VisitorType,
                       @Platform);

          COMMIT TRAN;
      END TRY-----------结束捕捉异常
      BEGIN CATCH------------有异常被捕获
          ROLLBACK TRAN;
          INSERT INTO dbo.LogTable
          VALUES      (Error_number(),--错误号
                       Error_severity(),--严重性
                       Error_state(),--错误状态号
                       Error_procedure(),--出现错误的存储过程或 触发器的名称
                       Error_line(),--导致错误的例程中的行号
                       Error_message()); --错误消息的完整文本
          PRINT '执行错误，请联系管理员！';
      --RETURN '执行错误，请联系管理员！';
      END CATCH--------结束异常处理
  END 

--执行方法
EXEC [dbo].[Fk_Reservation_P] 'FKNO',null,null,'','','','36.6','','','','','','','','','','','','','','',''; 
```

#### 无参数

#### 调用方法

```plsql
DECLARE @IDTest varchar(14);
EXEC [dbo].[xp_GetNo] 'FKNO',@IDTest OUTPUT; 
SELECT  @IDTest;
```

