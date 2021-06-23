# 签名方式
1.1. 将待签名参数按ASCII码正序排序

1.2. 将参数使用url键值对格式拼接成字符串

1.3. 将1.2中得到的字符串拼接上 key=机构密钥

1.4. 将1.3中得到的字符串用md5（32位）加密

1.5. 将1.4中得到的字符串转小写

**以php为例：**
```php
   // 传递的参数
   $params = 
     [
        "b" => 1,
        "a" => 2,
        "c" => 3,
        "token" => "分配给机构的身份认证值，每次必传"
     ];
   
   // ascii排序    
   ksort($params);
   
   // 加入key
   $params["key"] = "csw123456";
   
   // 拼接字符串
   $str = http_build_query($params);
   
   // 生成签名
   $sign = strtolower(md5(urldecode($str)));
   
   // get请求的最终url
   $url = "https://api.chaosw.com/xxx?b=1&a=2&c=3&token=xxx&sign=$sign";
   
   // post请求的最终参数
   $params = 
      [
        "b" => 1,
        "a" => 2,
        "c" => 3,
        "token" => "xxx",
        "sign" => $sign
      ];
```
# 全局参数及要求
##### 2.1 请求域名为 https://api.chaosw.com
##### 2.2 每次请求，token及sign是必须传入的参数，接口文档中不再单独将这两个参数列出
##### 2.3 响应的数据结构体为json对象
```json
 {
    "code": "响应码，200-成功，500-失败",
    "data": "响应的数据，json数组、对象或空值",
    "message": "响应提示信息"
 }
```

# 接口
## 1、注册
##### URL： */agency.api/register*

**请求方式：** POST

**请求参数：**
```json
{
    "username": "用户名,string，与手机号二选一必传",
    "mobile": "手机号，string，与用户名二选一必传",
    "realname": "学员真实姓名，string",
    "uid": "该学员在机构数据库中的主键id，string,必传",
    "password": "密码，string"
}
```

**返回值：**
```json
{
    "code": 200,
    "message": "请求成功!",
    "data": [
        {
            "icon": "考试图标url,string",
            "id": "考试id，int",
            "name": "考试名，string",
            "pid": "考试父id，int",
            "orderno": "排序，int或null",
            "is_org": "是否为自建考试，int，是-机构id，否-0",
            "status": "状态，1或0",
            "children": [
                {
                    "id": "考试id，int",
                    "name": "考试名，string",
                    "pid": "考试父id，int",
                    "orderno": "排序，int或null",
                    "is_org": "是否为自建考试，int，是-机构id，否-0",
                    "status": "状态，1或0"
                }
            ]
        }
    ]
}
```
**说明**

``用户名、手机号全系统唯一，如果用户名或手机号在其他机构注册过了，不能再注册，请更换``

## 2、登陆
##### URL： */agency.old_api/login*

**请求方式：** POST

**请求参数：**
```json
{
    "uid": "用户名或手机号,string，必传",
    "password": "密码，string，必传"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "请求成功!",
    "data": {
            "id": "学员id,int",
            "user_name": "学员用户名,string",
            "mobile": "学员手机号，string",
            "nick_name": "学员昵称，string",
            "email": "学员邮箱，string",
            "head_pic": "学员头像，string",
            "agency_contact": "机构联系方式，string",
            "login_time": "最近登录时间，string",
            "plaintextPassword": "学员明文密码，string"
        }
}
```
## 3、修改密码
##### URL： */agency.old_api/updatePass*

**请求方式：** POST

**请求参数：**
```json
{
    "uid": "注册时机构同步过来的学员id主键，万一注册时没有传过来，该值可传学员的用户名或手机号，string，必传",
    "pass": "密码，string，必传"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "密码修改成功"
}
```

## 4、获取考试信息
##### URL： */agency.old_api/getExams*

**请求方式：** POST

**请求参数：**
```json
{
    
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "请求成功!",
    "data": {
            "id": "考试id,int",
            "name": "考试名,string"
        }
}
```
## 5、获取考试下的科目
##### URL： */agency.old_api/getExamsToSubject*

**请求方式：** POST

**请求参数：**
```json
{
    "examId":"考试id，int，必传"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "请求成功!",
    "data": [{
            "subject_id": "科目id,int",
            "subject_name": "科目名,string",
            "sort": "排序，int"
        }]
}
```
## 6、获取考试下的班级
##### URL： */agency.old_api/getExamClass*

**请求方式：** POST

**请求参数：**
```json
{
    "examId":"考试id，int，必传"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "请求成功!",
    "data": [{
            "lesson_id": "试听课时id,int，没有时为0或null",
            "class_id": "班级id,int",
            "class_name": "班级名称，string",
            "type": "产品类型，string，固定值-class",
            "subject_id": "班级对应的科目id，int",
            "img": "班级缩略图，string",
            "type_id": "班级类型id，int",
            "year": "班级年份，int",
            "validity": "班级有效期，int，为时间戳则是固定有效期，为小于100的数则是表示自开通日期后生效的月数",
            "lesson_num": "课时数量，int",
            "guide_price": "指导价，double",
            "sort": "排序，int",
            "desc": "班级介绍，string"
    }]
}
```
## 7、获取考试下的套餐
##### URL： */agency.old_api/getExamPackage*

