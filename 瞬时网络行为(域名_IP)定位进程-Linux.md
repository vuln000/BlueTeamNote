经常会有这样的场景，已知某个域名或IP为恶意，有设备解析了恶意域名或通联了恶意IP。或者某个无出网业务的设备异常解析了域名、或通联了恶意IP。
此时的首要任务就是基于已知的信息（域名/IP）来找到发起请求的恶意进程，提取样本进行分析研判。对于持续的IP通联、问题较为简单、使用netstat、ss、lsof等工具找到对应恶意IP的pid后，去/proc/pid目录即可获取相关进程信息。
但是对于恶意进程使用了域名解析、尤其是使用了CDN、云函数的情况，外部IP地址可能持续在变化。再加上恶意进程可能存在沉睡机制，域名也非持续解析，几小时或者几天才解析一次。IP几个小时或者几天才连接一次。这样的情况也常常存在。我们使用netstat等命令，在执行瞬间往往看不到相关进程信息，难以定位恶意进程。
为了解决上述问题、这里记录基于非持续恶意域名、IP连接定位进程的两种思路。
# 通过解析行为定位进程
## IP地址固定的情况
对于固定的恶意IP通联，只是每次通联的时间不定的情况。使用bash脚本，持续通过命令获取IP通联及进程pid信息，在发现目标恶意IP通联出现时，保存进程状态和信息。
脚本如下(root权限，脚本仅在ubuntu下测试过，其他系统需测试)
```bash
#!/bin/bash

# 检查目录是否存在,不存在则创建
DUMP_DIR="/tmp/iphunter"
if [ ! -d "$DUMP_DIR" ]; then
  mkdir "$DUMP_DIR"
fi

# 检查输入是否合法
if [ -z "$1" ]; then
  echo "请输入要监控的IP地址"
  exit 1
fi
IP_ADDR="$1"

echo "开始监控IP地址: $IP_ADDR"

while true; do
  # 查找与该IP地址建立TCP连接的进程
  TCP_PIDS=$(ss -tnp | grep "$IP_ADDR" | awk '{print $6}' | cut -d',' -f2 | cut -d'=' -f2)
  
  # 查找与该IP地址建立UDP连接的进程
  UDP_PIDS=$(ss -unp | grep "$IP_ADDR" | awk '{print $5}' | cut -d',' -f2 | cut -d'=' -f2)
  
  # 合并TCP和UDP进程ID
  PIDS="$TCP_PIDS $UDP_PIDS"
  
  if [ -n "$PIDS" ]; then
    for PID in $PIDS; do
      # 获取进程信息
      PROC_PATH=$(readlink -f /proc/$PID/exe)
      DUMP_FILE="$DUMP_DIR/iphunter_$PID.dump"
      CMDLINE_FILE="$DUMP_DIR/iphunter_$PID.cmdline"
      
      # 输出进程信息
      echo "发现与IP地址 $IP_ADDR 建立连接的进程:"
      echo "PID: $PID"
      echo "程序路径: $PROC_PATH"
      
      # 使用 gcore 将进程内存转储到文件
      echo "正在转储进程内存至 $DUMP_FILE ..."
      sudo gcore -o $DUMP_FILE $PID > /dev/null 2>&1
      echo "内存转储完成"
      
      # 保存进程的命令行参数
      echo "正在保存进程的命令行参数至 $CMDLINE_FILE ..."
      cat /proc/$PID/cmdline > $CMDLINE_FILE
      echo "命令行参数保存完成"
      
      # 跳出循环并终止程序
      break 2
    done
  fi
  
  # 等待 0.1 秒后再次检查
  sleep 0.1
done

echo "程序已终止"
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/38747917/1723079999829-b8fe7034-0a34-4fd0-a9a1-c5e71e631d35.png#averageHue=%23421f37&clientId=u73de84f3-be66-4&from=paste&height=160&id=ua39f8ad0&originHeight=160&originWidth=552&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=86159&status=done&style=none&taskId=u64e76923-8f5c-4b31-8fa6-77831c73e2c&title=&width=552)
## 使用CDN、云函数等、IP地址变化的情况
对于这种情况，修改/etc/hosts文件，将恶意域名指向一个IP、然后使用上述域名等待TCP或者UDP连接。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/38747917/1723117404536-69f50f32-1c58-42db-977a-c29a5d93cc8b.png#averageHue=%2338142c&clientId=u6ef50a6b-501c-4&from=paste&height=213&id=uda43e6bf&originHeight=266&originWidth=1351&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=59258&status=done&style=none&taskId=u77e1e8a5-848e-40f6-8325-6205da0ffe8&title=&width=1080.8)
# 

