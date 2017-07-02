##继上次[51CTO学院 原理分析](http://github.com/yiShanXin/Android-hls)

这次我们开始新的旅途了.......先来两张效果图


<div >
<img src="https://ooo.0o0.ooo/2017/06/27/59522283eceda.png" width="300" height="460" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/595222dca81c0.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

==准备工作(工具)==

*  enjarify, JD-GUI   反编译 JAR 方便查看部分混搅代码
*  Wireshark  分析数据包
*  tcpdump    Android 端抓包工具


==准备工作(AES-128解密流程)==

*  明确软件 aes-128    加密类型??
*  APP下载视频目录      帮助分析加密类型
*  hooks              (Root后，抓取内存 byte[])



视频下载目录


<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/5952255434253.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

### 1. JD-GUI 代码如下，cbc/ecb 解密都出现了, 而且有一个写死 iv(偏移量)

<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/595223812f36b.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>


<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/595223a5c9227.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/595223a549ab3.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>


### 2. hook 内存地址如下

<div align=center>
<img src="
https://ooo.0o0.ooo/2017/06/27/595229d68661e.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/595224717998c.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

`根据 以上初步确定加密方式 aes-128-ecb `

### 3. 根据hook信息,确定了几组 byte[],反转成16进制字符串

-9, -111, -98, 40, -45, 6, 69, 75, 17, 75, -17, -36, -83, 104, -8, -4

-120,-15,-57,-11,88,22,8,-60,70,-49,-67,39,-19,-28,-22,100


### 4. 调用 openssl 解密转码

> openssl aes-128-ecb -v -p  -K 0f5d633720b150c0c5ffc4cdf3521998  -d -in 35ff61cefd757d45.ts -out 35ff61cefd757d45_out.ts


==经过两个半小时的实验,提示解密失败告终!泪崩😢😢😢😢😢😢.....,后来想着:"如果要实现本地播放，需要模拟 http 服务,这样才能访问呢！".接下来就用到 **tcpdump** 为了保证录制信息不会太多！，手机开启飞行模式，这样即使抓了多余的网络访问，也可以过滤掉错误的包信息==


<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/5952239a4d817.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

### 5. 重新播放手机缓存的视频 -> 抓包取样 -> 导出到电脑

>  ./tcudump -i any -p -s 0 -w /sdcard/capture.pcap

>  adb pull /sdcard/capture.pcap ~/developer/work

### 6. 用 Wireshark 打开,查看http m3u8链接！以及动态拼接解密(key)token 链接，在查看 JD-GUI 部分代码!初步解密方式是 aes-128-cbc,并且找到 IV

(IV 探索)接下来的问题通过 ==C Arrays== 显示知道正确的解密key! 0f5d633720b150c0c5ffc4cdf3521998 😉😉😉(放松一下!),然后再去找IV(偏移量),根据之前大量 hook(aes对象)信息，iv =102,108,100,115,106,102,111,100,97,115,106,105,102,117,100,115 大量出现，


<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/595225542fb94.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/59522534efa1f.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/59522534efa1f.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>



<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/5952254a9e420.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

解密 key

<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/595225542daaf.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>

> http://127.0.0.1:54652?originHost=u003d5pmp5rGl46iv4ryv54205r2y5oWn5pSv5pWt55Ws5oW05pWk4ryw4r2B5rmk54mv5qWk4r2k5oW05oSv5o2u4rmj5r2t4rmv54Gl5riu5rWv5r2j4r225qWk5pWv4r2t5pmf44yw45Sv44Sw44S245yu5oWs5rCv5q2l56Su5qG45qyA


### 7. 通过 byte 转16进制32位字符串，得到  666C64736A666F6461736A6966756473,写一个批量转换工具，看着控制台正确输出的信息(终于结束了，我的小心脏)


> for i in `cat down.m3u8| grep ts `;do sudo openssl aes-128-cbc -v -p -iv  666C64736A666F6461736A6966756473 -K 0f5d633720b150c0c5ffc4cdf3521998 -d -in $i -out "test16/"$i;done



<div align=center>
<img src="https://ooo.0o0.ooo/2017/06/27/5952227a3a84c.png" width="660" height="400" alt="Screen Shot 2017-06-15 at 10.26.26 AM.png"/>
</div>
































