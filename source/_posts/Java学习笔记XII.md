title: Java学习笔记XII
date: 2019-12-24 11:21:12
categories: 2019年12月
tags: [Java]

---

Java批量转换文件夹中的文件格式

<!-- more -->
用Java将某个文件夹中UTF-8格式的文件批量转换为GBK格式的文件，代码如下：

   private static String sourcePath = "/Users/liyiye/Documents/课程代码";// 文件夹源路径
   private static String destPath = "/Users/liyiye/Documents/ANSI格式课程代码";
   private static void utf8ToANSI(){
       try {
           File sourceDirectory = new File(sourcePath);
           File destDirectory = new File(destPath);
           if (!sourceDirectory.isDirectory()) {
               return;
           }
           // 获取文件夹中的所有.cpp文件，包括所有子级文件夹中的文件
           Collection<File> files = FileUtils.listFiles(sourceDirectory, new String[] { "cpp", "CPP" }, true);
           for (File file : files) {
               String absolutePath = file.getAbsolutePath();
               String newDir = absolutePath.replace(sourceDirectory.getName(), destDirectory.getName());
               // 把单个文件从utf-8编码转化到gbk编码，生成新文件，可以自动创建父级目录
               FileUtils.writeLines(new File(newDir), "GBK", FileUtils.readLines(file, "UTF-8"));
           }
           // 删除源目录,子文件都删除
           // FileUtils.deleteQuietly(sourceDirectory);
           // 把生成文件目录重命名成源目录名
           destDirectory.renameTo(new File(sourceDirectory.getAbsolutePath()));
           System.out.println("success");
       } catch (IOException e) {
           e.printStackTrace();
       }
   }





# 参考资料
【1】https://blog.csdn.net/qq1032355091/article/details/51803496
