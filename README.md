

![oasis sdk](https://oasgames.com/pc/en/dist/images/logo_en.png)

----



# android sdk

## 开发环境

Intellij IDEA 或 android studio 支持kotiln、java 接入 

## 接入步骤

### 1.将sdk-release.aar放入libs文件夹，下载sdk_config.xml文件放入src/main/res/values文件夹

### 2.gradle(kts) 配置

```kotlin
plugins {
    id("com.android.application")
    kotlin("android") //kotlin配置
    kotlin("android.extensions")
    id("com.google.gms.google-services") //goolge服务接入
}

repositories {
    maven(url = "https://dl.bintray.com/kittinunf/maven/")
    maven(url = "https://jitpack.io")
}

android {
  defaultConfig {
        minSdkVersion(19) //适配android最低版本
    		compileOptions {
        	sourceCompatibility = JavaVersion.VERSION_1_8
        	targetCompatibility = JavaVersion.VERSION_1_8
        }
    }

  }
}

android {
    lintOptions {
        this.isAbortOnError = false
    }
}

dependencies {
  implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.7")

    implementation("androidx.appcompat:appcompat:1.1.0")
    implementation("androidx.core:core-ktx:1.3.0")
    implementation("androidx.legacy:legacy-support-v4:1.0.0")

    implementation("androidx.constraintlayout:constraintlayout:1.1.3")
    implementation("com.google.android.material:material:1.1.0")
    implementation("com.tencent:mmkv-static:1.2.1")
    implementation("com.linecorp:linesdk:4.0.8")
    implementation("com.vk:androidsdk:2.2.3")
    implementation("com.twitter.sdk.android:twitter:3.3.0")
    implementation(files("libs/sdk-release.aar"))
    implementation("com.facebook.android:facebook-login:5.15.3")
    implementation("com.facebook.android:facebook-share:5.15.3")
    implementation("com.google.android.gms:play-services-auth:18.0.0")
    implementation("io.ktor:ktor-client-serialization-jvm:1.3.2")
    implementation("androidx.multidex:multidex:2.0.1")
    implementation("com.github.kittinunf.fuel:fuel:2.2.2")
    implementation("com.googlecode.libphonenumber:libphonenumber:8.12.9")
  
    //webview js支持
    implementation("com.github.lzyzsd:jsbridge:1.0.4")

    // Daniel says add these four line:
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0")
    implementation("androidx.lifecycle:lifecycle-livedata-ktx:2.2.0")
    implementation("androidx.activity:activity-ktx:1.1.0")

    // Adjust
    implementation("com.adjust.sdk:adjust-android:4.22.0")
    implementation("com.android.installreferrer:installreferrer:2.1")

    // Firebase 事件上报
    implementation("com.google.firebase:firebase-core:17.4.4")
    implementation("com.google.firebase:firebase-analytics:17.4.4")


    // Google支付
    implementation("com.android.billingclient:billing:3.0.0")

    // Glide 用于加载图片
    implementation("com.github.bumptech.glide:glide:4.9.0")
}

```

### 3.初始化(application中做初始化)

```kotlin
OasisGameKit.initialize(this, true, Language.CHINESE_SIMPLIFIED)
```

```java
OasisGameKit.INSTANCE.initialize(this, true, Language.CHINESE_SIMPLIFIED);
```

### 4.重写主activity中 onActivityResult函数

```kotlin
CallbackManager.onActivityResult(requestCode, resultCode, data)
```

```java
CallbackManager.INSTANCE.onActivityResult(requestCode, resultCode, data);
```

### 5.接口接入

登录接口

```kotlin
OasisGameKit.login(this).then {
	//非主线程
  	println("${it.playerId} login success")
}.otherwise {
    println("onLoginButtonClick ${it.message}")
}
```

```java
OasisGameKitJava.login(this, new OasisCallback<Player>() {
    @Override
    public void onSuccess(Player player) {
        Log.d(TAG, "login success " + player.getPlayerId());
    }

    @Override
    public void onFailure(@NotNull Throwable throwable) {
        Log.d(TAG, "login onFail " + throwable.getMessage());
    }

    @Override
    public void eventually() {
    }
});
```

账号切换通知

```kotlin
OasisGameKit.backgroundEvents.playerSwitched.then {
   println(it.newPlayer.playerId + "\n" + it.oldPlayer.playerId);
}
```

```java
OasisGameKitJava.getBackgroundEvents().addPlayerSwitchedCallback(new EventCallback<PlayerSwitchedEvent>() {
            @Override
            public void onCallback(PlayerSwitchedEvent playerSwitchedEvent) {
                Log.d(TAG, playerSwitchedEvent.getNewPlayer().getPlayerId() + "\n" + playerSwitchedEvent.getOldPlayer().getPlayerId());
            }
        });
```

日志上报

```kotlin
OasisGameKit.trackingKit.logEvent(
    "event_main_activity",
    null,
    TrackingKit.TrackingConsumer.DEFAULT or TrackingKit.TrackingConsumer.ADJUST or TrackingKit.TrackingConsumer.MDATA
)
```

```java
OasisGameKitJava.getTrackingKit().logEvent("event_main_activity", null, TrackingKit.TrackingConsumer.DEFAULT | TrackingKit.TrackingConsumer.ADJUST | TrackingKit.TrackingConsumer.MDATA);
```

设置角色接口(需在账号切换和登录成功时设置角色)

```kotlin
// set new GameRole
OasisGameKit.currentGameRole = GameRole("roleId_${System.currentTimeMillis()}", "1")

// update GameRole
OasisGameKit.currentGameRole.serverID = "1"
OasisGameKit.currentGameRole.serverName = "1服"
OasisGameKit.currentGameRole.serverType = GameRole.ServerType.ALL
OasisGameKit.currentGameRole.roleID = "${System.currentTimeMillis()}"
OasisGameKit.currentGameRole.roleName = "${System.currentTimeMillis()}"
OasisGameKit.currentGameRole.coins = 100
```

```java
OasisGameKitJava.setCurrentGameRole(new GameRole("roleId_" + System.currentTimeMillis(), "1"));
OasisGameKitJava.getCurrentGameRole().setServerID("1");
OasisGameKitJava.getCurrentGameRole().setServerName("1服");
OasisGameKitJava.getCurrentGameRole().setServerType(GameRole.ServerType.ALL);
OasisGameKitJava.getCurrentGameRole().setRoleID(System.currentTimeMillis() + "");
OasisGameKitJava.getCurrentGameRole().setRoleName(System.currentTimeMillis() + "");
OasisGameKitJava.getCurrentGameRole().setCoins(100);
```

获取支付列表

```kotlin
val list = ArrayList<String>()
list.add("normal.marstest.100")
list.add("normal.marstest.2000")
list.add("normal.marstest.1000")
list.add("rssweek.marstest.100")

val p = OasisGameKit.fetchProductCatalog(list)
p.then {
    it.forEach { info ->
        println("==========:${info.productId} - ${info.price}")
    }
}
p.otherwise {
    println("==========:${it.message}")
}
```

```java
ArrayList list = new ArrayList<String>();
list.add("normal.marstest.100");
list.add("normal.marstest.2000");
list.add("normal.marstest.1000");
list.add("rssweek.marstest.100");
OasisGameKitJava.fetchProductCatalog(list, new OasisCallback<List<StoreProductInfo>>() {
    @Override
    public void onSuccess(List<StoreProductInfo> storeProductInfos) {
        for (StoreProductInfo info : storeProductInfos) {
            Log.d(TAG, "getProductInfo onSuccess:" + info.getProductId() + "---" + info.getPrice());
        }
    }

    @Override
    public void onFailure(@NotNull Throwable throwable) {
        Log.d(TAG, "getProductInfo onFail:" + throwable.getMessage());
    }

    @Override
    public void eventually() {

    }
});
```

支付接口

```kotlin
val p =
    OasisGameKit.purchase(productId.text.toString(), "sdffssfsf")
p.then {
    runOnUiThread {
        Toast.makeText(
            this@MainActivity,
            "付款状态(${it.purchaseStatus}) 发钻状态(${it.deliveryStatus})",
            Toast.LENGTH_LONG
        ).show()
    }
}
p.otherwise {
    runOnUiThread {
        Toast.makeText(this@MainActivity, "付款失败(${it.message})", Toast.LENGTH_LONG).show()
    }
}
```

```java
OasisGameKitJava.purchase("normal.marstest.100", "sdffssfsf", new OasisCallback<PurchaseResult>() {
    @Override
    public void onSuccess(PurchaseResult purchaseResult) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(MainJavaActivity.this, "付款状态" + purchaseResult.getPurchaseStatus() + "---" + purchaseResult.getDeliveryStatus(), Toast.LENGTH_LONG).show();
            }
        });
    }

    @Override
    public void onFailure(@NotNull Throwable throwable) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(MainJavaActivity.this, "付款失败" + throwable.getMessage(), Toast.LENGTH_LONG).show();
            }
        });
    }

    @Override
    public void eventually() {

    }
});
```
