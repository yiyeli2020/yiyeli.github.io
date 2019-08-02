---
title: SpringBoot学习笔记VIII
date: 2019-8-1 17:12:12
categories: 2019年8月
tags: [SpringBoot]

---

CMS系统学习，对常用到的注解做详解

<!-- more -->
# CMS系统简介
内容管理系统（英语：content management system，缩写为 CMS）是指在一个合作模式下，用于管理工作流程的一套制度。该系统可应用于手工操作中，也可以应用到计算机或网络里。作为一种中央储存器（central repository），内容管理系统可将相关内容集中储存并具有群组管理、版本控制等功能。版本控制是内容管理系统的一个主要优势。
内容管理系统在物品或文案或数据的存储、掌管、修订（盘存）、语用充实、文档发布等方面有着广泛的应用。现在流行的开源CMS系统有WordPress、Joomla!、Drupal、Xoops、CmsTop等。

# 任务描述
saas-cms 的Respository中建立自己的分支，在controller/creditreview/TransportController中画出getAllProcessHistory函数的流程图，自己重新实现一个查询接口
数据库配置文件在resources.local/application.yml中
使用的数据库是：jdbc:mysql://10.143.248.78:3306/creditreview?


# 注解

handler method 参数绑定常用的注解,根据处理的Request的不同内容部分分为四类：（主要讲解常用类型）

A、处理request uri 部分（这里指uri template中variable，不含queryString部分）的注解：@PathVariable;

B、处理request header部分的注解：   @RequestHeader, @CookieValue;

C、处理request body部分的注解：@RequestParam,  @RequestBody;

D、处理attribute类型是注解： @SessionAttributes, @ModelAttribute;

## @PathVariable 

当使用@RequestMapping URI template 样式映射时， 即 someUrl/{paramId}, 这时的paramId可通过 @Pathvariable注解绑定它传过来的值到方法的参数上。

示例代码：


    @Controller
    @RequestMapping("/owners/{ownerId}")
    public class RelativePathUriTemplateController {

      @RequestMapping("/pets/{petId}")
      public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {    
        // implementation omitted
      }
    }
上面代码把URI template 中变量 ownerId的值和petId的值，绑定到方法的参数上。若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable("name")指定uri template中的名称。

##  @RequestHeader、@CookieValue

***@RequestHeader注解，可以把Request请求header部分的值绑定到方法的参数上。***

示例代码：

这是一个Request 的header部分：


    Host                    localhost:8080
    Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
    Accept-Language         fr,en-gb;q=0.7,en;q=0.3
    Accept-Encoding         gzip,deflate
    Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive              300

下面的代码，把request header部分的 Accept-Encoding的值，绑定到参数encoding上了， Keep-Alive header的值绑定到参数keepAlive上。


    @RequestMapping("/displayHeaderInfo.do")
    public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
                                  @RequestHeader("Keep-Alive") long keepAlive)  {

      //...

    }



***@CookieValue 可以把Request header中关于cookie的值绑定到方法的参数上。***

例如有如下Cookie值：

    JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
参数绑定的代码：

    @RequestMapping("/displayHeaderInfo.do")
    public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie)  {

      //...

    }
即把JSESSIONID的值绑定到参数cookie上。

##  @RequestParam, @RequestBody

### @RequestParam

A） 常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值；

B）用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容，提交方式GET、POST；

C) 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定；

示例代码：

    @Controller
    @RequestMapping("/pets")
    @SessionAttributes("pet")
    public class EditPetForm {

        // ...

        @RequestMapping(method = RequestMethod.GET)
        public String setupForm(@RequestParam("petId") int petId, ModelMap model) {
            Pet pet = this.clinic.loadPet(petId);
            model.addAttribute("pet", pet);
            return "petForm";
        }

        // ...


### @RequestBody

该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；

它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。

因为配置有FormHttpMessageConverter，所以也可以用来处理 application/x-www-form-urlencoded的内容，处理完的结果放在一个MultiValueMap<String, String>里，这种情况在某些特殊需求下使用，详情查看FormHttpMessageConverter api;

示例代码：

    @RequestMapping(value = "/something", method = RequestMethod.PUT)
    public void handle(@RequestBody String body, Writer writer) throws IOException {
      writer.write(body);

## @SessionAttributes, @ModelAttribute

### @SessionAttributes:

该注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。

该注解有value、types两个属性，可以通过名字和类型指定要使用的attribute 对象；

示例代码：

    @Controller
    @RequestMapping("/editPet.do")
    @SessionAttributes("pet")
    public class EditPetForm {
        // ...
    }


### @ModelAttribute

该注解有两个用法，一个是用于方法上，一个是用于参数上；

用于方法上时：  通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model；

用于参数上时： 用来通过名称对应，把相应名称的值绑定到注解的参数bean上；要绑定的值来源于：

A） @SessionAttributes 启用的attribute 对象上；

B） @ModelAttribute 用于方法上时指定的model对象；

C） 上述两种情况都没有时，new一个需要绑定的bean对象，然后把request中按名称对应的方式把值绑定到bean中。



用到方法上@ModelAttribute的示例代码：

    // Add one attribute
    // The return value of the method is added to the model under the name "account"
    // You can customize the name via @ModelAttribute("myAccount")

    @ModelAttribute
    public Account addAccount(@RequestParam String number) {
        return accountManager.findAccount(number);
    }

这种方式实际的效果就是在调用@RequestMapping的方法之前，为request对象的model里put（“account”， Account）；


用在参数上的@ModelAttribute示例代码：

    @RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
    public String processSubmit(@ModelAttribute Pet pet) {

    }
首先查询 @SessionAttributes有无绑定的Pet对象，若没有则查询@ModelAttribute方法层面上是否绑定了Pet对象，若没有则将URI template中的值按对应的名称绑定到Pet对象的各属性上。

## 在不给定注解的情况下，参数是怎样绑定的？

通过分析AnnotationMethodHandlerAdapter和RequestMappingHandlerAdapter的源代码发现，方法的参数在不给定参数的情况下：

若要绑定的对象时简单类型：  调用@RequestParam来处理的。 

若要绑定的对象时复杂类型：  调用@ModelAttribute来处理的。

这里的简单类型指java的原始类型(boolean, int 等)、原始类型对象（Boolean, Int等）、String、Date等ConversionService里可以直接String转换成目标对象的类型；



# 参考资料：
【1】https://blog.csdn.net/walkerJong/article/details/7946109
