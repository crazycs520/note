17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { mtype: 'stream',
17|chen-nc |   userid: 91947,
17|chen-nc |   role: 1,
17|chen-nc |   playmode: 1,
17|chen-nc |   classmode: 1000,
17|chen-nc |   message: '{"api":"join","actorId":91947,"actorName":"梁小娜","actorType":"teacher","appId":"","roomId":"4670_22","version":1}',
17|chen-nc |   unixtime: 1514282742123,
17|chen-nc |   index: 0 }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | ====进房＝＝＝ { mtype: 'stream',
17|chen-nc |   userid: 91947,
17|chen-nc |   role: 1,
17|chen-nc |   playmode: 1,
17|chen-nc |   classmode: 1000,
17|chen-nc |   message: '{"api":"join","actorId":91947,"actorName":"梁小娜","actorType":"teacher","appId":"","roomId":"4670_22","version":1}',
17|chen-nc |   unixtime: 1514282742123,
17|chen-nc |   index: 0 }
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 0,
17|chen-nc |   uuid: '9194710_1514282742143',
17|chen-nc |   mtype: 'class',
17|chen-nc |   userid: 91947,
17|chen-nc |   role: 1,
17|chen-nc |   classmode: 1000,
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"appid":"","roomId":"4670_22","userId":91947,"className":"FCS2.1  定义分组测试","classType":1000,"cmdtype":"start"}',
17|chen-nc |   save: '1' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 1,
17|chen-nc |   uuid: '9194711_1514282742144',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   userid: 91947,
17|chen-nc |   role: 1,
17|chen-nc |   classmode: 1000,
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"addStream","actorId":91947,"actorName":"梁小娜","actorType":"teacher","roomId":"4670_22","streamUrl":"rtmp:// 123.56.141.193:1935/live_zby/_4670_22_91947_3","streamType":3,"playMode":1,"version":1}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 2,
17|chen-nc |   uuid: '9194712_1514282742146',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   userid: 91947,
17|chen-nc |   role: 1,
17|chen-nc |   classmode: 1000,
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc","command":{"requestID":0,"needAck":0,"cmd":"openDoc","docID":"cf93f25de784d9561ee5668e00f57d0b","docUrl":"https://tv.firstleap.cn/gulp-3/index.html","onTesting":0,"targetUser":""},"version":"1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 3,
17|chen-nc |   uuid: '9194713_1514282742148',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   userid: 91947,
17|chen-nc |   role: 1,
17|chen-nc |   classmode: 1000,
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc","command":{"requestID":1,"needAck":0,"cmd":"gotoDocPage","docID":"cf93f25de784d9561ee5668e00f57d0b","pageIndex":1,"stepIndex":0,"onTesting":0,"targetUser":""},"version":"1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | { table: 'institution', where: { id: 0, join_mode: 1 } }
17|chen-nc | =====teacher=addStream===== { api: 'addStream',
17|chen-nc |   actorId: 91947,
17|chen-nc |   actorName: '梁小娜',
17|chen-nc |   actorType: 'teacher',
17|chen-nc |   roomId: '4670_22',
17|chen-nc |   streamUrl: 'rtmp:// 123.56.141.193:1935/live_zby/_4670_22_91947_3',
17|chen-nc |   streamType: 3,
17|chen-nc |   playMode: 1,
17|chen-nc |   version: 1 }
17|chen-nc | ---------redis---key----->>>> /control0:streams
17|chen-nc | =====teacher=openDoc===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 0,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'openDoc',
17|chen-nc |      docID: 'cf93f25de784d9561ee5668e00f57d0b',
17|chen-nc |      docUrl: 'https://tv.firstleap.cn/gulp-3/index.html',
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' }
17|chen-nc | =====teacher=gotoDocPage===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 1,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'gotoDocPage',
17|chen-nc |      docID: 'cf93f25de784d9561ee5668e00f57d0b',
17|chen-nc |      pageIndex: 1,
17|chen-nc |      stepIndex: 0,
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' } =====docPagesKey==== /control0:docpages
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 4,
17|chen-nc |   uuid: '9194714_1514282744925',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   userid: 91947,
17|chen-nc |   role: 1,
17|chen-nc |   classmode: 1000,
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"onlineDoc","command":{"requestID":2,"needAck":0,"cmd":"gotoDocPage","docID":"cf93f25de784d9561ee5668e00f57d0b","pageIndex":2,"stepIndex":0,"onTesting":0,"targetUser":""},"version":"1.0"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | =====teacher=gotoDocPage===== { api: 'onlineDoc',
17|chen-nc |   command: 
17|chen-nc |    { requestID: 2,
17|chen-nc |      needAck: 0,
17|chen-nc |      cmd: 'gotoDocPage',
17|chen-nc |      docID: 'cf93f25de784d9561ee5668e00f57d0b',
17|chen-nc |      pageIndex: 2,
17|chen-nc |      stepIndex: 0,
17|chen-nc |      onTesting: 0,
17|chen-nc |      targetUser: '' },
17|chen-nc |   version: '1.0' } =====docPagesKey==== /control0:docpages
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 5,
17|chen-nc |   uuid: '9194715_1514282747117',
17|chen-nc |   mtype: 'stream',
17|chen-nc |   userid: 91947,
17|chen-nc |   role: 1,
17|chen-nc |   classmode: 1000,
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"api":"removeStream","streamUrl":"rtmp:// 123.56.141.193:1935/live_zby/_4670_22_91947_3"}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
17|chen-nc | { index: 5,
17|chen-nc |   uuid: '9194715_1514282747117',
17|chen-nc |   mtype: 'class',
17|chen-nc |   userid: 91947,
17|chen-nc |   role: 1,
17|chen-nc |   classmode: 1000,
17|chen-nc |   targetid: '',
17|chen-nc |   message: '{"appid":"","cmdtype":"end","roomId":"4670_22","userId":91947}',
17|chen-nc |   save: '0' }
17|chen-nc | <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
17|chen-nc | =====teacher=removeStream===== { api: 'removeStream',
17|chen-nc |   streamUrl: 'rtmp:// 123.56.141.193:1935/live_zby/_4670_22_91947_3' }
17|chen-nc | /control0:streams
17|chen-nc | =====teacher=end=====
17|chen-nc | { table: 'institution', where: { id: 0, join_mode: 1 } }