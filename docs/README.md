[TOC]
# Clapton用户指南
@(Clapton)
## 简述
**Clapton** 是列表工具，通过页面配置的方式实现快速开发列表。

主要为了快速开发列表，提升开发效率。主要特性：
- **基于mybatis**：可以用mybatis语法标签
- **在线配置**：有web页面提供快速开发
- **列表配置刷新**
- **sql优化**

**Java 运行环境要求：1.6 or later**
**spring：2.5.6以上**
**web环境地址：** 
host 192.168.50.151 uc.vemic.com
开发环境 uc.vemic.com:89(跨境)开发环境 uc.vemic.com:91(开锣)
测试环境 uc.vemic.com:90
## 客户端接入
### pom
``` xml
		<dependency>
			<groupId>com.focustech.oss</groupId>
			<artifactId>clapton</artifactId>
			<version>1.1.2</version>
	    </dependency>
```

最新版本见maven仓库
### 依赖pom(可选)
``` xml
		<dependency>
			<groupId>org.apache.velocity</groupId>
			<artifactId>velocity</artifactId>
			<version>1.7</version>
		</dependency>
		<dependency>
			<groupId>ognl</groupId>
			<artifactId>ognl</artifactId>
			<version>3.1.7</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.3.2</version>
		</dependency>
		<!-- 下载excel 无需则可以不引入 -->
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi</artifactId>
			<version>3.14</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml</artifactId>
			<version>3.14</version>
		</dependency>
```
各业务系统根据自己实际情况自行添加
### web.xml
``` xml
    <servlet>
		<servlet-name>UiListView</servlet-name>
		<servlet-class>com.focustech.oss.clapton.http.UiListServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>UiListView</servlet-name>
		<url-pattern>/uiList/*</url-pattern>
	</servlet-mapping>
```
### candy配置
``` xml
	<bean id="sqlProvider" class="com.focus.candy.support.spring.SpringProvider">
		<property name="serviceInterface" value="com.focustech.oss.clapton.candy.SqlInterf"/>
		<property name="service">
			<bean class="com.focustech.oss.clapton.candy.SqlImpl" />
		</property>
	</bean>
```
candy注册中心配置根据实际项目而定
### spring xml配置
``` xml
	
	<bean id="uiFieSqlSessionFactory" class="com.focustech.oss.clapton.spring.MyBatisSessionFactoryBean">
		<!-- 数据源 多数据源配置多个SqlSessionFactory -->
		<property name="dataSource" ref="cbossDataSource" />
		<!-- 超时时间 -->
		<property name="timeout" value="38" />
		<!-- 配置文件路径 -->
		<property name="configLocation" value="/WEB-INF/sqlmap/config.xml" />
		<!-- sql文件路径 -->
		<property name="mapperLocations" value="/WEB-INF/sqlmap/*/ui-*.xml" />
		<!-- 插件 -->
		<property name="interceptors">
			<list>
				<ref bean="UiStatementHandlerInterceptor"/>
				<ref bean="UiSearchHandlerInterceptor"/>
			</list>
		</property>
	</bean>

	<!-- 插件 -->
	<bean id="UiStatementHandlerInterceptor" class="com.focustech.oss.clapton.ibatis.plugin.StatementHandlerInterceptor">
		<property name="dialect">
			<ref bean="UiOracle9iDialect" />
		</property>
	</bean>
	<bean id="UiSearchHandlerInterceptor" class="com.focustech.oss.clapton.ibatis.plugin.SearchHandlerInterceptor">
		<property name="dialect">
			<ref bean="UiOracle9iDialect" />
		</property>
	</bean>
	<!-- 数据库方言，目前只支持oracle -->
	<bean id="UiOracle9iDialect" class="com.focustech.oss.clapton.dialect.Oracle9iDialect">
	</bean>
	<!-- 系统通用查询参数添加入口 -->
	<bean id="addListParameter" class="com.focustech.cb.csb.uitool.impl.AddListParameter">
	</bean>
	<!-- 权限验证过滤器 -->
	<bean id="authorityFilter" class="com.focustech.cb.csb.uitool.filter.AuthorityFilter">
	</bean>
	<!-- 额外class，可在页面模板中使用 -->
	<bean id="externalClass" class="com.focustech.oss.clapton.extend.ExternalClass">
		<property name="clazzPackage">
		  <map>
		     <entry key="LocaleMessageUtils"><value>com.focustech.cb.csb.web.LocaleMessageUtils</value></entry>
		  </map>
	    </property>
	</bean>
	<!-- 下载excel配置文件 -->
	<bean id="excelUtil" class="com.focustech.oss.clapton.util.ExcelUtil">
		<property name="configPath" value="/WEB-INF/sqlmap/excel-config.xml" />
	</bean>
```
### config配置文件(可选)
``` xml
<?xml version="1.0" encoding="UTF-8" ?> 
    <!DOCTYPE mapper 
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
        	
<configuration >
	<!-- 引入自定义标签节点 -->
	<import name="roleData" class="com.focustech.cb.csb.uitool.sqlNode.RoleDataHandler"/>
	<import name="cbDataRange" class="com.focustech.cb.csb.uitool.impl.CBDataRangeHandler"/>
</configuration> 
```
### excel-config配置文件说明(可选)
``` xml
	<excels>
		<excel id="uiData" class="$clazz" sheetname="$LocaleMessageUtils.convertToLocaleContent($!{entity.title})">
			#foreach( $field in $entity.showFields)
				#if(${field.isHidden}!=1&&$field.fieldName!="_OPER_")
				<field name="$!{field.fieldName}" title="$LocaleMessageUtils.convertToLocaleContent($!{field.showText})"/>
				#end
			#end
		</excel>
	</excels>
```
excel先通过配置文件生成模板，在进行数据的导出
### 日志文件
``` xml
<appender name="cb_oss_clapton" class="org.apache.log4j.RollingFileAppender">
		<param name="File"
			value="/home/cb_oss/log/CSB/cb_oss_clapton.log" />
		<param name="append" value="true" />
		<param name="MaxFileSize" value="100MB" />
		<param name="MaxBackupIndex" value="10" />
		<param name="ImmediateFlush" value="true" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5p %C - %m%n" />
		</layout>
	</appender>
	<appender name="as_cb_oss_clapton" class="org.apache.log4j.AsyncAppender">
		<param name="BufferSize" value="512" />
		<appender-ref ref="cb_oss_clapton" />
	</appender>
	<logger name="clapton" additivity="false">
      <level value="debug"/>
      <appender-ref ref="as_cb_oss_clapton"/>
  	</logger>
```
log4j的配置，logback可同理参考
## 客户端开发
### sql
在对应的sql路径下建xml文件，与mybatis类似
``` xml
	<select id="6710675" resultType="java.util.HashMap"
		parameterType="java.util.HashMap"
		transformer="com.focustech.cb.csb.uitool.impl.NoticeListTransformer.transform">
		<!-- your sql here-->
		select column from table
	</select>
```
配置说明
- **id:**资源号，需要先申请，目前申请在svn Excel中
- **transformer(可选):**转换器，对sql查询的数据进行java转换
### web控制台配置说明
1、访问web系统 点击列表配置
2、点击新建
3、选择你要进行配置的系统以及ip
4、选择sqlId
5、进行页面配置
&nbsp;**entity一些参数说明：**
&nbsp;&nbsp;**标题:**列表顶部显示的标题
&nbsp;&nbsp;**模板:**给各个业务系统配置的id **跨境2 交易1**
&nbsp;&nbsp;**总数限制:**如设置10000，数据超过10000时，只显示10000，不填则不限制
&nbsp;&nbsp;**默认是否查询:** 关闭时，打开列表不进行自动查数据
&nbsp;&nbsp;**默认是否查询数量：**关闭时，打开列表不执行count语句
&nbsp;&nbsp;**查询后是否查询数量：**关闭时，查询不执行count语句
&nbsp;&nbsp;**翻页后是否查询数量：**关闭时，翻页不执行count语句
&nbsp;&nbsp;**刷新是否查询数量：**关闭时，刷新不执行count语句
&nbsp;**field一些参数说明：**
&nbsp;&nbsp;**字段:**sql column，可自定义，最终体现到列表页面的元素name
&nbsp;&nbsp;**表字段:**对应库表哪个字段，即查询时，会按这个值进行查询，若不填，则按sqlxml里标签拼接Sql
&nbsp;&nbsp;**查询类型:**精确(=) 模糊(like) 区间(<= >=) 属于(in)
&nbsp;&nbsp;**下拉:** 参数下拉选项，可以是Sql,可以是java方法，**注意：Sql中有动态参数时不能选Sql&缓存**
&nbsp;&nbsp;**显示控件：**
&nbsp;&nbsp;&nbsp;&nbsp;link：超链接显示	cstlink：自定义链接	checkbox：勾选框 1:勾选0:不勾选	
&nbsp;&nbsp;&nbsp;&nbsp;email：邮箱链接 date：时间格式 maxLength：最大长度
&nbsp;**button一些参数说明：**
&nbsp;&nbsp;**按钮类型:**链接，按钮；链接填url，按钮填javascript函数
6、拖动你想查询或显示的字段布局，完成列表
### 列表功能说明


