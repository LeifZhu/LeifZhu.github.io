---
title: 使用bash脚本监控程序输出并自动执行任务
date: 2018-09-02 21:32:19
tags: [linux, bash]
category: 人生经验
---
有时候我们需要执行这样的任务：当程序执行到一定的条件时就杀死进程，执行下一进程，这种“条件”通常是通过程序的输出来反馈的。我们不希望一直盯着屏幕来观察条件是否达到，用bash脚本来执行这种任务是一个明智的选择。
<!--more-->

## 问题描述
我要做的事情可以用下面的流程图描述。该任务的目标是检查不同的配置参数对训练运行速度的影响。

{% asset_img task.svg 任务描述 %}

## 使用bash脚本监控程序运行
bash脚本如下。思路是训练程序的输出重定向到文件mx_alexnet.log，然后每隔2秒用tail查看mx_alexnet.log的最后一行，检测loss的最新值, 如果达到target，就结束程序，然后重新开始。

```bash
#!/bin/bash

function run()
{
	interval=2
	target=1.5	
	logfile=mx_alexnet.log

	# set knobs
	export MXNET_CPU_WORKER_NTHREADS=$1
	export MXNET_CPU_PRIORITY_NTHREADS=$2
	export MXNET_CPU_NNPACK_NTHREADS=$3
	export MXNET_EXEC_ENABLE_INPLACE=$4

    # start training
	python code/train_imagenet.py --network alexnet \
				 --num-classes 8 \
				 --data-train /root/zl_workspace/dataset/imagenet8/mxnet-format/train_rec.rec \
				 --data-val /root/zl_workspace/dataset/imagenet8/mxnet-format/val_rec.rec \
				 --num-examples 10400 \
				 --num-epochs 20 \
				 --loss 'ce' \
				 --disp-batches 1 \
				 --batch-size 256 \
				 --lr 0.01 \
				 --lr-step-epochs 5,10 \
				 >$logfile 2>&1 &
	
	# stat the time
	begin_time=$(date +%s)
	sleep 10

	# monitor loss
	for((;;))
	do
		tail -n1 $logfile
		loss=$(tail -n1 $logfile | sed -n 's/.*\scross-entropy=\([0-9]\+.[0-9]\+\)*/\1/p') 
		if [ -n "$loss" ] 
		then
			if [ $(echo "${loss} < ${target}" | bc) -eq 1 ]
			then
				break
			else
				sleep $interval
			fi
		else
			sleep $interval
		fi
	done

	# record the time of runnig
	end_time=$(date +%s)
	run_time=$((end_time - begin_time))
	echo "$1, $2, $3, $4, $run_time" >> "$5"
	echo "training finshed! Cost ${run_time} s."

	# kill the process
	ps aux | grep "python code/train_imagenet.py" | grep -v "grep" | awk {'print $2'} | xargs kill
	echo "killed process successfully!"
	rm -f $logfile
	echo "finished removal successfully!"
}

# make sure the training is not going currently
ps aux | grep "python code/train_imagenet.py" | grep -v "grep" | awk {'print $2'} | xargs kill

# create a file to record running time
rfile=results.csv
touch $rfile
echo "mcwn, mcpn, mcnn, meei, run_time" > $rfile

for mcwn in 1 2 4 8; do
	for mcpn in 4 1 2 8; do
		for mcnn in 4 1 2 8; do
			for meei in true false; do
				if [ $(($mcwn + $mcpn + $mcnn)) -le 16 ]
				then
					run $mcwn $mcpn $mcnn $meei $rfile
					echo "waiting for restart..."
					sleep 10
				fi
			done
		done
	done
done
```

## 几个细节
### bash中比较浮点数的大小
bash本身是只支持比较整数的大小，要比较浮点数大小需要借助程序`bc`，所以上面中的代码中检测loss值用的是
```bash
if [ $(echo "${loss} < ${target}" | bc) -eq 1 ]
    ...
```
而不是
```bash
if [ ${loss} < ${target} ]
```
详情请 在terminal下`man bc`。
### bash中的复合逻辑表达式
注意脚本中的这一段
```bash
if [ -n "$loss" ] 
then
    if [ $(echo "${loss} < ${target}" | bc) -eq 1 ]
    then
        break
    else
        sleep $interval
    fi
else
    sleep $interval
fi
```
为什么不写成下面这样？
```bash
if [ -n "$loss" -a $(echo "${loss} < ${target}" | bc) -eq 1 ]
then
    break
else
    sleep $interval
fi
```
因为bash脚本中的复合逻辑表达式不像C语言中那样会被优化，当正则表达式没有匹配到值而使loss为空时，尽管`-n "$loss"`为真，`-a`后面的 `$(echo "${loss} < ${target}" | bc) -eq 1`仍然会执行，然后就会报错。

## bash script教程及几个常用的命令

bash教程：[中文](http://www.runoob.com/linux/linux-shell.html)-[英文](https://www.tutorialspoint.com/unix/)

本文中涉及的几个常用的命令
1. sed: [here](http://man.linuxde.net/sed)
2. grep: [here](http://man.linuxde.net/grep)
3. awk: [here](http://man.linuxde.net/awk)
4. xargs: [教程[1]](https://blog.gtwang.org/linux/xargs-command-examples-in-linux-unix/) - [教程[2]](http://man.linuxde.net/xargs)