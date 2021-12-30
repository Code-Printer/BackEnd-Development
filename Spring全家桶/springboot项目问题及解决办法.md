# 构建SpringBoot框架的web服务时遇到的问题  
## 创建SpringBoot项目时遇到Spring Initializr报错Error:connect timed out  
解决方法：在新建项目时，spring Initializr下选择阿里云的源，不选择默认，选择在Custom下输入https://start.aliyun.com     
## windows端启动项目时遇到的端口占用问题  
1.打开命令窗口,执行命令：netstat -ano。查看对应端口的PID   
2、命令窗口执行命令：tasklist，查看对应PID的应用名  
3、打开任务管理器，终止该进程  

## SpringBoot框架的web开发配置  
### 扫描路径配置  
控制类与启动类不在一个包，则需要手动指定扫描该包，一定要注意使用了手动指定后，就不存在默认扫描的包了，需要逐一手工指定。  
```java
@SpringBootApplication
//@ComponentScan(basePackages = { "com.example.springbootest", "Controlller" }) // 手动指定扫描的包
public class SpringbootestApplication {

    public static void main(String[] args) {

        SpringApplication.run(SpringbootestApplication.class, args);
    }

}
```
### Servlet相关配置  
修改对应的服务监听端口和项目域启动路径：在resource/application.properties文件中添加以下配置：  
```properties
#端口号
server.port=8081
#服务环境路径
server.servlet.context-path=/demo
#此时重启服务，在浏览器中输入http://localhost:8081/demo/user
```   
### 修改banner的内容  
1、通过配置修改banner的内容。首先访问一下网站,输入文本，生成相应的banner。    
```properties  
http://patorjk.com/software/taag/#p=display&f=Graffiti&t=demo
```  
2、在resource目录下新建banner.txt，并将上面内容复制到文件中。最后在resource/application.properties文件中添加以下配置内容：
```properties  
spring.banner.location=banner.txt
```
3、重新启动服务，即可看到自定义的banner。  
### 关闭banner显示功能  
关闭banner功能，只需要在在启动类的main方法中加入以下代码，重启服务即可：  
```java
    public static void main(String[] args) {
		SpringApplication application = new SpringApplication(DemoApplication.class);
		application.setBannerMode(Banner.Mode.OFF);
        application.run(args);
	}
```