[TOC]
clapton支持自定义sql标签，开发如下
##sql.xml
在sqlxml中，你首先要定义，你的自定义标签名字、属性、内容
``` xml
<select id="6710735" resultType="java.util.HashMap" parameterType="java.util.HashMap">
       select column from table
       <where>
        <roleData resourceId="6710735">
            1=-1
        </roleData>
       </where>
   </select>
```
如例子中，定义了标签名字roleData，属性resourceId，内容1=-1
##SqlNode
创建一个类实现SqlNode接口
``` java
/**
 * RoleDataSqlNode.java
 * 
 * @author suyuanbo
 */
public class RoleDataSqlNode implements SqlNode {

	//标签resourceId的值
	private Long resourceId;
	//内容
	private String text;

	public RoleDataSqlNode(Long resourceId, String text) {
		this.resourceId = resourceId;
		this.text = text;
	}

	@Override
	public boolean apply(DynamicContext context) {
		String scope;
		if (true) {
			// 可以查询所有对象
			scope = "1=1";
		} else {
			scope = text;
		}
		context.appendSql(scope);
		return true;
	}

}
```
##NodeHandler
创建一个类实现NodeHandler接口
``` java
/**
 * RoleDataHandler.java
 * 
 * @author suyuanbo
 */
public class RoleDataHandler implements NodeHandler {

    public RoleDataHandler() {
    }

    @Override
    public void handleNode(XNode nodeToHandle, List<SqlNode> targetContents, XMLScriptBuilder xmlScriptBuilder) {
	    //标签上自定义的属性
        String resourceId = nodeToHandle.getStringAttribute("resourceId");
        //标签里包含的内容
        String body = nodeToHandle.getStringBody("");
        //自定义SqlNode对象构造
        SqlNode roleDataSqlNode = new RoleDataSqlNode(Long.valueOf(resourceId), body);
        targetContents.add(roleDataSqlNode);
    }

}
```
##config.xml
```xml
<import name="roleData" class="com.focustech.cb.csb.uitool.sqlNode.RoleDataHandler"/>
```


