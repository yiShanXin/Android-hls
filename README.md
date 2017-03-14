
## 第一步 反编译 Apk,找到解密播放播放器模块
  
 * [enjarify](https://github.com/Storyyeller/enjarify) 生成混搅的jar。
 
      
        $ python3 -O -m enjarify.main 51cto.apk
       
       
     
 -  Http 抓包 User-Agent 这个看到(具体查看Http是什么详细解释)
     
 -  通过 JD-GUI 查看 用到了 ExoPlayer 播放器
     
        
## 第二步 了解 [ExoPlayer](https://github.com/google/ExoPlayer)



  * com.google.android.exoplayer2.source.hls.HlsChunkSource
            setEncryptionData
            newEncryptionKeyChunk
            
  * com.google.android.exoplayer2.source.hls.HlsChunkSource
  
  
  ## 准备 Xposed Hook 环境
    
    * 鉴于有的小伙伴 Android 设备没办法安装  Xposed环境
    
    * [Genymotion+Xposed](https://vimeo.com/156745941)
    
    * 通过 inspection 去Hook 方法名称, 减少不懂怎么Hook入门
    
    
  经过hook 数据后 , 发现解密key SharePreference 是存在是经过二次混搅过得,so！！！ 那就看播放界面是怎么处理 key的吧。机续通过 JD-GUI看代码吧！
  
  
  
  
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


* 写个批量解密 ts 切片脚本  如果没有 out_16 目录新建一个文件夹

     ----  
     for i in `cat high.m3u8| grep ts `;do openssl aes-128-ecb -v -p  -K 33535253533654533134475239524e36 -d -in $i -out "out_16/"$i;done
     ----
     
     
      
直接用 vlc 打开就是可以正常播放了，这次 Hook 之旅完成了.   
      
    
     
 
 
 
    
      
  
    
    
    
 
 
     
     
