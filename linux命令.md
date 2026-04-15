# linux命令

### pactl

```
#设置系统音量为 95% 的命令
pactl set-sink-volume @DEFAULT_SINK@ 95%
```
```
#查询默认音频输出设备的音量和静音状态
pactl get-sink-volume @DEFAULT_SINK@
```
输出示例：
```
Volume: front-left: 65536 / 100% / 0.00 dB,   front-right: 65536 / 100% / 0.00 dB
```
65536 是 PulseAudio 内部最大值（对应 100%）
如果显示 0 / 0%，说明音量被调到最低（无声）

```
#查询是否被静音
pactl get-sink-mute @DEFAULT_SINK@
```
输出示例：

```
Mute: no
```
如果显示 Mute: yes，说明被静音了，即使音量是 100% 也不会出声！

```
#声卡查询
pactl list sinks short
```

```
#手动测试
pactl set-default-sink alsa_output.usb-GeneralPlus_USB_Audio_Device-00.analog-stereo
```

```
#自动查usb声卡脚本
#!/bin/bash

# 自动找 USB 声卡（不需要手动写名字）
TARGET=$(pactl list sinks short | grep -i usb | awk 'NR==1 {print $2}')

# 如果找不到 USB 声卡，等待3秒再试
while [ -z "$TARGET" ]; do
    TARGET=$(pactl list sinks short | grep -i usb | awk 'NR==1 {print $2}')
    sleep 3
done

# 无限循环确保声音输出在 USB 声卡
while true; do
    CURRENT=$(pactl info 2>/dev/null | awk '/Default Sink/ {print $3}')

    if [[ "$CURRENT" != "$TARGET" ]]; then
        pactl set-default-sink "$TARGET" >/dev/null 2>&1

        # 迁移所有播放中的声音
        for s in $(pactl list sink-inputs short | awk '{print $1}'); do
            pactl move-sink-input "$s" "$TARGET" >/dev/null 2>&1
        done
    fi

    sleep 2
done
```



### tmux鼠标支持

```
vim ~/.tmux.conf
set -g mouse on
unbind -T root MouseDrag1Pane
```

#### tmux命令

```
tmux ls #查询
tmux attach -t action_player #进入
tmux kill-session -t action_player' #杀死

#按两下键盘： Ctrl + b   → 松开 → 再按   d  #藏到后台
```



### 开机自启的命令

### cron

 ***\*编辑任务\****

```
crontab -e
```

***\*任务格式\****

```
# 分 时 日 月 周 命令
*  *  *  *  *  command

# 开机执行（特殊时间）
@reboot command
```

```
@reboot /bin/bash -c 'cd /home/nvidia/workplace/simple_playaction && ./start_all.sh'
```

```
#cron需要手动加载ros2环境
source /opt/ros/humble/setup.bash
```

```
# cron需要禁止加载到会话，必须在后台运行，图形化会阻塞运行
#tmux attach-session -t $SESSION_NAME
```

### systemctl

```
# 用户级服务 
systemctl --user daemon-reload 

# 系统级服务（如果放/etc/systemd/system/） 
sudo systemctl daemon-reload
```



```
# 查看服务状态
systemctl --user status multi-services

# 停止服务
systemctl --user stop multi-services

# 重启服务
systemctl --user restart multi-services

# 查看实时日志
journalctl --user -u multi-services -f

# 禁用开机自启
systemctl --user disable multi-services
```



### 设置sudo无需密码


在终端中输入以下命令。`visudo` 命令会检查语法错误，比直接用文本编辑器打开更安全。

```
sudo visudo
```


在文件的**末尾**添加以下行。请将 `your_username` 替换为你自己的用户名。

```
your_username ALL=(ALL) NOPASSWD: ALL
```



### jetson agx 和蓝牙耳机连接一下又立马断开

```
Jetson AGX Xavier 连接蓝牙耳机秒断，90% 源于 Jetson 默认禁用蓝牙音频插件或电源管理 / 干扰，按以下顺序排查，通常能快速解决。

编辑蓝牙服务配置打开终端执行：
sudo vim /lib/systemd/system/bluetooth.service.d/nv-bluetooth-service.conf
找到行：
ExecStart=/usr/lib/bluetooth/bluetoothd -d --noplugin=audio,a2dp,avrcp
删除 --noplugin=audio,a2dp,avrcp，
改为：ExecStart=/usr/lib/bluetooth/bluetoothd -d
按 :wq 保存退出。


# 1. 通知 systemd 重新读取配置文件
sudo systemctl daemon-reload

# 2. 重启蓝牙服务
sudo systemctl restart bluetooth

# 3. 检查状态（确保显示 active (running)）
sudo systemctl status bluetooth


安装音频依赖并重启执行以下命令安装蓝牙音频模块并重启设备：
sudo apt update && sudo apt install -y pulseaudio-module-bluetooth
sudo reboot
```

### 方案二

```
以后遇到蓝牙能连键盘鼠标但连不上耳机/没声音的问题，请按以下最简步骤操作，90% 的情况只需这一步：
✅ 第一步：安装音频模块（最关键）

sudo apt update
sudo apt install --reinstall pulseaudio-module-bluetooth bluez bluez-tools
(加上 --reinstall 确保即使已安装也会重新配置)
✅ 第二步：重启相关服务

# 重启蓝牙服务
sudo systemctl restart bluetooth

# 重启音频服务（让 PulseAudio/PipeWire 重新识别蓝牙模块）
systemctl --user restart pulseaudio
# 或者如果是 PipeWire (新版 Ubuntu 默认):
systemctl --user restart pipewire pipewire-pulse
```

