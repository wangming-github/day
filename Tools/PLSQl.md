#### PL/SQl中文"?"乱码

新增系统环境变量一：LANG=zh_CN.GBK

新增系统环境变量二：
NLS_LANG=AMERICAN_AMERICA.ZHS16GBK 
或者是 
NLS_LANG=SIMPLIFIED CHINESE_CHINA.ZHS16GBK 

------



##### Oracle数据库建表完整sql

```plsql
-- CREATE TABLE  创建小程序卡片表
create table GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD
(
  ID_GIM_CLOUD_MINIPROGRAM_CARD     VARCHAR2(32) not null,
  TEMPLATE_NO                   VARCHAR2(50) not null,
  DESCRIPTION                   VARCHAR2(200),
  BELONG_TO                     VARCHAR2(50),
  TEMPLATE_PRIO                 VARCHAR2(3),
  LOAD_PERIOD_BEGIN             VARCHAR2(20),
  LOAD_PERIOD_END               VARCHAR2(10),
  SEND_PERIOD_BEGIN             VARCHAR2(10),
  SEND_PERIOD_END               VARCHAR2(10),
  TITLE                         VARCHAR2(20) not null,
  URL                           VARCHAR2(50) not null,
  COVER_URL                     VARCHAR2(50) not null,
  AGENT_FLAG                    char(2) not null,
  ID_GIM_AGENT_GROUP            VARCHAR2(32) not null,
  PUSH_URL_NO                   VARCHAR2(100),
  WX_TEMPLATE                   VARCHAR2(255),
  CREATED_BY                    VARCHAR2(100) not null,
  CREATED_DATE                  DATE not null,
  UPDATED_BY                    VARCHAR2(100) not null,
  UPDATED_DATE                  DATE not null
);


-- Add comments to the table 
comment on table GIM_CLOUD_MINIPROGRAM_CARD
  is '小程序卡片模板表';
-- Add comments to the columns 
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.ID_GIM_CLOUD_MINIPROGRAM_CARD
  is '主键';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.TEMPLATE_NO
  is '模板号';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.DESCRIPTION
  is '模板说明';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.BELONG_TO
  is '所属系统';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.TEMPLATE_PRIO
  is '优先级';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.LOAD_PERIOD_BEGIN
  is '取数时间段开始时间';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.LOAD_PERIOD_END
  is '取数时间段结束时间';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.SEND_PERIOD_BEGIN
  is '发送时间段开始时间';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.SEND_PERIOD_END
  is '发送时间段结束时间';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.TITLE
  is '标题';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.URL
  is '小程序路径';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.COVER_URL
  is '封面url';  
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.AGENT_FLAG
  is '坐席端是否可用';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.ID_GIM_AGENT_GROUP
  is '坐席组id';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.PUSH_URL_NO
  is '推送目的地url编号';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.WX_TEMPLATE
  is '对应的微信模板号';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.CREATED_BY
  is '创建人';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.CREATED_DATE
  is '创建时间';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.UPDATED_BY
  is '最后修改人';
comment on column GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD.UPDATED_DATE
  is '最后修改时间';


--CREAT PUBLIC SYNONYM  创建同义词
CREATE PUBLIC SYNONYM GIM_CLOUD_MINIPROGRAM_CARD FOR GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD;


-- Create/Recreate primary, unique and foreign key constraints   创建索引
CREATE unique INDEX GIMDATA.PK_GIM_CLOUD_MINIPROGRAM_CARD ON GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD (ID_GIM_CLOUD_MINIPROGRAM_CARD) INITRANS 16;


-- 主键
ALTER TABLE GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD ADD CONSTRAINT PK_GIM_CLOUD_MINIPROGRAM_CARD PRIMARY KEY (ID_GIM_CLOUD_MINIPROGRAM_CARD)
USING INDEX GIMDATA.PK_GIM_CLOUD_MINIPROGRAM_CARD;


--创建UM,BUTTON_ID联合索引
create index GIMDATA.INDEX_GRWX_UM_BUTTON_ID ON GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD (UM,BUTTON_ID) INITRANS 16;

-- Grant/Revoke object privileges  授权
GRANT SELECT,DELETE,UPDATE,INSERT ON GIMDATA.GIM_CLOUD_MINIPROGRAM_CARD TO NETSNCHRMSTJS;
GRANT SELECT, INSERT, UPDATE, DELETE ON GIM_CLOUD_MINIPROGRAM_CARD TO GIMLOGTMP,GIMROPR,GIMOPR,PUB_TEST,R_GIMDATA_DML;
GRANT SELECT ON GIM_CLOUD_MINIPROGRAM_CARD TO R_GIMDATA_QRY,DSPDSG,GBDSQP,MISIOCDSG,NEWCHDSG,R_GIMDATA_DEV_QRY,SASCCARDKTL,PA18SHOPESDMKTL,PADHDPSQP;

```



##### 建表语句2:

```plsql
	create table table_name(
        id numner(12),		
        text verchar2(255 CHAR) not null,--char类型,一个汉字占一个长度
        PID varchar2(32 BYTE) NOT NULL,  --byte类型,UTF8一个汉字占大约两个长度
        status number(1) DEFAULT 0 null  --添加默认值 如果为空默认值就为0
    )

    --添加主键
    ALTER TABLE "test"."table_name" ADD PRIMARY KEY ("ID");

    --添加注释
    comment on column table_name.id is '主键';
    comment on column table_name.text is '说明';
    comment on column table_name.status is '状态';


	--主键自增 ,1新建一个序列
	CREATE SEQUENCE cw_bl_id_increment  
	INCREMENT BY 1  
	START WITH 1  
	MAXVALUE 1.0E20  
	MINVALUE 1  
	NOCYCLE  
	CACHE 20  
	NOORDER  

	--主键自增 ,2创建一个触发器
	create or replace trigger 触发器名
	before insert on 表名
	for each row
	begin
	select 序列名.nextval into :new.id from dual;
	end;

	--添加字段
    ALTER TABLE table_name ADD (
        RS_SFTG NUMBER (1),
        RS_TGJE VARCHAR2 (255 CHAR)
    );

	--删除字段
	alter table table_name drop column RS_SFTG ;

```



###### 序列参数说明:

```plsql
CREATE SEQUENCE SEQNAME //序列名字         
INCREMENT BY 1   //每次自增1， 也可写非0的任何整数，表示自增，或自减  
START WITH 1     //以该值开始自增或自减  
MAXVALUE 1.0E20  //最大值；设置NOMAXVALUE表示无最大值  
MINVALUE 1       //最小值；设置NOMINVALUE表示无最大值  
CYCLE or NOCYCLE //设置到最大值后是否循环；  
CACHE 20         //指定可以缓存 20 个值在内存里；如果设置不缓存序列，则写NOCACHE  
ORDER or NOORDER //设置是否按照请求的顺序产生序列  

```

