# Use ImageMagick

## 安装

* Windows
    > `scoop install imagemagick`

* Linux
    >`sudo apt-get install imagemagick`

* MacOS
    >`brew install imagemagick`

---

## 用途

通过命令行来实现图片的简单调整操作以及一些批处理作业

---

## 命令(目前版本使用命令必须前置 magick )

> version 7.0.9-19

### 转换格式

* `magick convert example.png example.jpg`
  可以更改特定的格式
* `magick convert example.png -quality 95 example.jpg`
  可以调节图片质量 0-100, imagemagick 认为图片质量默认为92

### 修正分辨率

* `convert example.png -resize 200×100 example.png`
  可以修改图片分辨率, 同时 imagemagick 处理时尽量按照比例缩放,
  如果分辨率比例和原来比例不同, 会 `尽量贴近`, 若希望必须制定分辨率, 则需要在分辨率之后加 `!`
* 