* 获取新信道地址的接口是：test-api.weclassroom.com/service/chat/newim?i=22&t=1000

### 工作安排

#### VIPX对接励步TV

* 添加多机构支持，兼容以前的目录结构（）,老的资源删除

#### 励步需求

* ffmpeg 转码
* 文件管理，上完课的视频不删，磁盘不够时删除最久没有使用过的文件
* 下载限速
  * 虚拟老师自动下课和下课请求冲突, 机构ID
  * 课程名字




### 虚拟老师

1. 新增API接口（ POST ）：

​	调用阿里云oss转码_API		
​	调用ECS服务器转码_API

```json
// POST
//还没具体确定参数
body={
    record_id:6666
    video_resolution: "720P"
}
```

1. 新增 redis_publish 

```json
//收到 redis 转码 publish
{
    cmd:"transcoding",			//转码
	record_id: 6666,			//
	video_resolution: "720P",	// "1080P" / "720P"   /   "480P"   /  240
	result:  "sucess",   		
	transcode_video_url: "http://...record/4940_22/video_4940_22.flv", 	//转码后的视频url
	type: "oss"					// "oss" / "ecs" 的转码结果
}
```

1. 约课时 redis publish新增字段

```json
{
    "record_id":6666,
	"video_url_list": video_url_list,		//是一个obj
	"video_resolution": "720P"
}

video_url_list={
    "720P":"http://crazycs.com/720p.flv",
    "480P":"http://crazycs.com/480p.flv",
    "240P":"http://crazycs.com/240p.flv"
}
```

1. 视频转码状态存储——sqlite

```sql
CREATE TABLE IF NOT EXISTS video_list (
    record_id INTEGER,			
    resolution varchar(10),
    status varchar(20)
)
```

1. 代码逻辑

```js
收到 redis 约课 publish：
	if video_resolution == undefined  	//兼容以前版本，不传清晰度参数说明是以前的课
		//正常约课

	else if video_resolution in video_url_list	//相应的清晰度已经在 video_url_list 里面，可以直接下载
		// 正常约课
	else
		if 根据 record_id 和 清晰度 查询数据库有无相应的记录			 		  	
			//检查对应清晰度的状态
			if 转码中 
				把 class_id 加入到  redis 的 record_resolution 的list中
			else if 转码成功
				报警，返回约课失败
				//应该不存在这种状态，如果转码成功，约课时 video_url_list 里面就应该包含此 清晰度的 url 了 
			else if 转码超时
            	返回约课失败
            else if 转码失败
            	// 重试转码
                跟新 record_id , resolution 的状态设置为 转码中
                调用阿里云oss转码_API
                把 class_id 加入到  redis 的 record_resolution 的list中
				
		else	//本地没有 record_id 记录
			新增 record_id , resolution 记录到本地 , 并把状态设置为 转码中
			调用阿里云oss转码_API
			把 class_id 加入到  redis 的 record_resolution 的list中
```

```js
收到 redis 转码 publish

	if 根据 record_id 和 清晰度 查询数据库有无相应的记录
		if result =  sucess 	//转码成功	
			从 redis 的 record_resolution 的list中  取出class_id_list  ， 然后进入 正常约课  流程
			然后删除该记录，防止内存溢出	
		else	//转码失败
			if type == OSS 		//OSS失败
				调用ECS服务器转码_API
			else 				// ECS也失败
				从 redis 的 record_resolution 的list中   取出class_id_list  ， 然后进入 回调约课失败
	else:
		// 理论上不会有此情况
		报警 
```

1. 转码超时时间设置



### 推流端

1. record_id 作为视频文件夹和文件名 

   record_id=6666

   ```shell
   /home/video/
   		   6666/
   				 6666.flv			#以前没有 分辨率 参数时的文件名
   				 6666_720P.flv
   				 6666_480P.flv
   ```

2. 文件管理，数据存储在 SQLite 中 ， 当磁盘空间或者视频数量超过  .env 文件的设置后，删除旧文件， 当无旧文件可以删时，新的约课报失败

   ```python
   #视频表，做文件管理用
   class Video_Table(Base):
       __tablename__ = 'video_list'

       id = Column(String(250), primary_key=True)     # record_id
       num_cite = Column(Integer, nullable=True)  # +1 when order , -1 when over class
       site = Column(String(250), nullable=True)  # 存放路径
       update_time = Column(Float, nullable=True) # 最新更新时间，方便LRU算法删除旧视频

   #/class_id ---> video  对应表
   class Class_Video(Base):
       __tablename__ = 'class_list'

       class_id = Column(String(250), primary_key=True)
       video_id = Column(String(250), nullable=False)
       site     = Column(String(250), nullable=True)  # video store file site
   #视频文件的磁盘容量以及数量，每次启动时会扫描磁盘，启动后不会扫描
   class DIR_INFO(Base):
       __tablename__ = 'dir_info'

       dir = Column(String(250), primary_key=True)
       size = Column(Float, nullable=False)
       num  = Column(Integer, nullable=False)
   ```

3. 下载限速

   ```shell
   wget -c --limit-rate=300k http://.......
   ```

4. 配置文件 .env 新增参数

   ```shell
   #wget 命令路径
   wget_path=/usr/bin/wget
   #下载限速
   DOWNLOAD_RATE=1000k

   #max number of video
   MAX_VIDEO_NUM=1000

   #max size of all video ,   G
   MAX_VIDEO_SIZE=10
   ```



### ECS转码

- 下载

  ```
  下载哪个清晰度的视频来转码 ？
  ```

- 转码

  ```shell
  ffmpeg -i 3005.flv -acodec copy -s 320x240 3005_240P.flv
  ```

- 上传













推流测试

```
ECS_PASS:T3BxuVaO8W5OSZNG
ECS_IP:
       101.201.74.194(公)
       10.45.41.129(内)
     端口：4344
ECS_PASS:zLU9HWTcm8hcEcu5    
ECS_IP:
       101.201.154.149(公)
       端口：22
```

