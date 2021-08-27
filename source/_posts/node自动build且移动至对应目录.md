---
title: node自动build且移动至对应目录
date: 2021-08-27 14:53:18
subtitle:
categories: node
tags:
- node
- 构建
- 随笔
cover: /images/0bbd039fb5a54de2b240ab0fcd3af8ed.jpg
---


项目A的`libs`中引用了项目B `build`后的文件，在本地调试两个项目的bug时，对项目B做的修改无法立即在A中查看效果，需要 build之后，再移动对应的文件。十分的繁琐，这生产力必须要解放一波:smile:

## 打包
```javascript
const process = require('child_process');

// 利用execSync 执行shell命令
const buildScript = 'npm run build'

process.execSync(buildScript, function(error, res) {
  if(!error) console.log('🚀~ 打包完成 🚀~', res);
});
```

## 移动或复制

### execSync(fail)
惯性思维。还是想着使用 `execSync` 来执行`mv`或者`cp`的命令来移动/复制build文件

```javascript
  const mvScript = 'mv -a dist/. xxx/'
  process.execSync(mvScript, error => {
    if(!error) console.log('移动完成~');
  })
```
*注：windows执行`execSync`打开的是cmd命令行，而我使用的`linux`的命令。所以这样是行不通的*


### execFile执行bat程序(fail)
无法直接执行命令，那么使用`批处理程序`的 [move](https://jingyan.baidu.com/article/046a7b3e98c2fef9c37fa949.html)  命令
```javascript
// 运行批处理程序
process.execFile('temp.bat', [], {}, function(error, stdout, stderr) {
  console.log(error);
  console.log(stdout);
  console.log(stderr);
});
```
temp.bat
```bash
move dist\*.* xxx
```

*注：还是未成功，报错 文件名、目录名或卷标语法不正确。* 有待考究


### 化繁为简 fs模块
使用node的 `fa 文件系统`的[copyFile](http://nodejs.cn/api/fs.html#fs_fs_copyfile_src_dest_mode_callback)api
```javascript
const distDirPath = path.join(__dirname, 'dist')
// 要复制到的文件夹
const destDirPath = path.join(__dirname, '../项目A/libs')
const files = fs.readdirSync(distDirPath)

files.forEach(file => {
  fs.copyFile(path.join(distDirPath, file), path.join(destDirPath, file), (err) => {
    console.log("出错啦~")
  }) 
})
```

## 总结
node还有许多的模块，都十分有趣。光是复制文件的api就有
- `fs.copyFile` callback模式
- `fs.copyFileSync` 同步模式
- `fsPromises.copyFile` 

还有`.cp()`的三种模式(不过是 v16.7.0新增的api)

慢慢学习，变得更强~