[TOC]

##Transformer 
```java
public class Transformer {


    /**
     * 
     * @param datas 参数对象固定为ArrayList<Object>类型
     */
    public static void transform(ArrayList<Object> datas) {
        //进行datas结果集的装换操作
    }

}

```

##sql.xml
transformer属性填写刚刚建的class.method
```xml
<select id="6710675" resultType="java.util.HashMap"
		parameterType="java.util.HashMap"
		transformer="com.focustech.cb.csb.uitool.impl.NoticeListTransformer.transform">
```


**1、函数结果字段作为查询**
在以往的列表开发中，我们经常看到这样一些sql
``` sql
select *
  from (SELECT P.INB_PLAN_ID,
               P.PLAN_NAME,
               (P.ADD_TIME - 16 / 24) ADD_TIME_PST,
               (P.UPDATE_TIME - 16 / 24) UPDATE_TIME_PST,
               P.ADD_TIME,
               P.UPDATE_TIME,
               P.MSKU_QUANTITY,
               P.TOTAL_QUANTITY,
               P.STATUS,
               P.SITE_STORE_NAME
          FROM CB.CF_INB_PLAN P)
 where ADD_TIME_PST >= to_date('2017-09-24', 'yyyy-MM-dd')
```
我们来看一下执行计划

很明显，并没有走索引

