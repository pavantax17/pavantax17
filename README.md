USE msdb
GO

select distinct
    r.session_id [Session Id],
    r.blocking_session_id,
    r.command,
    r.last_wait_type,
    --r.percent_complete,
    CONVERT(CHAR(11),r.start_time,101) + CONVERT(CHAR( 5),r.start_time,114) [Session Start Time],
    DATEDIFF( MINUTE , r.start_time , getdate())  [Run Time Minutes], 
    sp.loginame [Login Name],
    db_name(r.database_id) as [Database Name],
    sp.hostname [Host Name],
    CAST((r.granted_query_memory*8.0)/1024.0 AS DECIMAL(10,2))as [MemoryUsed MB],
    --cpu,
    cpu_time,
    sp.program_name,
    t.text [Query]
    into #temp

from sys.dm_exec_requests r 
outer APPLY sys.dm_exec_sql_text(r.sql_handle) AS t 
JOIN sys.sysprocesses sp on sp.spid = r.session_id
where  sp.loginame <> '' and sp.loginame not in('') 
and DATEDIFF( MINUTE , r.start_time , getdate()) >= 3
and t.text <>'sp_server_diagnostics'
order by [Session Start Time] asc


insert into  msdb.dbo.Table1 
select a.* 
from   #temp a left join msdb.dbo.Table1 b 
on a.[Session Id] = b.[Session_Id] and a.[Session Start Time] = b.[Session_Start_Time] and a.[Login Name] = b.[Login_Name]
where b.[Session_Id] is null

update b
set 
b.[Run_Time_Minutes] = a.[Run Time Minutes],
b.[last_wait_type]= a.[last_wait_type],
b.[MemoryUsed_MB]=CASE WHEN b.[MemoryUsed_MB]>a.[MemoryUsed MB] THEN B.MemoryUsed_MB ELSE a.[MemoryUsed MB] END,
--b.[cpu]=CASE WHEN b.[cpu]>a.[cpu] THEN B.[cpu] ELSE a.[cpu] END,
b.cpu_time=CASE WHEN b.cpu_time>a.cpu_time THEN B.cpu_time ELSE a.cpu_time END
from   #temp a left join msdb.dbo.Table1 b 
on a.[Session Id] = b.[Session_Id] and a.[Session Start Time] = b.[Session_Start_Time] and a.[Login Name] = b.[Login_Name]
where b.[Session_Id] is not null

delete  from msdb.dbo.Table1  where [Session_Start_Time] < getdate()-90 
drop table #temp

USE [msdb]
GO

/****** Object:  Table [dbo].[Table1]    Script Date: 6/28/2022 3:46:29 PM ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[Table1](
    [Session_Id] [int] NOT NULL,
    [blocking_session_id] [int] NULL,
    [command] [varchar](100) NULL,
    [last_wait_type] [varchar](100) NULL,
    [Session_Start_Time] [datetime] NOT NULL,
    [Run_Time_Minutes] [int] NULL,
    [Login_Name] [varchar](40) NOT NULL,
    [Database_Name] [varchar](40) NULL,
    [HostName] [varchar](200) NULL,
    [MemoryUsed_MB] [float] NULL,
    [cpu_time] [bigint] NULL,
    [program_name] [varchar](200) NULL,
    [Query] [varchar](max) NULL,
 CONSTRAINT [PK_Session_Idnew] PRIMARY KEY CLUSTERED 
(
    [Session_Id] ASC,
    [Session_Start_Time] ASC,
    [Login_Name] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
