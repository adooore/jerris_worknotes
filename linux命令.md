# linux命令

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
