# mapslice - 图像瓦片切图工具

[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg)](http://standardjs.com/)

源项目地址：https://github.com/martinheidegger/mapslice

以下为中文说明翻译，以及一些使用细节上的补充说明，可直接使用的文件包请到这里下载：https://github.com/Ruimusume/mapslice/releases

mapslice是一种将图像切割成各种缩放级别的瓦片以用于交互式地图的工具。

有很多可用于高性能查看大图像的 Javascript 工具，但使用这些工具裁剪和切片图像可能会很麻烦。
mapslice会自动检测载入材料支持的图块大小，并创建所有可能的图块，以供常见的 javascript 地图工具
如 [polymaps](http://polymaps.org/), [kartograph](http://kartograph.org/) 或 [PanoJS](http://www.dimin.net/software/panojs/)。

## 安装

请先安装最新的 [node](http://nodejs.org/) 后在运行命令

cmd运行:

```bash
npm install mapslice
```
## 安装的目录位置

```bash
C:\Users\用户名\node_modules      //mapslice的支持包
C:\Users\用户名\package.json      //mapslice版本号信息
C:\Users\用户名\package-lock.json //mapslice支持包下载的地址信息
```

## 下载旧版本

请在@后面填入对应版本号，版本号查阅https://github.com/martinheidegger/mapslice/tags
```bash
npm install mapslice@1.3.0
```

## 命令行（原文说明可以忽视）

安装最新的 [node](http://nodejs.org/) 后，您可以使用 mapslice 作为命令行工具，方法是使用以下命令进行安装：

```bash
npm install mapslice -g
```

还请确保您已安装 [GraphicsMagick](http://www.graphicsmagick.org/README.html) 或 [ImageMagick](http://www.imagemagick.org/script/binary-releases.php) 并在命令行中可用！

满足以上条件后，使用以下命令运行mapslice：

```console
$ mapslice -f test.jpg
```

要获取更多文档，请运行不带参数的 mapslice：

```console
$ mapslice
```

## 脚本使用

请复制以下代码保存js格式放在node_modules文件夹旁边，然后还需要新建bat文件写入以下命令运行bat才能使用

```console
node slice.js
pause
```

注意：以下备注说明和原作者说明有改动

```JavaScript
const MapSlicer = require('mapslice')

// 此处传递的参数等于命令行参数
const mapSlicer = new MapSlicer({
  file: `myImage.jpg`,               // （必填）需要切片的大图片，格式智齿png和jpg
  output: `myImage/{z}/{y}/{x}.png`, // （默认值：从文件路径开始）输出文件模式，输出的格式最好改为jpg，如果是png的话3.0.0版本输出会很占用CPU甚至电脑卡顿什么都做不了
  outputFolder: './output',          // （默认值：从文件路径开始）要用于的输出。使用output或outputFolder，二选一不要同时使用！
  tileSize: 512,                     // （默认值：256）切片后的图片高宽像素大小
  imageMagick: true,                 // （默认值：false）如果（true），则使用ImageMagick而不是GraphicsMagick
  background: '#00000000',           // （默认值：'#FFFFFFF'）用于图块的背景颜色
  tmp: './temp',                     // （默认值：'.tmp'）用于存储辅助文件的临时目录，就是个文件夹命名而已
  parallelLimit: 3,                  // （默认值：5）同时运行的最大并行任务数（警告：进程可能会消耗大量内存！）
  minWidth: 200,                     // 设置瓦片的宽度像素单位，建议和tileSize值一样，你也可以添加或使用minHeight
  skipEmptyTiles: true,              // 跳过所有空瓦片
  bitdepth: 8,                       // (选填) 图片位深度，默认8，当然也有16、24、32实际输出瓦片没有太大影响
  dither: true,                      // (选填) dither抖动，建议改为false，输出的瓦片会有颗粒感，视觉上等于画质差
  colors: 128,                       // (选填) 取决于输出每个瓦片的质量，数值越高处理越慢效果越好，一般需要使用ImageMagick命令获取原图的颜色数量而定
  gm: require('gm'),                 // (选填) 指定GraphicsMagic库的替代方法
  signal: new (require('abort-controller'))().signal // (选填) 中止地图切片过程的信号，就是会在命令窗口显示无法输出的瓦片信息
})

mapSlicer.on('start', (files, options) => console.info(`Starting to process ${files} files.`))
mapSlicer.on('imageSize', (width, height) => console.info(`Image size: ${width}x${height}`))
mapSlicer.on('levels', (levels) => { console.info(`Level Data: ${levels}`) })
mapSlicer.on('warning', err => console.warn(err))
mapSlicer.on('progress', (progress, total, current, path)  => console.info(`Progress: ${Math.round(progress*100)}%`))
mapSlicer.on('end', () => console.info('Finished processing slices.') )
mapSlicer.start().catch(err => console.error(err))
```
最终文件摆放实例
```bash
node_modules
mapslice.js
mapslice.bat
```
## colors获取&参数设置

安装 [ImageMagick](http://www.imagemagick.org/script/binary-releases.php) 后，在原图目录上运行以下命令，即可知道图片颜色数量

```console
magick identify -format "%k" map.png
```

通过（原图颜色数量 / 瓦片数量）*8 = colors应该设置的数值，但是也要根据自身电脑处理情况去设置，因为会占用CPU使用

如果不想复杂计算颜色数量的话，建议设置2000~3000区间，输出的瓦片质量效果较好

### 注意

为了提高性能，mapslice 将每个缩放级别的预缩放版本存储在临时文件夹中，然后从中裁剪。这些临时文件可能会变得非常大，因为它们以低压缩和高质量存储在 sgi 文件中。

## License

[MIT](./LICENSE)

