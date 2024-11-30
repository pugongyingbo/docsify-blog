---
title: "字符串相似度"
date: 2019-06-09
keywords: 字符串相似度
description: ""
tags: Java
author: ""

---
#### 字符串相似度
Levenshtein Distance，又称编辑距离，指的是两个字符串之间，由一个转换成另一个所需的最少编辑操作次数。许可的编辑操作包括将一个字符替换成另一个字符，插入一个字符，删除一个字符。
>编辑距离的算法是首先由俄国科学家Levenshtein提出的，故又叫Levenshtein Distance。


```
/**
     * 比较两个字符串的相识度
     * 核心算法：用一个二维数组记录每个字符串是否相同，如果相同记为0，不相同记为1，每行每列相同个数累加
     * 则数组最后一个数为不相同的总数，从而判断这两个字符的相识度
     *
     * @param str
     * @param target
     * @return
     */
    private static int compare(String str, String target) {
        int d[][];              // 矩阵
        int n = str.length();
        int m = target.length();
        int i;                  // 遍历str的
        int j;                  // 遍历target的
        char ch1;               // str的
        char ch2;               // target的
        int temp;               // 记录相同字符,在某个矩阵位置值的增量,不是0就是1
        if (n == 0) {
            return m;
        }
        if (m == 0) {
            return n;
        }
        d = new int[n + 1][m + 1];
        // 初始化第一列
        for (i = 0; i <= n; i++) {
            d[i][0] = i;
        }
        // 初始化第一行
        for (j = 0; j <= m; j++) {
            d[0][j] = j;
        }
        for (i = 1; i <= n; i++) {
            // 遍历str
            ch1 = str.charAt(i - 1);
            // 去匹配target
            for (j = 1; j <= m; j++) {
                ch2 = target.charAt(j - 1);
                if (ch1 == ch2 || ch1 == ch2 + 32 || ch1 + 32 == ch2) {
                    temp = 0;
                } else {
                    temp = 1;
                }
                // 左边+1,上边+1, 左上角+temp取最小
                d[i][j] = min(d[i - 1][j] + 1, d[i][j - 1] + 1, d[i - 1][j - 1] + temp);
            }
        }
        return d[n][m];
    }
 
 
    /**
     * 获取最小的值
     */
    private static int min(int one, int two, int three) {
        return (one = one < two ? one : two) < three ? one : three;
    }
 
    /**
     * 获取两字符串的相似度
     */
    public static float getSimilarityRatio(String str, String target) {
        
    //去除空白字符、换行、标点符号 
     String regex = "[\\pP\\p{Punct}\\s]";
     return 1 - (float) compare(str.replaceAll(regex, ""), target.replaceAll(regex, "")) 
     / Math.max(str.length(), target.length());
     }
     
     public static void main(String[] args) {
    String str = "论据容易理解,把它们分成小块,并以相似的格式和句子结构呈现每一个论据";
    String target = "根据领域关系可以合并领域相同或相似的句子，得到句群及其领域";
    System.out.println("similarityRatio=" + getSimilarityRatio(str, target));

    }

```
