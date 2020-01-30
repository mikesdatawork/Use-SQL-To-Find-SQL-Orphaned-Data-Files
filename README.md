![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Use SQL To Find SQL Orphaned Data Files
**Post Date: April 9, 2018**





## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process



![Find Orphaned SQL Database Files]( https://mikesdatawork.files.wordpress.com/2018/04/image0011.png "Get Orphaned Database Files With SQL")
 
<p>With this little bit of SQL Logic you'll be able to read across ALL your data file locations (in the OS) and see what files are NOT associated with any live databases. This will help you find orphaned files, or any one-off backups, detached data files, or virtually any other 'files' which are located under the same folder as your data files. It's really useful.
Here's how it works. It builds a list of ALL your data file paths using sys.master_files. Then it uses those paths to extrapolate all known files under those drive locations. Then after a few temporary tables it runs a comparison between files that on the data file path locations and those that are actually connected to databases. What you get following that are files that are not associated to any databases; therefore they are either genuine orphaned database files, extraneous security files, random backup files or the usual built-in files you get upon installation of SQL Server such as for the system resource databases, replication etc.
At the bottom of the logic you can see where I am excluding some of those. Feel free to mod, or add your own. This is perfect for highly used environments where there is a multitude of detached data files, backup files, copied zips what have you.
Here's the code.</p>  



## SQL-Logic
```SQL
use [master];
set nocount on
 
if object_id('tempdb..#paths') is not null
drop table  #paths
create table    #paths ([path_id] int identity (1,1), [data_paths] varchar(255))
insert into #paths ([data_paths])
 
select distinct left([physical_name], len([physical_name]) - charindex('\', reverse([physical_name])) -0)
from sys.master_files
 
if object_id('tempdb..#found_files') is not null
    drop table  #found_files
create table    #found_files ([files] varchar(255), [file_path] varchar(255), [depth] int, [file] int)
 
declare @get_files  varchar(max)
set @get_files  = ''
select  @get_files  = @get_files +
'
insert into #found_files ([files], [depth], [file]) exec master..xp_dirtree ''' + [data_paths] + ''', 1,1;
update #found_files set [file_path] = ''' + [data_paths] + ''' where [file_path] is null;
' + char(10) from #paths
exec    (@get_files)
 
select
    'no_associated_database'= [files]
,   'path'          = [file_path]
from 
    #found_files
where
    [files] not in (select right([physical_name], charindex('\', reverse([physical_name])) - 1) from sys.master_files)
    and [files] not in
    (
        'mssqlsystemresource.mdf'
    ,   'mssqlsystemresource.ldf'
    ,   'distmdl.mdf'
    ,   'distmdl.ldf'
    )
    and [files] not like '%.cer'
```
Hope you find it helpful. 


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

     
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

