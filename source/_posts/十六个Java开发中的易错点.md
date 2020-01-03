title: 十六个Java开发中的易错点
date: 2020-1-3 16:11:12
categories: 2020年1月
tags: [Java]

---

十六个Java开发中的易错点

<!-- more -->

# 避免非空判断仅判断是否为empty
反例：

    public  static   void  main(String[] args){
        Map data=new HashMap();
        data=null;
        if(data.isEmpty()){
            return;
        }
    }

只判断data是否为isEmpty，但当数据为null时会报空指针异常

    Exception in thread "main" java.lang.NullPointerException
    	at Application.main(Application.java:454)

# log抛出要有详情和具体说明
反例：

    try {
      param = JSONObject.parseObject(methodParam,method.getParameterTypes()[0]);
    } catch (Exception e){
      throw new ParamException(e.getMessage());
    }
信息需要详细具体，参见
正例：

    try {
      ...
    } catch (IOException e){
      logger.error("....Failed",e);
      return null;
    }

# 配置文件尽量在配置文件定义，代码中不要配置

反例：

  String ip="127.0.0.1";
  Socket socket=new Socket(ip,6667);

正例：

  String ip= System.getProperty("myapplication.ip");
  Socket socket=new Socket(ip,6667);

# 精度转换

BigDecimal转double，需先转换为string再 BigDecimal（String s），再使用BigDecimal.valueOf()方法，避免精度丢失



# 参考资料
【1】https://www.hollischuang.com/archives/1360
