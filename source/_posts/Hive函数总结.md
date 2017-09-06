---
title: Hive函数总结
date: 2017-06-14 12:54:59
tags: Hive
toc: true
categories: 大数据技术
---
Hive提供了包括数学函数，类型转换函数，条件函数，字符函数，聚合函数，表生成函数等的内置函数。
<!-- more -->
## 1.数学函数 ##
函数名|返回类型|描述
---|---|---|
round(DOUBLE a)|DOUBLE|返回对a四舍五入的BIGINT值
round(DOUBLE a, INT d)|DOUBLE|返回DOUBLE型,保留d位小数的DOUBLE型的近似值
bround(DOUBLE a)|DOUBLE|银行家舍入法（1~4：舍，6~9：进，5->前位数是偶：舍，5->前位数是奇：进）
bround(DOUBLE a, INT d)|DOUBLE|银行家舍入法,保留d位小数
floor(DOUBLE a)|BIGINT|向下取整，最数轴上最接近要求的值的左边的值  如：6.10->6   -3.4->-4
ceil(DOUBLE a), ceiling(DOUBLE a)|BIGINT|求其不小于小给定实数的最小整数如：ceil(6) = ceil(6.1)= ceil(6.9) = 7
rand(), rand(INT seed)|DOUBLE|每行返回一个DOUBLE型随机数,seed是随机因子
exp(DOUBLE a), exp(DECIMAL a)|DOUBLE|返回e的a幂次方， a可为小数
n(DOUBLE a), ln(DECIMAL a)|DOUBLE|以自然数为底d的对数，a可为小数
log10(DOUBLE a), log10(DECIMAL a)|DOUBLE|以10为底d的对数，a可为小数
log2(DOUBLE a), log2(DECIMAL a)|DOUBLE|以2为底数d的对数，a可为小数
log(DOUBLE base, DOUBLE a)/log(DECIMAL base, DECIMAL a)|DOUBLE|以base为底的对数，base 与 a都是DOUBLE类型
pow(DOUBLE a, DOUBLE p)/ power(DOUBLE a, DOUBLE p)|DOUBLE|计算a的p次幂ap
sqrt(DOUBLE a), sqrt(DECIMAL a)|DOUBLE|计算a的平方根
bin(BIGINT a)|STRING|计算a的二进制STRING类型，a为BIGINT类型
hex(BIGINT a) hex(STRING a) hex(BINARY a)|STRING|计算十六进制
unhex(STRING a)|BINARY|hex的逆方法
conv(BIGINT num, INT from_base, INT to_base), conv(STRING num, INT from_base, INT to_base)|STRING|将BIGINT/STRING类型的num从from_base进制转换成to_base进制
pmod(INT a, INT b), pmod(DOUBLE a, DOUBLE b)|INT or DOUBLE|a对b取模
sin(DOUBLE a), sin(DECIMAL a)|DOUBLE|求a的正弦值
asin(DOUBLE a), asin(DECIMAL a)|DOUBLE|求d的反正弦值
cos(DOUBLE a), cos(DECIMAL a)|DOUBLE| 求余弦值
acos(DOUBLE a), acos(DECIMAL a)|DOUBLE| 求反余弦值
tan(DOUBLE a), tan(DECIMAL a)|DOUBLE| 求正切值
atan(DOUBLE a), atan(DECIMAL a)|DOUBLE| 求反正切值
degrees(DOUBLE a), degrees(DECIMAL a)|DOUBLE| 奖弧度值转换角度值
radians(DOUBLE a), radians(DOUBLE a)|DOUBLE| 将角度值转换成弧度值
positive(INT a), positive(DOUBLE a)|INT or DOUBLE| 返回a
negative(INT a), negative(DOUBLE a)|INT or DOUBLE| 返回a的相反数
sign(DOUBLE a), sign(DECIMAL a)|DOUBLE or INT| 如果a是正数则返回1.0，是负数则返回-1.0，否则返回0.0
e() |DOUBLE| 数学常数e
pi()|DOUBLE| 数学常数pi
factorial(INT a)|BIGINT| 求a的阶乘
cbrt(DOUBLE a)|DOUBLE| 求a的立方根
shiftleft(TINYINT/SMALLINT/INT a, INT b)/shiftleft(BIGINT a, INT?b)|INT/BIGINT| 按位左移
shiftright(TINYINT/SMALLINT/INT a, INTb)/shiftright(BIGINT a, INT?b)|INT/BIGINT| 按拉右移
shiftrightunsigned(TINYINT/SMALLINT/INTa, INT b),/shiftrightunsigned(BIGINT a, INT b)|INT/BIGINT| 无符号按位右移（<<<）
greatest(T v1, T v2, ...)|T| 求最大值
least(T v1, T v2, ...)|T| 求最小值

