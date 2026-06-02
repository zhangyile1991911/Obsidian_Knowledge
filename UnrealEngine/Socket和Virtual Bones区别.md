---
date: 2026-05-24
project: RE_Demo
tags:
  - UnrealEngine5
  - 骨骼
---
# Socket和VirtualBone区别
## Socket
- 可以在一个骨骼上添加多个socket
- 在runntime时无法修改相对位置/旋转，但可以在运行时候修改添加插槽物体

## Virtual Bones
- 只能在一个骨骼上添加一个虚拟骨骼
- 在Animation Blueprints中作为目标，比如IK目标或者注视

# 创建Socket流程
1. 打开SKM_Manny
2. 在SkeletonTree中所搜hand_r
3. 右击hand_r 添加Add Socket
4. 重命名插槽weapon_attach_r

# 新建Actor蓝图
1. 新建一个继承Actor蓝图
2. 添加Skeletal Mesh组件


# 挂在Actor蓝图到角色
1. 打开角色蓝图
2. 将Actor蓝图拖拽进角色蓝图
3. 将Actor蓝图放到角色Mesh下
4. 修改Actor的ParentSocket为weapon_attach_r

# 添加预览武器
1. 打开SKM_Manny
2. 找到weapon_attach_r槽位
3. 右击添加Add Preview Asset

# 添加预览姿势
1. PreviewSceneSetting中修改PreviewController选项为Use Specific Animation
2. Animation选择需要预览的姿势
3. 调整socket的位置和旋转
