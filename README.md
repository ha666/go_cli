# go_cli

#### 介绍
代码生成器，可以生成rpcx、gin、go-micro接口的完整解决方案。

#### 规则(坑)
1. 所有的表、字段、索引都必须要写注释
2. 表名不要包含info或list
3. 所有字段不允许为空
4. 必须有主键，而且必须是int、bigint、varchar类型
5. 同一个数据库中，主键名不能重复
6. 表名必须用下划线命名法
7. 字段名必须用驼峰命名法
8. 唯一索引提前创建好
9. deleteStatus字段(可选)用来表示是否删除:0未知，1未删除，2已删除
10. createTime字段(可选)必须有默认值CURRENT_TIMESTAMP
11. updateTime字段(可选)必须有默认值CURRENT_TIMESTAMP，并且设置on update CURRENT_TIMESTAMP
12. 支持的字段类型列表：bit、tinyint、smallint、int、bigint、float、decimal、double、numeric、char、nchar、varchar、nvarchar、text、longtext、mediumtext、enum、set、date、datetime、timestamp
13. 同一数据库中，相同的字段名不允许有不同的类型
14. 验证外键推断规则，不允许有重复的字段名
15. 未完待续……

## 步骤

#### 创建数据库
```sql
create database ha666db default character set utf8mb4 collate utf8mb4_general_ci;
```

#### 创建表、字段、索引
```sql
/*
 Navicat Premium Data Transfer

 Source Server         : 127.0.0.1
 Source Server Type    : MySQL
 Source Server Version : 80016
 Source Host           : 127.0.0.1:3306
 Source Schema         : ha666db

 Target Server Type    : MySQL
 Target Server Version : 80016
 File Encoding         : 65001

 Date: 29/05/2019 16:25:08
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for proj
-- ----------------------------
DROP TABLE IF EXISTS `proj`;
CREATE TABLE `proj` (
  `projId` int(11) NOT NULL AUTO_INCREMENT COMMENT '项目编号',
  `projName` varchar(32) COLLATE utf8mb4_general_ci NOT NULL COMMENT '项目名称',
  `userCode` varchar(32) COLLATE utf8mb4_general_ci NOT NULL COMMENT '用户编码',
  `deleteStatus` tinyint(4) NOT NULL COMMENT '是否删除',
  PRIMARY KEY (`projId`),
  KEY `idx_proj_projName` (`projName`) USING BTREE COMMENT '根据项目名称查询列表'
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='项目表';

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `userCode` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '用户编码，取自钉钉编码',
  `realName` varchar(16) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '姓名',
  `jobNumber` varchar(16) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '员工的工号',
  `jobPosition` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '职位信息',
  `hiredDate` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
  `avatar` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '头像url',
  `gender` tinyint(4) NOT NULL DEFAULT '0' COMMENT '性别，0未知，1男，2女',
  `userType` tinyint(4) NOT NULL DEFAULT '0' COMMENT '用户类型',
  `deleteStatus` tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除状态，0未知，1未删除，2删除',
  `createTime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updateTime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`userCode`) USING BTREE,
  UNIQUE KEY `uk_user_userCode` (`userCode`) USING BTREE COMMENT '根据用户编码查询唯一用户信息',
  KEY `idx_user_realName` (`realName`) USING BTREE COMMENT '根据姓名查询多条记录'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户信息表';

SET FOREIGN_KEY_CHECKS = 1;
```

#### 配置文件
config.yaml 格式：
```yaml
proj:
  package: "github.com/ha666/gin_demo"
  server_port: 9090
  interface_type: "gin"    #接口类型：rpcx、gin、go-micro
  rpcx:
    request_path: "gitee.com/ha666/ha666-common"
  gin:
    is_general_paging: false
  go_micro:
    micro_package: "com.ha666.ha666-server.srv.rbac"
    proto_path: "gitee.com/ha666/ha666-proto"
db:
  name: "ha666db"
  address: "127.0.0.1"
  port: 3306
  account: "root"
  password: "1234567890"
  conn_name: "ha666db"
  max_execution_time: 5000
  tables:
    - name: "user"
      wheres:
        - field: "likeRealName"
          format: "IsHanOrLetterOrNumber"
        - field: "jobNumber"
          format: "IsLetterOrNumber1"
        - field: "jobPosition"
          format: "IsHanOrLetterOrNumber"
        - field: "userType"
          format: "IsNumber"
          required: true
        - field: "startCreateTime"
        - field: "endCreateTime"
    - name: "proj"
        wheres:
          - field: "userId"
  white_tables:
    - name: "proj"
    - name: "user"
```
##### field

- like+字段名，表示这个字段模糊查询
- start+字段名，表示字段的开始时间
- end+字段名，表示字段的结束时间

##### format
值                |正则表达式                            |备注
------------------------|--------------------------------------------------------------|---------------------------
IsTaobaoNick            |(^[\\u4e00-\\u9fa5\\w_—\\-，。…·〔〕（）！@￥%……&*？、；‘“]*$)    | 是否淘宝用户名
IsSubTaobaoNick         |(^[\\u4e00-\\u9fa5\\w_—\\-，。…·〔〕（）！@￥%……&*？、；‘“:]*$)   | 是否淘宝用户名（子帐号）
IsVersion               |(^[0-9.]*$)                                                   | 是否版本号
IsUrl                   |(^[a-zA-z]+://[^\s]*$)                                        | 是否网址
IsNumber                |(^[0-9]*$)                                                    | 是否数字
IsMultipNumber          |(^[0-9,]*$)                                                   | 是否多数字(用逗号间隔)
IsLetterOrNumber        |(^[A-Za-z0-9_]*$)                                             | 判断是否由字母、数字、下划线组成
IsLetterOrNumber1       |(^[A-Za-z0-9_-]*$)                                            | 判断是否由字母、数字、下划线、中杠组成
IsHanOrLetterOrNumber   |^[A-Za-z0-9_\u4e00-\u9fa5-]*$                                 | 是否由汉字、字母、数字、下划线组成
IsGeneralString         |^[A-Za-z0-9_\\-#+./:\u4e00-\u9fa5]*$                          | 是否由汉字、字母、数字、下划线、中杠等组成
IsStandardTime          |                                                              | 是否标准时间格式
IsIPAddress             |                                                              | 是否IPv4地址
IsIntranetIP            |                                                              | 是否内网IP地址
IsEmail                 |^[_a-z0-9-]+(\\.[_a-z0-9-]+)*@[a-z0-9-]+(\\.[a-z0-9-]+)*(\\.[a-z]{2,4})$ | 是否email
IsMobile                |                                                              | 是否手机号
IsAllChineseChar        |                                                              | 字符串是否全中文字符
IsUtf8                  |                                                              | 是否utf-8编码字符串

#### 执行
- **windows**: go_cli.exe
- **mac**: ./go_cli

#### 手动复制
```go
根据gopath的规则把codes目录里的内容复制到自己的项目中
```

#### 第三方包下载地址
- 微云链接：[https://share.weiyun.com/5gHMdKN](https://share.weiyun.com/5gHMdKN?_blank)
- 百度云链接：[https://pan.baidu.com/s/1QbNnWUDWQF3a2abd7f42Xw　提取码：tamp](https://pan.baidu.com/s/1QbNnWUDWQF3a2abd7f42Xw?_blank)