## 2.集合函数 ##
函数名|返回类型|描述
---|---|---|
size(Map<K.V>)|int|求map的长度
size(Array<T>)|int|求数组的长度
map_keys(Map<K.V>)|array<K>|返回map中的所有key
map_values(Map<K.V>)|array<V>|返回map中的所有value
array_contains(Array<T>, value)|boolean|如该数组Array<T>包含value返回true。，否则返回false
sort_array(Array<T>)|array|按自然顺序对数组进行排序并返回

## 3.类型转换函数 ##
函数名|返回类型|描述
---|---|---|
binary(string/binary)|binary|将输入的值转换成二进制
cast(expr as <type>)|Expected "=" to follow "type"|将expr转换成type类型 如：cast("1" as BIGINT) 将字符串1转换成了BIGINT类型，如果转换失败将返回NULL

## 4.日期函数 ##
函数名|返回类型|描述
---|---|---|
from_unixtime(bigint unixtime[, string format])|string|将时间的秒值转换成format格式（format可为“yyyy-MM-dd hh:mm:ss”,“yyyy-MM-dd hh”,“yyyy-MM-dd hh:mm”等等）如from_unixtime(1250111000,"yyyy-MM-dd") 得到2009-03-12
unix_timestamp()|bigint|获取本地时区下的时间戳
unix_timestamp(string date)|bigint|将格式为yyyy-MM-dd HH:mm:ss的时间字符串转换成时间戳  如unix_timestamp('2009-03-20 11:30:01') = 1237573801
unix_timestamp(string date, string pattern)|bigint|将指定时间字符串格式字符串转换成Unix时间戳，如果格式不对返回0 如：unix_timestamp('2009-03-20', 'yyyy-MM-dd') = 1237532400
to_date(string timestamp)|string|返回时间字符串的日期部分
year(string date)|int|返回时间字符串的年份部分
quarter(date/timestamp/string)|int|返回当前时间属性哪个季度 如quarter('2015-04-08') = 2
month(string date)|int|返回时间字符串的月份部分
day(string date) dayofmonth(date)|int|返回时间字符串的天
hour(string date)|int|返回时间字符串的小时
minute(string date)|int|返回时间字符串的分钟
second(string date)|int|返回时间字符串的秒
weekofyear(string date)|int|返回时间字符串位于一年中的第几个周内  如weekofyear("1970-11-01 00:00:00") = 44, weekofyear("1970-11-01") = 44
datediff(string enddate, string startdate)|int|计算开始时间startdate到结束时间enddate相差的天数
date_add(string startdate, int days)|string|从开始时间startdate加上days
date_sub(string startdate, int days)|string|从开始时间startdate减去days
from_utc_timestamp(timestamp, string timezone)|timestamp|如果给定的时间戳并非UTC，则将其转化成指定的时区下时间戳
to_utc_timestamp(timestamp, string timezone)|timestamp|如果给定的时间戳指定的时区下时间戳，则将其转化成UTC下的时间戳
current_date|date|返回当前时间日期
current_timestamp|timestamp|返回当前时间戳
add_months(string start_date, int num_months)|string|返回当前时间下再增加num_months个月的日期
last_day(string date)|string|返回这个月的最后一天的日期，忽略时分秒部分（HH:mm:ss）
next_day(string start_date, string day_of_week)|string|返回当前时间的下一个星期X所对应的日期 如：next_day('2015-01-14', 'TU') = 2015-01-20  以2015-01-14为开始时间，其下一个星期二所对应的日期为2015-01-20
trunc(string date, string format)|string|返回时间的最开始年份或月份  如trunc("2016-06-26",“MM”)=2016-06-01  trunc("2016-06-26",“YY”)=2016-01-01   注意所支持的格式为MONTH/MON/MM, YEAR/YYYY/YY
months_between(date1, date2)|double|返回date1与date2之间相差的月份，如date1>date2，则返回正，如果date1<date2,则返回负，否则返回0.0  如：months_between('1997-02-28 10:30:00', '1996-10-30') = 3.94959677  1997-02-28 10:30:00与1996-10-30相差3.94959677个月
date_format(date/timestamp/string ts, string fmt)|string|按指定格式返回时间date 如：date_format("2016-06-22","MM-dd")=06-22

