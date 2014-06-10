# JPush API Java 语言客户端开发包

## 概述

这是 JPush REST API 的 Java 版本封装开发包，是由极光推送官方提供的，一般支持最新的 API 功能。

对应的 REST API 文档：<http://docs.jpush.cn/display/dev/REST+API>

API Javadoc：[API Docs](http://jpush.github.io/jpush-api-java-client/apidocs/)

## 安装

### maven 方式（3.0.0 版本 maven 库里还没有，请直接用 jar 的方式）
将下边的依赖条件放到你项目的 maven pom.xml 文件里。

```
<dependency>
    <groupId>cn.jpush.api</groupId>
    <artifactId>jpush-client</artifactId>
    <version>2.3.1</version>
</dependency>
```
### jar 包方式

请到 [Release页面](https://github.com/jpush/jpush-api-java-client/releases)下载相应版本的发布包。

### 依赖包
* [slf4j](http://www.slf4j.org/) / log4j (Logger)
* [gson](https://code.google.com/p/google-gson/) (Google JSON Utils)
* [guava](https://code.google.com/p/guava-libraries/) (Google Java Utils)

> 其中 slf4j 可以与 logback, log4j, commons-logging 等日志框架一起工作，可根据你的需要配置使用。

如果使用 Maven 构建项目，则需要在你的项目 pom.xml 里增加：

```
		<dependency>
			<groupId>com.google.code.gson</groupId>
			<artifactId>gson</artifactId>
			<version>2.2.4</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.5</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.5</version>
		</dependency>
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.16</version>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>17.0</version>
		</dependency>

		<dependency>
			<groupId>com.squareup.okhttp</groupId>
			<artifactId>mockwebserver</artifactId>
			<version>1.5.4</version>
			<scope>test</scope>
		</dependency>

```

如果不使用 Maven 构建项目，则项目 libs/ 目录下有依赖的 jar 可复制到你的项目里去。

### 构建本项目
建议直接使用 maven，执行命令：
```
maven package
```

## 使用样例

### 推送样例

> 以下片断来自项目代码里的文件：cn.jpush.api.examples.PushExample

```
        JPushClient jpushClient = new JPushClient(masterSecret, appKey);
        PushPayload payload = buildPushObject_all_all_alert();
        LOG.info("Paylaod JSON - " + payload.toString());

        PushResult result = jpushClient.sendPush(payload);
        if (result.isResultOK()) {
            LOG.debug(result.toString());
        } else {
            if (result.getErrorCode() > 0) {
                LOG.warn(result.getOriginalContent());
            } else {
                LOG.debug("Maybe connect error. Retry laster. ");
            }
        }

```

进行推送的关键在于构建一个 PushPayload 对象。以下示例一般的构建对象的用法。

* 快捷地构建推送对象：所有平台，所有设备，内容为 ALERT 的通知。
```
	public static PushPayload buildPushObject_all_all_alert() {
	    return PushPayload.alertAll(ALERT);
	}
```

* 构建推送对象：所有平台，推送目标是别名为 "alias1"，通知内容为 ALERT。
```
    public static PushPayload buildPushObject_all_alias_alert() {
        return PushPayload.newBuilder()
                .setPlatform(Platform.all())
                .setAudience(Audience.alias("alias1"))
                .setNotification(Notification.alert(ALERT))
                .build();
    }
```

* 构建推送对象：平台是 Android，目标是 tag 为 "tag1" 的设备，内容是 Android 通知 ALERT，并且标题为 TITLE。
```
    public static PushPayload buildPushObject_android_tag_alertWithTitle() {
        return PushPayload.newBuilder()
                .setPlatform(Platform.android())
                .setAudience(Audience.tag("tag1"))
                .setNotification(Notification.newBuilder()
                        .addPlatformNotification(AndroidNotification.newBuilder()
                                .setAlert(ALERT)
                                .setTitle(TITLE)
                                .build())
                        .build())
                .build();
    }
```

* 构建推送对象：平台是 iOS，推送目标是 "tag1", "tag_all" 的并集，推送内容同时包括通知与消息 - 通知信息是 ALERT，并且附加字段 from = "JPush"；消息内容是 MSG_CONTENT。通知是 APNs 推送通道的，消息是 JPush 应用内消息通道的。
```
    public static PushPayload buildPushObject_ios_tagAnd_alertWithExtrasAndMessage() {
        return PushPayload.newBuilder()
                .setPlatform(Platform.ios())
                .setAudience(Audience.tag_and("tag1", "tag_all"))
                .setNotification(Notification.newBuilder()
                        .addPlatformNotification(IosNotification.newBuilder()
                                .setAlert(ALERT)
                                .addExtra("from", "JPush")
                                .build())
                        .build())
                 .setMessage(Message.content(MSG_CONTENT))
                 .build();
    }
```

* 构建推送对象：平台是 Andorid 与 iOS，推送目标是 （"tag1" 与 "tag2" 的交集）并（"alias1" 与 "alias2" 的交集），推送内容是 - 内容为 MSG_CONTENT 的消息，并且附加字段 from = JPush。
```
    public static PushPayload buildPushObject_ios_audienceMore_messageWithExtras() {
        return PushPayload.newBuilder()
                .setPlatform(Platform.android_ios())
                .setAudience(Audience.newBuilder()
                        .addAudienceTarget(AudienceTarget.tag("tag1", "tag2"))
                        .addAudienceTarget(AudienceTarget.alias("alias1", "alias2"))
                        .build())
                .setMessage(Message.newBuilder()
                        .setMsgContent(MSG_CONTENT)
                        .addExtra("from", "JPush")
                        .build())
                .build();
    }
```

### 统计获取样例

> 以下片断来自项目代码里的文件：cn.jpush.api.examples.ReportsExample

```
JPushClient jpushClient = new JPushClient(masterSecret, appKey);
ReceivedsResult receivedsResult = jpushClient.getReportReceiveds("1708010723,1774452771");
LOG.debug("responseContent - " + receivedsResult.responseResult.responseContent);
if (receivedsResult.isResultOK()) {
    LOG.info("Receiveds - " + receivedsResult);
} else {
    if (receivedsResult.getErrorCode() > 0) {
        // 业务异常
        LOG.warn("Service error - ErrorCode: "
                + receivedsResult.getErrorCode() + ", ErrorMessage: "
                + receivedsResult.getErrorMessage());
    } else {
        // 未到达 JPush
        LOG.error("Other excepitons - "
                + receivedsResult.responseResult.exceptionString);
    }
}
```


## 版本更新

[Release页面](https://github.com/jpush/jpush-api-java-client/releases) 有详细的版本发布记录与下载。