---
title: Frame Buffer控制嵌入式设备显示屏
date: 2015-07-20 20:03
tags: [Linux kernel,TECH]
show_copyright : true
---
最近一段时间因为工作的原因,开始接触android和linux内核的一些东西.
有时间的话会陆续写一些关于这方面的博客,今天这片blog讲的是frame buffer

## 操作FrameBuffer的一般步骤

```
打开fb0设备文件(这个路径不一定,linux系统是放在/dev下面,嵌入式的系统可能放在/dev/graphics/下面);
用ioctl函数获取显示屏的位深、分辨率等信息,两个重要的结构体类型是fb_fix_screeninfo和fb_var_screeninfo;
mmap函数将FrameBuffer映射到用户空间;
读写FrameBuffer,进行绘图和图片显示等;
解除映射(munmap),关闭/dev/fb设备文件.
```
<!--more-->
## 看程序说话
``` c
#include <stdio.h>
#include <fcntl.h>
#include <stdlib.h>
#include <getopt.h>
#include <sys/ioctl.h>
#include <linux/fb.h>
#include <sys/mman.h>

struct fb_var_screeninfo vinfo;
struct fb_fix_screeninfo finfo;

/*fb0 can be found under /dev/graphics, not /dev */
const char *fb_path = "/dev/graphics/fb0";
static int *mapping = 0 ;

static void render(int xres, int yres, int color, int * mapping)
{
    int x=0;
    const int stride = finfo.line_length / 4;

    int *dest = (int *) (mapping) + vinfo.yoffset * stride + vinfo.xoffset;
    for( x=0; x< xres * yres; x++, dest++)
    {
        *dest = color;
    }
}

int main(int argc, char **argv)
{

    static int Param, fp, screensize;
    opterr = 0;

    fp = open(fb_path, O_RDWR);

    if (fp < 0)
    {
        printf("Can not open %s\n", fb_path);
        return -1;
    }

    if (ioctl(fp, FBIOGET_VSCREENINFO, &vinfo) || ioctl(fp, FBIOGET_FSCREENINFO, &finfo))
    {
        printf("Can not read infomation\n");
        return -1;
    }

    /*get the size of the whole screen in memory*/
    screensize = finfo.smem_len;
    /*map the framebuffer device to memory,and return void* */
    mapping =(int *) mmap(0, screensize, PROT_READ | PROT_WRITE, MAP_SHARED, fp, 0); 

    if ((char *) mapping == MAP_FAILED)
    {
        printf ("Can not map framebuffer device to memory.\n");
        return -1;
    }

    if ((Param = getopt(argc, argv, "rgb")) != -1)
    {
        switch(Param) 
        {
            case 'r':
                render(vinfo.xres, vinfo.yres, 0xffff0000, mapping);
                break;
            case 'g':
                render(vinfo.xres, vinfo.yres, 0xff00ff00, mapping);
                break;
            case 'b':
                render(vinfo.xres, vinfo.yres, 0xff0000ff, mapping);
                break;
        }
    }
    else
    {
        printf("Error: wrong parameter\n");
    }

    /*release the map*/
    munmap(mapping, screensize);
    close(fp);
    return 0;
}
```
### 注释
``` c
fp = open(fb_path, O_RDWR)
```
frame buffer设备文件一般是`/dev/fb0、/dev/fb1`...所以这一步就是打开这样的设备


``` c
if (ioctl(fp, FBIOGET_VSCREENINFO, &vinfo) || ioctl(fp, FBIOGET_FSCREENINFO, &finfo))
```
frame buffer提供了很多ioctl的命令,通过这些命令我们可以获取显示设备的分辨率等信息.
程序最开始声明的两个struct,第一个struct:`fb_var_screeninfo`描述的是显示卡的特性,重要的`xres`,`yres`,`bits_per_pixel`,`xoffset,yoffset`,尤其是`xres_virtual`和`yres_virtual`,博主之前就是被这两个参数摆了一道,后面我们会讲到.
第二个staruct:`fb_fix_screeninfo`描述显示卡的属性,系统运行时不能被修改,里面有`smem_len`等要注意

``` c
mapping =(int *) mmap(0, screensize, PROT_READ | PROT_WRITE, MAP_SHARED, fp, 0):
```
用`mmap`函数将`FrameBuffer`映射到用户空间,`mmap`这个函数可以自行google里面的参数意思

``` c
int *dest = (int *) (mapping) + vinfo.yoffset * stride + vinfo.xoffset
```
这里很重要,之前我也不知道,直接对mapping操作,发现屏幕有时候不会变色,后来查资料看到一篇博文[FB的一些概念](http://blog.csdn.net/yuanlulu/article/details/8621656)
原话就说的很好,这里借鉴一下,感谢博文作者yuanlulu
"其中重要的成员变量`xres`和`yres`定义在显示屏上真实显示的分辨率.而`xres_virtual`和`yres_virtual`是虚拟分辨率,它们定义的是显存分辨率,比如显示屏垂直分辨率是400,而虚拟分辨率是800.这就意味着在显存中存储着800行显示行,但是每次只能显示400行.但是显示哪400行呢?这就需要另外一个成员变量`yoffset`,当`yoffset`＝0时,从显存0行开始显示400行,如果`yoffset`＝30,就从显存31行开始显示400行.实际上这个技 术就是乒乓 buffer."
具体什么是乒乓buffer,感兴趣的可以自己查阅资料,这里就不多讲了.

``` c
munmap(mapping, screensize):
```
程序的最后,解除映射


OK,That's all.
