```javascript
socket.userId = data.userid;
socket.username = data.username;
socket.avatar = helpers.checkBase64Avatar(data.avatar, env.default_avatar);
socket.instId = data.instid;
socket.room = data.room;
socket.lid = roomArr[0];
socket.instId = roomArr[1];
socket.role = data.role;
socket.playmode = data.playmode;
socket.device = device;

```



17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { username: '梁小娜',
17|chen-nc |   room: '4572_22',
**17|chen-nc |   avatar: 'http://opas-file.b0.upaiyun.com/image/20161220/89c0378441a56b0401f22b31f7314f17.jpg',**
17|chen-nc |   userid: '91947',
17|chen-nc |   role: '1',
17|chen-nc |   playmode: 1,
17|chen-nc |   device: 1,
17|chen-nc |   instid: '22',
17|chen-nc |   classmode: 1000 }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | ====进房＝＝＝ { username: '梁小娜',
17|chen-nc |   room: '4572_22',
17|chen-nc |   avatar: 'http://opas-file.b0.upaiyun.com/image/20161220/89c0378441a56b0401f22b31f7314f17.jpg',
17|chen-nc |   userid: '91947',
17|chen-nc |   role: '1',
17|chen-nc |   playmode: 1,
17|chen-nc |   device: 1,
17|chen-nc |   instid: '22',
17|chen-nc |   classmode: 1000 }
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 0,
17|chen-nc |   uuid: '91947_1_0_1514273707370',
17|chen-nc |   mtype: 'class',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"appid":"","className":"FCS2.1 自定义分组测试","classType":"1000","cmdtype":"start","roomId":"4572_22","userId":"91947"}',
17|chen-nc |   save: '1' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 1,
17|chen-nc |   uuid: '91947_1_1_1514273707400',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"join", "actorId": "91947", "actorName": "梁小娜", "actorType": "teacher", "appId": "", "roomId": "4572_22", "version": 1 }',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 2,
17|chen-nc |   uuid: '91947_1_2_1514273707408',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"addStream", "appId": "", "roomId": "4572_22", "actorId": "91947", "streamUrl": "rtmp://123.56.141.193:1935/live_zby/_4572_22_91947_3", "streamType": "3", "playMode": "1",  "actorType": "teacher", "actorName": "梁小娜", "version": 1}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 3,
17|chen-nc |   uuid: '91947_1_3_1514273707420',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"requestID": 0,"needAck": 0,"cmd": "openDoc","docID": "a0f39e9479d67a9c7133a8437ef44846","docType": 4,"docUrl": "0.blank","docName": "whiteboard153448","onTesting": 0,"targetUser": ""}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 4,
17|chen-nc |   uuid: '91947_1_4_1514273707427',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"requestID": 1,"needAck": 0,"cmd": "openDoc","docID": "6009076d5c81d5db5791f67158c8356d","docType": 5,"docUrl": "https://tv.firstleap.cn/gulp-3/index.html","docName": "index.html","onTesting": 0,"targetUser": ""}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 5,
17|chen-nc |   uuid: '91947_1_5_1514273707434',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"requestID": 2,"needAck": 0,"cmd": "gotoDocPage","docID": "6009076d5c81d5db5791f67158c8356d","pageIndex": 1,"stepIndex": -1,"onTesting": 0,"targetUser": ""}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 6,
17|chen-nc |   uuid: '91947_1_6_1514273707439',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"requestID": 3,"needAck": 0,"cmd": "gotoDocPage","docID": "6009076d5c81d5db5791f67158c8356d","pageIndex": 1,"stepIndex": 0,"onTesting": 0,"targetUser": ""}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 7,
17|chen-nc |   uuid: '91947_1_7_1514273707444',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"requestID": 4,"needAck": 0,"cmd": "gotoDocPage","docID": "6009076d5c81d5db5791f67158c8356d","pageIndex": 1,"stepIndex": 0,"onTesting": 0,"targetUser": ""}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | { table: 'institution', where: { id: 22, join_mode: 1 } }
17|chen-nc | =====teacher=addStream===== { api: 'addStream',
17|chen-nc |   appId: '',
17|chen-nc |   roomId: '4572_22',
17|chen-nc |   actorId: '91947',
17|chen-nc |   streamUrl: 'rtmp://123.56.141.193:1935/live_zby/_4572_22_91947_3',
17|chen-nc |   streamType: '3',
17|chen-nc |   playMode: '1',
17|chen-nc |   actorType: 'teacher',
17|chen-nc |   actorName: '梁小娜',
17|chen-nc |   version: 1 }
17|chen-nc | ---------redis---key----->>>> /control4572_22:streams
17|chen-nc | =====teacher=openDoc===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 0,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'openDoc',
17|chen-nc |      docID: 'a0f39e9479d67a9c7133a8437ef44846',
17|chen-nc |      docType: 4,
17|chen-nc |      docUrl: '0.blank',
17|chen-nc |      docName: 'whiteboard153448',
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' }
17|chen-nc | =====teacher=openDoc===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 1,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'openDoc',
17|chen-nc |      docID: '6009076d5c81d5db5791f67158c8356d',
17|chen-nc |      docType: 5,
17|chen-nc |      docUrl: 'https://tv.firstleap.cn/gulp-3/index.html',
17|chen-nc |      docName: 'index.html',
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' }
17|chen-nc | =====teacher=gotoDocPage===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 2,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'gotoDocPage',
17|chen-nc |      docID: '6009076d5c81d5db5791f67158c8356d',
17|chen-nc |      pageIndex: 1,
17|chen-nc |      stepIndex: -1,
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' } =====docPagesKey==== /control4572_22:docpages
17|chen-nc | =====teacher=gotoDocPage===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 3,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'gotoDocPage',
17|chen-nc |      docID: '6009076d5c81d5db5791f67158c8356d',
17|chen-nc |      pageIndex: 1,
17|chen-nc |      stepIndex: 0,
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' } =====docPagesKey==== /control4572_22:docpages
17|chen-nc | =====teacher=gotoDocPage===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 4,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'gotoDocPage',
17|chen-nc |      docID: '6009076d5c81d5db5791f67158c8356d',
17|chen-nc |      pageIndex: 1,
17|chen-nc |      stepIndex: 0,
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' } =====docPagesKey==== /control4572_22:docpages
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 8,
17|chen-nc |   uuid: '91947_1_8_1514273747619',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"requestID": 5,"needAck": 0,"cmd": "gotoDocPage","docID": "6009076d5c81d5db5791f67158c8356d","pageIndex": 2,"stepIndex": 0,"onTesting": 0,"targetUser": ""}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | =====teacher=gotoDocPage===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 5,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'gotoDocPage',
17|chen-nc |      docID: '6009076d5c81d5db5791f67158c8356d',
17|chen-nc |      pageIndex: 2,
17|chen-nc |      stepIndex: 0,
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' } =====docPagesKey==== /control4572_22:docpages
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 9,
17|chen-nc |   uuid: '91947_1_9_1514273748901',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"requestID": 6,"needAck": 0,"cmd": "gotoDocPage","docID": "6009076d5c81d5db5791f67158c8356d","pageIndex": 3,"stepIndex": 0,"onTesting": 0,"targetUser": ""}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | =====teacher=gotoDocPage===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 6,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'gotoDocPage',
17|chen-nc |      docID: '6009076d5c81d5db5791f67158c8356d',
17|chen-nc |      pageIndex: 3,
17|chen-nc |      stepIndex: 0,
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' } =====docPagesKey==== /control4572_22:docpages
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 10,
17|chen-nc |   uuid: '91947_1_10_1514273753746',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"cmd": "docSendMessage","docID": "6009076d5c81d5db5791f67158c8356d","content":{"msg":"start.test", "body":{"docid":"6009076d5c81d5db5791f67158c8356d","tid":2,"starttime":1514273753730,"timeout":9}}}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | =====teacher=start.test===== 2
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 11,
17|chen-nc |   uuid: '91947_1_11_1514273762753',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"cmd": "docSendMessage","docID": "6009076d5c81d5db5791f67158c8356d","content":{"msg":"stop.test", "body":{"docid":"6009076d5c81d5db5791f67158c8356d","tid":2}}}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 12,
17|chen-nc |   uuid: '91947_1_12_1514273767063',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"requestID": 7,"needAck": 0,"cmd": "gotoDocPage","docID": "6009076d5c81d5db5791f67158c8356d","pageIndex": 4,"stepIndex": 0,"onTesting": 0,"targetUser": ""}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | =====teacher=gotoDocPage===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 7,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'gotoDocPage',
17|chen-nc |      docID: '6009076d5c81d5db5791f67158c8356d',
17|chen-nc |      pageIndex: 4,
17|chen-nc |      stepIndex: 0,
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' } =====docPagesKey==== /control4572_22:docpages
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 13,
17|chen-nc |   uuid: '91947_1_13_1514273767744',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc", "command": {"requestID": 8,"needAck": 0,"cmd": "gotoDocPage","docID": "6009076d5c81d5db5791f67158c8356d","pageIndex": 5,"stepIndex": 0,"onTesting": 0,"targetUser": ""}, "version": "1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | =====teacher=gotoDocPage===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 8,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'gotoDocPage',
17|chen-nc |      docID: '6009076d5c81d5db5791f67158c8356d',
17|chen-nc |      pageIndex: 5,
17|chen-nc |      stepIndex: 0,
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' } =====docPagesKey==== /control4572_22:docpages
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 14,
17|chen-nc |   uuid: '91947_1_14_1514273794459',
17|chen-nc |   mtype: 'class',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"appid":"","cmdtype":"end","roomId":"4572_22","userId":"91947"}',
17|chen-nc |   save: '1' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 15,
17|chen-nc |   uuid: '91947_1_15_1514273794509',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"removeStream", "streamUrl": "rtmp://123.56.141.193:1935/live_zby/_4572_22_91947_3"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | =====teacher=end=====
17|chen-nc | { table: 'institution', where: { id: 22, join_mode: 1 } }
17|chen-nc | =====teacher=removeStream===== { api: 'removeStream',
17|chen-nc |   streamUrl: 'rtmp://123.56.141.193:1935/live_zby/_4572_22_91947_3' }
17|chen-nc | /control4572_22:streams