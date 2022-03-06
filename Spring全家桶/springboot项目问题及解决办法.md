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
## SpringBoot的打包格式配置  
jar包：将项目看成一整个拼图，引入的 jar包 就是一个拼块，在项目中引入的依赖就是jar包，里面是编译后的class文件。    
war包：在 javaweb中通常都是将项目打包成war包再进行部署，里面包括写的代码编译成的class文件，依赖的包，配置文件，所有的页面文件。一个war包可以理解为一个web项目。  
设置Springboot项目的打包格式可以在pom.xml中设置  
```xml
    <groupId>com.xian</groupId>
    <artifactId>project</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>  //打包的格式在这里设置
    <name>project</name>
    <description>Demo project for Spring Boot</description>
```  
## 打包war包遇到报错
```  
通过Maven导出war包时报错：Failed to execute goal org.apache.maven.plugins:maven-war-plugin:2.2:war (default-war) on project Ocr: Error assembling WAR: webxml attribute is required (or pre-existing WEB-INF/web.xml if executing in update mode) -> [Help 1]  
```
<font color=red>解答：</font>这是系统没有找到 WEB-INF/web.xml文件，所以你需要在项目的main文件夹下，也就是和java的统计目录新建webapp文件夹，再在里面新建WEB-INF文件夹，最后在WEB-INF文件夹下新建web.xml文件，xml文件中不需要配置内容，默认内容就好了。之后执行maven的clean命令，清除之前打包的内容，然后执行package命令，重新打包就可以看到BUILD SUCCESS！