## 5.条件函数 ##
函数名|返回类型|描述
---|---|---|
if(boolean testCondition, T valueTrue, T valueFalseOrNull)|T|如果testCondition 为true就返回valueTrue,否则返回valueFalseOrNull ，（valueTrue，valueFalseOrNull为泛型） 
nvl(T value, T default_value)|T|如果value值为NULL就返回default_value,否则返回value
COALESCE(T v1, T v2, ...)|T|返回第一非null的值，如果全部都为NULL就返回NULL  如：COALESCE (NULL,44,55)=44/strong>
CASE a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END|T|如果a=b就返回c,a=d就返回e，否则返回f  如CASE 4 WHEN 5  THEN 5 WHEN 4 THEN 4 ELSE 3 END 将返回4
CASE WHEN a THEN b [WHEN c THEN d]* [ELSE e] END|T|如果a=ture就返回b,c= ture就返回d，否则返回e  如：CASE WHEN  5>0  THEN 5 WHEN 4>0 THEN 4 ELSE 0 END 将返回5；CASE WHEN  5<0  THEN 5 WHEN 4<0 THEN 4 ELSE 0 END 将返回0
isnull( a )|boolean|如果a为null就返回true，否则返回false
isnotnull ( a )|boolean|如果a为非null就返回true，否则返回false

