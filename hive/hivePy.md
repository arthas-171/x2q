# hive + python 联用
### [go back](/hive.md)      
### [go home](../README.md)    
 
## hive + python 联用
使用python+hive处理一些单纯sql不容易处理的问题, hive sql作为py的数据源 ,py的输出作为hive map输出,hive的聚合函数作为reduce,  
**实例**tagfortraveller.py                 
                                            
```python
#coding=utf-8
from math import radians, acos, atan, tan, sin, cos
import sys
import time
import datetime

from math import radians, cos, sin, asin, sqrt
device_number,area_id, from_prov,from_area, sex, age,date_id= '', '', '', '', '', '', ''
num=1


def Caltime(date1, date2):

    date1 = time.strptime(date1, "%Y%m%d")
    date2 = time.strptime(date2, "%Y%m%d")

    date1 = datetime.datetime(date1[0], date1[1], date1[2])
    date2 = datetime.datetime(date2[0], date2[1], date2[2])
    datestr=(date2 - date1).days
    return datestr

if __name__ == '__main__':

    for line in sys.stdin:

        # 如果速度超了，就去掉这条记录
        arr = line.strip().split('\t')
        if (device_number == ''):
            # 马上进行赋值
            num=1
            device_number = arr[0]
            area_id = arr[1]
            from_prov = arr[2]
            from_area = arr[3]
            sex = arr[4]
            age = arr[5]
            date_id = arr[6]
            print device_number + '\t' + area_id + '\t' + from_prov + '\t' + from_area + '\t' + sex + '\t' + age + '\t' + date_id + '\t' + str(num)
            continue
        else:
            #如果手机号相同
            if(arr[0]==device_number):
                #判断area_id一致
                if(arr[1]==area_id):
                    if(Caltime(date_id,arr[6])==1):
                        print arr[0] + '\t' + arr[1] + '\t' + arr[2] + '\t' + arr[3] + '\t' + arr[4] + '\t' + arr[5] + '\t' + arr[6] + '\t' + str(num)
                        device_number = arr[0]
                        area_id = arr[1]
                        from_prov = arr[2]
                        from_area = arr[3]
                        sex = arr[4]
                        age = arr[5]
                        date_id = arr[6]

                    else:
                        #如果时间不是差1天
                        num = num+1
                        print arr[0] + '\t' + arr[1] + '\t' + arr[2] + '\t' + arr[3] + '\t' + arr[4] + '\t' + arr[5] + '\t' + arr[6] + '\t' + str(num)
                        device_number = arr[0]
                        area_id = arr[1]
                        from_prov = arr[2]
                        from_area = arr[3]
                        sex = arr[4]
                        age = arr[5]
                        date_id = arr[6]
                else:
                    #如果area_id不一致直接输出
                    print arr[0] + '\t' + arr[1] + '\t' + arr[2] + '\t' + arr[3] + '\t' + arr[4] + '\t' + arr[5] + '\t' + arr[6] + '\t' + '1'
                    num = 1
                    device_number = arr[0]
                    area_id = arr[1]
                    from_prov = arr[2]
                    from_area = arr[3]
                    sex = arr[4]
                    age = arr[5]
                    date_id = arr[6]



            #如果手机号不同,直接将上条数据输出并对该条数据进行赋值
            else:

                print arr[0] + '\t' + arr[1] + '\t' + arr[2] + '\t' + arr[3] + '\t' + arr[4] + '\t' + arr[5] + '\t' + arr[6] + '\t' + '1'

                num = 1
                device_number = arr[0]
                area_id = arr[1]
                from_prov = arr[2]
                from_area = arr[3]
                sex = arr[4]
                age = arr[5]
                date_id = arr[6]

```
      
## 导入语句
add file /home/lf_cp_serv/wo_index/baiqj/practice/servers/tagfortraveller.py;   
执行sql
                                                            
```sql
select transform( device_number,area_id, from_prov,from_area,sex,age,date_id) using 'python tagfortraveller.py' 
as (device_number,area_id, from_prov,from_area,sex,age,date_id,num)
from
(select  device_number,area_id, from_prov,from_area, cust_sex sex, cust_age age,concat(month_id,day_id) date_id
from
ubd_serv_tour.DWA_V_D_CUS_AL_LIST_area
where month_id='201910' and day_id in ('01','03','05','06') and prov_id='030' and is_local='0' and from_area!=area_id
and device_number is not null and area_id is not null  and from_area is not null and from_prov is not null 
and cust_sex is not null  and cust_age is not null 
distribute by 
device_number,area_id
sort by 
device_number,area_id,date_id asc) t;
```                                                                                                        
**注意 hive transform关键字**     
hive transform (c1,c2,c3) using 'python xxx.py' as (c1,c2,c3)


#### 联系邮箱 xxx_xxx@aliyun.com

