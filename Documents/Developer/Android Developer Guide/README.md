# 시작하기

이 문서는 Android에서 Analytics SDK를 연동하기 위한 방법에 대해 설명합니다. Analytics SDK를 사용하기 위해서는 먼저 앱을 등록해야 합니다. 앱 등록 방법은 링크(http://cloud.toast.com/documents/2/)를 참고하세요.

**캠페인 연동 관련된 내용은 별도의 문서를 제공합니다. 이 문서에서는 클라이언트 구현에 대한 부분만 설명합니다. 전체적인 내용은 “캠페인 연동 가이드”를 참고하세요.**


# 프로젝트 설정


## SDK 다운로드

1. Analytics SDK 다운로드
    http://cloud.toast.com/documents/6/에서 Android SDK 파일을 다운로드 합니다.
2. 구글 서비스 API 다운로드
    Analytics SDK에서는 Advertising ID를 사용하기 위해서 Google Player Serverice가 필요합니다. 
    안드로이드 SDK Manager를 이용하여 최신 버전의 구글 플레이 서비스 API(google-play-service.jar) 를 다운로드 받습니다.
    (https://developer.android.com/sdk/installing/adding-packages.html)


## 프로젝트설정

### 라이브러리 dependency 설정
다운로드 받은 GameAnalyticsSDK_Android_v1.xxx.zip 파일의 압축을 풀어 아래의 그림과 같이 프로젝트의 libs 디렉토리에 복사합니다. (GameAnalytics SDK jar파일 1개와 성능/보안 지표 수집을 위한 native library가 있습니다)

(Proguard 사용시 다음 옵션을 추가해야 합니다. "-keep public class com.toast.android.analytics.external.leaktest.MemoryInfo { public *;}")

![Project Setting](https://raw.githubusercontent.com/ToastAnalytics/ToastAnalytics/master/Documents/Developer/Android%20Developer%20Guide/images/pg_aos_001.png)

다운로드 받은 구글 서비스 API 의 경우 jar 형태가 아닌 안드로이드 라이브러리 이므로, 해당 라이브러리를 이클립스로 import 해야 사용자 프로젝트에서 사용할 수 있습니다.
(http://developer.android.com/google/play-services/setup.html)

이클립스에 구글 서비스 API를 라이브러리로 import 한 후, 현재 작성중인 사용자 프로젝트 루트 폴더에 마우스 커서를 놓고 우클릭 합니다.

우클릭해서 나타나는 팝업창에서 Properties 메뉴를 누르고 위 그림에서 보이는 것처럼 팝업창이 뜨면, 팝업창 좌측 메뉴에 있는 Android를 선택한 후, 우측의 library 부분에서 [Add]버튼을 눌러 구글 서비스 API 라이브러리의 경로를 지정하도록 합니다.

### AndroidManifest 설정
Analytics SDK에서는 다음과 같은 Permission을 사용합니다.


```
<!-- 단말기 모뎀 상태(ex. 망 사업자명) 조회를 위해 사용합니다. -->
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<!-- 외부 스토리지 쓰기 권한. 미전송 내부적으로 사용하는 데이터 캐시를 위해 사용합니다 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.INTERNET" />
```


스토어 정보는 사용자가 앱을 다운로드한 마켓에 대한 정보를 추적하기 위한 설정입니다. “Google”, “TStore”, “Naver”등 마켓에 맞게 변경하여 사욥합니다. 마켓 문자열은 게임에서 임의로 정의하여 사용할 수 있습니다.

```
<manifest>
    ……
    <application>
        ……
        <!-- 스토어 정보 설정 -->
        <!-- android:value is Tstore, Olleh, Uplus, NaverApp etc  -->
        <meta-data
            android:name="com.toast.android.analytics.appstore"
            android:value="Google" />
        ……
    </application>
</manifest>
```
    

설치 유입경로 분석을 위해서는 Install Receiver 설정이 필요합니다. (Google Play를 통해서 앱을 설치하는 경우에만 동작합니다.)

```
<manifest>
    ……
    <application>
        ……
        <receiver
            android:name="com.toast.android.analytics.InstallReferrerReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="com.android.vending.INSTALL_REFERRER"/>
            </intent-filter>
        </receiver>
        ……
    </application>
</manifest>
```

    
동일한 Action을 처리하는 Broadcast Receiver는 앱 내에서 하나만 등록할 수 있습니다. 여러곳에서 Install Referrer를 사용하는 경우 반드시 아래 가이드를 따라서 설정해야 합니다.

Receiver는 하나만 등록하고 meta-data를 이용하여 다른 Receiver들을 추가합니다.

Install Receiver 설정이 잘못된 경우 신규 사용자 집계가 잘못될 수 있습니다.

다른 리시버를 추가로 등록하기 위해서는 아래 설정과 같이 meta-data로 주가합니다. 이때 android:value는 해당 리시버의 Class명을 적어줍니다.

```
<receiver 
    android:name="com.toast.android.analytics.InstallReferrerReceiver" 
    android:exported="true">
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER"/>
    </intent-filter>

    <meta-data android:name="forward1" android:value="Receiver-Class-Name" />
    <meta-data android:name="forward2" android:value=" Receiver-Class-Name" />
</receiver>
```


실행 유입경로 분석을 위해서는 Execution Receiver 설정이 필요합니다.

앱이 설치되어 있는 상태에서 프로모션을 통한 앱 실행이 발생하는 경우 효과 분석을 위해서 사용하는 Receiver입니다.
Install Receiver와 함께 등록해 줍니다.

```
<manifest>
  ……
  <application>
    ……
      
      <receiver android:name="com.toast.android.analytics.ExecutionReferrerReceiver" android:exported="true">
        <intent-filter>
          <action android:name="com.toast.android.analytics.toastpromotion.EXECUTION" />
        </intent-filter>
      </receiver>
        ……        
    <application>
</manifest>
```

 
# 필수연동

기본적인 통계 데이터 수집을 위해서 필수로 연동해야 하는 API입니다. 필수 연동 항목만 구현해도 대부분의 데이터를 볼 수 있습니다.
필수 연동 항목은 아래와 같습니다.

* 초기화 : initializeSDK
* 세션추적 : traceActivation, traceDeactivation
* 구매 (In App Purchase) : tracePurchase
* 재화 획득/사용 : traceMoneyAcquisition, traceMoneyConsumption
* 레벨업 : traceLevelUp
* 친구수 : traceFriendCount


## 초기화

SDK를 사용하기 위해서는 앱 등록 후 발행되는 “앱 아이디”와 “컴퍼니 아이디”가 필요합니다. 앱 등록 방법은 링크(http://cloud.toast.com/documents/2/)를 참고하세요.

initializeSDK 함수의 AppID는 앱 정보의 “AppKey”를, CompanyID는 “컴퍼니 아이디” 를 사용합니다.

![App Key](https://raw.githubusercontent.com/ToastAnalytics/ToastAnalytics/master/Documents/Developer/Android%20Developer%20Guide/images/pg_aos_002.png)

GameAnalytics SDK를 사용하기 위해서는 SDK 초기화를 먼저 수행해야 합니다.

GameAnalytics 클래스의 initializeSDK 함수는 SDK 초기화를 수행하는 함수입니다. 이 함수는 내부적으로 필요한 데이터(디바이스 정보, 앱 설정 정보)를 확인하고, 로그 전송을 위한 환경을 설정하는 작업을 수행합니다.


```
import com.nhnent.android.toast.analytics.GameAnalytics;

public class TestActivty extends Activity {

  @Override
  protected void onCreate(Bundle savedInstanceState)
  {
    super.onCreate();
    ……
    int result = GameAnalytics.initializeSdk(getApplicationContext(), “AppKey”, “CompanyID”, “AppVersion”, false);

    if(result != GameAnalytics.S_SUCCESS)
        Log.d(TAG, "initialize error " + GameAnalytics.getResultMessage(result));

    }

}
```
    
## 사용자 구분 기준 설정

**운영중에 사용자 구분 기준을 변경하면 변경 전/후 데이터의 연관 관계가 끊어지기 때문에 게임 오픈 이후에는 기준을 바꾸지 않아야 합니다.**

Analytics는 사용자를 구분하는 기준으로 Advertise ID 또는 User ID를 사용합니다. 두가지를 모두 사용할 수는 없고, 게임 정책에 따라 한가지를 선택하여야 합니다.

일반적으로 Advertise ID를 기준으로 사용합니다. 하지만 게임에서 특별한 요구사항이 있는 경우 User ID를 기준으로도 사용할 수 있습니다.

예를들어 Advertise ID를 사용하는 경우 하나의 디바이스에서 탈퇴->재가입 하는 경우에도 기존과 동일한 사용자로 집계됩니다. 반면 User ID를 사용하는 경우에는 신규 사용자로 집계됩니다.

또는 한명의 사용자가 두개의 디바이스를 사용하는 경우 Advertise ID를 사용하면 각각 다른 사용자로 집계되는 반면 User ID를 사용하는 경우 한명의 사용자로 집계됩니다.

이런 부분을 고려하여 게임에서 기준을 결정하여 사용합니다.

초기화 함수(initializeSDK)의 마지막 인자(use logging userid flag)로 이 값을 설정할 수 있습니다. Flag가 true 인 경우 User ID를 사용자 구분 기준으로 사용합니다. False로 설정하면 Advertise ID를 구분 기준으로 사용합니다.

아래 코드는 User ID를 사용자 구분 기준으로 사용하는 경우입니다.

```
import com.nhnent.android.toast.analytics.GameAnalytics;

public class TestActivty extends Activity {

  @Override
  protected void onCreate(Bundle savedInstanceState)
  {
    super.onCreate();
    ……
    int result = GameAnalytics.initializeSdk(getApplicationContext(), “AppID”, “CompanyID”, “AppVersion”, true);

    if(result != GameAnalytics.S_SUCCESS)
        Log.d(TAG, "initialize error " + GameAnalytics.getResultMessage(result));

    }
    ……
    // 게임에서 로그인 처리 완료
    ……
    // User ID를 사용자 구분 기준으로 사용하는 경우 User ID를 등록하는 함수.
    GameAnalytics.setUserId(“user_id”, true);
    ……
}
```
    
만약 초기화 함수의 마지막 인자(use logging userid flag)를 true로 설정한 경우 setLoggingUserId를 호출하여 User ID를 등록해야 합니다. Flag를 true로 설정하고 setLoggingUserId를 호출하지 않으면 이후에 호출하는 모든 API가 실패(E_LOGGING_USER_ID_EMPTY)를 Return 합니다.

setUserId의 두번째 인자는 Promotion이나 Campaign을 사용하는 경우 true입니다. 그렇지 않은 경에는 false 입니다.

setUserId 함수는 initializeSDK 호출 이후에 게임에서 로그인 성공한 후 게임에서 사용하는 userID를 획득한 직후에 호출하면 됩니다. userID는 게임에서 사용자를 구분하기 위해 사용하는 값을 사용하면 됩니다.


Advertise ID 관련 내용은 아래 링크를 참고하세요.
- Android : https://developer.android.com/google/play-services/id.html


## 세션 추적

DAU(Daily Active User)와 게임 체류 시간을 추적하기 위한 연동입니다. 

App 시작/종료, Background/Foreground 이동시 해당 액션에 맞는 API를 호출하여 측정할 수 있습니다. 

App이 처음 실행될때(initializeSDK 이후) 또는 background에서 foreground로 이동시 traceActivation을 호출하여 세션 추적을 시작합니다. 이후 App이 background로 들어가는 시점에 traceDeactivation을 호출하여 세션 추적을 멈춥니다. 

traceDeactivation을 호출하면 traceActivation과 traceDeactivation 사이의 시간을 계산하여 게임 이용 시간을 측정합니다. 또한 SDK 내부적으로 수행하던 작업도 traceDeactivation에서 중단합니다. 

Background/Foreground 이동시 위 함수를 호출하지 않으면 정확한 게임 이용 시간을 측정할 수 없기 때문에 이 API는 반드시 호출해야 합니다. 

DAU는 하루동안 traceActivation을 호출한 사용자(Advertise ID 또는 User ID 기준)의 중복을 제거한 숫자로 계산합니다.

```
import com.toast.android.analytics.GameAnalytics;

public class TestActivty extends Activity {

  @Override
  protected void onResume()
  {
    // foreground 상태로 전환된 것을 서버에게 알림
    GameAnalytics.traceActivation();
  }

  @Override
  protected void onPause()
  {
    // background 상태로 전환된 것을 서버에게  알림
    GameAnalytics.traceDeactivation();
  }
}
```


## 액션 추적

In-App Purchase, 머니 획득/사용, 레벨업, 친구 수 변경등 사용자의 Action에 대해 추적할 수 있습니다.

### In-App Purchase

In-App Purchase가 발생한 후 tracePurchase를 호출하여 매출 정보를 전송합니다. 

Currency는 ISO-4217(http://en.wikipedia.org/wiki/ISO_4217)에서 정의한 코드를 사용합니다. 

$0.99의 보석을 구매하는 경우 아래와 같이 사용합니다. 

(여기에서 “GEM_10”은 게임에서 정의한 Item의 Code입니다. Unit Cost는 해당 아이템의 단위 가격, Payment는 실제 사용자가 사용한 금액입니다. Level은 구매한 사용자의 Level을 입력합니다.)

```
GameAnalytics.tracePurchase("GEM_10", 0.99f, 0.99f, "USD", 10);
```


### 재화 획득/사용

게임내에서 재화의 획득/사용시 호출합니다. 1차 재화, 2차 재화의 변동량을 추적합니다. 일반적으로 1차 재화는 In-App Purchase를 통해서 구매하는 재화(ex. 보석, 루비등) 입니다. 2차 재화는 1차 재화를 이용하여 구매하는 재화(ex. 체리, 하트등) 입니다. 

IAP를 통해서 보석 10개를 구매한 경우 아래와 같이 사용합니다. 

(“CODE_IAP”는 게임에서 정의한 Code입니다. 1차 재화인 경우 Type은 0, 2차 재화인 경우 1을 사용합니다)

GameAnalytics.traceMoneyAcquisition("CODE_IAP", "0", 10, 10);

보석 10개를 이용하여 체리 100개를 구매한 경우는 아래와 같이 사용합니다.

```
// 1차 재화 사용
GameAnalytics.traceMoneyConsumption("CODE_USE_GEM", "0", 10, 10);

// 2차 재화 획득
GameAnalytics.traceMoneyAcquisition("CODE_BUY_CHERRY", "1", 100, 10);
```

1차 재화를 사용하여 2차 재화를 구입한 경우 실제 ‘1차 재화 감소’->‘2차 재화 증가’가 발생합니다. 하지만 2차 재화를 구입하기 위해서 1차 재화를 사용하는 경우 별도의 재화 소모로 판정하지 않고 싶은 경우 ‘2차 재화 획득’ 로그만 전송하여도 됩니다.

### 레벨업

사용자 레벨이 변경되는 경우 traceLevelUp을 호출합니다. 참고로 대부분의 액션 추적 API는 레벨별 액션 추적을 위해서 사용자 Level을 같이 받습니다. 

사용자 레벨이 10으로 변경되는 경우 아래와 같이 호출합니다. 한 사용자의 레벨은 반드시 증가해야만 합니다. 감소하는 경우 정확한 데이터 측정이 불가능합니다. 

예를들어 “Candy Crush Sage”와 같이 스테이지로 진행 되는 게임에서 스테이지를 레벨로 사용하는 경우에는 해당 스테이지에 최초 진입할때만 레벨업 로그를 남겨야 합니다. 만약 이전 스테이지로 다시 돌아가서 플레이 하는 경우에는 레벨업 로그를 남기지 않습니다.

또한 다른 API에 전달되는 level 값도 현재 진행중인 스테이지가 아닌 사용자의 최고 스테이지를 레벨 값으로 사용해야 합니다.

```
GameAnalytics.traceLevelUp(10);
```


### 친구

사용자의 친구 숫자를 등록합니다. 일반적으로 앱 실행 후 친구 정보 로딩이 완료된 시점에 호출하면 됩니다.

```
GameAnalytics.traceFriendCount(100);
```


# 선택적 연동

기본연동을 통해 제공되는 기본 지표 이외에 게임에서 추가적으로 측정하고 싶은 지표가 있는경우, 혹은 Analytics에서 제공하는 Campaign 기능을 사용하는 방법에 대해 설명합니다.

추가적으로 연동할 수 있는 항목은 아래와 같습니다.

* Campaign 연동 : setCampaignListener, (show/hide)Campaign
* Custom Event 수집 : traceEvent
* 소요시간 측정 : traceStartSpeed, traceEndSpeed


## 캠페인 연동

### 캠페인 연동 사전 준비
캠페인 연동 및 실행을 위해서는 별도의 가이드를 제공하고 있습니다.
Toast Analytics의 “캠페인 실행” 메뉴의 “페이지 가이드”를 참고하세요.
http://analytics.toast.com/promotion/share/document/4.2_Campaign_run.pdf

### 푸시 연동
Analytics SDK에서 캠페인을 위해서 GCM을 사용합니다. GCM 사용을 위해서는 Analytics 관리자 페이지에 Project Number와 API Key를 등록해야 합니다. 또한 SDK에는 SenderID를 등록해야 합니다. 

GCM에 관련된 전반적인 내용은 Google에서 제공하는 “Google Cloud Message” 문서를 참고하세요. 

http://developer.android.com/google/gcm/gs.html 

Analytics 관리자 페이지에 정보를 등록하는 방법은 “Getting Started” 문서에서 “푸시 설정” 항목을 참고하세요. 

SDK에 SenderID는 “setGcmSenderId” API를 동해 등록할 수 있습니다.

```
import com.nhnent.android.toast.analytics.GameAnalytics;

public class TestActivty extends Activity {

  @Override
  protected void onCreate(Bundle savedInstanceState)
  {
    super.onCreate();
    ……
    int result = GameAnalytics.initializeSdk(getApplicationContext(), “AppKey”, “CompanyID”, “AppVersion”, false);

    if(result != GameAnalytics.S_SUCCESS)
        Log.d(TAG, "initialize error " + GameAnalytics.getResultMessage(result));

    }
    GameAnalytics.setGcmSenderId(“123456789012”);
    ……
```

GCM 사용을 위해서는 AndroidManifest.xml 파일에 추가로 permission과 reciever를 등록해야 합니다.

추가로 필요한 permission은 아래와 같습니다. (여기에서 “APP.PACKAGE.NAME”은 사용자의 패키지명으로 변경해야 합니다.)

```
<!--  gcm 푸시 권한 설정 -->
<uses-permission android:name="android.permission.GET_ACCOUNTS" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
<uses-permission android:name="android.permission.VIBRATE" />
<permission
          android:name="APP.PACKAGE.NAME.permission.C2D_MESSAGE"
          android:protectionLevel="signature" />
<uses-permission android:name="APP.PACKAGE.NAME.permission.C2D_MESSAGE" />
    
추가로 등록해야 하는 Receiver는 아래와 같습니다. (역시 “APP.PACKAGE.NAME”은 사용자의 패키지명으로 변경해야 합니다.)
<manifest>
  ……
  <application>
    ……
    <!-- gcm 리시버 설정 -->
    <receiver 
        android:name="com.toast.android.analytics.gcm.GcmBroadcastReceiver"
        android:permission="com.google.android.c2dm.permission.SEND" >
            <intent-filter android:priority=”1”>
            <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            <category android:name="APP.PACKAGE.NAME" />
            </intent-filter>
    </receiver>
    <service android:name="com.toast.android.analytics.gcm.GcmIntentService" />
    …… 
    </application>
</manifest>
```

서버에서 푸시(GCM) 메시지를 받으면 Analytics SDK는 Notification Bar에 알림을 표시합니다. 이 알림을 터치하면 앱을 실햅합니다. 이때 아무런 설정을 하지 않으면 해당 앱의 Main Activity를 띄웁니다.

Main Activity가 아닌 다른 Activity를 실행하기 위해서는 “setPushIntent”함수를 통해서 명시적으로 지정해야 합니다.

```
import com.nhnent.android.toast.analytics.GameAnalytics;

public class TestActivty extends Activity {

  @Override
  protected void onCreate(Bundle savedInstanceState)
  {
    super.onCreate();
    ……
    int result = GameAnalytics.initializeSdk(getApplicationContext(), “AppID”, “CompanyID”, “AppVersion”, false);

    if(result != GameAnalytics.S_SUCCESS)
        Log.d(TAG, "initialize error " + GameAnalytics.getResultMessage(result));

    }

    GameAnalytics.setGcmSenderId(“123456789012”);
    ……

    Intent intent = new Intent(getApplicationContext(), GCMActivity.class);
    GameAnalytics.setPushIntent(intent);
    ……
} 
```

덧붙여 AndroidManifest.xml에 있는 Activity의 launchMode를 singeTop, singleTask, singleInstance로 설정하는 경우 새로운 Activity를 만들지 않고 기존 것을 재사용 합니다.

이때 Push에 대한 정보가 담긴 Intent 정보를 제대로 전달받기 위해서는 해당 Activity에서 “onNewIntent”를 명시적으로 Override 하여야 합니다.

```
public class NEAFlatSampleActivity extends Activity {
    
  @Override
  protected void onNewIntent(Intent intent)
  {
    super.onNewIntent(intent);
    setIntent(intent);
  }
}
```


SDK 에서 기본으로 제공하는 GCM 푸시 리시버를 사용하지 않고, 사용자가 직접 구현하는 경우 Manifest의 설정은 아래와 같이 변경되어야 합니다.

또한 GCM 푸시를 받아 처리하는 로직(리시버, 서비스, Notification) 또한 사용자가 직접 구현해야 합니다.

```
<manifest>
  ……
  <application>
    ……
    <!-- gcm 리시버 설정 -->
    <receiver
      android:name="com.toast.android.analytics.gcm.GcmBroadcastReceiver"
      android:permission="com.google.android.c2dm.permission.SEND" >
        <intent-filter android:priority=”1”>
          <action android:name="com.google.android.c2dm.intent.RECEIVE" />
          <category android:name="APP.PACKAGE.NAME" />
        </intent-filter>
    </receiver>
    <service android:name="com.toast.android.analytics.gcm.GcmIntentService" />
    ……

    <!--  gcm push 관련 사용자 정의 설정 -->
    <receiver 
      android:name=".GcmBroadcastReceiver"
      android:permission="com.google.android.c2dm.permission.SEND" >
      <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <category android:name="com.example.aflatgametest" />
      </intent-filter>
    </receiver>
    <service android:name=".GcmIntentService" />

    ……
    </application>
</manifest>
```
    
위에서 보이는 것처럼 SDK 용으로 설정된 리시버, 서비스 설정 아래에 사용자 정의 설정을 신규로 추가하도록 합니다.

푸시 payload 구성시 “cid” 필드는 사용자 정의 푸시에서 사용해서는 안됩니다. “cid” 필드는 SDK 에서 제공하는 기본 푸시에서 내부적으로 사용하는 태그로 캠페인 정보를 전달하기 위해 사용하고 있습니다.


### 캠페인 Listener 구현 및 등록

SDK는 일정한 주기로 캠페인 서버와 통신하여 캠페인 및 보상정보를 가져옵니다. 만약에 현재 사용자에게 진행할 캠페인이 있거나 사용자가 받을 보상 정보가 있는 경우 CampaignListener를 통해서 알려줍니다.

따라서 캠페인 정보를 받기 위해서는 CampaignListener를 구현해야 합니다.

```
public interface CampaignListener {
    void OnCampaignVisibilityChanged(string adspaceName, bool show);
    void OnCampaignLoadSuccess(string adspaceName);
    void OnCampaignLoadFail(string adspaceName, int errorCode, string errorMessage);
    void OnMissionComplete( missionList)
    void OnPromotionVisibilityChanged(bool show)
}
```


각 Callback은 아래와 같은 경우 호출됩니다.

* OnCampaignVisibilityChanged : showCampaign, hideCampaign를 호출하여 캠페인 관련 팝업이나 배너가 보이거나 사라질때 호출됩니다.
* OnMissionComplete : 사용자가 캠페인/프로모션을 진행하고 정해진 미션을 달성하여 보상 정보가 있을때 호출됩니다. 여기에서 받은 정보를 이용하여 게임서버를 통해서 보상을 지급해야 합니다. 캠페인 보상 관련 프로세스는 “캠페인 적용 가이드”문서를 참고하세요.
* OnCampaignLoadSuccess, OnCampaignLoadFail : 서버에서 가져 온 캠페인 정보 파싱 결과를 알려줍니다. 게임에서는 이 Callback에서 특별한 처리를 할 필요는 없습니다. 로그 확인을 위해 제공하는 Callback입니다.
* OnPromotionVisibilityChanged : Promotion을 사용하는 경우 Promotion View가 열리고 닫힐때 호출됩니다.

구현한 CampaignListener는 setCampaignListener를 이용하여 등록합니다.


```
function setListener() {
  ……
  GameAnalytics.setCampaignListener(new MyCampaignListener() {
    public void OnCampaignVisibilityChanged(string adspaceName, bool show) {
      ……
    }

    public void OnCampaignLoadSuccess(string adspaceName) {
      // for Debugging
    }

    public void OnCampaignLoadFail(string adspaceName, int errorCode, string errorMessage) {
      // for Debugging
    }

    public void OnMissionComplete(List<string> missionList) {
      // missionList String에는 key/value 정보가 구분자 ‘|’으로 제공됩니다.
      // 이 값을 가지고 게임서버를 통해 promotion server에서 검증하여 사용자에게
      // 보상을 지급합니다.
    }
    
    public void OnPromotionVisibilityChanged(bool show) {
      // Promotion View가 열릴때 show가 true로 전달됩니다.
      // show가 true인 경우 게임 렌더링 성능을 낮추는 작업을 할 수 있습니다.
      // 반대로 show가 false인 경우 게임 렌더링 성능을 다시 원래대로 돌릴 수 있습니다.
    }
  });
}
```

### 캠페인 Show/Hide
현재 사용자에게 진행 중인 캠페인이 있는 경우 Analytics 웹사이트에서 등록한 캠페인 팝업/배너를 보여주고, 노출된 팝업/배너를 숨기는 메소드입니다. 파라미터인 adspaceName은 Analytics 웹사이트에서 캠페인 등록시에 정의한 adspace 이름을 사용하면 됩니다. Adspace란 팝업/배너가 나타나는 게임 내의 특정 위치를 의미합니다.

showCampaign() 메소드는 해당 adspace를 사용하는 캠페인이 없으면 아무런 동작도 하지 않으므로, 캠페인 팝업/배너를 노출할 것으로 예상되는 게임 내의 여러 지점에 각각 다른 adspaceName으로 함수를 호출해두면 이후에 별도의 게임 클라이언트 수정없이도 게임 운영자가 Analytics 웹사이트에서 캠페인을 등록하는 작업만으로 쉽게 팝업/배너를 노출할 수 있게 됩니다.

Adspace 등록 방법은 “캠페인 테스트 가이드”를 참고하세요.

```
public static int showCampaign(String adspaceName, Activity activity)
public static int showCampaign(String adspaceName, Activity activity, int animation, int lifeTime) 
public static int hideCampaign(String adspaceName)
public static int hideCampaign(String adspaceName, int animation)
```

## 커스텀 이벤트 사용

게임별로 특정 이벤트를 정의하여 분석하고 싶은 경우 사용합니다.

예를들어 Fever Time Item을 사용하는 경우 아래와 같이 사용합니다. 사용된 모든 코드는 게임에서 정의하여 사용합니다. 아래 예제는 특정 스테이지에서 아이템 변동 내용을 추적하기 위해 정의한 코드입니다.

```
GameAnalytics.traceEvent("ITEM", "ITEM_USE", "FEVER", "STAGE_10", 1, 10);
```

특정 레벨에서 보스 배틀 결과를 추적할때도 사용할 수 있습니다.

```
GameAnalytics.traceEvent("STAGE", "STAGE_BOSS_VICTORY", "DRAGON_VALLEY", "BOSS_MOB", 1, 10);
```

이 외에도 다양한 용도로 게임에 특화된 이벤트 추적에 사용할 수 있습니다.

traceEvent에 사용하는 String Type 파라미터(event type, event code, param1, param2)는 각각 50byte까지 사용할 수 있습니다. 그리고 event 하위에 발생 가능한 param1 은 300개까지, 또 param1 하위에 발생 가능한 param2는 200개까지 사용할 수 있습니다.

자세한 내용은 Toast Analytics 사이트에서 가이드를 참고하세요. (커스텀 이벤트 페이지 우측 상단의 “페이지 가이드”를 클릭하면 다운받을 수 있습니다)


## 프로모션 연동

게임에서 타 게임과 프로모션 연동시 사용합니다.

### 프로모션 연동 사전 준비
프로모션 연동 및 실행을 위해서는 별도의 가이드를 제공하고 있습니다.

### 프로모션 버튼 추가
Analytics 설정 페이지에서 프로모션 정보를 등록하고 버튼 이미지를 등록하면 SDK 초기화 할때 버튼을 다운로드 받아서 Device에 저장합니다.

게임에서는 버튼 이미지를 이용하여 적절한 위치에 버튼을 생성하여야 합니다.

버튼을 생성하기 전에 먼저 프로모션이 가능한 상태인지 확인합니다. 관리자가 Analytics 웹페이지에서 프로모션을 사용하지 않겠다고 설정하거나, 버튼 이미지 다운로드에 실패 또는 프로모션 서비스 점검이나 장애시에는 프로모션 진행이 불가능한 상태이므로 버튼을 화면에 표시하지 않아야 합니다.“isPromotionAvailable()”으로 프로모션 가능 여부를 확인할 수 있습니다. 프로모션 진행이 가능한 상태이면 “getPromotionButtonImagePath()&rdqou; 함수를 이용하여 버튼 이미지 경로를 확인해서 게임에서 적절한 위치에 버튼을 생성합니다. (버튼 이미지는 PNG 형식으로 저장합니다.)

(Analytics 설정에 등록한 이미지를 사용하지 않고 게임 배포 패키지에 포함된 이미지를 사용하여 버튼을 생성도 가능합니다. 단 이때도 프로모션 가능 여부를 반드시 확인-isPromotionAvailable- 후에 버튼을 생성해야 합니다.)

```
if (GameAnalytics.isPromotionAvailable() == true) {
    String buttonImagePath = GameAnalytics.getPromotionButtonImagePath();
    Drawable d = new BitmapDrawable(activity.getResources(), buttonImagePath);
    ……
} else {
    Debug.Log ("Hide Toast Promotion Button");
}
```

### 프로모션 실행
프로모션 버튼을 터치하는 경우 “launchPromotionPage()”를 호출하여 프로모션을 실행합니다.

```
private void touchPromotionButton() {
    ……
    GameAnalytics.launchPromotionPage();
    ……
}
```

“launchPromotionPage()”에서는 내부적으로 WebView를 사용하여 프로모션 컨텐츠를 제공합니다. 이 기능을 사용하기 위해서는 AndroidManifet.xml 설정에 해당 Activity를 추가해야 합니다.

가로/세로 화면 고정을 위해서는 android:screenOrientation을 이용하여 설정합니다. (http://developer.android.com/guide/topics/manifest/activity-element.html#screen)

```
<activity
    android:configChanges="keyboard|keyboardHidden|orientation|
        screenLayout|uiMode|screenSize|smallestScreenSize"
    android:name="com.toast.android.analytics.PromotionActivity" />
```

또한 앱 설치/실행시 광고 효과 측정을 위해서 Install Receiver를 등록해야 합니다.

Install Receiver는 프로모션을 통해 앱을 설치한 경우(Google Play만 가능) 효과 측정을 위해 필요합니다. Execution Receiver는 앱이 설치되어 있는 경우 프로모션을 통해 실행 효과 측정을 위해 필요합니다. “2.2.2. AndroidManifest 설정”을 참고하세요.

### 보상
사용자에게 프로모션 페이지가 노출된 경우 보상을 지급합니다. 또한 프로모션을 통해 앱을 설치하거나 정해진 미션을 달성한 경우에도 보상을 지급합니다.

보상에 대한 정보는 “CampaignListener”의 “onMissionComplete”를 통해서 전달 됩니다.

보상 처리 절차는 “프로모션 적용및 운영 가이드”문서를 참고하세요.


## 소요시간 측정

특정 구간에 소요되는 시간을 측정할 수 있습니다. 예를들어 튜토리얼에 소요되는 시간을 측정하고 싶은경우, Scene 전환에 걸리는 시간을 측정하고 싶은경우등 시간 측정이 필요한 임의의 구간에 사용할 수 있습니다.

Intro Scene 로딩 시간을 측정하고 싶은 경우 아래와 같이 사용합니다. “INTRO_LOADING”은 특정 구간에 대해서 게임에서 정의하는 값입니다.

```
private void onStart() {
    GameAnalytics.traceStartSpeed("INTRO_LOADING");
}

private void onLoadCompleted() {
    GameAnalytics.traceEndSpeed("INTRO_LOADING");
}
```


# SDK 설정

## 디버그 모드 활성화
개발중에 SDK 로그 확인을 위해서 로그 출력 여부를 설정할 수 있습니다.

이 함수는 initializeSDK 이전/이후 모두 호출 가능합니다. 기본 값은 setDebugMode(false)입니다.

Log tags는 “Analytics:”로 시작합니다. Eclipse에서 log cat filter를 “Analytics”로 지정하면 SDK에서 발생하는 로그를 확인할 수 있습니다.)

```
private void Start () {
     ……
     GameAnalytics.setDebugMode(true);
     ……

     int result = GameAnalytics.initializeSdk ("APPKEY", "COMPANYID", "VERSION", false);

     if (result != 0 ) {
        // SDK 초기화 실패
     }
     ……
}
```

디버그 모드가 활성화된 경우 로그 전송 내용을 확인할 수 있습니다. 로그를 전송하고 그에 대한 응답 로그를 확인하여 로그가 정상적으로 전송되었는지 확인할 수 있습니다. 아래와 같은 로그 스트링이 있으면 수집된 데이터가 정상적으로 서버로 전송된 것입니다. (***은 상황에 따라서 다른 값입니다)

```
Android : server response (***) : 200 OK
```


## 디바이스 정보 확인
SDK에서 수집하는 Device 정보를 확인할 수 있습니다.

현재 확인 가능한 값은 Device ID, Push Token, Campaign User ID입니다. 이 값들은 캠페인 연동 테스트시 필요합니다. 자세한 내용은 “캠페인 연동 가이드”를 참고하세요.

Device 정보를 확인하기 위해 사용하는 Key 값입니다.
- public static final String DEVICE_INFO_DEVICEID = “deviceId”;
- public static final String DEVICE_INFO_TOKEN = “token”;
- public static final String DEVICE_INFO_CAMPAIGN_USERID = ”campaignUserId”;

```
private void printDeviceInfo() {
     String deviceID = GameAnalytics.getDeviceInfo(DEVICE_INFO_DEVICEID);
     String pushToken = GameAnalytics.getDeviceInfo(DEVICE_INFO_TOKEN);
     String campaignUserID = GameAnalytics.getDeviceInfo(DEVICE_INFO_CAMPAIGN_USERID);
     ……
}
```

## SDK 버전 확인
SDK 버전은 &lduo;getVersion()” 함수를 통해 확인할 수 있습니다.

```
public static String getVersion()
```
