# rsstolink
已完结

## 作用
1.~~因为alist离线下载遇到文件夹太难受了，10个上传只能成功三个，故用这个链接解析来使用单个链接合集新建任务(**发现还是没啥用，只能等aliast啥时候重视起来修复一下**)~~

## 一键部署到Vercel
[![一键部署到Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/petra1026/animation_rss_downloadlink_extraction&env=VUE_APP_API_URL&envDescription=你的api地址，可以点击旁边的Learn More部署，并填写到右边输入框&envLink=https://github.com/petra1026/RSS-to-JSON/tree/master)


2.遇见没有合集的的番剧可以用这个小工具快速获取多个种子

### 构建准备
```
npm install
```

### 开发
```
npm run serve
```

### 构建
在构建之前，请在.nev中配置你的后端Api地址。附后端项目地址:[RSS-to-JSON](https://github.com/petra1026/RSS-to-JSON)
```
npm run build

```

