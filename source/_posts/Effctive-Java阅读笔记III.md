title: Effctive-Java阅读笔记III
date: 2019-12-31 15:06:12
categories: 2019年12月
tags: [Design Pattern，Java]

---

第二章阅读：创建和销毁对象
05.依赖注入优于硬连接资源（hardwiring resources）


<!-- more -->
　许多类依赖于一个或多个底层资源。例如，拼写检查器依赖于字典。将此类类实现为静态工具类并不少见 （详见第 4 条）:

    // Inappropriate use of static utility - inflexible & untestable!
    public class SpellChecker {
        private static final Lexicon dictionary = ...;

        private SpellChecker() {} // Noninstantiable

        public static boolean isValid(String word) { ... }
        public static List<String> suggestions(String typo) { ... }
    }

同样地，将它们实现为单例的做法也并不少见（详见第 3 条）：

    // Inappropriate use of singleton - inflexible & untestable!
    public class SpellChecker {
        private final Lexicon dictionary = ...;

        private SpellChecker(...) {}
        public static INSTANCE = new SpellChecker(...);

        public boolean isValid(String word) { ... }
        public List<String> suggestions(String typo) { ... }
    }



#参考资料
