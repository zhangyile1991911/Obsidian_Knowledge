---
date: 2026-05-24
course: RE_Demo
tags:
  - Locomotion
  - AnimBluePrint
  - UnrealEngine5
---

## 安装插件
- Animation Locomotion Library
- Animation Warping

## 收集角色数据
- 使用BlueprintThreadSafeUpdateAnimation线程安全方法收集角色数据
- 使用PropertyAccess访问角色数据


## 结构图

```plantuml
interface ALI_AnimLayerInterface
{
    FullBody_Idle();
    FullBody_Start();
    FullBody_Cycle();
    FullBody_Stop();
}
class ABP_CharacterBase
{
    
}

class ABP_ItemBaseLayer
{
    
}

ABP_ItemBaseLayer ..|> ALI_AnimLayerInterface
```


