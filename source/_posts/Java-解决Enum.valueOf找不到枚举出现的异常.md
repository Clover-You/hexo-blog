title: Java 解决Enum.valueOf找不到枚举出现的异常
categories: 随笔
tags: [JAVA,随笔]
date: 2022-01-02 18:21:06
---
由于`Enum.valueOf`匹配不到枚举时会出现异常，这个可以用`try...catch`来解决，但是这样会导致代码往臃肿的道路上越走越远。
本文与其说是解决**Enum.valueOf找不到枚举出现的异常**还不如说是换了解决方案：
现在有这么一个枚举
```java
/**
 * 类型操作接口
 */
public interface TypeEnum {
  Boolean ret(); // 表示是某个类型时返回结果
}

public enum ImageEnumImpl implements TypeEnum {
  /**
   * jpeg类型的图片
   */
  JPEG {
    @Override
    public Boolean ret() {
      return true;
    }
  },
  /**
   * jpg类型的图片
   */
  JPG {
    @Override
    public Boolean ret() {
      return true;
    }
  },
  /**
   * gif类型的图片
   */
  GIF {
    @Override
    public Boolean ret() {
      return true;
    }
  },
  /**
   * bmp类型的图片
   */
  BMP {
    @Override
    public Boolean ret() {
      return true;
    }
  },
  /**
   * png类型的图片
   */
  PNG {
    @Override
    public Boolean ret() {
      return true;
    }
  },

}

```
服务端需要通过枚举来判断支持上传的文件类型，可以使用`Enum.valueOf`来判断，而且很香
```java
 if (ImageEnumImpl.valueOf(exhibitionName).ret()){}
```
但是如果枚举类中并不存在这个类型就会抛出一个异常，导致无法用`if`的方式来判断，可以使用以下代码来操作，当枚举不存在时返回`null`
```java
 private ImageEnumImpl getIfPresent(String name) {
    return Enums.getIfPresent(ImageEnumImpl.class, name).orNull();
 }
```
使用这个方法，这样代码就好看多了
```java
if (getIfPresent(exhibitionName) == null) {
    return AjaxResult.error(400, "请上传正确的图片文件，如：jpg、png、bmp、gif！");
}
```