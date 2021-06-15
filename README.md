- 👋 Hi, I’m @zyy-960616
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
zyy-960616/zyy-960616 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->


ok
---01重新归类二级分类
drop table if exists temp_rp_ct_dpk_fee_xds_heji_1;
create table temp_rp_ct_dpk_fee_xds_heji_1
(
statis_month                          varchar(8)
,month                                varchar(30)
,prov_code                            varchar(30)
,city_code                            varchar(30)
,serv_number                          varchar(30) 
,good_sec_type                        varchar(30) 
,good_sec_type_cn                     int  
,receivable_amount                    int
,fiveG_type                           varchar(50)
,good_sec_type_new                    varchar(50)
);                       
insert into table temp_rp_ct_dpk_fee_xds_heji_1 
select month,prov_code,city_code,serv_number,good_sec_type,good_sec_type_cn,receivable_amount, fiveg_type,
(case when good_sec_type in ('1','2','3','22','29') then 1  
  when good_sec_type in ('4','21') then 2 
  when good_sec_type in ('6') then 3 
  when good_sec_type in ('10') then 4 
  when good_sec_type in ('11') then  5  
  when good_sec_type in ('12') then 6 
  when good_sec_type in ('5','7','8','9','17','18','19','20','23','24','28')  then 7 
  when good_sec_type in ('14','16','26') then 8 
  when good_sec_type in ('13','15','25','27') then 9 
  else good_sec_type  end ) as good_sec_type_new
from 
(select month,prov_code,city_code,serv_number,good_sec_type,good_sec_type_cn, receivable_amount, fiveg_type
from csap662.tb_rp_ct_bill_account_analysis_dpk_packet_button_model_month
where statis_month='202104'
group by month,prov_code,city_code,serv_number,good_sec_type,good_sec_type_cn, receivable_amount, fiveg_type)a



---02长时包偏好
drop table if exists temp_rp_ct_dpk_fee_xds_heji_2;
create table temp_rp_ct_dpk_fee_xds_heji_2
(
statis_month                          varchar(8)
,month                                varchar(30)
,prov_code                            varchar(30)
,city_code                            varchar(30)
,serv_number                          varchar(30) 
,fiveG_type                           varchar(50)
,good_sec_type_short                  varchar(50)
,good_sec_type_long                   varchar(50)
);                       
insert into table temp_rp_ct_dpk_fee_xds_heji_2 
select month ,prov_code,city_code,serv_number,max(fiveG_type) fiveG_type,max(good_sec_type_long) good_sec_type_long,
max(good_sec_type_short) good_sec_type_short
from(select month ,serv_number,
case when good_sec_type in ('1','2','3','4','5','6','7','8','9','10','11','12','21','22','28','29') then 1 else 0 end good_sec_type_short,
case when good_sec_type  in ('13','14','15','16','17','18','19','20','23','24','25','26','27') then 1 else 0 end good_sec_type_long,
case when fiveG_type in ('1') then 1 else 0 end  fiveG_type
from csap662.tb_rp_ct_bill_account_analysis_dpk_packet_button_model_month
 where statis_month='202104' )a
group by prov_code,city_code


----03拼接长时包
drop table if exists temp_rp_ct_dpk_fee_xds_heji_3;
create table temp_rp_ct_dpk_fee_xds_heji_3
(
statis_month                          varchar(8)
,month                                varchar(30)
,prov_code                            varchar(30)
,city_code                            varchar(30)
,serv_number                          varchar(30) 
,good_sec_type                        varchar(30) 
,good_sec_type_cn                     int  
,receivable_amount                    int
,fiveG_type                           varchar(50)
,good_sec_type_new                    varchar(50)
,good_sec_type_short                  varchar(30)
,good_sec_type_long                   varchar(30) 
);                       
insert into table temp_rp_ct_dpk_fee_xds_heji_3 
select month,prov_code,city_code,a.serv_number,good_sec_type,good_sec_type_cn, good_sec_type_new,receivable_amount, 
fiveg_type,good_sec_type_short,good_sec_type_long
from
(select month,prov_code,city_code,serv_number,good_sec_type,good_sec_type_cn,receivable_amount,good_sec_type_new
from temp_rp_ct_dpk_fee_xds_heji_1 )a
left join
(select month ,serv_number,fiveG_type,good_sec_type_short, good_sec_type_long
from temp_rp_ct_dpk_fee_xds_heji_2)b
on a.serv_number=b.serv_number
and a.month=b.month



