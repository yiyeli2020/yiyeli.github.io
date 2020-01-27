title: Java学习笔记XIII
date: 2019-12-24 11:21:12
categories: 2019年12月
tags: [Java]

---

Java中Calendar，LocalDateTime的用法和区别
Java中length,length(),size()区别

<!-- more -->


例如使用LocalDateTime：

    private static void localDateTimeTest() throws ModuleBusinessException{
        LocalDateTime currentTime=LocalDateTime.now();
        LocalDateTime suspendLoanTime=LocalDateTime.of(2020,1,23,10,0,0);
        LocalDateTime recoverLoanTime=LocalDateTime.of(2020,1,31,9,59,59);
        System.out.println(currentTime);
        System.out.println(suspendLoanTime);
        System.out.println(recoverLoanTime);
        if(currentTime.isAfter(suspendLoanTime) && currentTime.isBefore(recoverLoanTime)){
            throw new ModuleBusinessException("尊敬的用户您好，1月23日10:00-1月31日9:59暂停XX，1月31日10:00恢复XX，届时会短信通知到您，需要您再次来到APP继续XX");
        }
    }

而使用Calendar时有个容易被坑的地方，calendar.set()中月份字段从0开始：

    private static void dateTimeTest() throws ModuleBusinessException {
        Calendar calendar=Calendar.getInstance();
        Date currentTime=calendar.getTime();
        calendar.set(2020,0,23,10,0,0);//（月份字段从0开始）
        Date suspendLoanTime=calendar.getTime();
        calendar.set(2020,0,31,9,59,59);//（月份字段从0开始）
        Date recoverLoanTime=calendar.getTime();
        System.out.println(currentTime);
        System.out.println(suspendLoanTime);
        System.out.println(recoverLoanTime);
        if(!currentTime.after(suspendLoanTime) || !currentTime.before(recoverLoanTime)){
            throw new ModuleBusinessException("尊敬的用户您好，1月23日10:00-1月31日9:59暂停XX，1月31日10:00恢复XX，届时会短信通知到您，需要您再次来到APP继续XX");
        }
    }

java中的length属性是针对数组说的,比如说声明了一个数组,想知道这个数组的长度则用到了length这个属性.

java中的length()方法是针对字符串String说的,如果想看这个字符串的长度则用到length()这个方法.

java中的size()方法是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看


# 参考文献：
【1】https://blog.csdn.net/sk880609/article/details/7524006
