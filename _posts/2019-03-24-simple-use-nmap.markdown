# nmap的简单使用

```bash
nmap 192.168.1.105
nmap localhost # 扫描自己的系统
nmap 192.168.199.172 192.168.199.160 # 同时扫描多个
nmap -p 22,80 localhost # 扫描特定端口
nmap -sn 192.168.1.0/24 # 跳过端口检测，只执行简单的ping来检查那些系统在开机运行
namp -O localhost # 识别目标系统的系统类型
nmap -sV localhost # -sV: Probe open ports to determine service/version info
```
