---
layout:     post
title:      JavaMail使用
subtitle:   JavaMail中各配置参数收集整理
date:       2018-10-14
author:     KaithyXu
header-img: img/JavaMail.jpg
catalog: true
tags:
    - JavaMail
---
## JavaMail使用


### 介绍

SpringBoot 提供了一个封装好的工具类JavaMailSender来实现邮件的发送，但是在application.properties中如何配置邮件服务的参数是个比较大的问题，所以自己打算收集这些参数，做个整理，以便后续再用需要使用Java Mail时不会一头雾水。


### 解题思路

 利用快排算法先对数组进行排序，然后再根据数组下标取得第K大的元素，其中，在快排递归的时候，先判断数组中间位置与k的大小，根据比较结果选择递归的方向以减少不必要的递归消耗。
 application.properties中比较常用的邮件服务设置：
```
spring.mail.default-encoding=UTF-8
spring.mail.host=smtp.qq.com
spring.mail.username=username
spring.mail.password=password
spring.mail.protocol=smtp

#登录服务器是否需要认证
spring.mail.properties.mail.smtp.auth=true

#SSL证书Socket工厂
spring.mail.properties.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory

#使用qq邮箱smtp服务器提供的465端口
spring.mail.properties.mail.smtp.socketFactory.port=465

```
SpringBoot会自动加载配置文件中的配置信息，具体的加载动作在包
```
org.springframework.boot.autoconfigure.mail
```
中的MailSenderPropertiesConfiguration类中实现。
在SpringBoot中具体使用参考如下代码：
````
package cn.com.kaithy.xu.SkillCompetitionDemo.service;

import cn.com.kaithy.xu.SkillCompetitionDemo.entity.MailServer;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.MailException;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.mail.javamail.MimeMessagePreparator;
import org.springframework.stereotype.Service;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

import java.io.File;

@Slf4j
@Service
public class MailSendService {

    /**
     * 自动注入已经含有配置文件中指定的属性的JavaMailSender
     */
    @Autowired
    private JavaMailSender mailSender;

    /**
     * Thymeleaf模版引擎
     */
    @Autowired
    private TemplateEngine templateEngine;

    /**
     * 邮件的发送方，可以是邮件的username，也可以自定义
     */
    @Value("${spring.mail.username}")
    private String username;

    /**
     * 发送简单的文本信息
     * @param message
     * @param recipients
     */
    public void sendsimpleMessage(String message, String... recipients) {

        String[] destination = recipients;

        /**
         * 使用MimeMessageHelper来协助填充发送方以及邮件内容以及邮件主题等
         */
        MimeMessagePreparator messagePreparator = mimeMessage -> {
            MimeMessageHelper messageHelper = new MimeMessageHelper(mimeMessage);
            messageHelper.setFrom(username);
            messageHelper.setText(message);
            messageHelper.setSubject("Simple Message Demo");
            messageHelper.setTo(destination);
        };

        try {
            mailSender.send(messagePreparator);
        }catch (MailException e) {
            log.error(e.getLocalizedMessage(),e);
            throw new RuntimeException("发送邮件失败");
        }
        System.out.println("邮件发送成功!");
    }

    /**
     * 组装模版内容，并且通过模版引擎生成邮件正文
     * @param title
     * @param message
     * @param appendFile 邮件附件
     * @param recipients
     */
    public void sendTemplateMessage(String title, String message, File appendFile, String... recipients) {
        String[] destination = recipients;

        MimeMessagePreparator messagePreparator = mimeMessage -> {
            mimeMessage.addHeader("X-Mailer","Microsoft Outlook Express 6.00.2900.2869");
            MimeMessageHelper messageHelper = new MimeMessageHelper(mimeMessage);
            messageHelper.setFrom(username);
            messageHelper.setText(build(title,message),true);
            messageHelper.setSubject("Template Message Demo");
            messageHelper.setTo(destination);
            FileSystemResource resource = new FileSystemResource(appendFile);
            messageHelper.addAttachment(appendFile.getName(),resource);
        };

        try {
            //发送邮件
            mailSender.send(messagePreparator);
        }catch (MailException e) {
            log.error(e.getLocalizedMessage(),e);
            throw new RuntimeException("发送邮件失败");
        }
        System.out.println("邮件发送成功!");
    }

