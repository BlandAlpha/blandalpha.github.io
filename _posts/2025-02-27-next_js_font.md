---
title: Next.js网页如何在shadcn下自定义字体
author: guoao
category: [笔记]
tags: [计算机, 笔记, 前端开发, Next.js, shadcn, SaaS]
image:
   path: https://images.unsplash.com/photo-1530435460869-d13625c69bbf
---

## 如何自定义字体？ (Google Font)

本人正在初学现代网页前端技术架构。在使用 `Next.js` 与 [shadcn](https://ui.shadcn.com/) 构建网页时，教程告诉我可以使用 `next/font/google` 方法更为便捷而高性能地更改网页字体。方法如下：

1. 打开 `layout.js` 文件，在最上方输入：

```js
import { DM_Sans } from "next/font/google";
```

当然这里的 `DM_Sans` 字体可以更换为 Google Font 中的任何一款字体。

2. 在下方，创建一个 `const` 字体类型：

```js
import { DM_Sans } from "next/font/google";
// 其他 import...

const dm_sans = DM_Sans({ 
  subsets: ['latin'],
  display: 'swap',  // 这一行据说可以提高字体加载效率，不过我没测试过
})
```

3. 为你的 `body` 或 `main` 标签添加类名：

```js
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body
        className={cn('min-h-screen antialiased', dm_sans.className)}
      >
        <Navbar />
        {children}
      </body>
    </html>
  );
}
```

如果你使用了 `shadcn` 的全局样式，则你需要像上面一样加入 `dm_sans` 的类名。

> 注意！你需要记得把 `cn` 内部的 `font-san` 删除掉！不然这会覆盖你后面的自定义字体。
{: .prompt-danger}

这样你应该就可以完成网页字体的全局替换了。

## 为什么我这样做完之后字体仍没有变化？

在经过查询之后发现问题可能出在这些地方：

### `.next` 文件夹

在2024年末左右，reddit、stackoverflow以及其他论坛大量出现了 Google 字体突然无法加载的情况。高票回答大多都指出了 next.js 的缓存问题。如果你遇到了字体无法加载的情况，请跟随以下步骤：

1. 关闭浏览器里所有本地网页实例；
2. 命令行输入 `^C` 关闭 `dev` 服务器；
3. 删除项目文件夹下的 `.next` 文件夹；
4. 重新运行 `mpn dun dev`。

经过以上处理可以清除 next.js 的本地字体缓存。

### 字体名称错误或覆盖

如果你经过上一步操作也没有解决问题，则可能是你字体名输错了，或者被覆盖。

上文已经提到，如果你不小心忘记删掉 `font-sans`：

```js
className={cn('min-h-screen font-sans antialiased', dm_sans.className)}
```

那么系统默认字体就会覆盖掉你之后定义的字体。解决方法就是删除他。
