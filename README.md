
## 第一步 反编译 Apk,找到解密播放器模块
  
 - [enjarify](https://github.com/Storyyeller/enjarify) 生成混搅的jar (来自 Google 推荐)。
 
      
        $ python3 -O -m enjarify.main 51cto.apk
       
       
     
 -  Http 抓包 User-Agent 这个看到(具体查看Http是什么详细解释)
     
 -  通过 JD-GUI 查看 用到了 ExoPlayer 播放器

    ![51cto Code](https://github.com/yiShanXin/Android-hls/blob/master/images/QQ%E6%88%AA%E5%9B%BE20170314112708.png)
     
        
## 第二步 了解 [ExoPlayer](https://github.com/google/ExoPlayer)


![51cto hls](https://github.com/yiShanXin/Android-hls/blob/master/images/QQ%E6%88%AA%E5%9B%BE20170314103643.png)


  * com.google.android.exoplayer2.source.hls.HlsChunkSource
            
            setEncryptionData

            newEncryptionKeyChunk
            
  
  * com.google.android.exoplayer2.source.hls.HlsChunkSource
  
## 第四步 准备 Xposed Hook 环境


    
   -  鉴于有的小伙伴 Android 设备没办法安装 Xposed环境 , 以及需要自己写基于 Xposed Hook代码 ，下面有个视频教程可以达到这个 hook 效果
    
   -  [Genymotion+Xposed](https://vimeo.com/156745941) 视频附带模拟器 安装环境，以及资源下载地址 ,
   > 可能需要<font color=red size=4>梯子(Fan qiang),</font> 文件是在google 服务器
    
   -  通过 Inspeckage 去Hook 方法名称, 减少不懂怎么Hook入门 
    
   # java Hook 代码 片段 (PS:不方便刚入门想Hook同学)所以推荐工具
   
      static void hook(HookItem item, ClassLoader classLoader) {
        try {
            Class<?> hookClass = findClass(item.className, classLoader);

            if (hookClass != null) {

                if (item.method != null && !item.method.equals("")) {
                    for (Method method : hookClass.getDeclaredMethods()) {
                        if (method.getName().equals(item.method) && !Modifier.isAbstract(method.getModifiers())) {
                            XposedBridge.hookMethod(method, methodHook);
                        }
                    }
                } else {
                    for (Method method : hookClass.getDeclaredMethods()) {
                        if(!Modifier.isAbstract(method.getModifiers())) {
                            XposedBridge.hookMethod(method, methodHook);
                        }
                    }
                }

                if (item.constructor) {
                    for (Constructor<?> constructor : hookClass.getDeclaredConstructors()) {
                        XposedBridge.hookMethod(constructor, methodHook);
                    }
                }

            } else {
                log(TAG + "class not found.");
            }
        } catch (Error e) {
            Module.logError(e);
        }
    }
    
    
  经过hook 数据后 , 发现解密key SharePreference 是存在是经过二次混搅过得,so ！！！ 那就看播放界面是怎么处理 key的吧。机续通过 JD-GUI看代码吧！
  
  
  ![51cto player](https://github.com/yiShanXin/Android-hls/blob/master/images/QQ%E6%88%AA%E5%9B%BE20170314103455.png)
  
  
  - com.google.android.exoplayer.core.PlayerActivity
  
  - com.google.android.exoplayer.core.PlayerFragment
     
      onPlayOnLine (处理在线播放)
      
      onPlayLocal (处理本地缓存播放)
      
      playSpecificChapter 



 
找到了关键点，只需要 Hook这个信息，就可以知道 key  = 3SRSS6TS14GR9RN6， 但是 openssl 解密必须是一个32位的十六进制字符串，自己写个字符串转16进制的方法，
 
 
 
     public static String string2HexString(String strPart) {
         StringBuffer hexString = new StringBuffer();
           for (int i = 0; i < strPart.length(); i++) {
              int ch = (int) strPart.charAt(i);
              String strHex = Integer.toHexString(ch);
              hexString.append(strHex);
           }
         return hexString.toString();
    }


 
  - 得到结果  33535253533654533134475239524e36 
  
  - 加密 类型 aes-128-ecb 
  
  ![51cto vlc 加解密](https://github.com/yiShanXin/Android-hls/blob/master/images/QQ%E6%88%AA%E5%9B%BE20170314103657.png)


## 第五步 写个批量解密 ts 切片脚本  如果没有 out_16 目录新建一个文件夹


      `for i in `cat high.m3u8| grep ts `;do openssl aes-128-ecb -v -p  -K 33535253533654533134475239524e36 -d -in $i -out "out_16/"$i;done`


>  shell 批量解析m3u8 来自于同事<font color=red size=4>Feng  同学</font>的代码

> 说明 上图代码段如果不知道这么写的道理 建议好好 opnssl help， 以及源代码处理大致逻辑，这个原理很重要

![51cto vlc 播放](https://github.com/yiShanXin/Android-hls/blob/master/images/QQ%E5%9B%BE%E7%89%8720170314102038.jpg)  
      
直接用 vlc 打开就是可以正常播放了，这次 Hook 之旅完成了.   
      
    
     
 
 
 
    
      
  
    
    
    
 
 
     
     