    /**
     * 模版引擎生成邮件内容
     * @param title
     * @param message
     * @return
     */
    public String build(String title, String message) {
        Context context = new Context();
        context.setVariable("title",title);
        context.setVariable("message",message);
        return templateEngine.process("mailTemplate",context);
    }

}

````

### mail.properties中各个参数整理
一下主要是整理mail.properties中的各项属性的用途，前缀由于统一都是spring.mail.properties,所以就忽略不写


| 名称 | 类型 | 功能 |
| --- | --- | --- |
| mail.smtp.timeout | int | Socket数据读取超时时间 |
| mail.smtp.writetimeout | int | Socket数据写入超时时间 |
| mail.smtp.connectiontimeout | int | Socket 连接超时时间 |
|mail.smtp.auth | Boolean | 使用提供的用户名与密码去做验证，默认是为false |
| mail.smtp.starttls.enable | Boolean | 使用 TLS协议 连接 true表示运行TLS连接 |
| mail.smtp.socketFactory | SocketFactory | 指定用于创建SMTP Socket的类的实例对象，这个类一定是实现了javax.net.Socket接口 |
| mail.smtp.socketFactory.class | String | 指定用于创建SMTP SOCKET 的类的名称，这个类必须是实现了javax.net.SocketFactory接口 |
| mail.smtp.socketFactory.port | Int | 指定SOCKET 连接的端口号，默认为25 |
| mail.smtp.socketFactory.fallback | Boolean | 当设置为true时，若无法使用指定的套接字工厂类创建Socket的情况下，会使用java.net.Socket类来创建Socket |
| mail.smtp.ssl.trust | String | 当这个参数设置后，但是又没有指明具体的SocketFactory，就会启用MailSSlSocketFactory，这个参数用于设置信任名单，受信任的host将绕过SSL，通配符* 表示信任所有的host，如果要具体指定信任的host，用空格作为多个host的分隔符，如果不设置host，那么信任的host都取决于服务器提供的证书 |
| mail.smtp.ssl.protocols | String | 设置使用SSL连接后的协议名称，多个用空格分隔 |
| mail.smtp.ssl.ciphersuites | String | 指定用SSL连接时的SSL密码套件 |
| mail.smtp.proxy.host | String | 指明用于连接邮件服务器的HTTP代理的host |
| mail.smtp.proxy.port | String | 指定HTTP代理的端口，默认是80 |
| mail.smtp.proxy.user | String | HTTP代理服务器连接的用户名 |
| mail.smtp.proxy.password | String | HTTP代理服务器连接的密码 |
| mail.smtp.socks.host | String | 指定将用于连接到邮件服务器的SOCKS5代理服务器的主机名 |
| mail.smtp.socks.port | String | 指定将用于连接到邮件服务器的SOCKS5代理服务器的端口 |
| mail.smtp.mailextension | String | 扩展字符串追加到MAIL命令。 扩展字符串可用于指定标准SMTP服务扩展以及特定于供应商的扩展。通常，应用程序应使用SMTPTransport方法supportsExtension来验证服务器是否支持所需的服务扩展。 请参阅RFC 1869和其他定义特定扩展名的RFC。 |
| mail.smtp.userset | Boolean | 如果设置为true，则在isConnected方法中使用RSET命令而不是NOOP命令。 在某些情况下，在执行许多NOOP命令后，sendmail的响应速度会很慢。 使用RSET可以避免此sendmail问题。 默认为false。 |
| mail.smtp.noop.strict | Boolean | 如果设置为true（默认值），则坚持使用NOOP命令的250响应代码来指示成功。 isConnected方法使用NOOP命令来确定连接是否仍然有效。 一些较旧的服务器在成功时返回错误的响应代码，一些服务器根本不执行NOOP命令，因此总是返回失败代码。 将此属性设置为false可处理以这种方式损坏的服务器。 通常，当服务器超时连接时，它将发送421响应代码，客户端将其视为对它发出的下一个命令的响应。 超时连接时，某些服务器会发送错误的故障响应代码。 处理以这种方式损坏的服务器时，请勿将此属性设置为false。 |
| mail.smtp.auth.mechanisms |  String | 使用参数中提供的认证机制，默认的认证机制包括 “LOGIN PLAIN DIGEST-MD5 NTLM”等机制除了XOAUTH2认证机制 |
| mail.smtp.auth.login.disable | boolean | 若该参数设置为true，则会阻止使用AUTH LOGIN认证机制进行用户认证，默认是false |
| mail.smtp.auth.plain.disable | boolean | 若设置为true，则阻止使用AUTH PLAIN认证机制进行用户认证，默认是false |
| mail.smtp.auth.digest-md5.disable | boolean | 若设置为true，这禁止使用AUTH DIGEST-MD5认证机制，默认为false |
| mail.smtp.auth.ntlm.disable | boolean | 若设置为true则禁止使用AUTH NTLM机制，默认false | 
| mail.smtp.auth.ntlm.domain |  String| NTLM 认证域 |
| mail.smtp.auth.ntlm.flags | int | NTLM认证协议的具体标志 |
| mail.smtp.auth.xoauth2.disable | boolean | 设置为true则禁止使用AUTHENTICATE XOAUTH2 认证机制，由于OAUTH2 协议需要特殊的访问授权码以代替密码，这项机制默认是false，禁止的，若要使用该机制，此参数设置为false，同时在mail.smtp.auth.mechanisms参数中添加XOAUTH2 |
| mail.smtp.submitter | String | 在MAIL FROM命令的AUTH标记中使用的提交者。 通常由邮件中继用于传递有关邮件原始提交者的信息。 另请参见SMTPMessage的setSubmitter方法。 邮件客户端通常不使用此功能。 |
| mail.smtp.dsn.notify | String | 设置RCPT命令的NOTIFY选项，其他的还有 NEVER，SUCCESS，FAILUERE，DELAY等选项以及相关组合 |
| mail.smtp.dsn.ret | String | MAIL 命令中RET选项，与之相关的还有FULL和HDRS选项 |
| mail.smtp.allow8bitmime | boolean | 如果设置为true，并且服务器支持8BITMIME扩展，则使用“ quoted-printable”或“ base64”编码的邮件文本部分如果遵循RFC2045规则的8bit文本，则转换为使用“ 8bit”编码。 |
| mail.smtp.sendpartial | boolean | 若设置为true，无论目标地址是否是有效的都会发送邮件，并将部分异常封装进SendFailedException中，若设置为false（默认也是为false），无效的地址是不会发送邮件的 |
| mail.smtp.sasl.enable | boolean | 若设置为true，尝试使用javax.security.sasl包中的认证机制进行登录认证，默认为false |
| mail.smtp.sasl.mechanisms |  String| 参数值为SASL机制名称，用空格或逗号分隔，指明使用的SASL认证机制 |
|mail.smtp.sasl.authorizationid | String | 使用提供的认证ID用于SASL认证，若没有设置则使用用户名来进行认证 |
| mail.smtp.sasl.realm | String | 设置与DIGEST-MD5一期使用的认证域 |
| mail.smtp.sasl.usecanonicalhostname | boolean | 若设置为true，这用InetAddress.getCanonicalHostName返回的规范的host去传递给SASL机制，用于代替普通的host进行连接，默认为false |
| mail.smtp.quitwait | boolean | 若设置为false，QUIT命令会被发送并且立刻关闭连接，若设置为true（默认）传输通道会等待QUIT的命令响应 |
| mail.smtp.reportsuccess | boolean | 如果设置为true，则使传输为每个成功的地址包含一个SMTPAddressSucceededException。 还要注意，这将导致从SMTPTransport的sendMessage方法抛出SendFailedException，即使所有地址正确且消息已成功发送也是如此。 |
|  |  |  |


****TLS协议：安全传输层协议（TLS）用于在两个通信应用程序之间提供保密性和数据完整性。****

****密码套件（Cipher suite）是传输层安全（TLS）/安全套接字层（SSL）网络协议中的一个概念****

****AUTH命令是ESMTP命令（SMTP服务的扩展）用于客户端与服务器之间的认证，AUTH命令会将客户端的用户名与密码发送给服务端进行认证，同时AUTH命令也可以结合一些其他的关键字如 PLAIN，LOGIN，CRAM-MD5，DIGEST-MD5等等来选择所要使用的认证机制，下一篇文章我会试着翻阅资料把这几个机制的原理记录下来****