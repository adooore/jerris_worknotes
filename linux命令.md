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