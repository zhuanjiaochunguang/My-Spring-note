# My-Spring-note
生成验证码的方式有很多，个人认为较为灵活方便的是Kaptcha ，他是基于SimpleCaptcha的开源项目。使用Kaptcha 生成验证码十分简单并且参数可以进行自定义。只需添加jar包配置下就可以使用。kaptcha所有配置都可以通过web.xml来完成，如果项目使用了Spring MVC，那么实现方式会略有不同。

一、Servlet项目

1、添加jar包依赖

   maven项目，在pom.xml中添加dependency

<!-- kaptcha -->  
<dependency>  
    <groupId>com.google.code.kaptcha</groupId>  
    <artifactId>kaptcha</artifactId>  
    <version>2.3.2</version>  
</dependency>
  非maven项目，在官网下载kaptcha的jar包，然后添加到项目lib库中。

  下载地址：http://code.google.com/p/kaptcha/downloads/list

2、配置web.xml

<servlet>  
    <servlet-name>Kaptcha</servlet-name>  
    <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>  
</servlet>  
<servlet-mapping>  
    <servlet-name>Kaptcha</servlet-name>  
    <url-pattern>/kaptcha.jpg</url-pattern> 
</servlet-mapping>
    注：url-pattern 自定义

 

    kaptcha的参数都有默认值，如果要配置kaptcha，在init-param增加响应的参数即可
<servlet>  
    <servlet-name>Kaptcha</servlet-name>  
    <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>  
    <init-param>  
        <param-name>kaptcha.image.width</param-name>  
        <param-value>200</param-value>  
        <description>Width in pixels of the kaptcha image.</description>  
    </init-param>  
    <init-param>  
        <param-name>kaptcha.image.height</param-name>  
        <param-value>50</param-value>  
        <description>Height in pixels of the kaptcha image.</description>  
    </init-param>  
    <init-param>  
        <param-name>kaptcha.textproducer.char.length</param-name>  
        <param-value>4</param-value>  
        <description>The number of characters to display.</description>  
    </init-param>  
    <init-param>  
        <param-name>kaptcha.noise.impl</param-name>  
        <param-value>com.google.code.kaptcha.impl.NoNoise</param-value>  
        <description>The noise producer.</description>  
    </init-param>  
</servlet>
3、jsp代码

<script type="text/javascript"> 
$(function(){  //生成验证码         
    $('#kaptchaImage').click(function () {  
    $(this).hide().attr('src', '/code/captcha-image?' + Math.floor(Math.random()*100) ).fadeIn(); });      
});   
 
window.onbeforeunload = function(){  
    //关闭窗口时自动退出  
    if(event.clientX>360&&event.clientY<0||event.altKey){     
        alert(parent.document.location);  
    }  
};  
             
function changeCode() {  //刷新
    $('#kaptchaImage').hide().attr('src', '/code/captcha-image?' + Math.floor(Math.random()*100) ).fadeIn();  
    event.cancelBubble=true;  
}  
</script> 
 
<div class="form-group">  
   <label>验证码 </label> 
   <input name="j_code" type="text" id="kaptcha" maxlength="4" class="form-control" />
   <br/> 
   <img src="/code/captcha-image" id="kaptchaImage"  style="margin-bottom: -3px"/>       
   <a href="#" onclick="changeCode()">看不清?换一张</a>  
</div>
二、Spring mvc 中使用kaptcha

1、spring 配置文件 applicationContext.xml


<bean id="captchaProducer" class="com.google.code.kaptcha.impl.DefaultKaptcha">  
        <property name="config">  
            <bean class="com.google.code.kaptcha.util.Config">  
                <constructor-arg>  
                    <props>  
                        <prop key="kaptcha.border">yes</prop>  
                        <prop key="kaptcha.border.color">105,179,90</prop>  
                        <prop key="kaptcha.textproducer.font.color">blue</prop>  
                        <prop key="kaptcha.image.width">125</prop>  
                        <prop key="kaptcha.image.height">45</prop>  
                        <prop key="kaptcha.textproducer.font.size">45</prop>  
                        <prop key="kaptcha.session.key">code</prop>  
                        <prop key="kaptcha.textproducer.char.length">4</prop>  
                        <prop key="kaptcha.textproducer.font.names">宋体,楷体,微软雅黑</prop>  
                    </props>  
                </constructor-arg>  
            </bean>  
        </property>  
    </bean>
2、Controller的实现

import java.awt.image.BufferedImage;  
import javax.imageio.ImageIO;  
import javax.servlet.ServletOutputStream;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import javax.servlet.http.HttpSession;   
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Controller;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.servlet.ModelAndView;  
import com.google.code.kaptcha.Constants;  
import com.google.code.kaptcha.Producer;  
  
@Controller  
@RequestMapping("/code")  
public class CaptchaController {  
       
    @Autowired  
    private Producer captchaProducer = null;  
   
    @RequestMapping(value = "captcha-image")  
    public ModelAndView getKaptchaImage(HttpServletRequest request, HttpServletResponse response) throws Exception {  
        HttpSession session = request.getSession();  
        String code = (String)session.getAttribute(Constants.KAPTCHA_SESSION_KEY);  
        System.out.println("验证码: " + code );  
           
        response.setDateHeader("Expires", 0);  
        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");  
        response.addHeader("Cache-Control", "post-check=0, pre-check=0");  
        response.setHeader("Pragma", "no-cache");  
        response.setContentType("image/jpeg");  
         
        String capText = captchaProducer.createText();  
        session.setAttribute(Constants.KAPTCHA_SESSION_KEY, capText);  
         
        BufferedImage bi = captchaProducer.createImage(capText);  
        ServletOutputStream out = response.getOutputStream();  
        ImageIO.write(bi, "jpg", out);  
        try {  
            out.flush();  
        } finally {  
            out.close();  
        }  
        return null;  
    }  
}
3、kaptcha可配置项
kaptcha.border  是否有边框  默认为true  我们可以自己设置yes，no  
kaptcha.border.color   边框颜色   默认为Color.BLACK  
kaptcha.border.thickness  边框粗细度  默认为1  
kaptcha.producer.impl   验证码生成器  默认为DefaultKaptcha  
kaptcha.textproducer.impl   验证码文本生成器  默认为DefaultTextCreator  
kaptcha.textproducer.char.string   验证码文本字符内容范围  默认为abcde2345678gfynmnpwx  
kaptcha.textproducer.char.length   验证码文本字符长度  默认为5  
kaptcha.textproducer.font.names    验证码文本字体样式  默认为new Font("Arial", 1, fontSize), new Font("Courier", 1, fontSize)  
kaptcha.textproducer.font.size   验证码文本字符大小  默认为40  
kaptcha.textproducer.font.color  验证码文本字符颜色  默认为Color.BLACK  
kaptcha.textproducer.char.space  验证码文本字符间距  默认为2  
kaptcha.noise.impl    验证码噪点生成对象  默认为DefaultNoise  
kaptcha.noise.color   验证码噪点颜色   默认为Color.BLACK  
kaptcha.obscurificator.impl   验证码样式引擎  默认为WaterRipple  
kaptcha.word.impl   验证码文本字符渲染   默认为DefaultWordRenderer  
kaptcha.background.impl   验证码背景生成器   默认为DefaultBackground  
kaptcha.background.clear.from   验证码背景颜色渐进   默认为Color.LIGHT_GRAY  
kaptcha.background.clear.to   验证码背景颜色渐进   默认为Color.WHITE  
kaptcha.image.width   验证码图片宽度  默认为200  
kaptcha.image.height  验证码图片高度  默认为50
