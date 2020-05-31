---
title: CSS实现背景拉伸
date: 2020-05-30 14:13:40
categories:
- web前端
tags:
- CSS

#使用hexo-typora-image进行联动,由于设置了根目录的.yml文件的post_asset_folder: true，以及typora的图片复制目录为./${filename}/		,所以这里直接为空就行了。
#这个就作为以后的模板了
typora-copy-images-to: '{{filename}}'

---




```html
body{
    background-image: url("/bg.jpg");		//加载背景图片
    background-repeat: no-repeat;			//设置图片不重复
    background-size: 100% 100%;			//将宽度与高度均设为百分百，显然你自己也可以根据情况设置width还是height。
    background-attachment: fixed;		//这是完成背景拉伸的最关键的一环，将背景固定，使背景不随滚动条走
}
```

