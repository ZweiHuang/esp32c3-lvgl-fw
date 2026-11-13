# 给 ESP32-C3 + 1.28 寸圆屏(GC9A01)编译带 LVGL 的 MicroPython

官方 MicroPython 不带 LVGL，`lvgl-micropython/lvgl_micropython` 也不发布 ESP32-C3
的预编译包，所以这个仓库里只有一个 GitHub Actions 工作流：它在云端把
`ESP32_GENERIC_C3` + `gc9a01` 显示驱动的固件编译出来，产出一个可以直接烧录的
合并镜像 `lvgl_micropy_ESP32_GENERIC_C3-4.bin`。

## 用法（三步）

1. 在 GitHub 上新建一个仓库（名字随意，比如 `esp32c3-lvgl-fw`），把本目录里的
   `.github/workflows/build.yml` 原样上传（注意 `.github` 是隐藏目录，网页上传时
   要在 "Upload files" 里把整个 `build.yml` 拖进去，并在路径里补上
   `.github/workflows/`；用 git 提交更省事）。
2. 打开仓库的 **Actions** 页签，左侧选 "编译 LVGL MicroPython (ESP32-C3 + GC9A01)"，
   点 **Run workflow**。首次跑 15~25 分钟（第二次因为有缓存会快很多）。
3. 跑完后点进这次运行，在页面底部 **Artifacts** 里下载
   `lvgl-micropython-esp32c3-gc9a01`，解压得到 `build/lvgl_micropy_ESP32_GENERIC_C3-4.bin`。

## 烧录

合并镜像从 `0x0` 开始写，会覆盖整块 flash（原有的 MicroPython 和用户文件都会没有）：

```
esptool --chip esp32c3 --port COM3 --baud 921600 --erase-all write_flash 0x0 lvgl_micropy_ESP32_GENERIC_C3-4.bin
```

## 编译参数说明

- `BOARD=ESP32_GENERIC_C3`：厂家文档确认 CS=10 / DC=2 / MOSI=7 / SCLK=6，这颗板子
  没有 RST 和背光控制脚，运行时用 `reset_pin=None, backlight_pin=None` 即可，
  所以不需要自定义板子文件。
- `DISPLAY=gc9a01`：只把这个显示驱动编进去（省 flash；不带触摸，INDEV 一个都不编）。
- `--flash-size=4`：板子是 4MB 内置 flash。
- `--enable-jtag-repl=y --enable-uart-repl=y`：REPL 同时挂在原生 USB(GPIO18/19，
  也就是电脑的 COM3) 和 UART0(GPIO20/21，CH340 那条 COM4) 上，两条 USB 口都能连。
- `montserrat 24/28` 字体：上游默认只开 12/14/16，240x240 圆屏用 16 太小，所以
  sed 打开 24 和 28。