---04合并用户新分类金额
drop table if exists temp_rp_ct_dpk_fee_xds_heji_4;
create table temp_rp_ct_dpk_fee_xds_heji_4
(
statis_month                          varchar(8)
,month                                varchar(30)
,prov_code                            varchar(30)
,city_code                            varchar(30)
,serv_number                          varchar(30) 
,good_sec_type                        varchar(30) 
,good_sec_type_sum                    int 
,receivable_amount_sum                int
,fiveG_type                           varchar(50)
,good_sec_type_new                    varchar(50)
,good_sec_type_short                  varchar(30)
,good_sec_type_long                   varchar(30) 
);                       
insert into table temp_rp_ct_dpk_fee_xds_heji_4 
select month ,prov_code,city_code,serv_number,good_sec_type_long,good_sec_type_short,good_sec_type_new,fiveG_type,
sum(good_sec_type_cn) good_sec_type_sum,sum(receivable_amount) receivable_amount_sum 
from  temp_rp_ct_dpk_fee_xds_heji_3
group by prov_code,city_code,serv_number,good_sec_type_short,good_sec_type_long,good_sec_type_new,fiveG_type;



 

----05价格排序
drop table if exists temp_rp_ct_dpk_fee_xds_heji_5;
create table temp_rp_ct_dpk_fee_xds_heji_5
(
statis_month                          varchar(8)
,month                                varchar(30)
,prov_code                            varchar(30)
,city_code                            varchar(30)
,serv_number                          varchar(30) 
,good_sec_type                        varchar(30) 
,good_sec_type_sum                    int 
,receivable_amount_sum                int
,fiveG_type                           varchar(50)
,good_sec_type_new                    varchar(50)
,good_sec_type_short                  varchar(30)
,good_sec_type_long                   varchar(30)
,priority                             varchar(30)
);                       
insert into table temp_rp_ct_dpk_fee_xds_heji_5
select month,prov_code,city_code,serv_number,good_sec_type_long,good_sec_type_short,good_sec_type_new,fiveG_type, good_sec_type_sum,receivable_amount_sum,
row_number()over(partition by prov_code,city_code,serv_number order by (good_sec_type_sum) desc) priority  
from temp_rp_ct_dpk_fee_xds_heji_4



---6拼接金额
drop table if exists temp_rp_ct_dpk_fee_xds_heji_6;
create table temp_rp_ct_dpk_fee_xds_heji_6
(
statis_month                          varchar(8)
,month                                varchar(30)
,prov_code                            varchar(30)
,city_code                            varchar(30)
,serv_number                          varchar(30) 
,good_sec_type                        varchar(30) 
,good_sec_type_sum                    int 
,receivable_amount_sum                int
,fiveG_type                           varchar(50)
,good_sec_type_new                    varchar(50)
,good_sec_type_short                  varchar(30)
,good_sec_type_long                   varchar(30)
,priority                             varchar(30)
,receivable_amount_all                int
);  
insert into table temp_rp_ct_dpk_fee_xds_heji_6
select a.month ,a.prov_code,a.city_code,a.serv_number,good_sec_type_long,good_sec_type_short,good_sec_type_new,fiveG_type, 
good_sec_type_sum,receivable_amount_sum,receivable_amount_all, priority  
from 
(select month,prov_code,city_code,serv_number,good_sec_type_long,good_sec_type_short,good_sec_type_new,fiveG_type, good_sec_type_sum,receivable_amount_sum, priority  
where statis_month='202104'
from temp_rp_ct_dpk_fee_xds_heji_5)a
left join 
(select month ,prov_code,city_code,serv_number, sum(receivable_amount_sum) receivable_amount_all
from temp_rp_ct_dpk_fee_xds_heji_5
where statis_month='202104'
group by prov_code,city_code,serv_number)b
on a.serv_number=b.serv_number
and a.prov_code=b.prov_code
and a.city_code=b.city_code

