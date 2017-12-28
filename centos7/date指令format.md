#man date可以看到date的help文件
　　#date 获取当前时间
　　#date -d "-1week" +%Y%m%d 获取上周日期（day,month,year,hour）
　　#date--date="-24 hour" +%Y%m%d 同上
　　date_now=`date+%s` shell脚本里面赋给变量值
　　%% 输出%符号
　　%a 当前域的星期缩写 (Sun..Sat)
　　%A 当前域的星期全写 (Sunday..Saturday)
　　%b 当前域的月份缩写(Jan..Dec)
　　%B 当前域的月份全称 (January..December)
　　%c 当前域的默认时间格式 (Sat Nov 04 12:02:33 EST 1989)
　　%C n百年 [00-99]
　　%d 两位的天 (01..31)

　　%D 短时间格式 (mm/dd/yy)

　　%e 短格式天 ( 1..31)

　　%F 文件时间格式 same as %Y-%m-%d

　　%h same as %b

　　%H 24小时制的小时 (00..23)

　　%I 12小时制的小时 (01..12)

　　%j 一年中的第几天 (001..366)

　　%k 短格式24小时制的小时 ( 0..23)

　　%l 短格式12小时制的小时 ( 1..12)

　　%m 双位月份 (01..12)

　　%M 双位分钟 (00..59)

　　%n 换行

　　%N 十亿分之一秒(000000000..999999999)

　　%p 大写的当前域的上下午指示 (blank in many locales)

　　%P 小写的当前域的上下午指示 (blank in many locales)

　　%r 12小时制的时间表示（时:分:秒,双位） time, 12-hour (hh:mm:ss [AP]M)
　　%R 24小时制的时间表示 （时:分,双位）time, 24-hour (hh:mm)
　　%s 自基础时间 1970-01-01 00:00:00 到当前时刻的秒数(a GNU extension)
　　%S 双位秒 second (00..60);
　　%t 横向制表位(tab)
　　%T 24小时制时间表示(hh:mm:ss)
　　%u 数字表示的星期（从星期一开始 1-7）
　　%U 一年中的第几周 星期天为开始 (00..53)
　　%V 一年中的第几周 星期一为开始 (01..53)
　　%w 一周中的第几天 星期天为开始 (0..6)
　　%W 一年中的第几周 星期一为开始 (00..53)
　　%x 本地日期格式 (mm/dd/yy)
　　%X 本地时间格式 (%H:%M:%S)
　　%y 两位的年(00..99)
　　%Y 年 (1970…)
　　例子：编写shell脚本计算离自己生日还有多少天？

[plain] view plain copy
read -p"Input your birthday(YYYYmmdd):" date1
　　m=`date--date="$date1" +%m` #得到生日的月
　　d=`date--date="$date1" +%d` #得到生日的日
　　date_now=`date+%s` #得到当前时间的秒值
　　y=`date +%Y` #得到当前时间的年
　　birth=`date--date="$y$m$d" +%s` #得到今年的生日日期的秒值
　　internal=$(($birth-$date_now)) #计算今日到生日日期的间隔时间
　　if ["$internal" -lt "0" ]; then #判断今天的生日是否已过
　　birth=`date--date="$(($y+1))$m$d" +%s` #得到明天的生日日期秒值
　　internal=$(($birth-$date_now))#计算今天到下一个生日的间隔时间
　　fi
　　echo "Thereis :$((einternal/60/60/24)) days." #输出结果，秒换算为天



创建以当前时间为文件名的

mkdir `date+%Y%m%d`

备份以时间做为文件名的

tar cvf./htdocs`date +%Y%m%d`.tar ./*



date命令如何获得上星期的日期?

lastmonth=`date+%Y-%m --date="-1 month"`"-01";

然后 awk -v 作为一个参数这样传进去简单点。

#date -d "-1week" "+%Y%m%d %A"

或者：#date -d "last week" +%Y%m%d

举一反三:date命令获得前天，昨天，明天，后天，上个月，下个月的日期

   date -d "-2 day" "+%Y%m%d %A"   或 date --date="2 days ago""+%Y%m%d %A"

   date -d "-1 day" "+%Y%m%d %A"   或 date --date="1 day ago""+%Y%m%d %A"

   date -d "1 day" "+%Y%m%d %A"

   date -d "2 day" "+%Y%m%d %A"

   date -d "-1 month" "+%Y%m%d %A"

   date -d "1 month" "+%Y%m%d %A"

echo `date -d"-1 month" "+%Y-%m-%d %X"`



还有其他用法:

date+%s   可以得到UNIX的时间戳;

已知时间戳，求当前日期
date --date="19700101 +$(($((`date +%s`))+(8*60*60))) sec" +%Y%m%d" "%H:%M:%S
PS:从1970年01月01日开始算， +$(($((`date +%s`))+(8*60*60))) sec 表示过了多少秒，其中(8*60*60)表示8个小时，时区问题，我国时间与世界时间相差8小时。

转自：http://linux.chinaunix.net/bbs/forum-2-1.html
