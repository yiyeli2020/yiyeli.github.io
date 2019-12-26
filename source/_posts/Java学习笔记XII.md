title: Java学习笔记XII
date: 2019-12-24 11:21:12
categories: 2019年12月
tags: [Java]

---

使用Java批量转换文件夹中的文件编码格式，删除文件中的代码

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

批量删除代码中的注释

    private static int count = 0;

    /**
     * 删除文件中的各种注释，包含//、/* * /等
     *
     * @param charset
     *            文件编码
     * @param file
     *            文件
     */
    public static void clearComment(File file, String charset) {
        try {
            // 递归处理文件夹
            if (!file.exists()) {
                return;
            }

            if (file.isDirectory()) {
                File[] files = file.listFiles();
                for (File f : files) {
                    clearComment(f, charset); // 递归调用
                }
                return;
            } else if (!file.getName().endsWith(".cpp")) {
                // 非java文件直接返回
                return;
            }
            System.out.println("-----开始处理文件：" + file.getAbsolutePath());

            // 根据对应的编码格式读取
            BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(file), charset));
            StringBuffer content = new StringBuffer();
            String tmp = null;
            while ((tmp = reader.readLine()) != null) {
                content.append(tmp);
                content.append("\n");
            }
            reader.close();
            String target = content.toString();
            // String s =
            // target.replaceAll("\\/\\/[^\\n]*|\\/\\*([^\\*^\\/]*|[\\*^\\/*]*|[^\\**\\/]*)*\\*\\/",
            // ""); //本段正则摘自网上，有一种情况无法满足（/* ...**/），略作修改
            String s = target.replaceAll("\\/\\/[^\\n]*|\\/\\*([^\\*^\\/]*|[\\*^\\/*]*|[^\\**\\/]*)*\\*+\\/", "");
            // System.out.println(s);
            // 使用对应的编码格式输出
            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(file), charset));
            out.write(s);
            out.flush();
            out.close();
            count++;
            System.out.println("-----文件处理完成---" + count);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void clearComment(String filePath, String charset) {
        clearComment(new File(filePath), charset);
    }

    public static void clearComment(String filePath) {
        clearComment(new File(filePath), "GBK");
    }

    public static void clearComment(File file) {
        clearComment(file, "GBK");
    }
    public  static   void  main(String[] args){
        clearComment(destPath);
    }



# 参考资料
【1】https://blog.csdn.net/qq1032355091/article/details/51803496
【2】https://www.cnblogs.com/hfultrastrong/p/7689630.html
