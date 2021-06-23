# 电子邮件功能  
## 应用场景  
1、注册验证  
2、营销推送  
3、监控报警  

## SpringBoot实现发送电子邮件  
以网易邮箱为发送方为例
1、在网易邮箱的设置中开启SMTP协议服务,记住授权码。  
![result](https://static01.imgkr.com/temp/c12bcc0f610e44c39d480ae6e53293ca.png)  

2、注册发件邮箱并设置客户端授权码，在application.properties配置参数  
```properties  
# 邮箱配置
spring.mail.host=smtp.163.com
# 你的163邮箱
spring.mail.username=ispringboot@163.com
# 注意这里不是邮箱密码，而是SMTP授权密码
spring.mail.password=isb001
spring.mail.port=25  //端口
spring.mail.protocol=smtp  //协议
spring.mail.default-encoding=UTF-8  //编码格式
```   
3、pom.xml依赖spring-boot-starter-mail模块  
```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-mail</artifactId>
</dependency>

<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
   <groupId>org.projectlombok</groupId>
   <artifactId>lombok</artifactId>
</dependency> 
```  
4、实现服务端代码  
```java
package org.ijiangtao.tech.spring.boot.mail.imail.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.io.File;

@Service
public class MailService {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Value("${spring.mail.username}")
    private String from;

    @Autowired
    private JavaMailSender mailSender;

    /**
     * 简单文本邮件
     * @param to 接收者邮件
     * @param subject 邮件主题
     * @param contnet 邮件内容
     */
    public void sendSimpleMail(String to, String subject, String contnet){

        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(contnet);
        message.setFrom(from);

        mailSender.send(message);
    }

    /**
     * HTML 文本邮件
     * @param to 接收者邮件
     * @param subject 邮件主题
     * @param contnet HTML内容
     * @throws MessagingException
     */
    public void sendHtmlMail(String to, String subject, String contnet) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();

        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(contnet, true);
        helper.setFrom(from);

        mailSender.send(message);
    }

    /**
     * 附件邮件
     * @param to 接收者邮件
     * @param subject 邮件主题
     * @param contnet HTML内容
     * @param filePath 附件路径
     * @throws MessagingException
     */
    public void sendAttachmentsMail(String to, String subject, String contnet,
                                    String filePath) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();

        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(contnet, true);
        helper.setFrom(from);

        FileSystemResource file = new FileSystemResource(new File(filePath));
        String fileName = file.getFilename();
        helper.addAttachment(fileName, file);

        mailSender.send(message);
    }

    /**
     * 图片邮件
     * @param to 接收者邮件
     * @param subject 邮件主题
     * @param contnet HTML内容
     * @param rscPath 图片路径
     * @param rscId 图片ID
     * @throws MessagingException
     */
    public void sendInlinkResourceMail(String to, String subject, String contnet,
                                       String rscPath, String rscId) {
        logger.info("发送静态邮件开始: {},{},{},{},{}", to, subject, contnet, rscPath, rscId);

        MimeMessage message = mailSender.createMimeMessage();
        MimeMessageHelper helper = null;

        try {

            helper = new MimeMessageHelper(message, true);
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(contnet, true);
            helper.setFrom(from);

            FileSystemResource res = new FileSystemResource(new File(rscPath));
            helper.addInline(rscId, res);
            mailSender.send(message);
            logger.info("发送静态邮件成功!");

        } catch (MessagingException e) {
            logger.info("发送静态邮件失败: ", e);
        }

    }

}
```  
5、测试段代码  
```java
package org.ijiangtao.tech.spring.boot.mail.imail.service;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

import javax.annotation.Resource;
import javax.mail.MessagingException;

@RunWith(SpringRunner.class)
@SpringBootTest
public class MailServiceTest {

    @Autowired
    private MailService mailService;

    @Resource
    private TemplateEngine templateEngine;

    @Test
    //简单文本邮件测试
    public void sendSimpleMail() {
        mailService.sendSimpleMail("接收方的邮箱","测试spring boot imail-主题","测试spring boot imail - 内容");
    }

    @Test
    //发送html文本邮件测试
    public void sendHtmlMail() throws MessagingException {

        String content = "<html>\n" +
                "<body>\n" +
                "<h3>hello world</h3>\n" +
                "<h1>html</h1>\n" +
                "<body>\n" +
                "</html>\n";

        mailService.sendHtmlMail("接收方的邮箱号","这是一封HTML邮件",content);
    }

    @Test
    //发送可带附件的文本邮件
    public void sendAttachmentsMail() throws MessagingException {
        String filePath = "/ijiangtao/软件开发前景.docx";
        String content = "<html>\n" +
                "<body>\n" +
                "<h3>hello world</h3>\n" +
                "<h1>html</h1>\n" +
                "<h1>附件传输</h1>\n" +
                "<body>\n" +
                "</html>\n";
        mailService.sendAttachmentsMail("接收方的邮箱号","这是一封HTML邮件",content, filePath);
    }

    @Test
    //图片邮件测试
    public void sendInlinkResourceMail() throws MessagingException {
        //TODO 改为本地图片目录
        String imgPath = "/ijiangtao/img/blob/dd9899b4cf95cbf074ddc4607007046c022564cb/blog/animal/dog/dog-at-work-with-computer-2.jpg?raw=true";
        String rscId = "admxj001";
        String content = "<html>" +
                "<body>" +
                "<h3>hello world</h3>" +
                "<h1>html</h1>" +
                "<h1>图片邮件</h1>" +
                "<img src='cid:"+rscId+"'></img>" +
                "<body>" +
                "</html>";

        mailService.sendInlinkResourceMail("接收方的邮箱号","这是一封图片邮件",content, imgPath, rscId);
    }

    @Test
    //自定义HTML文本邮件测试
    public void testTemplateMailTest() throws MessagingException {
        Context context = new Context();
        context.setVariable("id","ispringboot"); //模板html中a标签的herf路径中的id变量填充

        String emailContent = templateEngine.process("emailTeplate", context); //使用模板引擎类对象引入html模板
        mailService.sendHtmlMail("接收方的邮箱号","这是一封HTML模板邮件",emailContent);

    }
}
```    
## 创建自定义的邮件内容html模板  
我们使用thymeleaf作为模板引擎。  
自定义的emailTeplate.html模板如下：
```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0"/>
    <meta http-equiv="X-UA-Compatible" content="ie=edge"/>
    <title>注册-测试邮件模板</title>
</head>
<body>
    你好，感谢你的注册，这是一封验证邮件，请点击下面的连接完成注册，感谢您的支持。
    <a href="#" th:href="@{https://github.com/{id}(id=${id})}">激活账户</a>
</body>
</html>
```
