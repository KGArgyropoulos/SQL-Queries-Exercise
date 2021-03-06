**For the entity-relationship diagram in the picture [schema.png], try to find the appropriate SQL queries**

**Exercise 1**
- Find IDs from the customers who are French and have made an order after 1998-07-02.

select C_CUSTKEY as 'customer_key' \
from customer \
where C_NATIONKEY in(select N_NATIONKEY \
					from nation \
					where N_NAME="FRANCE" \
					and C_CUSTKEY in(select O_CUSTKEY \
									 from orders \
									 where O_ORDERDATE>'19980702'))

**Exercise 2**
- Find the names of the ASIAN suppliers who have supplied European customers.

select S_NAME as 'supplier_name' \
from supplier \
where S_NATIONKEY in(select N_NATIONKEY \
					from nation \
					where N_REGIONKEY in(select R_REGIONKEY \
										 from region \
										 where R_NAME="ASIA")) \
and S_SUPPKEY in(select L_SUPPKEY \
				from lineitem \
				where L_ORDERKEY in(select O_ORDERKEY \
									from orders \
									where O_CUSTKEY in(select C_CUSTKEY \
														from customer \
														where C_NATIONKEY in (select N_NATIONKEY \
																			  from nation \
																			  where N_REGIONKEY in (select R_REGIONKEY \
																			  from region \
																			  where R_NAME="EUROPE")))))

**Exercise 3**
- Find the number of the suppliers who delivered an order on (receiptdate=) 1998-07-21 and brand "brand#22".

select count(distinct L_SUPPKEY) as 'suppliers' \
from lineitem \
where L_RECEIPTDATE='19980721' and L_PARTKEY in(select PS_PARTKEY \
												from partsupp \
												where PS_PARTKEY in(select P_PARTKEY \
												from part \
												where P_BRAND="brand#22"))

**Exercise 4**
- Check (yes/no) if there is an order on 2014-12-12, from a French customer to a German supplier.
- *No use of Flow Control Operators (if,case,etc)

select 'yes' as answer \
from customer \
where C_CUSTKEY in(select O_CUSTKEY \
					from orders \
					where O_ORDERKEY in(select L_ORDERKEY \
										from lineitem \
										where L_SUPPKEY in(select S_SUPPKEY \
															from supplier \
															where S_NATIONKEY in(select N_NATIONKEY \
																				 from nation \
																				 where N_NATIONKEY="GERMANY")))) \
and C_NATIONKEY in(select N_NATIONKEY \
				   from nation \
				   where N_NAME="FRANCE") \
and C_CUSTKEY in(select O_CUSTKEY \
				 from orders \
				 where O_ORDERDATE='20141212') \
UNION \
select 'no' as answer \
from customer \
where C_CUSTKEY in(select O_CUSTKEY \
					from orders \
					where O_ORDERKEY in(select L_ORDERKEY \
									    from lineitem \
										where L_SUPPKEY in(select S_SUPPKEY \
													       from supplier \
															where S_NATIONKEY in(select N_NATIONKEY \
																				from nation \
																				where N_NATIONKEY<>"GERMANY")))) \
and C_NATIONKEY in(select N_NATIONKEY \
				   from nation \
				   where N_NAME<>"FRANCE") \
and C_CUSTKEY in(select O_CUSTKEY \
				 from orders \
				 where O_ORDERDATE<>'20141212')

**Exercise 5**
- Find the average supply cost(PS_SUPPLYCOST) of the part named "yellow smoke firebrick chiffon rosy".

select p_name,avg(PS_SUPPLYCOST) as 'avg_supplycost' \
from part,partsupp \
where P_PARTKEY=PS_PARTKEY and p_name="yellow smoke firebrick chiffon rosy"

**Exercise 6**
- Find the names and the phone numbers of the customers who have the same supplier in all of their orders.

select C_NAME as customer_name,C_PHONE as customer_phone \
from customer \
where C_CUSTKEY in(select O_CUSTKEY \
				   from orders \
                   where O_ORDERKEY in(select L_ORDERKEY \
								       from lineitem l \
                                       where not exists(select * \
														from lineitem l1 \
                                                        where l.L_SUPPKEY<>l1.L_SUPPKEY and l.L_ORDERKEY=l1.L_ORDERKEY)))

**Exercise 7**
- Find supplier's and client's nationalities for orders made between 1998-07-24 and 2011-09-10, as long as these orders include at least 5 items.

select distinct n2.N_NAME as 'supplier_nation',n1.N_NAME as 'customer_nation' \
from nation n1,nation n2,customer c1,supplier s1 \
where n1.N_NATIONKEY=c1.C_NATIONKEY \
and n2.N_NATIONKEY=s1.S_NATIONKEY \
and c1.C_CUSTKEY in(select O_CUSTKEY \
				    from orders \
                    where O_ORDERDATE>'19980724' \
					and O_ORDERDATE<'20110910' \
				    and O_ORDERKEY in(select L_ORDERKEY \
								      from lineitem \
                                      where L_SUPPKEY=s1.S_SUPPKEY)) \
group by n2.N_NAME,c1.C_CUSTKEY \
having 5<=(select count(L_PARTKEY) \
		   from lineitem \
           where L_ORDERKEY in(select O_ORDERKEY \
						       from orders \
                               where O_CUSTKEY=c1.C_CUSTKEY))

**Exercise 8**
- For every supplier that has exactly 5 kinds of items available, find his name, his id and the number of orders he has made.

select S_SUPPKEY,S_NAME,count(distinct L_ORDERKEY) as 'orders' \
from supplier s,partsupp ps,lineitem l,orders o \
where s.S_SUPPKEY=ps.PS_SUPPKEY \
and ps.PS_SUPPKEY=l.L_SUPPKEY \
and l.L_ORDERKEY=o.O_ORDERKEY \
and ps.PS_SUPPKEY in(select ps1.PS_SUPPKEY \
				     from partsupp ps1\
                     where ps1.PS_SUPPKEY=s.S_SUPPKEY \
                     having count(*)=5) \
group by s.S_SUPPKEY

**Exercise 9**
- Find the names and the phone numbers of the customers who have made orders in every country.

select C_NAME,C_PHONE \
from customer c \
where not exists(select * \
				 from supplier s,nation \
                 where s.S_NATIONKEY=N_NATIONKEY \
and not exists(select * \
				from customer c1,orders,lineitem \
            	where c1.C_CUSTKEY=c.C_CUSTKEY \
				and c1.C_CUSTKEY=O_CUSTKEY and O_ORDERKEY=L_ORDERKEY and L_SUPPKEY=s.S_SUPPKEY))

**Exercise 10**
- Find the names and the phone numbers of the customers who have made orders only from German suppliers and the ones who have made more than one order between 1998-01-01 and 1999-12-31.

select C_NAME,C_PHONE \
from customer c \
where not exists(select* from orders o \
		 where c.C_CUSTKEY=O_CUSTKEY \
		 and exists(select * from lineitem,partsupp,supplier,nation \
				    where o.O_ORDERKEY=L_ORDERKEY \
				    and L_SUPPKEY=PS_SUPPKEY \
		 		    and S_SUPPKEY=PS_SUPPKEY \
		 		    and S_NATIONKEY=N_NATIONKEY \
		 	    	and N_NAME!="GERMANY" )) \
UNION \
	select C_NAME,C_PHONE \
	from customer,orders \
	where C_CUSTKEY=O_CUSTKEY and O_ORDERDATE>'19980101' and O_ORDERDATE<'19991231' \
	group by C_NAME \
	having count(O_ORDERKEY)>1
