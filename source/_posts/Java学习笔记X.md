title: Java学习笔记X
date: 2019-12-17 18:05:12
categories: 2019年12月
tags: [Java]

---

Java中前后端交互例如使用ajax异步传值。

<!-- more -->
AJAX（Asynchronous JavaScript and XML）意思就是用JavaScript执行异步网络请求。如果仔细观察一个Form的提交，你就会发现，一旦用户点击“Submit”按钮，表单开始提交，浏览器就会刷新页面，然后在新页面里告诉你操作是成功了还是失败了。如果不幸由于网络太慢或者其他原因，就会得到一个404页面。

这就是Web的运作原理：一次HTTP请求对应一个页面。

如果要让用户留在当前页面中，同时发出新的HTTP请求，就必须用JavaScript发送这个新请求，接收到数据后，再用JavaScript更新页面，这样一来，用户就感觉自己仍然停留在当前页面，但是数据却可以不断地更新。

前端代码：

  function sendMessage(tokenType){
     console.log(tokenType);
     var serverURL="[[${serverURL}]]";//从后端的map中获取
     //token服务地址
     var url = "/getToken";
     $.ajax({
         type : "post",
         data :{
         "tokenType":tokenType
         },
         dataType:"json",
         async : true,
         url : url,
         success:function(token){
               console.log(token)
               window.frames[0].postMessage(token.data,serverURL);
         },
         error: function() {
            alert("获取token失败，请稍后再试！");
          }
       });
后端代码：

    @Value("${cm.bussinessName}")//从application.yml中获取
    private String bussinessName;
    @Value("${cm.bussinessID}")
    private String bussinessID;
    @Value("${cm.sysID}")
    private String sysID;
    @Value("${cm.userName}")
    private String userName;
    @Value("${cm.serverURL}")
    private String serverURL;

    @ApiOperation(value = "cm5页面接入接口")
    @GetMapping("/cm5")
    public String getCm5(ModelMap map) {
     String scanTargetUrl= String.format("%s/interaction/queryFile?clientId=$$$&batchId=###&bussinessId=%s&bussTableName=%s&userID=%s&sysid=%s",serverURL,bussinessID,bussinessName,userName,sysID);
     String uploadTargetUrl=String.format("%s/interaction/operateParam?clientId=$$$&batchId=###&bussinessId=%s&bussTableName=%s&userID=longruntest&sysid=%s",serverURL,bussinessID,bussinessName,sysID);
     map.put("scanTargetUrl",scanTargetUrl);
     map.put("uploadTargetUrl",uploadTargetUrl);
     map.put("serverURL",serverURL);
     return "/cm5/cm5";
    }

    @PostMapping("/getToken")
    @ResponseBody
    public String getToken( String tokenType){
        CMPlusClient cmclient = cm5UtilService.getCmPlusClient();
        return cmclient.getToken(tokenType);
    }

由于后端传递的targetUrl中的&会被转义为&amp;，所以添加过滤函数

     function unescapeHTML(a){
      a = "" + a;
      return a.replace(/&lt;/g, "<").replace(/&gt;/g, ">").replace(/&amp;/g, "&").replace(/&quot;/g, '"').replace(/&apos;/g, "'");
    }

前端页面cm5.html中要注意js脚本的路径是否存在：
    <script src="static/js/jquery-1.11.1.min.js" type="text/javascript"></script>

# 参考资料
【1】https://my.oschina.net/smfx1314/blog/3053056
【2】https://www.jianshu.com/p/35020e29206c
【3】http://xbhong.top/2018/02/28/ajax/
