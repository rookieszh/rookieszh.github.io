---
title: Code Refactoring Note I
date: 2020-03-27 17:51:06
tags:
    - Programming
    - Refactoring
description: "本节笔记介绍了 what is code refactoring，并以if-else的例子来说明如何实际运用 code refactoring。"
---

## 什么是重构

### 误区与定义
误区：把任何形式的代码清理当做重构。
定义：对软件内部结构的一种调整，目的是在不改变软件可观察行为的前提下，提高其可理解性，降低修改成本。

### 软件工程质量
外部质量
* Functionality(功能性)
* Usability(可用性)
* Reliability(可靠性)
* Performance(性能)
* Supportability/Safety(维护性、安全性)

内部质量
* 代码质量(不可见)

### 为何需要重构

#### 技术债务
> 开发团队在设计或架构选型时从短期效应的角度选择了一个易于实现的方案，但从长远来看，这种方案会带来消极的影响，亦即开发团队所欠的债务。 ——Ward Cunningham

#### 破窗效应

#### 重构的收益
重构改进软件的设计——保持架构活力
重构使软件更容易理解——易理解>易维护>高效率
重构帮助找到BUG——结构清晰更容易发现问题
重构提高编程速度——更快迭代

#### 重构的挑战
1. 工期延误：Baby step、长期收益
2. 破坏功能：单元测试很重要、使用Eclipse的自动化重构工具
3. 性能下降
深奥的代码并不能给编译器提供更好的优化线索
影响系统效率的关键通常只在10%的代码上
调优一段整洁的代码，比调优一段“被优化过”的代码容易的多

## Java 中如何减少 if-else 嵌套
场景：
1. 异常逻辑处理：只能一个分支是正常流程
2. 不同状态处理：所有分支都是正常流程

### 接口分层
> 把接口分为外部和内部接口，所有空值判断放在外部接口完成；而内部接口传入的变量由外部接口保证不为空，从而减少空值判断。

### 利用多态
> 利用多态，把业务判断消除，各子类分别关注自己的实现，并实现子类的创建方法，避免用户了解过多的类。
> 但使用多态对原来代码修改过大，最好在设计之初就使用多态方式。
```java
// 重构前
private static final int TYPE_LINK = 0;
private static final int TYPE_IMAGE = 1;
private static final int TYPE_TEXT = 2;
private static final int TYPE_IMAGE_TEXT = 3;
public class ShareItem {
    int type;
    String title;
    String content;
    String imagePath;
    String link;
}
public interface ShareListener {
    int STATE_SUCC = 0;
    int STATE_FAIL = 1;
    void onCallback(int state, String msg);
}

public void share(ShareItem item, ShareListener listener) {
    // do something
    shareImpl(item, listener);
}
private void shareImpl (ShareItem item, ShareListener listener) {
    if (item.type == TYPE_LINK) {
        // 分享链接
        if (!TextUtils.isEmpty(item.link) && !TextUtils.isEmpty(item.title)) {
            doShareLink(item.link, item.title, item.content, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else if (item.type == TYPE_IMAGE) {
        // 分享图片
        if (!TextUtils.isEmpty(item.imagePath)) {
            doShareImage(item.imagePath, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else {
        listener.onCallback(ShareListener.STATE_FAIL, "不支持的分享类型");
    }
}

// 重构后
public abstract class ShareItem {
    int type;
    public ShareItem(int type) {
        this.type = type;
    }
    public abstract void doShare(ShareListener listener);
}
public class Link extends ShareItem {
    String title;
    String content;
    String link;
    public Link(String link, String title, String content) {
        super(TYPE_LINK);
        this.link = !TextUtils.isEmpty(link) ? link : "default";
        this.title = !TextUtils.isEmpty(title) ? title : "default";
        this.content = !TextUtils.isEmpty(content) ? content : "default";
    }
    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}
public class Image extends ShareItem {
    String imagePath;
    public Image(String imagePath) {
        super(TYPE_IMAGE);
        this.imagePath = !TextUtils.isEmpty(imagePath) ? imagePath : "default";
    }
    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}

public void share(ShareItem item, ShareListener listener) {
    // do something
    shareImpl(item, listener);
}
private void shareImpl (ShareItem item, ShareListener listener) {
    item.doShare(listener);
}
```

### Map分支
> 把分支状态信息预先缓存在Map里，直接get获取具体值，消除分支。
```java
private Map<Integer, Class<? extends ShareItem>> map = new HashMap<>();

private void init() {
    map.put(TYPE_LINK, Link.class);
    map.put(TYPE_IMAGE, Image.class);
    map.put(TYPE_TEXT, Text.class);
    map.put(TYPE_IMAGE_TEXT, ImageText.class);
}

public ShareItem createShareItem(int type) {
    try {
        Class<? extends ShareItem> shareItemClass = map.get(type);
        return shareItemClass.newInstance();
    } catch (Exception e) {
        return new DefaultShareItem(); // 返回默认实现，不要返回null
    }
}
```

### 合并条件表达式
```java
// 重构前
double disablityAmount(){
    if(_seniority < 2)
        return 0;
    if(_monthsDisabled > 12)
        return 0;
    if(_isPartTime)
        return 0;
}

// 重构后

double disablityAmount(){
    if(_seniority < 2 || _monthsDisabled > 12 || _isPartTime)
        return 0;
}
```

### 提前return
> 改为平行关系，而非包含关系
> 尽可能地维持正常流程代码在最外层
```java
// 重构前
double getPayAmount(){
    double result;
    if(_isDead) {
        result = deadAmount();
    }else{
        if(_isSeparated)
            result = separatedAmount();
        else{
            if(_isRetired)
                result = retiredAmount();
            else
                result = normalPayAmount();
        }
    }
    return result;
}

// 重构后
double getPayAmount(){
    if(_isDead)
        return deadAmount();
    if(_isSeparated)
        return separatedAmount();
    if(_isRetired)
        return retiredAmount();
    return normalPayAmount();
}
```
> 将条件反转使异常情况先退出，让正常流程维持在主干流程。
```java
// 重构前
public double getAdjustedCapital(){
    double result = 0.0;
    if(_capital > 0.0 ){
        if(_intRate > 0 && _duration >0){
            resutl = (_income / _duration) *ADJ_FACTOR;
        }
    }
    return result;
}

// 重构后
// (_income / _duration) *ADJ_FACTOR 是主流程，根据优化原则（尽可能地维持正常流程代码在最外层），放到最外层
public double getAdjustedCapital(){
    if(_capital <= 0.0 ){
        return 0.0;
    }
    if(_intRate <= 0 || _duration <= 0){
        return 0.0;
    }
    return (_income / _duration) *ADJ_FACTOR;
}
```

### if-else 封装
```java
double getPayAmount(){
    Object obj = getObj();
    if (obj.getType == 1) {
        return getType1Money(obj);
    }
    else if (obj.getType == 2) {
        return getType2Money(obj);
    }
}

double getType1Money(Object obj){
    ObjectA objA = obj.getObjectA();
    return objA.getMoney()*obj.getNormalMoneryA();
}

double getType2Money(Object obj){
    ObjectB objB = obj.getObjectB();
    return objB.getMoney()*obj.getNormalMoneryB()+1000;
}
```
