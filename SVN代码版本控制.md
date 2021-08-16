# SVN  
## SVN的作用  
1、全称为Subversion，是代码版本管理工具
2、可以记住每次的修改
3、查看所有修改的记录
4、恢复任何历史版本
5、恢复已经删除的文件
## SVN的使用与安装  
### SVN服务端的安装  
1、SVN服务器端的下载地址：https://www.visualsvn.com/server/download/  
2、SVN客户端下载地址：https://tortoisesvn.net/downloads.html  
3、安装客户端需要注意：  
![result](https://static01.imgkr.com/temp/7a5364963ad5454dad8ce2e4497561f2.png)  
4、客户端与服务器端安装成功的标志：  
![result](https://static01.imgkr.com/temp/32c24d76c0974990b83858ded02f3818.png)  
### SVN服务器端(VisualSVN Server)的使用   
 1、新建版本库  
 ![result](https://static01.imgkr.com/temp/9b3fc5b109f5435fa4a97d6f449f0e91.png)  
 ![result](https://static01.imgkr.com/temp/5874dc3b0d6c4cceaa88ad81af4c639f.png)  
 ![result](https://static01.imgkr.com/temp/b34c5878654249c69595232131327c59.png)  
2、向SVN服务器中导入项目  
(1)拷贝服务器的版本库地址  
![result](https://static01.imgkr.com/temp/09de3e7349c747a7acc3a4b70da50446.png)  
![result](https://static01.imgkr.com/temp/d4e9d42d286c4d32b1a91ce5be44ffcd.png)  
(2)在本地选择要上传的项目文件  
![result](https://static01.imgkr.com/temp/2f71d0d9d7cb43f2803614fe88a0de59.png)  
(3)将第一步复制的地址黏贴到对应位置   
![result](https://static01.imgkr.com/temp/d09b53478aa2463daa282c4104ae2b2d.png)  
3、本地同步SVN服务器的指定项目  
(1)在服务器中复制要同步的项目的地址  
![result](https://static01.imgkr.com/temp/4e7a4fa8296e4dadab71f96b7687a919.png)  
(2)在磁盘中创建一个目录用来存放从服务器同步的项目，在此目录空白处右键  
![result](https://static01.imgkr.com/temp/748abd121c344710abc36a8e3bf5c29c.png)  
(3)输入远程地址，并设置项目的输出目录  
![result](https://static01.imgkr.com/temp/c4576c0a08514412a6ab34e2ab6e5b11.png)  
4、本地修改内容提交到服务器(commit)  
 先add增加到提交列表，在SVN Commit提交到远程SVN服务器  
 ![result](https://static01.imgkr.com/temp/c4576c0a08514412a6ab34e2ab6e5b11.png)    
 5、更新服务器内容到本地  
 ![result](https://static01.imgkr.com/temp/3b82037f55db45209b9c58810f33f513.png)  
### SVN版本控制的冲突与解决  
1、冲突产生：假设A、B两个用户都在版本号为100的时候，更新了hello.txt这个文件，A用户在修改完成之 后提交hello.txt到服务器，提交成功，这个时候hello.txt文件的版本号已经变成101 了；同时B 用户在版本号为100的hello.txt文件上作修改，修改完成之后提交到服务器时，由于不是在当 前最新的101版本上作的修改，所以导致提交失败；此时用户B去更新文件，如果B用户和A 用户修改了文件的同一行代码，就会出现冲突。  
2、冲突文件存在形式：冲突文件hello.txt会衍生出如下四个文件：
(1) 上次的版本 hello.txt.r100
(2) 别人已提交更新后的版本 hello.txt.r101
(3) 自己更新后的版本 hello.txt.mine
(4) 冲突文件自身 hello.txt
解决冲突后，只会剩下hello.txt解决冲突后的文件。  
3、冲突的解决：  
(1)、B用户在磁盘中的项目目录下执行update命令  
![result](https://static01.imgkr.com/temp/e0a8a1ebb79447e083ff52ecac190108.png)  
(2)、B用户的项目目录下出现如下文件   
![result](https://static01.imgkr.com/temp/6d94de653501466f80f7a9e30f75d5d6.png)   
(3)在冲突文件右键,弹出编辑冲突的窗口
![result](https://static01.imgkr.com/temp/ec8515ee86f044d192ce434da30efab7.png)  
![result](https://static01.imgkr.com/temp/60962bfa924442238a70836d3b1aea98.png)  
(4)如果要使用服务器版本，在Theirs窗口右键选择Use this text block；如果要使用本地版本，在 Mine窗口右键选择Use this text block；最终选择的都会保存在合并窗口
![result](https://static01.imgkr.com/temp/325a1a0407df4fceb30f4eddfa6de8e1.png)  
(5)修改完成后，进行如下图所示的操作解决冲突,冲突解决后，只剩下解决冲突后的文件，衍生出的文件自动消失,B用户再次执行commit操作，可提交至服务器。  
![result](https://static01.imgkr.com/temp/73a66cb525ac47609c6d70b3ce354238.png)  


 


  