如果我们换一种写法
``` sql
SELECT P.INB_PLAN_ID,
       P.PLAN_NAME,
       (P.ADD_TIME - 16 / 24) ADD_TIME_PST,
       (P.UPDATE_TIME - 16 / 24) UPDATE_TIME_PST,
       P.ADD_TIME,
       P.UPDATE_TIME,
       P.MSKU_QUANTITY,
       P.TOTAL_QUANTITY,
       P.STATUS,
       P.SITE_STORE_NAME
  FROM CB.CF_INB_PLAN P
 where P.ADD_TIME >= to_date('2017-09-24', 'yyyy-MM-dd') + 16 / 24
```
我们来看一下执行计划

走索引！
以往列表工具达不到这种效果。
所以，在clapton中，如遇到这样情况，应该如下写法
```xml
<select id="6900087" resultType="java.util.HashMap" parameterType="java.util.HashMap">
		SELECT P.INB_PLAN_ID,
		  P.PLAN_NAME,
		  (P.ADD_TIME   -16/24) ADD_TIME_PST,
		  (P.UPDATE_TIME-16/24) UPDATE_TIME_PST,
		  P.ADD_TIME,
		  P.UPDATE_TIME,
		  P.MSKU_QUANTITY,
		  P.TOTAL_QUANTITY,
		  P.STATUS,
		  P.SITE_STORE_NAME
		FROM CB.CF_INB_PLAN P
		<where>
			<if test="ge_ADD_TIME_PST != null">
				P.ADD_TIME >= to_date(#{ge_ADD_TIME_PST}, 'yyyy-MM-dd') + 16/24
			</if>
		</where>
	</select>
```
**2、子查询字段作为查询**
看这样一个sql
```sql
select * from (
SELECT TA.TRIPLE_AGREEMENT_ID,
       TA.TRIPLE_AGREEMENT_NO,
       TA.SERVICE_PROVIDER_ID DEPT_NAME,
       (SELECT FULLNAME
          FROM CBOSS.OSS_ADMIN_USER U
         WHERE U.USER_ID = TA.CONTRACTOR_ID) SIGN_NAME,
       TA.SIGN_DATE,
       TA.STATUS
  FROM CBOSS.FIE_TRIPLE_AGREEMENT TA
) where SIGN_NAME like '%张三%'
```
看一下执行计划

再看这样一个sql
```sql
SELECT TA.TRIPLE_AGREEMENT_ID,
       TA.TRIPLE_AGREEMENT_NO,
       TA.SERVICE_PROVIDER_ID DEPT_NAME,
       (SELECT FULLNAME
          FROM CBOSS.OSS_ADMIN_USER U
         WHERE U.USER_ID = TA.CONTRACTOR_ID) SIGN_NAME,
       TA.SIGN_DATE,
       TA.STATUS
  FROM CBOSS.FIE_TRIPLE_AGREEMENT TA
  where  TA.CONTRACTOR_ID ='12000094'
```
看一下执行计划

明显可见，走索引，耗费和基数减少。即使在没有索引的情况下，基数也会减少。

思考：用户需要模糊查询叫'张三'的数据么？
其实用户并不需要模糊查询，想要的只是'张三'这个人的数据。变换思路，先用人员控件把'张三'的id查出，在传入sql查询
