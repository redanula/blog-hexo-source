title: SQL--数据导入
date: 2017-08-01 14:34:56
tags: SQL
---

之前写API及处理后台数据的时候写了一些脚本，记录一下。

快捷数据表操作的存储过程脚本。eg: exec 'Z_InitData','目标表T1','源表T2'

        Create PROCEDURE Z_InitData
        	@tbname nvarchar(100),
        	@fromtbname nvarchar(100)
        AS
        BEGIN
        	-- SET NOCOUNT ON added to prevent extra result sets from
        	-- interfering with SELECT statements.
        	SET NOCOUNT ON;
            declare @sql nvarchar(max)
        	declare @col nvarchar(max)
        	set @sql = 'select @col = stuff((select '',''+name from syscolumns where id=object_id( '''+ @tbname +''') order by name for xml path('''')),1,1,'''')'  
        	exec sp_executesql @sql,N'@col nvarchar(max) output',@col output
        	set @sql = ' truncate table '+ @tbname +';'
        	set @sql = @sql + ' set IDENTITY_INSERT '+ @tbname +' on ;'
        	set @sql = @sql + ' insert into '+ @tbname + '('+ @col +') select '+@col+ ' from ' + @fromtbname
        	set @sql = @sql + '; set IDENTITY_INSERT '+ @tbname +' off;'
        	print(@sql)
        	exec(@sql)
        END

---
打印表字段，方便insert。eg: exec 'Z_Print','目标表T1'

        ALTER PROCEDURE Z_Print
        	@tbname nvarchar(100) ='TTT'
        AS
        BEGIN
        	-- SET NOCOUNT ON added to prevent extra result sets from
        	-- interfering with SELECT statements.
        	SET NOCOUNT ON;
            declare @sql nvarchar(max)
        	declare @col nvarchar(max)
        	set @sql = 'select @col = stuff((select '',''+name from syscolumns where id=object_id( '''+ @tbname +''') order by name for xml path('''')),1,1,'''')'  
        	exec sp_executesql @sql,N'@col nvarchar(max) output',@col output
        	set @sql = ' insert into '+ @tbname + '('+ @col +') '
        	print(@sql)
        END

