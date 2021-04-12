README
===
iOS 10버전 이상 권장합니다.

FRAMEWOKR 추가
===

![img1](https://user-images.githubusercontent.com/73524723/114341872-b2f89500-9b95-11eb-9109-266d621d034c.png)


__TARGET > General > Frameworks, Libraries, and Embedded Content__ 에 DILO에서 제공한 framework를 Drag & Drop해서 포함시킨후 __Embed__ 형태로 사용.


![img2](https://user-images.githubusercontent.com/73524723/114341896-c3107480-9b95-11eb-8cb4-eb0463e0adad.png)

__TARGETS > Build Phases > Link Binary With Libraries__ 에 DILO에서 제공한 framework를 추가한 후 Status 값 형태를 __Required__ 로 선택.

광고 요청 샘플코드(SWFIT)
===
```swift
import DiloSDK

class ViewController: UIViewController {
  // 광고 관리자 인스턴스
  let adManager: AdManager = AdManager.sharedInstance
  
  override func viewDidLoad() {
    super.viewDidLoad()
    
    // (optional) Companion 광고를 사용하는경우
    // 설정하지 않을경우 companion 광고 노출하지 않음 
    adManager.setCompanionSlot(companionView)
    
    // (optional) 광고 스킵버튼을 사용하는경우
    // 스킵버튼을 받는 목적 -> 스킵가능 여부에 따른 버튼 표시/숨김 자동 
    // 설정하지 않고 직접구현 가능 @see AdManager#onSkipEnabled
    adManager.setSkipButton(companionView)
    
    // 광고요청시 받아온 광고가 0개일경우 호출 (호출되었을경우 본컨텐츠 재생권장) 
    adManager.onNoFill {
      ...
    }
    
    // 광고요청 후 응답된 광고가 있을경우 호출 
    adManager.onAdReady {
      ...
      self.adManager.start()
    }
    
    // 재생중인 광고가 스킵이 가능할경우 호출 // adIndex -> zero base index
    adManager.onSkipEnabled { (adIndex: Int) in
      ...
    }

    // 광고 재생시 광고에대한 정보 리스너 설정
    // 광고가 n개있을경우 n번 호출 
    adManager.onAdStart { (adInfo: AdInfo) in
      ...
    }

    // 광고가 재생중일때 진행상태를 0.3초마다 호출
    adManager.onTimeUpdate { (progress: DiloSDK.Progress) in
      ...
    }

    // 재생중인 광고가 일시중지되었을 경우 
    adManager.onPause {
      ...
    }
  
    // 일시중지되었던 광고가 재개되었을경우 
    adManager.onResume {
      ...
    }
    
    // 광고가 재생완료되었을경우 호출 
    // 광고가 n개 있을경우 n번 호출 
    adManager.onAdCompleted {
      ...
    }

    // 모든광고가 재생되었을경우 한번 호출 
    adManager.onAllAdsCompleted {
      ...
    }

    // 에러발생시 호출 (호출되었을경우 본컨텐츠 재생권장) 
    adManager.onError { (errorMessage: String) in
      ...
    } 
  }

  // 광고 요청 프로세스
  @IBAction func adRequestAction() {

    // 광고 요청 파라미터
    let adRequestParam = RequestParam(
      // 앱 번들 식별자(Bundle.main.bundleIdentifier) 
      bundleId: bundleId,

      // DILO와 협의된 컨텐츠의 식별값
      // 예) https://test.com/audio/episode.mp3 
      epiCode: epiCode,

      // 광고타입 설정. RequestParam.ProductType 참조
      // .DILO -> 오디오만 나오는 광고
      // .DILO_PLUS -> 오디오 + 컴패니언 광고
      // .DILO_PLUS_ONLY -> 오디오 + 컴패니언 광고 (컴패니언이 무조건 포함)
      productType: productType,

      // 광고 갯수 타입 설정. RequestParam.FillType 참조
      // .SINGLE -> 단일 광고 요청 drs에 맞는 길이의 광고요청
      // .MULTI -> 아래 drs값의 길이에 맞게 광고요청
      // .SIINGLE_ANY -> drs와 상관없이 6초, 10초, 15초 랜덤의 단일 광고요청
      fillType: fillType,

      // 광고시간 설정
      // RequestParam.FillType.SINGLE_ANY일 경우 nil설정 
      drs: drs
    )

    do {
      // 위에 설정한 값을 토대로 광고요청
      // 필수값 설정이 잘못되었을 경우 에러(AdRequestError 참조)
      // .InvalidRequestOption -> 요청정보가 잘못되었을 경우
      // .InvalidADServerURL -> 광고 서버 URL이 잘못설정 되었을 경우(DILO에 문의)
      try admanager.requestAd(adRequestParam) { (result: Bool) in // 광고요청 결과
        // true 광고요청에 받아온 광고가 1개이상 있음
        // false 광고요청에 받아온 광고가 0개 
        if result {
          // (optional) 광고를 받아온 후 바로 광고재생을 원할경우 
          self.admanager.start()
        } 
      }
    } catch let err {
      ...
    } 
  }

  // 광고 스킵요청
  @IBAction func adSkipAction() {
    // 현재 재생중인 광고 스킵요청
    // 광고스킵이 된 경우 true
    // 스킵이 가능하나 스킵 가능한시간이 아닐경우 false 리턴 // 스킵이 불가능한 광고는 항상 false 리턴
    let skipResult = adManager.skip()
    ...
  }
  
  // 광고재생 중지/재생
  @IBAction func resumeOrStopAction() {
    // 중지되었을경우 0
    // 재생시 1
    let result = adManager.playOrPause() 
    ...
  }

  // 컴패니언광고를 사용하고 리로드가 필요한경우 
  @IBAction func reloadCompanionAd() {
    adManager.reloadCompanion(companionView)
  }

  // 광고 종료(사용자 Action에 따라 호출하는 것은 권장되지 않음) 
  @IBAction func adStopAction() {
    // 광고 종료
    adManager.stop()
  }
}

```

광고 요청 샘플코드(Objective-C)
===
```objectivec
#import <DiloSDK/DiloSDK-Swift.h>
@interface ViewController()
// 광고 관리자 인스턴스
@property AdManager *adManager; ...
@end
@implementation ViewController

...

- (void) viewDidLoad {
  [super viewDidLoad];
  self.adManager = AdManager.sharedInstance;
  // (optional) Companion 광고를 사용하는경우 
  // 설정하지 않을경우 companion 광고 노출하지 않음
  [self.adManager setCompanionSlot: companionView];

  // (optional) 광고 스킵버튼을 사용하는경우
  // 스킵버튼을 받는 목적 -> 스킵가능 여부에 따른 버튼 표시/숨김 자동 
  //설정하지 않고 직접구현 가능 @see AdManager#onSkipEnabled
  [self.adManager setSkipButton: skipButton];
  
  // 광고요청시 받아온 광고가 0개일경우 호출 (호출되었을경우 본컨텐츠 재생권장) 
  [self.adManager onNoFill: ^() {
    ...
  }];

  // 광고요청 후 응답된 광고가 있을경우 호출 
  [self.adManager onAdReady: ^() {
    ...
    [self.adManager start];
  }];
  
  // 재생중인 광고가 스킵이 가능할경우 호출
  // adIndex -> zero base index
  [self.adManager onSkipEnabled: ^(NSInteger adIndex) {
    ...
  }];

  // 광고 재생시 광고에대한 정보 리스너 설정
  // 광고가 n개있을경우 n번 호출
  [self.adManager onAdStart: ^(AdInfo *adInfo) {
    ...
  }];
  
  // 재생중인 광고가 일시중지되었을 경우 
  [self.adManager onPause: ^() {
    ...
  }];

  // 일시중지되었던 광고가 재개되었을경우 
  [self.adManager onResume: ^() {
    ...
  }];
  
  // 광고 재생시 광고에대한 정보 리스너 설정
  // 광고가 n개있을경우 n번 호출
  [self.adManager onTimeUpdate: ^(Progress *progress) {
    ...
  }];
  
  // 광고가 재생완료되었을경우 호출
  // 광고가 n개 있을경우 n번 호출 
  [self.adManager onAdCompleted: ^() {
    ...
  }];
  
  // 모든광고가 재생되었을경우 한번 호출 
  [self.adManager onAllAdsCompleted: ^() {
    ...
  }];
  
  // 에러발생시 호출 (호출되었을경우 본컨텐츠 재생권장) 
  [self.adManager onError: ^(NSString *errorMessage)
    ...
  }]; 
}

// 광고 요청 프로세스
- (IBAction) adRequest {
  // 광고 요청 파라미터
  RequestParam *adRequest = [[RequestParam alloc]
  
    // 앱 번들 식별자(NSBundle.mainBundle.bundleIdentifier) 
    initWithBundleId: bundleId
    
    // DILO와 협의된 컨텐츠의 식별값
    // 예) https://test.com/audio/episode.mp3
    epiCode: epiCode
  
    // 광고시간 설정
    // FillTypeSINGLE_ANY일 경우 nil설정
    drs: drs
    
    // 광고타입 설정. ProductType 참조
    // ProductTypeDILO -> 오디오만 나오는 광고
    // ProductTypeDILO_PLUS -> 오디오 + 컴패니언 광고
    // ProductTypeDILO_PLUS_ONLY -> 오디오 + 컴패니언 광고 (컴패니언이 무조건 포함) 
    productType: productType
    
    // 광고 갯수 타입 설정. FillType 참조
    // FillTypeSINGLE -> 단일 광고 요청 drs에 맞는 길이의 광고요청
    // FillTypeMULTI -> 위 drs값의 길에 맞게 광고요청
    // FillTypeSINGLE_ANY -> drs와 상관없이 6초, 10초, 15초 랜덤의 단일 광고요청 
    fillType: fillType
  ]
  
  NSError *err;
  // 위에 설정한 값을 토대로 광고요청
  // 필수값 설정이 잘못되었을 경우 에러(AdRequestError 참조)
  // .InvalidRequestOption -> 요청정보가 잘못되었을 경우
  // .InvalidADServerURL -> 광고 서버 URL이 잘못설정 되었을 경우(DILO에 문의) 
  [self.adManager requestAd: adRequest error: &err :^(bool result) {
    // 광고요청 결과
    // true 광고요청에 받아온 광고가 1개이상 있음 
    // false 광고요청에 받아온 광고가 0개
    if (result) {
      [self.adManager start];
    }
  }];
}

// 광고 스킵요청
- (IBAction) adSkipAction {
  // 현재 재생중인 광고 스킵요청
  // 광고스킵이 된 경우 true
  // 스킵이 가능하나 스킵 가능한시간이 아닐경우 false 리턴 
  // 스킵이 불가능한 광고는 항상 false 리턴
  bool skipResult = [self.adManager skip]; 
  ...
}

// 광고재생 중지/재생
- (IBAction) resumeOrStopAction {
  // 중지되었을경우 0
  // 재생시 1
  bool result = [self.adManager playOrPause]; 
  ...
}

// 컴패니언광고를 사용하고 리로드가 필요한경우 
- (IBAction) reloadCompanion {
  [self.adManager reloadCompanion: companionView];
}

// 광고 종료(사용자 Action에 따라 호출하는 것은 권장되지 않음) 
- (IBAction) adStopAction {
  // 광고종료
  [self.adManager stop];
}
```

클래스 명세(SWIFT)
===

Class AdManager
---

Property
|Name|Type|Description|
|---|:---:|:---|
|sharedInstance|`AdManager`|싱글톤 인스턴스|

Method
|Name|Parameter|Type|Description|
|---|:---:|:---:|:---|
|setCompanionSlot|\_: `UIVIew`|`Void`|컴패니언 view 설정 <br />\*argument로 넘긴 UIView에 대한 숨김/보임 처리는<br />SDK에서 자동으로 처리.<br />처음 set되었을 경우, 기본적으로 숨김처리 상태|
|setSkipButton|\_: `UIVIew`|`Void`|광고 스킵버튼 설정|
|requestAd|\_: `RequestParam`<br />\_: (`Bool`) -> `Void`|`Void`|광고요청<br />콜백 리스너를 통해서 요청결과값 리턴 throws `AdRequestError`|
|start| |`Void`|광고 재생|
|playOrPause| |`Void`|광고 중지/재생|
|skip| |`Bool`|현재 재생중인 광고 스킵요청<br />스킵이 불가능할 경우 `false` 리턴<br />\*스킵이 불가능한 경우<br />1. 스킵이 n초이고 광고 프로세스가 n보다 작을때<br />2. 스킵자체가 불가능한 광고일 경우<br /><br />\*setSkipButton(UIView)을 이용하여 스킵버튼을 지정한경우 직저적으로 구현하지 않아도 됩니다.|
|stop| |`Void`|광고 종료<br />받아온 광고를 모두 종료한다.<br />\*사용자가 직접호출할 수 없도록 권장함.|
|reloadCompanion|\_: `UIView`|`Void`|현재 광고에 포함되어있는 컴패니언을 다시 불러온다|
|onAdReady|\_: () -> `Void`|`Void`|광고요청 후 응답된 광고가 1개 이상일 경우 호출|
|onAdStart|\_: (`AdInfo`) -> `Void`|`Void`|광고가 시작되었을 경우 리스너 호출.<br />\*광고가 n개일 경우 n번 호출|
|onTimeUpadate|\_: (`DiloSDK.Progress`) -> `Void`|`Void`|광고가 재생중일때 0.3초마다 리스너 호출|
|onPause|\_: () -> `Void`|`Void`|재생중인 광고가 일시중지 되었을 경우 호출<br />\*playOrpause() 함수를 직접 호출 했을경우에만 발생하는 이벤트.|
|onResume|\_: () -> `Void`|`Void`|일시중지 되었던 광고가 재개되었을 경우 호출<br />\*playOrPuase() 함수를 직접 호출 했을경 우에만 발생하는 이벤트.|
|onSkipEnabled|\_: (`int`) -> `Void`|`Void`|현재 재생중인 광고가 스킵이 가능해졌을 경 우 리스너 호출|
|onAdCompleted|\_: () -> `Void`|`Void`|재생중인 광고가 완료되었을때 호출.<br />광고개 n개있을경우 n번 호출|
|onAllAdsCompleted|\_: () -> `Void`|`Void`|응답받은 모든 광고가 재생완료 되었을 경우 호출|
|onError|\_: () -> `Void`|`Void`|광고 실행중 에러가 발생했을 경우 호출|
