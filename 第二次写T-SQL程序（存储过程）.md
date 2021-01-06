

```plsql
USE [DEVELOP]

GO

/****** Object:  StoredProcedure [dbo].[Fk_Reservation_P]    Script Date: 2021/1/6 21:51:57 ******/
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
          SET @IDCard = '420303198012120101';
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

          IF @@ROWCOUNT = 1
            BEGIN
                SELECT '恭喜，预约成功！请耐心等待审核结果。';
            END
          ELSE
            BEGIN
                SELECT '数据提交失败！';
            END

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
          
          --PRINT '执行错误，请联系管理员！';
          SELECT '系统错误，请联系管理员！';
      END CATCH--------结束异常处理
  END 

```



