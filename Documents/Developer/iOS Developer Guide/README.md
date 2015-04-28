# 시작하기

이 문서는 iOS에서 Analytics SDK를 연동하기 위한 방법에 대해 설명합니다. Analytics SDK를 사용하기 위해서는 먼저 앱을 등록해야 합니다. 앱 등록 방법은 링크 (http://cloud.toast.com/documents/2/)를 참고하세요. 
캠페인 연동 관련된 내용은 별도의 문서를 제공합니다. 이 문서에서는 클라이언트 구현에 대한 부분만 설명합니다. 전체적인 내용은 “캠페인 연동 가이드”를 참고하세요.

# 프로젝트 설정

## SDK 다운로드

http://cloud.toast.com/documents/6/에서 Android SDK 파일을 다운로드 합니다.

## 프로젝트설정

라이브러리 dependency 설정
다운로드 받은TAGAnalytics.h 파일을 프로젝트에 포함시키고, libTAGAnalytics.a 파일을 “Linked Frameworks and Libraries”에 추가합니다. “AdSuppert.framework”, “libsqlite3.dylib”, “CoreTelephony.framework”도 추가합니다.

필수연동
기본적인 통계 데이터 수집을 위해서 필수로 연동해야 하는 API입니다. 필수 연동 항목만 구현해도 대부분의 데이터를 볼 수 있습니다.

필수 연동 항목은 아래와 같습니다.
초기화 : initializeSDK
세션추적 : traceActivation, traceDeactivation
구매 (In App Purchase) : tracePurchase
재화 획득/사용 : traceMoneyAcquisition, traceMoneyConsumption
레벨업 : traceLevelUp
친구수 : traceFriendCount
초기화

SDK를 사용하기 위해서는 앱 등록 후 발행되는 “앱 인증Key”와 “컴퍼니 아이디”가 필요합니다. 앱 등록 방법은 링크(http://cloud.toast.com/documents/2/)를 참고하세요.

GameAnalytics SDK를 사용하기 위해서는 SDK 초기화를 먼저 수행해야 합니다.
GameAnalytics 클래스의 initializeSDK 함수는 SDK 초기화를 수행하는 함수입니다. 이 함수는 내부적으로 필요한 데이터(디바이스 정보, 앱 설정 정보)를 확인하고, 로그 전송을 위한 환경을 설정하는 작업을 수행합니다.
- (BOOL)application:(UIApplication *)application
didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  ……
  [TAGAnalytics initializeSdk:@”AppKey”
			 companyId:@”CompanyID”
			appVersion:@”AppVersion”
		 useLoggingUserId:NO];
  ……
}
        
사용자 구분 기준 설정
** 운영중에 사용자 구분 기준을 변경하면 변경 전/후 데이터의 연관 관계가 끊어지기 때문에 게임 오픈 이후에는 기준을 바꾸지 않아야 합니다. 
Analytics는 사용자를 구분하는 기준으로 Advertise ID 또는 User ID를 사용합니다. 두가지를 모두 사용할 수는 없고, 게임 정책에 따라 한가지를 선택하여야 합니다. 
일반적으로 Advertise ID를 기준으로 사용합니다. 하지만 게임에서 특별한 요구사항이 있는 경우 User ID를 기준으로도 사용할 수 있습니다. 
예를들어 Advertise ID를 사용하는 경우 하나의 디바이스에서 탈퇴->재가입 하는 경우에도 기존과 동일한 사용자로 집계됩니다. 반면 User ID를 사용하는 경우에는 신규 사용자로 집계됩니다. 
또는 한명의 사용자가 두개의 디바이스를 사용하는 경우 Advertise ID를 사용하면 각각 다른 사용자로 집계되는 반면 User ID를 사용하는 경우 한명의 사용자로 집계됩니다. 
이런 부분을 고려하여 게임에서 기준을 결정하여 사용합니다. 
초기화 함수(initializeSDK)의 마지막 인자(use logging userid flag)로 이 값을 설정할 수 있습니다. Flag가 true 인 경우 User ID를 사용자 구분 기준으로 사용합니다. False로 설정하면 Advertise ID를 구분 기준으로 사용합니다.
아래 코드는 User ID를 사용자 구분 기준으로 사용하는 경우입니다.
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:
 (NSDictionary *)launchOptions
{
  ……
  int result = [TAGAnalytics initializeSdk:@”AppKey”
					 companyId:@”CompanyID”
					appVersion:@”AppVersion”
				 useLoggingUserId:YES];
  if (result != 0) {
	 // Initialize Fail
  }

  // 게임에서 로그인 처리 완료
  ……
  // User ID를 사용자 구분 기준으로 사용하는 경우 User ID를 등록하는 함수.
  [TAGAnalytics setLoggingUserId:@”user id”]; 
……
}
        
만약 초기화 함수의 마지막 인자(use logging userid flag)를 true로 설정한 경우 setLoggingUserId를 호출하여 User ID를 등록해야 합니다. Flag를 true로 설정하고 setLoggingUserId를 호출하지 않으면 이후에 호출하는 모든 API가 실패(E_LOGGING_USER_ID_EMPTY)를 Return 합니다.
setUserId에서 “useCampaignOrPromotion”은 Promotion이나 Campaign을 사용하는 경우 true입니다. 그렇지 않은 경에는 false 입니다.
setUserId 함수는 initializeSDK 호출 이후에 게임에서 로그인 성공한 후 게임에서 사용하는 userID를 획득한 직후에 호출하면 됩니다. userID는 게임에서 사용자를 구분하기 위해 사용하는 값을 사용하면 됩니다.
Advertise ID 관련 내용은 아래 링크를 참고하세요. 
https://developer.apple.com/LIBRARY/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/index.html

세션 추적

DAU(Daily Active User)와 게임 체류 시간을 추적하기 위한 연동입니다. 
App 시작/종료, Background/Foreground 이동시 해당 액션에 맞는 API를 호출하여 측정할 수 있습니다. 
App이 처음 실행될때(initializeSDK 이후) 또는 background에서 foreground로 이동시 traceActivation을 호출하여 세션 추적을 시작합니다. 이후 App이 background로 들어가는 시점에 traceDeactivation을 호출하여 세션 추적을 멈춥니다. 
traceDeactivation을 호출하면 traceActivation과 traceDeactivation 사이의 시간을 계산하여 게임 이용 시간을 측정합니다. 또한 SDK 내부적으로 수행하던 작업도 traceDeactivation에서 중단합니다. 
Background/Foreground 이동시 위 함수를 호출하지 않으면 정확한 게임 이용 시간을 측정할 수 없기 때문에 이 API는 반드시 호출해야 합니다. 
DAU는 하루동안 traceActivation을 호출한 사용자(Advertise ID 또는 User ID 기준)의 중복을 제거한 숫자로 계산합니다.
- (void)applicationWillResignActive:(UIApplication *)application
{
	[TAGAnalytics traceDeactivation];
}

- (void)applicationDidBecomeActive:(UIApplication *)application
{
	[TAGAnalytics traceActivation];
}
        
액션 추적

In-App Purchase, 머니 획득/사용, 레벨업, 친구 수 변경등 사용자의 Action에 대해 추적할 수 있습니다.

1. In-App Purchase
In-App Purchase가 발생한 후 tracePurchase를 호출하여 매출 정보를 전송합니다. 
Currency는 ISO-4217(http://en.wikipedia.org/wiki/ISO_4217)에서 정의한 코드를 사용합니다. 
$0.99의 보석을 구매하는 경우 아래와 같이 사용합니다. 
(여기에서 “GEM_10”은 게임에서 정의한 Item의 Code입니다. Unit Cost는 해당 아이템의 단위 가격, Payment는 실제 사용자가 사용한 금액입니다. Level은 구매한 사용자의 Level을 입력합니다.)
[TAGAnalytics tracePurchase:@”GEM_10”
payment:0.99
unitCost:0.99
currency:@”USD”
 level:10];
        
2. 재화 획득/사용
게임내에서 재화의 획득/사용시 호출합니다. 1차 재화, 2차 재화의 변동량을 추적합니다. 일반적으로 1차 재화는 In-App Purchase를 통해서 구매하는 재화(ex. 보석, 루비등) 입니다. 2차 재화는 1차 재화를 이용하여 구매하는 재화(ex. 체리, 하트등) 입니다.
IAP를 통해서 보석 10개를 구매한 경우 아래와 같이 사용합니다. 
(“CODE_IAP”는 게임에서 정의한 Code입니다. 1차 재화인 경우 Type은 0, 2차 재화인 경우 1을 사용합니다)
[TAGAnalytics traceMoneyConsumption:@”CODE_IAP”
	  type:@”0”
consumptionAmount:10
	 level:10];
        
보석 10개를 이용하여 체리 100개를 구매한 경우는 아래와 같이 사용합니다.
// 1차 재화 사용
[TAGAnalytics traceMoneyConsumption:@”CODE_USE_GEM”
				  type:@”0”
		  consumptionAmount:10
				 level:10];

// 2차 재화 획득
[TAGAnalytics traceMoneyAcquisition:@”CODE_BUY_CHERRY”
				  type:@”0”
		  consumptionAmount:10
				 level:10];
        
1차 재화를 사용하여 2차 재화를 구입한 경우 실제 ‘1차 재화 감소’->‘2차 재화 증가’가 발생합니다. 하지만 2차 재화를 구입하기 위해서 1차 재화를 사용하는 경우 별도의 재화 소모로 판정하지 않고 싶은 경우 ‘2차 재화 획득’ 로그만 전송하여도 됩니다.

3. 레벨업
사용자 레벨이 변경되는 경우 traceLevelUp을 호출합니다. 참고로 대부분의 액션 추적 API는 레벨별 액션 추적을 위해서 사용자 Level을 같이 받습니다. 
사용자 레벨이 10으로 변경되는 경우 아래와 같이 호출합니다. 한 사용자의 레벨은 반드시 증가해야만 합니다. 감소하는 경우 정확한 데이터 측정이 불가능합니다. 
예를들어 “Candy Crush Sage”와 같이 스테이지로 진행 되는 게임에서 스테이지를 레벨로 사용하는 경우에는 해당 스테이지에 최초 진입할때만 레벨업 로그를 남겨야 합니다. 만약 이전 스테이지로 다시 돌아가서 플레이 하는 경우에는 레벨업 로그를 남기지 않습니다, 
또한 다른 API에 전달되는 level 값도 현재 진행중인 스테이지가 아닌 사용자의 최고 스테이지를 레벨 값으로 사용해야 합니다.
[TAGAnalytics traceLevelUp:10];
4. 친구
사용자의 친구 숫자를 등록합니다. 일반적으로 앱 실행 후 친구 정보 로딩이 완료된 시점에 호출하면 됩니다.
[TAGAnalytics traceFriendCount:100]; 
선택적 연동

기본연동을 통해 제공되는 기본 지표 이외에 게임에서 추가적으로 측정하고 싶은 지표가 있는경우, 혹은 Analytics에서 제공하는 Campaign 기능을 사용하는 방법에 대해 설명합니다.

추가적으로 연동할 수 있는 항목은 아래와 같습니다.
Campaign 연동 : setCampaignListener, (show/hide)Campaign
Custom Event 수집 : traceEvent
소요시간 측정 : traceStartSpeed, traceEndSpeed
캠페인 연동

1. 캠페인 연동 사전 준비
캠페인 연동 및 실행을 위해서는 별도의 가이드를 제공하고 있습니다. 
Toast Analytics의 “캠페인 실행” 메뉴의 “페이지 가이드”를 참고하세요. 
(http://analytics.toast.com/promotion/share/document/4.2_Campaign_run.pdf)

2. 푸시 연동
Analytics SDK에서 캠페인을 위해서 APNS를 사용합니다. APNS를 사용하기 위해서는 Analytics 관리자 페이지에 인증서를 등록해야 합니다.
Analytics 관리자 페이지에 정보를 등록하는 방법은 “Getting Started” 문서에서 “푸시 설정” 항목을 참고하세요.
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)
launchOptions
{
  ……
  // SDK 초기화
  [TAGAnalytics initializeSdk:@”AppKey”
			 companyId:@”CompanyID”
			appVersion:@”AppVer”
		 useLoggingUserId:NO];
  ……

  // 푸시를 받은 경우 푸시 데이터를 SDK에 전달
  if (launchOptions != nil)
  {
	NSDictionary* dictionary = [launchOptions objectForKey:UIApplicationLaunchOptions
	RemoteNotificationKey];
	if (dictionary != nil)
	  [TAGAnalytics setPushData:dictionary];
  }

  // 푸시 토큰을 APNS에 요청한다.
  if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0)
	{
	[[UIApplication sharedApplication] registerUserNotificationSettings:[UIUserNotificationSettings 
	settingsForTypes:(UIUserNotificationTypeSound | UIUserNotificationTypeAlert |
	UIUserNotificationTypeBadge) 
	categories:nil]];
	[[UIApplication sharedApplication] registerForRemoteNotifications];
  }
  else
  {
	[[UIApplication sharedApplication] registerForRemoteNotificationTypes:
	(UIUserNotificationTypeBadge | UIUserNotificationTypeSound | 
	UIUserNotificationTypeAlert)];
  }
  ……
}

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:
  (NSData *) deviceToken
{
  // 푸시 토큰이 정상적으로 등록된 경우 토큰을 SDK에 전달한다.
  [TAGAnalytics tracePushToken:deviceToken];
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
{
  // 푸시를 받은 경우 푸시 데이터를 SDK에 전달
  [TAGAnalytics setPushData:userInfo];
}
        
3. 캠페인 Listener 구현 및 등록
SDK는 일정한 주기로 캠페인 서버와 통신하여 캠페인 및 보상정보를 가져옵니다. 만약에 현재 사용자에게 진행할 캠페인이 있거나 사용자가 받을 보상 정보가 있는 경우 TAGCampaignDelegate를 통해서 알려줍니다.
따라서 캠페인 정보를 받기 위해서는 TAGCampaignDelegate를 구현해야 합니다.
@protocol TAGCampaignDelegate <NSObject>
-(void)analyticsDidMissionComplete:(NSArray*)missionList;
-(void)analyticsDidCampaignVisibilityChange:(NSString*)adspaceName show:(BOOL)show;
-(void)analyticsDidCampaignLoadSuccess:(NSString*)adspaceName;
-(void)analyticsDidCampaignLoadFail:(NSString*)adspaceName errorCode:(int)errorCode 
  errorMessage:(NSString*)errorMessage;
-(void)analyticsDidPromotionVisibilityChanged:(BOOL)show;
@end
        
각 Callback은 아래와 같은 경우 호출됩니다
analyticsDidCampaignVisibilityChange: showCampaign, hideCampaign을 호출하여 캠페인 관련 팝업이나 배너가 보이거나 사라질때 호출됩니다.
analyticsDidMissionComplete: 사용자가 캠페인/프로모션을 진행하고 정해진 미션을 달성하여 보상 정보가 있을때 호출됩니다. 여기에서 받은 정보를 이용하여 게임서버를 통해서 보상을 지급해야 합니다. 캠페인 보상 관련 프로세스는 “캠페인 적용 가이드”문서를 참고하세요
analyticsDidCampaignLoadSuccess, analyticsDidCampaignLoadFail: 서버에서 가져 온 캠페인 정보 파싱 결과를 알려줍니다. 게임에서는 이 Callback에서 특별한 처리를 할 필요는 없습니다. 로그 확인을 위해 제공하는 Callback입니다. 
구현한TAGCampaignDelegate는 setCampaignDelegate를 이용하여 등록합니다.
analyticsDidPromotionVisibilityChange: iOS에서는 현재 사용하지 않습니다.
- (void)viewDidLoad {
  ……
  [TAGAnalytics setCampaignDelegate:self];
  ……
}

……

-(void)analyticsDidMissionComplete:(NSArray *)missionList
{
  // missionList Array는 NSString으로 mission string입니다.
  // mission string은 key/value 정보가 구분자 ‘|’으로 제공됩니다.
  // 이 값을 가지고 게임서버를 통해 promotion server에서 검증하여 사용자에게
  // 보상을 지급합니다.
}

- (void)analyticsDidCampaignVisibilityChange:(NSString *)adspaceName show:(BOOL)show
{
  // 배너,팝업 Visibility Change
}

- (void)analyticsDidCampaignLoadSuccess:(NSString *)adspaceName
{
  // for Debugging
}

- (void)analyticsDidCampaignLoadFail:(NSString *)adspaceName errorCode:(int)errorCode errorMessage:
(NSString *)errorMessage
{
  // for Debugging
}
        
4. 캠페인 Show/Hide
현재 사용자에게 진행 중인 캠페인이 있는 경우 Analytics 웹사이트에서 등록한 캠페인 팝업/배너를 보여주고, 노출된 팝업/배너를 숨기는 메소드입니다. 파라미터인 adspaceName은 Analytics 웹사이트에서 캠페인 등록시에 정의한 adspace 이름을 사용하면 됩니다. Adspace란 팝업/배너가 나타나는 게임 내의 특정 위치를 의미합니다. 
showCampaign() 메소드는 해당 adspace를 사용하는 캠페인이 없으면 아무런 동작도 하지 않으므로, 캠페인 팝업/배너를 노출할 것으로 예상되는 게임 내의 여러 지점에 각각 다른 adspaceName으로 함수를 호출해두면 이후에 별도의 게임 클라이언트 수정없이도 게임 운영자가 Analytics 웹사이트에서 캠페인을 등록하는 작업만으로 쉽게 팝업/배너를 노출할 수 있게 됩니다. 
Adspace 등록 방법은 “캠페인 테스트 가이드”를 참고하세요.
+(int)showCampaign:(NSString*)adspaceName parent:(UIView*)parent;
+(int)showCampaign:(NSString*)adspaceName parent:(UIView*)parent animation:(int)animation 
  lifeTime:(int)lifeTime;
+(int)hideCampaign:(NSString*)adspaceName;
+(int)hideCampaign:(NSString*)adspaceName animation:(int)animation;
        
커스텀 이벤트 사용

게임별로 특정 이벤트를 정의하여 분석하고 싶은 경우 사용합니다.
예를들어 Fever Time Item을 사용하는 경우 아래와 같이 사용합니다. 사용된 모든 코드는 게임에서 정의하여 사용합니다. 아래 예제는 특정 스테이지에서 아이템 변동 내용을 추적하기 위해 정의한 코드입니다.
[TAGAnalytics traceEvent:@”ITEM”
eventCode:@”ITEM_USE”
param1:@”FEVER”
param2:@”STAGE_10”
value:1
level:10];
        
특정 레벨에서 보스 배틀 결과를 추적할때도 사용할 수 있습니다.
[TAGAnalytics traceEvent:@”STAGE”
eventCode:@”STAGE_BOSS_VICTORY”
param1:@”DRAGON_VALLEY”
param2:@”BOSS_MOB”
value:1
level:10];
        
이 외에도 다양한 용도로 게임에 특화된 이벤트 추적에 사용할 수 있습니다. 
traceEvent에 사용하는 String Type 파라미터(event type, event code, param1, param2)는 각각 50byte까지 사용할 수 있습니다. 그리고 event 하위에 발생 가능한 param1 은 300개까지, 또 param1 하위에 발생 가능한 param2는 200개까지 사용할 수 있습니다. 
자세한 내용은 Toast Analytics 사이트에서 가이드를 참고하세요. (커스텀 이벤트 페이지 우측 상단의 “페이지 가이드”를 클릭하면 다운받을 수 있습니다)

소요시간 측정

특정 구간에 소요되는 시간을 측정할 수 있습니다. 예를들어 튜토리얼에 소요되는 시간을 측정하고 싶은경우, Scene 전환에 걸리는 시간을 측정하고 싶은경우등 시간 측정이 필요한 임의의 구간에 사용할 수 있습니다. 
Intro Scene 로딩 시간을 측정하고 싶은 경우 아래와 같이 사용합니다. “INTRO_LOADING”은 특정 구간에 대해서 게임에서 정의하는 값입니다.
- (void) onStart() {
  [TAGAnalytics traceStartSpeed:@"INTRO_LOADING"];
}

- (void) onLoadCompleted() {
  [TAGAnalytics traceEndSpeed:@"INTRO_LOADING"];
}
        
SDK 설정

1. 디버그 모드 활성화
개발중에 SDK 로그 확인을 위해서 로그 출력 여부를 설정할 수 있습니다. 
이 함수는 initializeSDK 이전/이후 모두 호출 가능합니다. 기본 값은 setDebugMode(false)입니다. 
Log tags는 “Analytics:”로 시작합니다. Eclipse에서 log cat filter를 “Analytics”로 지정하면 SDK에서 발생하는 로그를 확인할 수 있습니다.)
- (void) Start () {
  ……
 [TAGAnalytics setDebugEnabled:YES];
  ……
}
        
디버그 모드가 활성화된 경우 로그 전송 내용을 확인할 수 있습니다. 로그를 전송하고 그에 대한 응답 로그를 확인하여 로그가 정상적으로 전송되었는지 확인할 수 있습니다. 아래와 같은 로그 스트링이 있으면 수집된 데이터가 정상적으로 서버로 전송된 것입니다. (***은 상황에 따라서 다른 값입니다) 
iOS : RequestWorkerThread::didReceiveResponse - <NSHTTPURLResponse: ***> { URL: *** } { status code: 200,

2. 디바이스 정보 확인
SDK에서 수집하는 Device 정보를 확인할 수 있습니다.
현재 확인 가능한 값은 Device ID, Push Token, Campaign User ID입니다. 이 값들은 캠페인 연동 테스트시 필요합니다. 자세한 내용은 “캠페인 연동 가이드”를 참고하세요.
private void printDeviceInfo() {
NSString *deviceID = [TAGAnalytics deviceInfoWithKey:DEVICE_INFO_DEVICEID];
NSString *pushToken = [TAGAnalytics deviceInfoWithKey:DEVICE_INFO_TOKEN;
NSString *cUserID = [TAGAnalytics deviceInfoWithKey:DEVICE_INFO_CAMPAIGN_USERID];
 ……
}
        
3. SDK 버전 확인
SDK 버전은 “getVersion()” 함수를 통해 확인할 수 있습니다.
[TAGAnalytics version];
API Reference

클라이언트 SDK API 목록

클라이언트 SDK는 표 01과 같은 API를 제공한다. 각 API에 대한 상세 설명은 표0.2 아래 내용을 참고 하시기 바랍니다.

[표 01 클라이언트 SDK API 목록]
클라이언트 SDK API 목록
API	설명
resultStringFromCode	SDK API 호출 결과값을 오류 메시지로 변환한다
initializeSdk	클라이언트 SDK 모듈을 초기화
hideCampaign	해당 Campaign 뷰를 숨긴다.
hideReward	해당 Reward 뷰를 숨긴다.
setCampaignDelegate	Campaign 노출 관련 상태를 비동기적으로 통지받는 델리게이트 등록
showCampaign	해당 Campaign 뷰의 노출을 요청한다. 관련 리소스가 준비되지 않은 준비가될 때 까지 노출이 지연되며, 노출시 setCampaignDelegate에 등록한 델리게이트를 통해 통지 받을수 있다.
showReward	해당 Reward 의 노출을 요청한다. 관련 리소스가 준비되지 않은않은 준비가될 때 까지 노출이 지연되며, 노출시 setCampaignDelegate에 등록한 리스너를 통해 통지 받을수 있다.
setDebugModeEnabled	디버깅 메시지를 디버깅 콘솔에 노출한다. 릴리즈 버전의 경우 반드시 false로 세팅하도록 한다.
setCampaignUserId	게임에서 사용하는 사용자 User Id를 세팅한다.(캠페인용)
setLoggingUserId	게임에서 사용하는 사용자 User Id를 세팅한다.(로깅용)
traceActivation	앱이 포그라운드로 활성화 될 때 호출한다.
traceDeactivation	앱이 백그라운드로 비활성화 될 때 호출한다.
traceFriendCount	게임내 사용자의 친구수를 알려줄 때 호출한다.
tracePurchase	앱내에서 아이템을 구매했을 때 호출한다.
traceMoneyAcquisition	Money를 획득했을 때 호출한다
traceMoneyConsumption	Money를 소진했을 때 호출한다.
traceLevelUp	레벨업이 되었을 때 호출한다.
traceEvent	특정한 이벤트가 발생했을 때 호출한다.
traceStartSpeed	특정 구간의 시간 소모를 측정하고자할 때 시작 시점에 호출한다.
traceEndSpeed	특정 구간의 시간 소모를 측정하고자할 때 끝 시점에 호출한다.
[표 02 클라이언트 SDK 공통 Return Value]
클라이언트 SDK 공통 Return Value 목록
Return Value	값	설명
S_SUCCESS	0x0000	성공
W_ALREADY_INITIALIZED	0x1000	SDK가 이미 초기화됨.
E_NOT_INITIALIZED	0x8000	SDK가 초기화 되지 않은 상태에서 api 가 호출됨.
E_SESSION_CLOSED	0x8001	traceStart 가 호출되지 않은 상태에서 api 가 호출됨.
E_INVALID_PARAMS	0x8002	유효하지 않은 인자값이 전달됨.
E_ALREADY_EXISTS	0x8003	동일한 screenCode 값으로 traceStartSpeed가 2회이상 호출됨.
E_INTERNAL_ERROR	0x8004	내부에러
E_INSUFFICIENT_OPERATION	0x8005	traceStartSpeed가 호출되지 않은 screenCode값으로 traceEndSpeed가 호출됨.
E_APP_ID_IS_EMPTY	0x8006	SDK 초기화시 필수 입력값인 앱 ID 값이 NULL임.
E_ENTERPRISE_ID_IS_EMPTY	0x8007	SDK 초기화시 필수 입력값인 Company ID 값이 NULL임.
E_APP_VERSION_IS_EMPTY	0x8008	SDK 초기화시 필수 입력값인 앱 버전값이 NULL임.
E_TOKEN_EMPTY	0x8009	디바이스 토큰 값이 NULL 임.
E_PARENT_EMPTY	0x800A	UIView가 NULL인 상태로 showCampaign/showReward를 호출했음.
E_LOGGING_USER_ID_IS_EMPTY	0x800B	initializeSdk 에서 useLoggingUserId를 YES로 설정한후 setLogginUserId를 호출하지 않은 상태에서 trace 메소드를 호출했음.
E_CAMPAIGN_SHOW_EXPIRED	0x7000	만료기간이 지난 캠페인에 대해 요청함.
E_CAMPAIGN_SHOW_ALREADY	0x7001	이미 표출되고 있는 캠페인 뷰를 show 요청하거나, 표출되거나 큐잉되지 않은 캠페인 뷰를 hide 요청함.
E_CAMPAIGN_SHOW_PENDING	0x7002	이미 다른 캠페인 뷰가 표출되고 있어서 캠페인 뷰 show 요청이 큐잉됨.
E_CAMPAIGN_SHOW_FAIL	0x7003	캠페인 뷰의 리소스중 일부를 로드할수 없음.
E_CAMPAIGN_SHOW_BLOCKED	0x7004	사용자에 의해 차단되었거나 완료된 캠페인에 대해 show 요청을 했음.
E_CAMPAIGN_NOTEXIST	0x7005	존재하지 않는 adspaceName나 promotionId로 show 요청을 했음.
E_CAMPAIGN_DISABLED	0x7006	사용자가 광고ID를 사용을 비활성화 했음.
E_CAMPAIGN_USER_ID_IS_EMPTY	0x7007	사용자 user id가 세팅되지 않았음.
resultStringFromCode

Description
int형 SDK API 호출 결과값을 오류 메시지로 변환한다.

Syntax
            + (NSString*)resultStringFromCode:(int)resultCode;
        
Parameters
resultCode 
[in] API 호출 결과로 받은 리턴값

Return Values
입력값에 해당하는 오류 메시지.
[참고]
표 0 2 클라이언트 SDK 공통 Return Value를 참조한다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceActivation];
            if (result != 0)
            {
                  NSLog(@”traceActivation fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
None

initializeSdk

Description
SDK 를 초기화 하는 API로써 다른 모든 API 호출에 선행되어야한다.

Syntax
            + (int)initializeSdk:(NSString*)appId enterprizeId:(NSString*)entId appVersion:
               (NSString*)appVersion useLoggingUserId:(BOOL)useLoggingUserId;
        
Parameters
appId 
[in] 앱의 고유 ID값.
entId 
[in] 앱을 소유한 회사의 고유 ID값.
appVersion 
[in] 앱의 버젼. 
useLoggingUserId 
[in] user id를 사용해 로그 추적을 사용할지 유무.
[참고]
appId와 entId 값은 앱 등록 사이트에서 앱별/회사별로 부여받는 고유의 값이다.
userLoggingUserId 를 YES로 설정하면 setLoggingUserId 로 user id를 설정하기 전까지 모든 trace 메소드는 실패하게된다.
Return Values
호출 성공 여부를 가리키는 값.
[참고]
이 API 호출은 항상 성공한다.
Remarks
None

Example Code
            int result = [TAGAnalytics initializeSdk:@”test_app_id” enterprizeId:@”test_ent_id” 
            appVersion:@”1.0.0” useLoggingUserId:YES];
            if (result != 0)
            {
                  NSLog(@”initializeSdk fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
None

hideCampaign

Description
노출된 캠페인뷰를 숨기도록 한다.

Syntax
            + (int)hideCampaign:(NSString*)adspaceName animation(int)animation;
        
Parameters
adspaceName 
[in] 캠페인 노출 위치를 가리키는 고유 값.
[참고]
showCampaign Queue에 동일한 adspaceName 으로 중첩되어 요청되어 있을경우 가장 오래된 campaign 순으로 숨김 처리한다.
animation 
[in] 캠페인 숨김시 애니메이션 효과.
[참고]
ANIMATION_NONE : 애니메이션 효과 없음.
ANIMATION_FADE : Fade-Out 효과로 숨긴다.
ANIMATION_SLIDE : Slide 효과로 숨긴다.
Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_CAMPAIGN_SHOW_ALREADY : 없는 캠페인뷰에 대한 숨김 요청.
Remarks
None

Example Code
            int result = [TAGAnalytics hideCampaign:@”SCENE_00” animation:ANIMATION_SLIDE];
            if (result != 0)
            {
                  NSLog(@”hideCampaign fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
None

hideReward

Description
노출된 보상뷰를 숨기도록 한다.

Syntax
            + (int)hideReward:(NSString*)rewardId animation(int)animation;
        
Parameters
rewardId 
[in] 캠페인 보상을 가리키는 고유 값.
[참고]
showReward Queue에 동일한 rewardId 로 중첩되어 요청되어 있을경우 가장 오래된 reward 순으로 숨김 처리한다.
animation 
[in] 보상뷰 숨김시 애니메이션 효과.
[참고]
ANIMATION_NONE : 애니메이션 효과 없음.
ANIMATION_FADE : Fade-Out 효과로 숨긴다.
ANIMATION_SLIDE : Slide 효과로 숨긴다.
Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_CAMPAIGN_SHOW_ALREADY : 없는 보상뷰에 대한 숨김 요청.
Remarks
None

Example Code
            int result = [TAGAnalytics hideReward:@”55” animation:ANIMATION_SLIDE];
            if (result != 0)
            {
                  NSLog(@”hideCampaign fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
showReward, showCampaign, hideCampaign

setCampaignDelegate

Description
캠페인 노출 관련 상태 정보를 비동기적으로 통지받는 리스너를 등록한다.

Syntax
            + (void)setCampaignDelegate:(id<TAGCampaignDelegate> campaignDelegate;
        
Parameters
campaignDelegate 
[in] 캠페인 노출 상태가 바뀔 때 호출될 델리게이트.
[참고]
onReward : 리워드 Id 리스트가 수신되었음.
onCampaignVisibilityChanged : 캠페인의 노출 상태가 바뀜.
onCampaignLoadSuccess : 캠페인을 노출하는데 필요한 리소스 준비가 완료됨.
onCampaignLoadFail : 캠페인을 노출하는데 필요한 리소스를 준비하는동안 오류가 발생하였음.
Return Values
호출 성공 여부를 가리키는 값.
[참고]
이 API 호출은 항상 성공한다.
Remarks
None

Example Code
            int result = [TAGAnalytics setCampaignListener:listener];
            if (result != 0)
            {
                  NSLog(@”setCampaignListener fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
showCampaign, hideCampaign

showCampaign

Description
캠페인 뷰를 노출하도록 한다.

Syntax
            + (int)showCampaign:(NSString*)adspaceName parent:(UIView*)parent 
               animation(int)animation lifetime:(int)lifeTime;
        
Parameters
adspaceName 
[in] 캠페인 노출 위치를 가리키는 고유 값.
[참고]
캠페인은 동시에 1개의 뷰만 노출된다. 만약 여러 개의 캠페인을 노출 요청한다면 현재 노출된 뷰 이외의 것들은 Queue에 저장되고 현재 노출된 뷰가 사라지면 자동으로 순서에 따라 노출되게된다.
parent 
[in] 캠페인뷰가 노출될 부모 UIView.
animation 
[in] 캠페인 숨김시 애니메이션 효과.
[참고]
ANIMATION_NONE : 애니메이션 효과 없음.
ANIMATION_FADE : Fade-In 효과로 나타난다.
ANIMATION_SLIDE : Slide 효과로 나타난다.
lifeTime 
[in] 캠페인 노출후 일정시간 이후 자동 숨김
[참고]
0 : 자동 숨김 없음.
-	N(0보다 큰 정수) : N 밀리세컨트 이후 자동 숨김...
Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_CAMPAIGN_SHOW_ALREADY : 이미 동일한 캠페인뷰가 노출된 상태.
E_CAMPAIGN_SHOW_EXPIRED : 만료된 캠페인에 대한 요청.
E_CAMPAIGN_SHOW_PENDING : 이미 다른 캠페인뷰가 노출중인 상태이며 앞서 요청된 캠페인뷰가 hide되면 노출되게됨.
Remarks
None

Example Code
            int result = [TAGAnalytics showCampaign:@”SCENE_00” parent:self animation:ANIMATION_SLIDE lifetime:3000];
            if (result != 0)
            {
                  NSLog(@”showCampaign fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
hideCampaign, showReward, hideReward

showReward

Description
보상뷰를 노출하도록 한다.

Syntax
            + (int)showReward:(NSString*)rewardId parent:(UIView*)parent animation(int)animation lifetime:(int)lifeTime;
        
Parameters
rewardId 
[in] 보상뷰를 가리키는 고유 값.
[참고]
보상뷰는 동시에 1개의 뷰만 노출된다. 만약 여러 개의 보상뷰를 노출 요청한다면 현재 노출된 뷰 이외의 것들은 Queue에 저장되고 현재 노출된 뷰가 사라지면 자동으로 순서에 따라 노출되게된다.
parent
[in] 보상뷰가 노출될 부모 UIView.

animation
[in] 보상뷰 숨김시 애니메이션 효과.
[참고]
ANIMATION_NONE : 애니메이션 효과 없음.
ANIMATION_FADE : Fade-In 효과로 나타난다.
ANIMATION_SLIDE : Slide 효과로 나타난다.
lifeTime
[in] 보상뷰 노출후 일정시간 이후 자동 숨김.
[참고]
0 : 자동 숨김 없음.
N(0보다 큰 정수) : N 밀리세컨트 이후 자동 숨김...
Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_CAMPAIGN_SHOW_ALREADY : 이미 동일한 보상뷰뷰가 노출된 상태.
E_CAMPAIGN_SHOW_EXPIRED : 만료된 보상에 대한 요청.
E_CAMPAIGN_SHOW_PENDING : 이미 다른 보상뷰가 노출중인 상태이며 앞서 요청된 보상뷰가 hide되면 노출되게됨.
Remarks
None

Example Code
            int result = [TAGAnalytics showReward:@”55” parent:self animation:ANIMATION_SLIDE lifetime:3000];
            if (result != 0)
            {
                  NSLog(@”showCampaign fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
hideReward, showCampaign, hideCampaign

setDebugModeEnabled

Description
디버깅 메시지를 디버깅 콘솔에 출력할지 여부를 지정한다. 릴리즈 버전의 경우 반드시 false로 세팅하도록 한다.

Syntax
            + (void)setDebugModeEnabled:(BOOL)enable;
        
Parameters
enable 
[in] 디버깅 메시지 출력 여부.

Return Values
None

Remarks
None

Example Code
            [TAGAnalytics setDebugModeEnabled:YES];
        
See Also
None

setCampaignUserId

Description
캠페인용으로 게임에서 사용하는 사용자 User Id를 세팅한다.

Syntax
            + (int)setCampaignUserId:(NSString*)userId;
        
Parameters
userId 
[in] 사용자 User Id.

Return Values
호출 성공 여부를 가리키는 값.
[참고]
이 API 호출은 항상 성공한다.
Remarks
None

Example Code
            int result = [TAGAnalytics setCampaignUserId:@”user@domaign.com”];
            if (result != 0)
            {
                  NSLog(@” setUserId fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
None

setLoggingUserId

Description
로깅용으로 게임에서 사용하는 사용자 User Id를 세팅한다.

Syntax
           + (int)setLoggingUserId:(NSString*)userId;
        
Parameters
userId 
[in] 사용자 User Id.

Return Values
호출 성공 여부를 가리키는 값.
[참고]
이 API 호출은 항상 성공한다.
Remarks
None

Example Code
            int result = [TAGAnalytics setLoggingUserId:@”user@domaign.com”];
            if (result != 0)
            {
                  NSLog(@” setUserId fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
None

traceActivation

Description
앱이 포그라운드로 활성화 될 때 호출한다.

Syntax
            + (int)traceActivation;
        
Parameters
None

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_NOT_INITIALIZED : SDK가 초기화 되지 않은 상태에서 호출되었다.
E_SESSION_CLOSED : traceStart 가 호출되지 않은 상태에서 호출되었다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceActivation];
            if (result  != 0)
            {
                  NSLog(@”traceActivation fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initializeSdk, traceStart, traceDeactivation

traceDeactivation

Description
앱이 백라운드로 비활성화 될 때 호출한다.

Syntax
            + (int)traceDeactivation;
        
Parameters
None

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_NOT_INITIALIZED : SDK가 초기화 되지 않은 상태에서 호출되었다.
E_SESSION_CLOSED : traceStart 가 호출되지 않은 상태에서 호출되었다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceDeactivation];
            if (result  != 0)
            {
                  NSLog(@”traceDeactivation fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initializeSdk, traceStart, traceActivation

traceFriendCount

Description
앱내에서 친구수를 로깅할때 호출한다.

Syntax
            + (int)traceFriendCount:(int)friendCount;
        
Parameters
friendCount 
[in] 친구수.

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_NOT_INITIALIZED : SDK가 초기화 되지 않은 상태에서 호출되었다.
E_SESSION_CLOSED : traceStart 가 호출되지 않은 상태에서 호출되었다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceFriendCount:10];
            if (result  != 0)
            {
                  NSLog(@” traceFriendCount fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initializeSdk

tracePurchase

Description
앱내에서 아이템을 구매했을 때 호출한다.

Syntax
            + (int)tracePurchase:(NSString*)itemCode payment:(flat)payment unitCost:(float)unitCost 
               currency:(NSString*)currency level:(int)level;
        
Parameters
itemCode 
[in] 상품 코드
payment 
[in] 지불 금액
unitCost 
[in] 상품 단가
currency 
[in] 통화
level 
[in] 레벨

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_NOT_INITIALIZED : SDK가 초기화 되지 않은 상태에서 호출되었다.
E_SESSION_CLOSED : traceStart 가 호출되지 않은 상태에서 호출되었다.
Remarks
None

Example Code
            int result = [TAGAnalytics tracePurchase:@”item00” payment:0.99 unitCost:0.99 currency:@”USD” level:5];
            if (result  != 0)
            {
                  NSLog(@”tracePurchase fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initializeSdk, traceStart

traceMoneyAcquisition

Description
앱내에서 Money를 획득했을 때 호출한다.

Syntax
            + (int)traceMoneyAcquisition:(NSString*)usageCode type:(NSString*)type 
               acquisitionAmount:(int)acquisitionAmount level:(int)level;
        
Parameters
usageCode 
[in] 사용처 코드.
type 
[in] 재화 타입(0:1차 재화, 1:2차 재화).
acquisitionAmount 
[in] 획득 수량.
level 
[in] 레벨

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_NOT_INITIALIZED : SDK가 초기화 되지 않은 상태에서 호출되었다.
E_SESSION_CLOSED : traceStart 가 호출되지 않은 상태에서 호출되었다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceMoneyAcquisition:@”usage00” type@”0” acquisitionAmount:3 level:1];
            if (result  != 0)
            {
                  NSLog(@”traceMoneyAcquisition fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initinitializeSdk, traceStart, traceMoneyConsumption

traceMoneyConsumption

Description
앱내에서 Money를 소진했을 때 호출한다.

Syntax
            + (int)traceMoneyConsumption:(NSString*)usageCode type:(NSString*)type 
               consumptionAmount:(int)consumptionAmount level:(int)level;
        
Parameters
usageCode 
[in] 사용처 코드.
type 
[in] 재화 타입(0:1차 재화, 1:2차 재화).
consumptionAmount 
[in] 소비 수량.
level 
[in] 레벨

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_NOT_INITIALIZED : SDK가 초기화 되지 않은 상태에서 호출되었다.
E_SESSION_CLOSED : traceStart 가 호출되지 않은 상태에서 호출되었다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceMoneyAcquisition:@”usage00” type@”0” consumptionAmount:1 level:1];
            if (result  != 0)
            {
                  NSLog(@”traceMoneyAcquisition fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initializeSdk, traceStart, traceMoneyAcquisition

traceLevelUp

Description
레벨업이 되었을 때 호출한다.

Syntax
            + (int)traceLevelUp:(int)level;
        
Parameters
level 
[in] 레벨

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_NOT_INITIALIZED : SDK가 초기화 되지 않은 상태에서 호출되었다.
E_SESSION_CLOSED : traceStart 가 호출되지 않은 상태에서 호출되었다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceLevelUp:5];
            if (result  != 0)
            {
            NSLog(@”traceLevelUp fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initializeSdk, traceStart

traceEvent

Description
특정한 이벤트가 발생했을을 때 호출한다.

Syntax
            + (int)traceEvent:(NSString*)eventType eventCode
            :(NSString*)eventCode param1:(NSString*)param1 param2:(NSString*)param2 
             value:(NSString*)value level:(int)level;
        
Parameters
eventType 
[in] 이벤트 유형.
eventCode 
[in] 이벤트 코드.
param1 
[in] 파라미터1.
param2 
[in] 파라미터2.
value 
[in] 값.
level 
[in] 레벨.

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_NOT_INITIALIZED : SDK가 초기화 되지 않은 상태에서 호출되었다.
E_SESSION_CLOSED : traceStart 가 호출되지 않은 상태에서 호출되었다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceEvent:@”event_type” eventCode:@”event_code” 
            param1:@”param1” param2:@”param2” value:@”value” level:5];
            if (result  != 0)
            {
                  NSLog(@”traceEvent fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initializeSdk, traceStart

traceStartSpeed

Description
특정 구간의 시간 소모를 측정하고자 할 때 시작 시점에 호출한다.

Syntax
            + (int)traceStartSpeed:(NSString*)intervalName;
        
Parameters
intervalName 
[in] 특정 구간을 가리키는 키값.

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_ALREADY_EXISTS : 이미 존재하는 screenCode로 중복 호출되었다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceStartSpeed:@”SCENE00_LOAD”];
            if (result  != 0)
            {
                  NSLog(@”traceStartSpeed fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initializeSdk, traceStart, traceEndSpeed

tracEndSpeed

Description
특정 구간의 시간 소모를 측정하고자 할 때 끝 시점에 호출한다.

Syntax
            + (int)traceEndSpeed:(NSString*)intervalName;
        
Parameters
intervalName 
[in] 특정 구간을 가리키는 키값.

Return Values
호출 성공 여부를 가리키는 값.
[참고]
E_NOT_INITIALIZED : SDK가 초기화 되지 않은 상태에서 호출되었다.
E_SESSION_CLOSED : traceStart 가 호출되지 않은 상태에서 호출되었다.
E_INSUFFICIENT_OPERATION : 해당 screenCode에 대해 traceStartSpeed 가 호출되지 않았다.
Remarks
None

Example Code
            int result = [TAGAnalytics traceEndSpeed:@”SCENE00_LOAD”];
            if (result  != 0)
            {
                  NSLog(@”traceEndSpeed fails : %@”, [TAGAnalytics resultStringFromCode:result]);
            }
        
See Also
initializeSdk, traceStart, traceStartSpeed

FAQ
네트워크 환경이 안좋아도 로그 수집에 문제가 없나요?

모든 로그는 내부적으로 백업되게됩니다. 네트워크 환경이 좋지 않아 전송되지 않은 로그는 네트워크 환경이 좋아지게되면 자동적으로 재전송하게됩니다. 단, 24시간이 지난 로그는 자동 폐기되게 됩니다.
로그 데이터를 전송하는중 앱이 종료되면 어떻게 되나요?

모든 로그는 내부에 백업되어 있기 때문에 앱이 종료되어 전송에 실패한 로그는 다음에 앱이 실행되면 자동으로 재전송되게 됩니다. 단, 24시간이 지난 로그는 자동 폐기되게 됩니다.
안드로이드의 경우 앱의 종료 시점을 명확히 알기 어려운데 앱의 종료 로그 수집에 문제 없나요?

안드로이드에서 앱의 종료 로그가 명확하지 않아도 다른 로그들의 전송 패턴을 분석하여 앱의 종료 시점을 합리적으로 추론하여 분석에 반영하고 있습니다.
수집가능한 데이터에는 어떤 것들이 있나요?

앱설치 정보, 아이템 구매/획득/소비 정보, 레벨업정보, 이벤트 추적, 클라이언트 씬 로딩시간, 캠페인 노출/클릭/달성 정보를 수집할 수 있으며, 회원ID 와 같은 유저 정보 또한 수집할 수 있습니다.
SDK 에서 기본적으로 수집하는 데이터 이외에 사용자 정의 데이터도 수집가능한가요?

사용자 정의 데이터 타입에 대해서는 수집이 불가능합니다.
