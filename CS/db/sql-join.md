# sql join语句

## INNER JOIN(JOIN) 内连接/等值连接
获取两个表中字段匹配关系的记录。如tb_auth和tb_job可以以user_id作为连接字段。  
一般使用ON来设定连接条件，使用WHERE进行结果的过滤，但ON也可以用WHERE代替。

![内连接示意图](https://img-blog.csdn.net/20170613174645585?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9ibzg5NDU1MTAw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



## LEFT JOIN 左连接（左外连接）
获取左表所有记录，即使右表没有对应匹配的记录。

![左连接](https://img-blog.csdn.net/20170613174955995?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9ibzg5NDU1MTAw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## RIGHT JOIN 右连接
与LEFT JOIN相反，获取右表所有记录，即使左表没有对应的记录。

![右连接](https://img-blog.csdn.net/20170613175027043?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYm9ibzg5NDU1MTAw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


ref：
[MYSQL数据库-join连接讲解](https://blog.csdn.net/bobo89455100/article/details/73181096)