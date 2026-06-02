---
date: 2026-05-24
project: RE_Demo
tags:
  - UnrealEngine5
  - Debug技巧
  - 时间膨胀
---
## 控制台中直接输入
```
slomo 0.5 # 设置为 50% 速度（慢动作） 
slomo 1.0 # 恢复正常速度 
slomo 2.0 # 双倍加速
```


### 代码脚本控制

1. 打开LevelBlueprint
2. 调用SetGlobalTimeDilation

![[UnrealEngine_TimeDilation.png|458]]
