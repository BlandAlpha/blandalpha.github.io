---
title: Hello World
author: guoao
category: [日常]
tags: [杂谈]
math: true
image:
   path: https://images.unsplash.com/photo-1548679847-1d4ff48016c7
   alt: 封面图
---

>TODO: 更改Favicon，更改英文字体到Rubik，减少Social链接
{: .prompt-info}

尝试将这个页面发布到Github pages上去……上一个模板部署失败了，这次祝我好运！

## 代码渲染测试

```python
num=[]
i=2
for i in range(2,100):
   j=2
   for j in range(2,i):
      if(i%j==0):
         break
   else:
      num.append(i)
print(num)
```

This is an `in-line code`

## 公式渲染测试

单独成行的公式：

$$
y=ax^2+bx+c
$$

行内公式：$$ y=ax^2 $$

## 图片渲染测试

![WebPic](https://images.unsplash.com/photo-1503460589709-a01a75418735){: .shadow}
_一张在线图片_

## 提示块测试

>这是一则小提示
{: .prompt-info}

## 视频嵌入测试

### YouuTube视频嵌入

{% include embed/youtube.html id='9XpBKjPNxTw' %}

### Bilibili视频嵌入

{% include embed/bilibili.html id='BV15W421P7cK' %}