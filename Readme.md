# mapslice - 图像裁剪和切片工具

[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg)](http://standardjs.com/)

源项目地址：https://github.com/martinheidegger/mapslice

以下为中文说明翻译

mapslice是一种将图像切割成各种缩放级别的切片以用于交互式地图的工具。

有很多可用于高性能查看大图像的 Javascript 工具，但使用这些工具裁剪和切片图像可能会很麻烦。
mapslice会自动检测载入材料支持的图块大小，并创建所有可能的图块，以供常见的 javascript 地图工具
如 [polymaps](http://polymaps.org/), [kartograph](http://kartograph.org/) 或 [PanoJS](http://www.dimin.net/software/panojs/).

## 安装

via npm:

```bash
$ npm install mapslice
```

## 命令行

安装最新的 [node](http://nodejs.org/), 后，您可以使用 mapslice 作为命令行工具，方法是使用以下命令进行安装：

```console
$ npm install mapslice -g
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

```JavaScript
const MapSlicer = require('mapslice')

// The parameters passed here are equal to the command-line parameters
const mapSlicer = new MapSlicer({
  file: `myImage.jpg`,               // (required) Huge image to slice
  output: `myImage/{z}/{y}/{x}.png`, // (default: derived from file path) Output file pattern
  outputFolder: './output',          // (default: derived from file path) Output to be used for. Use either output or outputFolder, not both!
  tileSize: 512,                     // (default: 256) Tile-size to be used
  imageMagick: true,                 // (default: false) If (true) then use ImageMagick instead of GraphicsMagick
  background: '#00000000',           // (default: '#FFFFFFFF') Background color to be used for the tiles. More: http://ow.ly/rsluD
  tmp: './temp',                     // (default: '.tmp') Temporary directory to be used to store helper files
  parallelLimit: 3,                  // (default: 5) Maximum parallel tasks to be run at the same time (warning: processes can consume a lot of memory!)
  minWidth: 200,                     // See explanation about Size detection below
  skipEmptyTiles: true,              // Skips all the empty tiles
  bitdepth: 8,                       // (optional) See http://aheckmann.github.io/gm/docs.html#dither
  dither: true,                      // (optional) See http://aheckmann.github.io/gm/docs.html#bitdepth
  colors: 128,                       // (optional) See http://aheckmann.github.io/gm/docs.html#colors
  gm: require('gm'),                 // (optional) Alternative way to specify the GraphicsMagic library
  signal: new (require('abort-controller'))().signal // (optional) Signal to abort the map slicing process
})

mapSlicer.on('start', (files, options) => console.info(`Starting to process ${files} files.`))
mapSlicer.on('imageSize', (width, height) => console.info(`Image size: ${width}x${height}`))
mapSlicer.on('levels', (levels) => { console.info(`Level Data: ${levels}`) }) // see TypeScript declaration for more details
mapSlicer.on('warning', err => console.warn(err))
mapSlicer.on('progress', (progress, total, current, path)  => console.info(`Progress: ${Math.round(progress*100)}%`))
mapSlicer.on('end', () => console.info('Finished processing slices.') )
mapSlicer.start().catch(err => console.error(err))

## Size detection and scaling

To render the image in its fullest glory, mapslice assumes that you want to preserve the original image-quality and chooses input-size as its starting point from which the quality should be reduced. However: If you have a fixed-size map-user-interface then you might want the smallest image quality to fit this user-interface-design in order to assure that its is beautifully visible. To produce tiles that fit this needs you can use the "minWidth" or "minHeight" property which fits the map to have its lowest size matching exactly your required size:

```console
$ mapslice -f test.jpg -w=1000
```

Will fit the smallest size to be exactly 1000 pixels wide and zoom up from there.

### 注意


为了提高性能，mapslice 将每个缩放级别的预缩放版本存储在临时文件夹中，然后从中裁剪。这些临时文件可能会变得非常大，因为它们以低压缩和高质量存储在 sgi 文件中。

## License

[MIT](./LICENSE)

