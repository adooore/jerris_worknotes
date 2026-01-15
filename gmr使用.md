GVHMR从视频生成人体跟随数据

```
(gvhmr) ti5robot@chenjunrui:/mnt/d/123_github/gitlab/170d/Ti5Humanoid/code/T170D/action/GVHMR-main/GVHMR-main$ python tools/demo/demo.py --video=docs/example_video/tennis.mp4 -s
```



软连接两个文件夹

```
(gmr) ti5robot@chenjunrui:/mnt/d/123_github/gitlab/170d/Ti5Humanoid/code/T170D/action/GMR-master/GMR-master$ ln -s /mnt/d/123_github/gitlab/170d/Ti5Humanoid/code/T170D/action/GVHMR-main/GVHMR-main/inputs/checkpoints/body_models /mnt/d/123_github/gitlab/170d/Ti5Humanoid/code/T170D/action/GMR-master/GMR-master/assets/body_models
```



GMR生成pkl文件

```
(gmr) ti5robot@chenjunrui:/mnt/d/123_github/gitlab/170d/Ti5Humanoid/code/T170D/action/GMR-master/GMR-master$ python scripts/gvhmr_to_robot.py --gvhmr_pred_file /mnt/d/123_github/gitlab/170d/Ti5Humanoid/code/T170D/action/GVHMR-main/GVHMR-main/outputs/demo/tennis/hmr4d_results.pt --robot unitree_g1 --save_path outputs/tennis_g1.pkl
```

播放pkl文件

```
(gmr) ti5robot@chenjunrui:/mnt/d/123_github/gitlab/170d/Ti5Humanoid/code/T170D/action/GMR-master/GMR-master$python scripts/vis_robot_motion.py --robot unitree_g1 --robot_motion_path outputs/testbb.pkl --record_video --video_path outputs/testbb.mp4
```



动作不对极可能是模型限位问题
