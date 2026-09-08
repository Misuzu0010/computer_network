可以认为 每一个器官都代表着一个能力 能力由角色身上的GameAbilitySystem管理 每装备一个器官就挂载这个能力 再同意通过物理系统叠加

分板块：
- InputService	读取移动、跳跃、鼠标瞄准、槽位按键、交互、重置、暂停

- PlayerMotor2D	地面移动、短跳、空中控制、速度限制、落地检测

- PlayerGroundSensor	判断玩家是否稳定站地面，供换装和补气使用

- PlayerLoadout	管理两个装备槽位，不负责具体能力

- PlayerInventory	保存已经收集的器官 ID

- OrganAbilityController	根据当前装备创建和运行能力实例

- TongueAbility	射线命中、舌头连接、牵引、拉环操作

- LungAbility	气流范围、墙体遮挡、物体推动、玩家反冲、气量

- WingAbility	滑翔状态、重力降低、水平控制

- PhysicsEffectAccumulator	汇总牵引、反冲、滑翔等效果，
统一在物理帧中应用

- InteractionTarget	统一交互提示和目标高亮

- LightMovable	标记哪些物体可以被肺和舌头推动

- ResettableObject	为木箱、电池、拉环等对象提供区域重置能力

- ZoneController	管理 L01—L04 的进入、完成和检查点

- RunState	管理本轮流程中的全局状态

- DialogueDirector	对白队列、优先级、跳过、一次性播放

- ObjectiveController	管理当前目标，目标变化由关卡事件触发

- CompletionController	处理回收台提交和结尾流程

- HUD	槽位、气量、目标、提示、字幕、功能卡


OrganAbilityController 负责管理史莱姆身上的能力

可供拥有的能力
TongueAbility
LungAbility
WingAbility

再通过 PhysicsEffectAccumulator 统一处理叠加/不叠加之后的物理效果

LightMovable 用于管理本物体是否可以被移动 即虚接口继承 然后标记可拖拽

ResettableObject 继承本接口的能力 内置一个刷新函数 刷新所有字段为默认值

ZoneController 挂载切换条件判断脚本

RunState 相当于是一个Tick中的快照（？

DialogueDirector 负责管理对话进程 我可以简单类比成为UE中的DataAsset数据层逻辑





