# Aws Aurora's bug - selecting multiple columns in select-where-in clause.

This query has no problem in AWS rds - MySQL 5.7.x. 

    select * from PRODUCT where (product_id, product_type) in ((1, 'p'), (2, 'd'));

The plan is like this. a index is used.

    id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
    ---
    1	SIMPLE	PRODUCT	NULL	range	product	product	16	NULL	2	100.00	Using where
    
MySQL Version is : 

    version()
    5.7.10-log
    
After I migrated MySQL 5.7 to Aurora, the query became so slow. It took 1 ~ 2 secs. The plan was not normal. Index wasn't used.

    id select_type table type possible_keys key key_len ref rows Extra
    -------------------
    1 SIMPLE PRODUCT ALL NULL NULL NULL NULL 1475470 Using where
    
The index hint of MySQL doesn't work. The plan can't use index.

    explain
    select * from PRODUCT FORCE key (product) where (product_id, product_type) in ((1, 'p'), (2, 'd'));
    
    id select_type table type possible_keys key key_len ref rows Extra
    -------------------
    1 SIMPLE PRODUCT ALL NULL NULL NULL NULL 1475470 Using where
    
Aurora supports only MySQL 5.6.x, not 5.7.x now. The MySQL version of Aurora's write instance and read instance are the same, 5.6.10

    version()
    5.6.10-log

When query is changed like this, 

    explain
    select * from PRODUCT 
    where 
    (product_id = 1 and product_type = 'p' ) or 
    (product_id = 2 and product_type = 'd');
  
index is used normaly.

    id	select_type	table	type	possible_keys	key	key_len	ref	rows	Extra
    ---
    1	SIMPLE	PRODUCT	range	product	product	16	NULL	2	Using index condition


It's MySQL 5.6.x problem not Aurora. When I installed MySQL-5.6.x on AWS. the query had the same problem, not use index. And It's MySQL 5.6.x bug. (http://bugs.mysql.com/bug.php?id=35819)



