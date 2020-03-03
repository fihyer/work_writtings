## PC总管理后台

### 用户管理（个人用户管理）

> 总后台用来管理系统平台微信小程序端的个人（业主）用户信息。需微信小程序端配合交互。


个人用户信息包括：
- 用户姓名: 用户姓名可以通过微信小程序端“个人中心”-“修改个人信息”中授权信息获得。
  - [x] 打开微信小程序，点击“我的”，第一次使用的用户默认无头像、用户名为“鸿巢用户”。
  - [ ] 点击“修改个人信息”，点击“授权信息”，自动获取微信昵称和用户头像。点击“提交”后，微信端提示修改“修改信息成功“，同时总后台个人用户管理列表增加新的记录，姓名为用户微信昵称，注册时间为操作“提交”时间。
  - [ ] 在微信小程序“修改个人信息”中以相同的信息提交则提示“修改失败”；修改“昵称”或“头像”中的任何信息（需与原来信息不一样）再次提交则提示“修改信息成功”
  - [ ] 总后台“个人用户管理”列表中点击查看注册用户详情，其中“姓名”，“手机号码”， “地址”， “性别”均为空。
      - [ ] 对用户信息进行修改：任意输入姓名，和电话号码（必须有，但可以是任意不正确的电话号码）点击“保存”后，页面提示“保存成功，“个人用户管理”列表信息更新。
      - [ ] 打开微信小程序端，个人信息无改变，”我的“页面右上方的“绑定手机号码”按钮消失。
- 手机号码:
  - [ ] 在微信小程序“我的”页面，点击右上方“绑定手机号码”，弹出获取手机号码信息确认页面
      - [ ] 点击拒绝显示“绑定手机号成功”
      - [x] 点击允许显示“绑定手机号成功”，小程序“我的”页面右上方“绑定手机号码”按钮消失；同时总后台“个人用户管理”列表中对应用户手机号码信息更新成功。
  - [ ] 总后台“个人用户管理”列表中再次点击查看注册用户详情，其中“姓名”， “地址”， “性别”为空。
      - [ ] 对用户信息进行修改：填入姓名、地址，选择性别后保存，页面提示保存成功。“个人用户管理”列表中用户信息同步更新。
      - [ ] 再次打开小程序端查看个人信息：用户名依旧为原来的信息，但订单中发单人信息会及时更新。点击“修改个人信息”需要再次输入不同的“昵称”进行提交，提交成功，但总管理后台信息不发生改变！**【只要在后台用户详情中进行过修改，则微信小程序修改个人信息，只在小程序端显示有效，后台无信息变动】**
- 住址: 无获取方式，智能通过后台进行手动输入。个人用户地址信息与小程序上选择商品发布任务时地址无任何关联。
- 性别: 只能通过后台进行手动选择。
- 注册时间: 自动获取，用户**第一次修改个人信息时提交的时间**以及**第一次绑定手机号码的时间**



###

## 培训管理后台

### 服务人员管理

> 服务人员管理分为三个阶段：查看前端（微信小程序）申请输入服务人员信息；查看在培训中服务人员信息（通过初步审核的服务人员）；查看已上岗服务人员信息（培训完毕后，签署电子协议，并通过后台管理人员审核批准上岗）。

- 待培训服务人员
  - [ ] 在微信小程序端（**首页** 和 **我的**）中“平台服务人员招募”
  - [ ] 阅读入驻协议，点击同意入驻协议后，填写基本信息，其中包括（全部为必填内容）:
      - 姓名
      - 电话
      - 联系方式
- 培训中服务人员
  - [ ] 
- 已上岗服务人员