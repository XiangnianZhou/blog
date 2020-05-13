从网上下载的电影字幕文件，大部分是srt，ass\ssa 这类的文本文件，不过有些是压缩包。需要批量解压这些文件，并清理不合格文件和目录。
我大概想了想，用 shell 脚本应该是比较轻量、简单而且代码量小的方式了，于是写了这个我**平生第一个有用的 shell 脚本**。

脚本功能：遍历 subtitle 目录，然后在电影目录下的英文（eng）目录执行一些操作：
*  如果是压缩文件，解压
*  清理目录
*  清理压缩包
*  清理多文件目录
*  清理非指定格式文件
*  更新数据库

```sh
#!/bash/bin
source ./mysql.sh

for dir in `ls subtitle`
do
    echo 正在处理 $dir
    dir_eng=subtitle/$dir/eng
    if [ -d $dir_eng ]; then
        file=`ls $dir_eng `
        # 解压 zip 和 rar, overwrite mode
        if [[ "$file" =~ zip$ ]]; then
            unzip -q  -o "$dir_eng/*.zip" -d $dir_eng
        elif [[ "$file" =~ rar$ ]]; then
            rar  x -inul -o+ "$dir_eng/*.rar" -d $dir_eng 
        fi
        echo  '  已解压'

        #### 解压后,处理目录:  ####

        # 1. 删除 eng 下的所有目录
        dir_in_eng=`ls -l $dir_eng | grep -E "^d"`
        if [ "$dir_in_eng" != "" ]; then
            find $dir_eng/* -type d  | xargs rm -rf
        fi

        # 2. 删除所有压缩文件
        rm -f $dir_eng/*.{zip,rar}

        # 3. 清空多文件目录
        file_num=`ls $dir_eng | wc -l`
        if [ $file_num -gt 1 ]; then
            rm $dir_eng/*
        fi

        # 4. 不是 srt 和 ass/ssa 的文件删除
        # grep 貌似不支持环视, -P 使用 Perl 语法
        err_file=`ls $dir_eng | grep -Po '.*(?<!srt|ass|ssa)$'`
        if [ -n "$err_file" ]; then
            rm -f $dir_eng/$err_file
        fi

        echo '  文件清理已完成'
    fi

    # 5. 空目录需要更新数据库
    file_num=`ls $dir_eng | wc -l`
    if  [ "$file_num" = "0" ]; then
        mysql $SQLINFO "UPDATE subtitle SET subtitle_eng='' WHERE zmk_id = $dir"
        echo "  数据库已更新"
    fi
    echo -e "  任务已结束\n"
done
```
