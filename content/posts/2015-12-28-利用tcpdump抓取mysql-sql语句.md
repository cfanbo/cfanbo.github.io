---
title: 利用tcpdump抓取mysql sql语句
author: admin
type: post
date: 2015-12-28T13:58:05+00:00
url: /archives/16424
categories:
 - 程序开发

---
这个脚本是我之前在网上无意间找个一个利用tcpdump 抓包工具获取mysql流量，并通过过滤把sql 语句输入。

脚本不是很长，但是效果很好。

```
#!/bin/bash
#this script used montor mysql network traffic.echo sql
tcpdump -i eth0 -s 0 -l -w - dst port 3306 | strings | perl -e '
while(<>) { chomp; next if /^[^ ]+[ ]*$/;
    if(/^(SELECT|UPDATE|DELETE|INSERT|SET|COMMIT|ROLLBACK|CREATE|DROP|ALTER|CALL)/i)
    {
        if (defined $q) { print "$q\n"; }
        $q=$_;
    } else {
        $_ =~ s/^[ \t]+//; $q.=" $_";
    }
}'
```

下面是执行脚本的输出：

```
SELECT b.id FROM module as a,rights as b where a.id=b.module_id and b.sid='179' and a.pname like 'vip/member_order_manage.php%'
SELECT count(id) as cc,sum(cash) as total from morder_stat_all  where (ymd BETWEEN '1312214400' and '1312336486') and depart_id=5 an
d order_class=2
select id,name from media where symd='0000-00-00'
select id,name from depart where s_flag=' '  and onoff=1 order by sno
select id,name from plank where depart_id=5  and onoff=1 order by no
select id,name from grp where plank_id=0  and onoff=1 order by no
select id,CONCAT(pname,'-',name) as name from pvc order by pname
select id,CONCAT(no,'-',name) as name from local where pvc_id=0 order by no
select id,name from product_breed
select color_name from product_color where id=5
select id,name from product where id = '0'
select * from morder_stat_all  where (ymd BETWEEN '1312214400' and '1312336486') and depart_id=5 and order_class=2 order by ymd DESC
 LIMIT 0,50
select urlkey from sys_config where id=1
select name from morder where id=7195793
select no,name from staff where id=5061
select product_id,amt,price0 from order_product where order_id = 7195793
select concat_ws('/',name,NULLIF((select color_name as cn from product_color where id=color_id),''),NULLIF((select style_name from p
roduct_style where id=style_id),'')) as name,spec,weight,price from product where id = 16938
select concat_ws('/',name,NULLIF((select color_name as cn from product_color where id=color_id),''),NULLIF((select style_name from p
roduct_style where id=style_id),'')) as name,spec,weight,price from product where id = 19005
select name from morder where id=7195768
select no,name from staff where id=221
select product_id,amt,price0 from order_product where order_id = 7195768
select concat_ws('/',name,NULLIF((select color_name as cn from product_color where id=color_id),''),NULLIF((select style_name from p
roduct_style where id=style_id),'')) as name,spec,weight,price from product where id = 18978
select concat_ws('/',name,NULLIF((select color_name as cn from product_color where id=color_id),''),NULLIF((select style_name from p
roduct_style where id=style_id),'')) as name,spec,weight,price from product where id = 18282
select concat_ws('/',name,NULLIF((select color_name as cn from product_color where id=color_id),''),NULLIF((select style_name from p
roduct_style where id=style_id),'')) as name,spec,weight,price from product where id = 19740
```

从上面的日志可以看出，脚本的功能还是很强大吧 。

转自： [http://www.cnblogs.com/ylqmf/archive/2012/07/11/2586741.html](http://www.cnblogs.com/ylqmf/archive/2012/07/11/2586741.html)