## 6.字符函数 ##
函数名|返回类型|描述
---|---|---|
ascii(string str)|int|返回str中首个ASCII字符串的整数值
base64(binary bin)|string|将二进制bin转换成64位的字符串
concat(string/binary A, string/binary B...)|string|对二进制字节码或字符串按次序进行拼接
context_ngrams(array<array<string>>, array<string>, int K, int pf)|array<struct<string,double>>|与ngram类似，但context_ngram()允许你预算指定上下文(数组)来去查找子序列，具体看StatisticsAndDataMining(这里的解释更易懂)
concat_ws(string SEP, string A, string B...)|string|与concat()类似，但使用指定的分隔符喜进行分隔
concat_ws(string SEP, array<string>)|string|拼接Array中的元素并用指定分隔符进行分隔
decode(binary bin, string charset)|string|使用指定的字符集charset将二进制值bin解码成字符串，支持的字符集有：'US-ASCII', 'ISO-8859-1', 'UTF-8', 'UTF-16BE', 'UTF-16LE', 'UTF-16'，如果任意输入参数为NULL都将返回NULL
encode(string src, string charset)|binary|使用指定的字符集charset将字符串编码成二进制值，支持的字符集有：'US-ASCII', 'ISO-8859-1', 'UTF-8', 'UTF-16BE', 'UTF-16LE', 'UTF-16'，如果任一输入参数为NULL都将返回NULL
find_in_set(string str, string strList)|int|返回以逗号分隔的字符串中str出现的位置，如果参数str为逗号或查找失败将返回0，如果任一参数为NULL将返回NULL回
format_number(number x, int d)|string|将数值X转换成"#,###,###.##"格式字符串，并保留d位小数，如果d为0，将进行四舍五入且不保留小数
get_json_object(string json_string, string path)|string|从指定路径上的JSON字符串抽取出JSON对象，并返回这个对象的JSON格式，如果输入的JSON是非法的将返回NULL,注意此路径上JSON字符串只能由数字 字母 下划线组成且不能有大写字母和特殊字符，且key不能由数字开头，这是由于Hive对列名的限制
in_file(string str, string filename)|boolean|如果文件名为filename的文件中有一行数据与字符串str匹配成功就返回true
instr(string str, string substr)|int|查找字符串str中子字符串substr出现的位置，如果查找失败将返回0，如果任一参数为Null将返回null，注意位置为从1开始的
length(string A)|int|返回字符串的长度
locate(string substr, string str[, int pos])|int|查找字符串str中的pos位置后字符串substr第一次出现的位置
lower(string A) lcase(string A)|string|将字符串A的所有字母转换成小写字母
lpad(string str, int len, string pad)|string|从左边开始对字符串str使用字符串pad填充，最终len长度为止，如果字符串str本身长度比len大的话，将去掉多余的部分
ltrim(string A)|string|去掉字符串A前面的空格
ngrams(array<array<string>>, int N, int K, int pf)|array<struct<string,double>>|返回出现次数TOP K的的子序列,n表示子序列的长度，具体看StatisticsAndDataMining (这里的解释更易懂)
parse_url(string urlString, string partToExtract [, string keyToExtract])|string|返回从URL中抽取指定部分的内容，参数url是URL字符串，而参数partToExtract是要抽取的部分，这个参数包含(HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, and USERINFO,例如：parse_url('http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1', 'HOST') ='facebook.com'，如果参数partToExtract值为QUERY则必须指定第三个参数key  如：parse_url('http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1', 'QUERY', 'k1') =‘v1’
printf(String format, Obj... args)|string|按照printf风格格式输出字符串
regexp_extract(string subject, string pattern, int index)|string|抽取字符串subject中符合正则表达式pattern的第index个部分的子字符串，注意些预定义字符的使用，如第二个参数如果使用'\s'将被匹配到s,'\\s'才是匹配空格
regexp_replace(string INITIAL_STRING, string PATTERN, string REPLACEMENT)|string|按照Java正则表达式PATTERN将字符串INTIAL_STRING中符合条件的部分成REPLACEMENT所指定的字符串，如里REPLACEMENT这空的话，抽符合正则的部分将被去掉  如：regexp_replace("foobar", "oo/ar", "") = 'fb.' 注意些预定义字符的使用，如第二个参数如果使用'\s'将被匹配到s,'\\s'才是匹配空格
repeat(string str, int n)|string|重复输出n次字符串str
reverse(string A)|string|反转字符串
rpad(string str, int len, string pad)|string|从右边开始对字符串str使用字符串pad填充，最终len长度为止，如果字符串str本身长度比len大的话，将去掉多余的部分
rtrim(string A)|string|去掉字符串后面出现的空格
sentences(string str, string lang, string locale)|array<array<string>>|字符串str将被转换成单词数组，如：sentences('Hello there! How are you?') =( ("Hello", "there"), ("How", "are", "you") )
space(int n)|string|返回n个空格
split(string str, string pat)|array|按照正则表达式pat来分割字符串str,并将分割后的数组字符串的形式返回
str_to_map(text[, delimiter1, delimiter2])|map<string,string>|将字符串str按照指定分隔符转换成Map，第一个参数是需要转换字符串，第二个参数是键值对之间的分隔符，默认为逗号;第三个参数是键值之间的分隔符，默认为"="
substr(string/binary A, int start) substring(string/binary A, int start)|string|对于字符串A,从start位置开始截取字符串并返回
substr(string/binary A, int start, int len) substring(string/binary A, int start, int len)|string|对于二进制/字符串A,从start位置开始截取长度为length的字符串并返回
substring_index(string A, string delim, int count)|string|截取第count分隔符之前的字符串，如count为正则从左边开始截取，如果为负则从右边开始截取
translate(string/char/varchar input, string/char/varchar from, string/char/varchar to)|string|将input出现在from中的字符串替换成to中的字符串 如：translate("MOBIN","BIN","M")="MOM"
trim(string A)|string|将字符串A前后出现的空格去掉
unbase64(string str)|binary|将64位的字符串转换二进制值
upper(string A) ucase(string A)|string|将字符串A中的字母转换成大写字母
initcap(string A)|string|将字符串A转换第一个字母大写其余字母的字符串
levenshtein(string A, string B)|int|计算两个字符串之间的差异大小  如：levenshtein('kitten', 'sitting') = 3
soundex(string A)|string|将普通字符串转换成soundex字符串

## 7.聚合函数  ##
函数名|返回类型|描述
---|---|---|
count(\*), count(expr), count(DISTINCT expr[, expr...])|INT|统计总行数
sum(col), sum(DISTINCT col)|DOUBLE|sum(col),表示求指定列的和，sum(DISTINCT col)表示求去重后的列的和
avg(col), avg(DISTINCT col)|DOUBLE|avg(col),表示求指定列的平均值，avg(DISTINCT col)表示求去重后的列的平均值
min(col)|DOUBLE|求指定列的最小值
max(col)|DOUBLE|求指定列的最大值
variance(col), var_pop(col)|DOUBLE|求指定列数值的方差
var_samp(col)|DOUBLE|求指定列数值的样本方差
stddev_pop(col)|DOUBLE|求指定列数值的标准偏差
stddev_samp(col)|DOUBLE|求指定列数值的样本标准偏差
covar_pop(col1, col2)|DOUBLE|求指定列数值的协方差
covar_samp(col1, col2)|DOUBLE|求指定列数值的样本协方差
corr(col1, col2)|DOUBLE|返回两列数值的相关系数
percentile(BIGINT col, p)|DOUBLE|返回col的p%分位数
## 8.表生成函数 ##
函数名|返回类型|描述
---|---|---|
explode(array<TYPE> a)|Array Type|对于a中的每个元素，将生成一行且包含该元素
explode(ARRAY)|N rows|每行对应数组中的一个元素
explode(MAP)|N rows|每行对应每个map键-值，其中一个字段是map的键，另一个字段是map的值
posexplode(ARRAY)|N rows|与explode类似，不同的是还返回各元素在数组中的位置
stack(INT n, v_1, v_2, ..., v_k)|N rows|把M列转换成N行，每行有M/N个字段，其中n必须是个常数
json_tuple(jsonStr, k1, k2, ...)|tuple|从一个JSON字符串中获取多个键并作为一个元组返回，与get_json_object不同的是此函数能一次获取多个键值
parse_url_tuple(url, p1, p2, ...)|tuple|返回从URL中抽取指定N部分的内容，参数url是URL字符串，而参数p1,p2,....是要抽取的部分，这个参数包含HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, USERINFO, QUERY:<KEY>

## 9.示例代码 ##

创建表  
```
create table table1(A string, B string ,C string)  
ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.MultiDelimitSerDe'  
WITH SERDEPROPERTIES ("field.delim"="@|@")  
STORED AS TEXTFILE  
tblproperties('creator'='yangql','created_at'='2017-05-24')  ;
```

--数据  
```
190@|@1030,1031,1032,1033,1190@|@select id
191@|@1030,1031,1032,1033,1190@|@select id
```
--加载数据
```
load data local inpath '/home/yangql/proc/hive/testData.txt' into table table1;
```
--查询
```
hive (test)> select * from table1;
OK
table1.a	table1.b	table1.c
190	1030,1031,1032,1033,1190	select id
191	1030,1031,1032,1033,1190	select id
Time taken: 0.072 seconds, Fetched: 2 row(s)
```
- lateral view:lateral view用于和split、explode等UDTF一起使用的，能将一行数据拆分成多行数据，在此基础上可以对拆分的数据进行聚合，lateral view首先为原始表的每行调用UDTF，UDTF会把一行拆分成一行或者多行，lateral view在把结果组合，产生一个支持别名表的虚拟表。  
执行过程是先执行from到 as cloumn的列过程，在执行select 和where后边的语句；
```
hive (test)> select t1.a,t2.b,t1.b,t1.c from table1 t1 lateral view explode(split(t1.b,',')) t2 as b;
OK
t1.a	t2.b	t1.b	t1.c
190	1030	1030,1031,1032,1033,1190	select id
190	1031	1030,1031,1032,1033,1190	select id
190	1032	1030,1031,1032,1033,1190	select id
190	1033	1030,1031,1032,1033,1190	select id
190	1190	1030,1031,1032,1033,1190	select id
191	1030	1030,1031,1032,1033,1190	select id
191	1031	1030,1031,1032,1033,1190	select id
191	1032	1030,1031,1032,1033,1190	select id
191	1033	1030,1031,1032,1033,1190	select id
191	1190	1030,1031,1032,1033,1190	select id
```
- explode
```
hive (test)> select explode(split(b,',')) from table1;
OK
col
1030
1031
1032
1033
1190
1030
1031
1032
1033
1190
Time taken: 0.073 seconds, Fetched: 10 row(s)
```
- posexplode
```
hive (test)> select posexplode(split(b,',')) from table1;
OK
pos	val
0	1030
1	1031
2	1032
3	1033
4	1190
0	1030
1	1031
2	1032
3	1033
4	1190
Time taken: 0.077 seconds, Fetched: 10 row(s)
```
