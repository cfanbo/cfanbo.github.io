---
title: 如何根据rowid获取extent_id
author: admin
type: post
date: 2009-05-16T04:25:09+00:00
excerpt: 我们知道，rowid是由四部分组成的，分别是data_object_id,file_id,block_number和row_number，通过oracle提供的dbms_rowid包可以很方便的将一串rowid解析出上述四部分的内容。然后根据这些信息，则可以获取其extent_id。
url: /archives/1399
IM_contentdowned:
 - 1
categories:
 - 数据库

---
我们知道，rowid是由四部分组成的，分别是data\_object\_id,file\_id,block\_number和row\_number，通过oracle提供的dbms\_rowid包可以很方便的将一串rowid解析出上述四部分的内容。然后根据这些信息，则可以获取其extent_id。

```
SYS@datac>declare
  2  v_block_id number;
  3  v_file_id number;
  4  v_object_id number;
  5  v_extent_id number;
  6  v_object_name varchar2(30);
  7  v_owner varchar2(30);
  8  v_rowid varchar2(20):='AAACrKAAXAAAAzUAAH';
  9  begin
 10  select dbms_rowid.ROWID_BLOCK_NUMBER(v_rowid),
 11         dbms_rowid.ROWID_RELATIVE_FNO(v_rowid),
 12         dbms_rowid.ROWID_OBJECT(v_rowid)
 13   into v_block_id,v_file_id,v_object_id
 14  from dual;
 15
 16  select owner,object_name
 17    into v_owner,v_object_name
 18  from dba_objects
 19  where data_object_id=v_object_id;
 20
 21  select extent_id into v_extent_id
 22  from dba_extents
 23  where owner=v_owner
 24  and segment_name=v_object_name
 25  and file_id=v_file_id
 26  and v_block_id between block_id and block_id+blocks-1;
 27
 28  dbms_output.put_line('         rowid: '||v_rowid);
 29  dbms_output.put_line('       file_id: '||v_file_id);
 30  dbms_output.put_line('      block_id: '||v_block_id);
 31  dbms_output.put_line('data_object_id: '||v_object_id);
 32  dbms_output.put_line('         owner: '||v_owner);
 33  dbms_output.put_line('   object_name: '||v_object_name);
 34  dbms_output.put_line('     extent_id: '||v_extent_id);
 35  end;
 36  /
         rowid: AAACrKAAXAAAAzUAAH
       file_id: 23
      block_id: 3284
data_object_id: 10954
         owner: NINGOO
   object_name: TEST
     extent_id: 5
```

将上述代码打包到一个shell脚本中，rowid通过参数传入，则可以更方便日常环境中使用。工欲善其事，必先利其器，**将经验转化为工具，利用工具提升效率，才能做一个Lazy DBA**。

```
$ tbsql rowid AAACrKAAXAAAAzUAAH
         rowid: AAACrKAAXAAAAzUAAH
       file_id: 23
      block_id: 3284
data_object_id: 10954
         owner: NINGOO
   object_name: TEST
     extent_id: 5

$ tbsql rowid AAACrKAAZAAABiiAAR
         rowid: AAACrKAAZAAABiiAAR
       file_id: 25
      block_id: 6306
data_object_id: 10954
         owner: NINGOO
   object_name: TEST
     extent_id: 7
```