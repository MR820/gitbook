```shell
#!/bin/sh
# config
RANGE=299
DELAY=60
LOC_WORK=/etl
LOC_DATA_DTA=${LOC_WORK}/dta
TSK_TBL=gwcxxx
#15天的秒数
DAYS=15
DAYSNUM=$(($DAYS*86400))
HOST="10.19.106.109"
USER="cusorder"
PASS="gwcxxx"
PORT=3306
# function
# main
##fun rlog
function rlog(){
RDKNAME="SH-21377-wsd-gwcxxx"
RDRANGE=5
RDLIMIT=20
RDRUNAT=$1
RDCHKAT=$(date '+%Y-%m-%d %H:%M:%S')
#curl "http://10.0.2.2/st?r=set&k=${RDKNAME};&v=${RDRANGE}!${RDLIMIT}!${RDRUNAT}!${RDCHKAT};" >/dev/null 2>&1
}
##end rlog
while [ 1 ]
do
    # 1 get time from file
    timepre_t=`head -1 update.txt` #文件中记录的时间 head类比tail
    timepre_s=$(date +%s -d "$timepre_t") #日期时间转为unix时间
    timepre15_s=$(($timepre_s-$DAYSNUM)) #15天之前
    timepre15_t=$(date '+%Y%m%d%H%M%S' -d "1970-01-01 UTC $timepre15_s seconds"); #unix时间戳格式化 20200512000000
    timepre15_d=$(date '+%Y%m%d' -d "1970-01-01 UTC $timepre15_s seconds"); #20200512
    timepre=$(date +%Y%m%d%H%M%S -d "$timepre_t") #日期时间(2020-05-27 00:00:00)格式化 20200527000000
    datepre=$(date +%Y%m%d -d "$timepre_t") # 20200527
    # 2 get cur work time (need change range->300 min->3)
    timecur_s=$(($timepre_s + $RANGE))
    timecur_t=$(date '+%Y-%m-%d %H:%M:%S' -d "1970-01-01 UTC $timecur_s seconds");
    timecur=$(date +%Y%m%d%H%M%S -d "$timecur_t")
    # 3 get next time
    timenxt_s=$(($timepre_s + $RANGE + 1))
    timenxt_t=$(date '+%Y-%m-%d %H:%M:%S' -d "1970-01-01 UTC $timenxt_s seconds");
    # 4 get now sys time
    timenow_t=$(date '+%Y-%m-%d %H:%M:%S' -d '-2 min')
    timenow_s=$(date +%s -d "$timenow_t")

    LOC_DATA_SAV=${LOC_WORK}/dta/${datepre}
    PRE15_TSK_CSV=${LOC_WORK}/dta/${timepre15_d}/${TSK_TBL}_${timepre15_t}
    TSK_CSV="${LOC_DATA_SAV}/${TSK_TBL}_${timepre}"
    if [ ! -d "$LOC_DATA_SAV" ]; then #[ ! -d "/etl/dta/20200507" ]判断是否存在目录
	    mkdir "$LOC_DATA_SAV" #不存在就创建
    fi
    # 5 if have task
    if [ $timecur_s -lt $timenow_s ]; then #判断执行时间<当前时间
        # 5-1 do by 5 minute
        echo "!!!begin<$(date)> [${timepre_t}, ${timecur_t}]"
        sed -e "s/#1/$timepre_t/g" -e "s/#2/$timecur_t/g" gwcxxx.sql > $TSK_CSV.sql #sed -e "s/old/new/g" 取代 变量替换
        if [ -e $TSK_CSV.csv ]; then #判断文件是否存在
          echo "!!!dup"
          echo "$TSK_CSV.csv exists!"
        else
          echo "!!!exp"
          mysql -h $HOST -P $PORT -u $USER --skip-column-names --raw --password=$PASS < $TSK_CSV.sql > $TSK_CSV.csv #生成csv文件 --skip-column-names:输出结果无列名
        fi

        if [ "$?" == "0" ]; then #$?:执行上一条指令的返回值 0表示没有错误
            echo "$timenxt_t" > update.txt #下一次数据库开始时间
            rlog "$timenxt_t" #rlog方法，脚本健康检查
	          echo "$TSK_CSV.csv|$(cat $TSK_CSV.csv | wc -l)|$(/bin/ls -l $TSK_CSV.csv | cut -d' ' -f 5)" > $TSK_CSV.flg #/etl/dta/20200618/gwcxxx_20200618100500.csv|3|141   cut -d' ' -f 5 以' '隔开，提取第五个
            sleep 1
            echo "!!!done <$(date)>done"
        else
            echo "   error for wait"
            sleep $DELAY
        fi
        #删除15天前的csv flg sql文件
        if [ -e $PRE15_TSK_CSV.csv ]; then
              rm -rf $PRE15_TSK_CSV.*
        echo "!!!file $PRE15_TSK_CSV.csv rm-succeed <$(date)>done!!!"
            sleep 1
        else
            echo  "!!!file $PRE15_TSK_CSV.csv -notFind <$(date)>!!!"
            sleep 1
        fi
    else
        # 4-8 delay
        echo "!!!sleep<$(date)>"
        sleep $DELAY
    fi
#exit
#if [ "${timepre_t:0:10}" == "2012-04-14" ]; then
# exit
#fi
done


```