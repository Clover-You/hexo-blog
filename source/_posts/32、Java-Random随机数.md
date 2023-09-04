title: 32、Java Random随机数
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 16:57:42
---
### 产生随机数

随机产生一个int类型的取值范围内的数字

```java
Random random = new Random();
int randomNumber = random.nextInt();
```

生成一个0～100的随机数

```java
Random random = new Random(); 
int randomNumber = random.nextInt(101);
```



### 产生5个不同的随机数

记录查找重复、生成随机数次数

```java
private static int findIndexCount;
private static int randomRunCount;
```

创建随机数

```java
public static int[] randomNumber(int max, int count) {
  int[] nums = new int[count];
  int i = 0;
  while (i < nums.length) {
    randomRunCount++;
    // 创建一个随机数
    Random random = new Random();
    int randomNum = (int) random.nextInt(max); // 得到数字
    int index = findIndex(nums, randomNum);// 查找重复
    if (index == -1) {// -1表示没有找到
      nums[i++] = randomNum;
    }
  }
  return nums;
}
```

查找重复

```java
public static int findIndex(int[] src, int Pos) {
  for (int i = 0; i < src.length; i++) {
    findIndexCount++;
    if (Pos == src[i]) {
      return i;
    }
  }
  return -1;
}
```

使用`randomNumber`

```java
int[] nums = randomNumber(6, 5);
Arrays.sort(nums);

System.out.println("查重使用了：" + findIndexCount + "次");
System.out.println("生成了：" + randomRunCount + "次");
```

\-- `查重使用了：107次`

\-- `生成了：32次`

\-- `1`
\-- `2`
\-- `3`
\-- `4`
\-- `5`