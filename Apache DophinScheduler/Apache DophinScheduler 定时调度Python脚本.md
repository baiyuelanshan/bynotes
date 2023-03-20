
1.创建租户
---
安全中心 -> 租户管理 -> 创建租户
这一步是将操作系统的账户与队列关联。

![[Pasted image 20230317175245.png]]


![[Pasted image 20230320104255.png]]
`这里的ubuntu是操作系统的一个账户`


2.指定用户的租户
---
安全中心 -> 用户管理  -> 编辑， 指定admin用户的租户

![[Pasted image 20230320104338.png]]



3.创建Python环境
---
安全中心 -> 环境管理 -> 创建环境

![[Pasted image 20230320100741.png]]

![[Pasted image 20230320161759.png]]


4.创建项目
--
项目管理 -> 创建项目
![[Pasted image 20230320101317.png]]

![[Pasted image 20230320101358.png]]

5.创建工作流
--

![[Pasted image 20230320101544.png]]

点击刚创建的项目 `test_project`


工作流定义 -> 创建工作流

![[Pasted image 20230320101753.png]]


![[Pasted image 20230320102213.png]]

拖拽python组件到画板上，指定环境名称，编辑Python脚本的内容

![[Pasted image 20230320161606.png]]

![[Pasted image 20230320102430.png]]
点击画板右上角的保存按钮


![[Pasted image 20230320102523.png]]



6.上线项目
--
![[Pasted image 20230320103117.png]]



7.设置调度时间
---

![[Pasted image 20230320162407.png]]

![[Pasted image 20230320110541.png]]


8.上线定时管理
--

![[Pasted image 20230320103700.png]]

![[Pasted image 20230320110633.png]]


9.查看日志
--
可以在任务实例查看运行日志


![[Pasted image 20230320105849.png]]