----7
drop table if exists temp_rp_ct_dpk_fee_xds_heji_7;
create table temp_rp_ct_dpk_fee_xds_heji_7
(
statis_month                          varchar(8)
,month                                varchar(30)
,prov_code                            varchar(30)
,city_code                            varchar(30)
,serv_number                          varchar(30) 
,fiveG_type                           varchar(50)
,good_sec_type_short                  varchar(30)
,good_sec_type_long                   varchar(30)
,receivable_amount_all                int
,good_sec_type_hour                   varchar(30)
,good_sec_type_day1                   varchar(30)
,good_sec_type_day3                   varchar(30)
,good_sec_type_day7                   varchar(30)
,good_sec_type_weekday                varchar(30)
,good_sec_type_monday                 varchar(30)
,good_sec_type_other                  varchar(50)
,good_sec_type_con,                   varchar(30)
,good_sec_type_mon,                   varchar(30)
,good_sec_type_null,                  varchar(30)
,good_sec_type_hour_sum               varchar(30)
,good_sec_type_day1_sum               varchar(30)
,good_sec_type_day3_sum               varchar(30)
,good_sec_type_day7_sum               varchar(30)
,good_sec_type_weekday_sum            varchar(30)
,good_sec_type_monday_sum             varchar(30)
,good_sec_type_other_sum              varchar(50)
,good_sec_type_con_sum                varchar(30)
,good_sec_type_mon_sum                varchar(30)
,good_sec_type_null_sum               varchar(30)
,good_sec_type_hour_amount            varchar(30)
,good_sec_type_day1_amount            varchar(30)
,good_sec_type_day3_amount            varchar(30)
,good_sec_type_day7_amount            varchar(30)
,good_sec_type_weekday_amount         varchar(30)
,good_sec_type_monday_amount          varchar(30)
,good_sec_type_other_amount           varchar(50)
,good_sec_type_con_amount             varchar(30)
,good_sec_type_mon_amount             varchar(30)
,good_sec_type_null_amount            varchar(30)
,good_sec_type_hour_priority          varchar(30)
,good_sec_type_day1_priority          varchar(30)
,good_sec_type_day3_priority          varchar(30)
,good_sec_type_day7_priority          varchar(30)
,good_sec_type_weekday_priority       varchar(30)
,good_sec_type_monday_priority        varchar(30)
,good_sec_type_other_priority         varchar(50)
,good_sec_type_con_priority           varchar(30)
,good_sec_type_mon_priority           varchar(30)
,good_sec_type_null_priority          varchar(30)
insert into table temp_rp_ct_dpk_fee_xds_heji_7
select month ,prov_code,city_code,serv_number,good_sec_type_long,good_sec_type_short,fiveG_type,receivable_amount_all, 
case when good_sec_type_new='1' then 1 else 0 end good_sec_type_hour,
case when good_sec_type_new='2' then 1 else 0 end good_sec_type_day1,
case when good_sec_type_new='3' then 1 else 0 end good_sec_type_day3,
case when good_sec_type_new='4' then 1 else 0 end good_sec_type_day7,
case when good_sec_type_new='5' then 1 else 0 end good_sec_type_weekday,
case when good_sec_type_new='6' then 1 else 0 end good_sec_type_monday,
case when good_sec_type_new='7' then 1 else 0 end good_sec_type_other,
case when good_sec_type_new='8' then 1 else 0 end good_sec_type_con,
case when good_sec_type_new='9' then 1 else 0 end good_sec_type_mon,
case when good_sec_type_new='9999' then 1 else 0 end good_sec_type_null,
case when good_sec_type_new='1' then good_sec_type_sum else 0 end good_sec_type_hour_sum,
case when good_sec_type_new='2' then good_sec_type_sum else 0 end good_sec_type_day1_sum,
case when good_sec_type_new='3' then good_sec_type_sum else 0 end good_sec_type_day3_sum,
case when good_sec_type_new='4' then good_sec_type_sum else 0 end good_sec_type_day7_sum,
case when good_sec_type_new='5' then good_sec_type_sum else 0 end good_sec_type_weekday_sum,
case when good_sec_type_new='6' then good_sec_type_sum else 0 end good_sec_type_monday_sum,
case when good_sec_type_new='7' then good_sec_type_sum else 0 end good_sec_type_other_sum,
case when good_sec_type_new='8' then good_sec_type_sum else 0 end good_sec_type_con_sum,
case when good_sec_type_new='9' then good_sec_type_sum else 0 end good_sec_type_mon_sum,
case when good_sec_type_new='9999' then good_sec_type_sum else 0 end good_sec_type_null_sum,
case when good_sec_type_new='1' then receivable_amount_sum else 0 end good_sec_type_hour_amount,
case when good_sec_type_new='2' then receivable_amount_sum else 0 end good_sec_type_day1_amount,
case when good_sec_type_new='3' then receivable_amount_sum else 0 end good_sec_type_day3_amount,
case when good_sec_type_new='4' then receivable_amount_sum else 0 end good_sec_type_day7_amount,
case when good_sec_type_new='5' then receivable_amount_sum else 0 end good_sec_type_weekday_amount,
case when good_sec_type_new='6' then receivable_amount_sum else 0 end good_sec_type_monday_amount,
case when good_sec_type_new='7' then receivable_amount_sum else 0 end good_sec_type_other_amount,
case when good_sec_type_new='8' then receivable_amount_sum else 0 end good_sec_type_con_amount,
case when good_sec_type_new='9' then receivable_amount_sum else 0 end good_sec_type_mon_amount,
case when good_sec_type_new='9999' then good_sec_type_sum else 0 end good_sec_type_null_amount,
case when good_sec_type_new='1' then priority else 0 end good_sec_type_hour_priority,
case when good_sec_type_new='2' then priority else 0 end good_sec_type_day1_priority,
case when good_sec_type_new='3' then priority else 0 end good_sec_type_day3_priority,
case when good_sec_type_new='4' then priority else 0 end good_sec_type_day7_priority,
case when good_sec_type_new='5' then priority else 0 end good_sec_type_weekday_priority,
case when good_sec_type_new='6' then priority else 0 end good_sec_type_monday_priority,
case when good_sec_type_new='7' then priority else 0 end good_sec_type_other_priority,
case when good_sec_type_new='8' then priority else 0 end good_sec_type_con_priority,
case when good_sec_type_new='9' then priority else 0 end good_sec_type_mon_priority,
case when good_sec_type_new='9999' then priority else 0 end good_sec_type_null_priority
from  temp_rp_ct_dpk_fee_xds_heji_6



-----8
drop table if exists temp_rp_ct_dpk_fee_xds_heji_8;
create table temp_rp_ct_dpk_fee_xds_heji_8
(
statis_month                          varchar(8)
,month                                varchar(30)
,prov_code                            varchar(30)
,city_code                            varchar(30)
,serv_number                          varchar(30) 
,fiveG_type                           varchar(50)
,good_sec_type_short                  varchar(30)
,good_sec_type_long                   varchar(30)
,receivable_amount_all                int
,good_sec_type_hour                   varchar(30)
,good_sec_type_day1                   varchar(30)
,good_sec_type_day3                   varchar(30)
,good_sec_type_day7                   varchar(30)
,good_sec_type_weekday                varchar(30)
,good_sec_type_monday                 varchar(30)
,good_sec_type_other                  varchar(50)
,good_sec_type_con,                   varchar(30)
,good_sec_type_mon,                   varchar(30)
,good_sec_type_null,                  varchar(30)
,good_sec_type_hour_sum               varchar(30)
,good_sec_type_day1_sum               varchar(30)
,good_sec_type_day3_sum               varchar(30)
,good_sec_type_day7_sum               varchar(30)
,good_sec_type_weekday_sum            varchar(30)
,good_sec_type_monday_sum             varchar(30)
,good_sec_type_other_sum              varchar(50)
,good_sec_type_con_sum                varchar(30)
,good_sec_type_mon_sum                varchar(30)
,good_sec_type_null_sum               varchar(30)
,good_sec_type_hour_amount            varchar(30)
,good_sec_type_day1_amount            varchar(30)
,good_sec_type_day3_amount            varchar(30)
,good_sec_type_day7_amount            varchar(30)
,good_sec_type_weekday_amount         varchar(30)
,good_sec_type_monday_amount          varchar(30)
,good_sec_type_other_amount           varchar(50)
,good_sec_type_con_amount             varchar(30)
,good_sec_type_mon_amount             varchar(30)
,good_sec_type_null_amount            varchar(30)
,good_sec_type_hour_priority          varchar(30)
,good_sec_type_day1_priority          varchar(30)
,good_sec_type_day3_priority          varchar(30)
,good_sec_type_day7_priority          varchar(30)
,good_sec_type_weekday_priority       varchar(30)
,good_sec_type_monday_priority        varchar(30)
,good_sec_type_other_priority         varchar(50)
,good_sec_type_con_priority           varchar(30)
,good_sec_type_mon_priority           varchar(30)
,good_sec_type_null_priority          varchar(30)
insert into table temp_rp_ct_dpk_fee_xds_heji_8
select month,prov_code,city_code,serv_number,good_sec_type_long,good_sec_type_short,fiveG_type,receivable_amount_all,
max(good_sec_type_hour) good_sec_type_hour ,max(good_sec_type_day1) good_sec_type_day1 ,max(good_sec_type_day3) good_sec_type_day3,max(good_sec_type_day7) good_sec_type_day7,
max(good_sec_type_weekday) good_sec_type_weekday,max(good_sec_type_monday) good_sec_type_monday,max(good_sec_type_other) good_sec_type_other,max(good_sec_type_con) good_sec_type_con,
max(good_sec_type_mon) good_sec_type_mon,max(good_sec_type_null) good_sec_type_null,
max(good_sec_type_hour_sum) good_sec_type_hour_sum,max(good_sec_type_day1_sum) good_sec_type_day1_sum,max(good_sec_type_weekday_sum) good_sec_type_weekday_sum,
max(good_sec_type_day3_sum) good_sec_type_day3_sum,max(good_sec_type_day7_sum) good_sec_type_day7_sum,
max(good_sec_type_other_sum) good_sec_type_other_sum,max(good_sec_type_con_sum) good_sec_type_con_sum,max(good_sec_type_mon_sum) good_sec_type_mon_sum,
max(good_sec_type_null_sum) good_sec_type_null_sum,max(good_sec_type_monday_sum) good_sec_type_monday_sum,
max(good_sec_type_hour_amount) good_sec_type_hour_amount,max(good_sec_type_day1_amount) good_sec_type_day1_amount,
max(good_sec_type_day3_amount) good_sec_type_day3_amount,max(good_sec_type_day7_amount) good_sec_type_day7_amount,max(good_sec_type_weekday_amount) good_sec_type_weekday_amount,
max(good_sec_type_monday_amount) good_sec_type_monday_amount,max(good_sec_type_other_amount) good_sec_type_other_amount,max(good_sec_type_con_amount) good_sec_type_con_amount,
max(good_sec_type_mon_amount) good_sec_type_mon_amount,max(good_sec_type_null_amount) good_sec_type_null_amount,
max(good_sec_type_hour_priority) good_sec_type_hour_priority,max(good_sec_type_day1_priority) good_sec_type_day1_priority,
max(good_sec_type_day3_priority) good_sec_type_day3_priority,max(good_sec_type_day7_priority) good_sec_type_day7_priority,max(good_sec_type_weekday_priority) good_sec_type_weekday_priority,
max(good_sec_type_monday_priority) good_sec_type_monday_priority,max(good_sec_type_other_priority) good_sec_type_other_priority,max(good_sec_type_con_priority) good_sec_type_con_priority,
max(good_sec_type_mon_priority) good_sec_type_mon_priority,max(good_sec_type_null_priority) good_sec_type_null_priority
from  temp_rp_ct_dpk_fee_xds_heji_7
group by prov_code,city_code,serv_number,good_sec_type_long,good_sec_type_short,fiveG_type,receivable_amount_all