**请求方式：** POST

**请求参数：**
```json
{
    "examId":"考试id，int，必传",
    "page": "页码，默认为1，非必传"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "请求成功!",
    "data": [{
            "lesson_id": "试听课时id,int，没有时为0或null",
            "exam_id": "考试id,int",
            "package_id": "套餐id，int",
            "pacakge_name": "套餐名称，string",
            "type": "产品类型，string，固定值为package",
            "img": "套餐缩略图，string",
            "type_id": "类型id，int",
            "year": "班级年份，int",
            "validity": "班级有效期，int，为时间戳则是固定有效期，为小于100的数则是表示自开通日期后生效的月数",
            "lesson_num": "课时数量，int",
            "guide_price": "指导价，double",
            "sort": "排序，int",
            "desc": "套餐介绍，string",
            "classes": ["套餐包含的班级id，int"]
    }]
}
```
## 8、获取老师
##### URL： */agency.old_api/getTeacher*

**请求方式：** POST

**请求参数：**
```json
{
    
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "请求成功!",
    "data": [{
            "teacher_id": "老师id,int",
            "teacher_name": "老师名,string",
            "title": "老师头衔，string",
            "img": "老师头像，string",
            "desc": "老师介绍，string",
            "exams": [{
                "exam_id": "主讲考试id，int", 
                "exam_name": "主讲考试名，string"
            }],
            "subjects": ["主讲科目id，int"],
            "main_subject": "主讲科目名，string"
    }]
}
```
## 9、获取班级下课时
##### URL： */agency.old_api/getClassLesses*

**请求方式：** POST

**请求参数：**
```json
{
    "classId":"班级id，int，必传",
    "type": "一个班级下有多年份的课时，如只需最近年份课时该值传1，非必传"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "请求成功!",
    "data": [{
            "lesson_id": "课时id,int",
            "lesson_name": "课时名称,string",
            "class_id": "课时对应的班级id，int",
            "teacher_id": "主讲老师id，int",
            "duration": "课时时长，int，秒数",
            "is_free": "是否免费，1-免费，0-收费",
            "sort": "排序，int",
            "year": "班级年份，int"
    }]
}
```
## 10、获取课时播放H5页面
##### URL： */agency.old_api/get_video*

**请求方式：** POST

**请求参数：**
```json
{
    "lesson_id":"课时id，int，必传",
    "uid": "注册时机构同步过来的学员id主键，万一注册时没有传过来，该值可传学员的用户名或手机号，string，必传"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "请求成功!",
    "data": {
            "video_url": "视频播放H5页面地址,string"
    }
}
```
**说明**

``web对接可用iframe嵌入返回的视频播放H5页面地址，app对接可用webview嵌入返回的视频播放H5页面地址，有过期时间，请勿保存，需要实时获取``

## 11、获取课时讲义
##### URL： */agency.old_api/getPptUrl*

**请求方式：** POST

**请求参数：**
```json
{
    "lesson_id":"课时id，int，必传",
    "uid": "注册时机构同步过来的学员id主键，万一注册时没有传过来，该值可传学员的用户名或手机号，string，必传"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "请求成功!",
    "data": "讲义下载链接地址"
}
```
**说明**

``讲义下载地址有过期时间，请勿保存，需要实时获取``

## 12、开课
##### URL： */agency.api/operate_course*

**请求方式：** POST

**请求参数：**
```json
{
    "id": "班级或套餐id，int,必传",
    "type": "产品类型，id参数为班级id时固定值为class，id参数为套餐id时固定值为package，string,必传",
    "uid": "注册时机构同步过来的学员id主键，万一注册时没有传过来，该值可传学员的用户名或手机号"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "开课成功",
    "data": null
}
```
## 13、退课申请
##### URL： */agency.api/refund_course*

**请求方式：** POST

**请求参数：**
```json
{
    "id": "班级或套餐id，int,必传",
    "type": "产品类型，id参数为班级id时固定值为class，id参数为套餐id时固定值为package，string,必传",
    "uid": "注册时机构同步过来的学员id主键，万一注册时没有传过来，该值可传学员的用户名或手机号",
    "remark": "退课申请的理由描述"
}
```

**返回值：**

```json
{
    "code": 200,
    "message": "退课申请成功",
    "data": null
}
```

## 14、获取学员已购班级
##### URL： */agency.user/classes*
**请求方式：** POST

**请求参数：**
```json
{
    "uid": "注册时机构同步过来的学员id主键，万一注册时没有传过来，该值可传学员的用户名或手机号",
    "exam_id": "考试id，int,非必传",
    "subject_id": "科目id，int，非必传"
}
```
**返回值：**
```json
{
    "code": 200,
    "message": "请求成功",
    "data": [
        {
            "class_id": "班级id，int",
            "class_name": "班级名称，string",
            "exam_id": "考试id，int",
            "exam_name": "考试名，string",
            "target_type_name": "班型，string",
            "year": "年份，int",
            "validity": "到期时间，YYYY-MM-dd，string",
            "lesson_num": "课时数,int",
            "rating": "评分，int",
            "member_num": "已购人数，int",
            "desc": "简介，string",
            "img": "班级缩略图，string",
            "lesson_id": "试听课时id，int"
        }
    ]
}
```