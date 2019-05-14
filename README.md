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
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for proj
-- ----------------------------
DROP TABLE IF EXISTS `proj`;
CREATE TABLE `proj` (
  `projId` int(11) NOT NULL AUTO_INCREMENT COMMENT '项目编号',
  `userId` int(11) NOT NULL COMMENT '用户编号',
  `deleteStatus` tinyint(4) NOT NULL COMMENT '是否删除',
  PRIMARY KEY (`projId`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='项目表';

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `userId` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户编号，主键，自增',
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
  PRIMARY KEY (`userId`),
  UNIQUE KEY `uk_user_userCode` (`userCode`) USING BTREE COMMENT '根据用户编码查询唯一用户信息',
  KEY `idx_user_realName` (`realName`) USING BTREE COMMENT '根据姓名查询多条记录'
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户信息表';

-- ----------------------------
-- Records of user
-- ----------------------------
BEGIN;
INSERT INTO `user` VALUES (1, 'ha666', 'ha666HA666', 'T-000000', '暂无', '2019-04-30 12:37:23', '', 0, 1, 1, '2019-04-30 12:37:23', '2019-04-30 12:37:23');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

#### 配置文件
config.yaml 格式：
```yaml
proj:
  name: "ha666-server"
  package_prefix: "gitee.com/ha666/"
  interface_type: "rpcx"    #接口类型：rpcx、gin、go-micro
  server_port: 12985
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

```

#### 执行
- **windows**: go_cli.exe
- **mac**: ./go_cli

#### 手动复制
```go
根据gopath的规则把codes目录里的内容复制到自己的项目中
```
