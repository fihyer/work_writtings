# 后台信息统计台

[TOC]

## 总管理后台

### 快捷入口
- 企业审核
- 新增广告
- 企业快速充值
- 发布文章
- 企业用户消费导出
- 个人用户消费导出
- 服务人员收入导出
- 后台管理员配置
- 企业管理员配置
- 培训管理员配置
- 服务分类管理
- 系统基础设置
- 提现配置

### 统计
- 企业入驻统计: 从系统总后台角度使用这角度看，需要知道当前企业总数，以及各个时期企业申请情况（申请入驻量），同时针对申请的企业，其中审核通过的企业量。
  - 企业总数：直接从企业数据表进行总数统计
  - 新增企业申请量：例如在本周内申请入驻企业有30家。如果获取一定时间段内新增企业申请数量？在`tr_company_apply`表中，通过审核的企业字段`is_check`从0变为1，同时企业信息将被加入`tr_company`数据表。`tr_company_apply`中有申请日期`add_time`, 可以对`tr_company_apply`表中时间段内企业数量进行统计得出某一时间段内新增企业申请数量。
  - 新增（审核通过）企业：参考上一个统计信息（"新增企业申请量"），可通过统计`tr_company`数据表中的某时间段内的企业数量获得。
- 用户统计
  - 新增用户（某时间段内）：根据用户表`tr_user`中用户的`add_time`，统计获得末段时间内新增用户量。
  - 平台总用户：可通过统计用户数据表`tr_user`获得平台总用户。
  - 活跃用户：如何定义活跃用户？1、用户经常登录使用平台；2、在1成立的前提下，用户经常下单（下单量）。那么在`tr_user`数据表中，虽然每个用户拥有`last_time`字段（用户最后登录时间),但无法计算出用户的使用频率（暂且假设用户登录就是使用平台了不存在误操作登录）。是否可以新建一个表(`登录统计表`: `用户ID`,`登录时间`, `登录次数`在上一次基础上依次+1)专门用来统计用户登录情况：用户每次登录将登录时间记录到该表中，同时统计用户登录次数。如果在`登录统计表`中选出末段时间，找出登录数量大于一定量（月登录15次，周登录4次，日登陆2次以上）的用户进行统计，得出某段时间内活跃用户量。再次延伸可加上2 - 用户的下单情况，更加精确得出有效活跃量（目前可暂不考虑这个指标）。
- 验收申诉管理统计
  - 平台申诉总数: 可通过统计申诉表`tr_appeal`获得平台总的申诉数量。
  - 已解决申诉数量: _包含以下两条，待商议_ 
  - 新增申诉数量:
  - 新增申诉已解决:
- 企业充值收入统计
  - 充值企业总数:
  - 充值总金额:
- 用户消费统收入统计
  - 平台下单用户总数
  - 用户消费总金额
- 服务人员财务统计
  - 接单服务人员总数
  - 服务人员总收入
  - 服务人员提现总额
- 平台总任务量统计
  - 平台订单总量
  - 接单量
  - 完成率

## 企业后台

### 快捷入口
- 新增产品
- 发布任务
- 新增验收人员
- 导出充值记录
- 导出消费记录
- 后台规则添加
- 计算系数设置
- 新增管理人员

### 统计
- 企业产品统计
  - 企业产品总量
  - 企业新增产品
  - 企业活跃产品（发布成任务的产品topK）: 活跃产品
  - 企业不会越产品（最不常发布的产品topK）
- 订单统计
  - 企业累计总订单量
  - 企业新增订单（正对莫一时期：本周、本月、本年……）
  - 接单量/接单率
- 验收评价管理
  - 手动验收订单数量
  - 系统自动验收订单数量
- 申诉统计
  - 总申诉量
  - 企业自处理申诉量
  - 移交后台申诉量
  - 新增申诉量
  - 企业自处理新增申诉
  - 移交后台新增申诉
- 财务统计
  - 充值总金额
  - 消费总金额
  - 本月、周、年消费


## 培训后台

### 快捷入口
- 添加产品
- 添加后台规则
- 新增技能分类
- 更新入驻协议
- 新增管理员

### 统计
- 服务人员统计
  - 已上岗服务人员总数
  - 新增服务人员申请（本周、本月、本年……）
  - 待审核服务人员总数
  - 培训中服务人员总数
- 产品统计
  - 服务后台产品总量
  - 新增产品
  - 活跃产品TopK
  - 不活跃产品TopK
- 订单统计
  - 发布任务总量
  - 接单量
  - 完成量
- 技能统计
  - 技能分类数量（大小类）
  - 新增技能分类