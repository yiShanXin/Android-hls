
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
 
 
     
     
