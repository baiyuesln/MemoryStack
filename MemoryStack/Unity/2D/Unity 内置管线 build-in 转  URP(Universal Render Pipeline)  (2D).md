
#URP

1 , 在packge manager ->untiy registry 中 下载 Universal RP
registry

![[Pasted image 20241204163550.png]]

2 在Asset中,右键create, rendering 创建 URP Asset(with 2D) 和URP 2d Renderer

![[Pasted image 20241204163745.png]]
3,添加Renderer到Asset 的Renderer List中
![[Pasted image 20241204163951.png]]
4,在Edit -> **Project Settings**中选择**Graphics**，指定默认渲染管线为刚才新建的URP Assets。注意是拖拽URP Assets
	![[Pasted image 20241204164202.png]]
5,Windows -> Rendering -> Render Pipeline Converter
![[Pasted image 20241204164329.png]]

6,然后勾选所有材质,直接Converter 成功.