README
===
iOS 11버전 이상 권장합니다.

FRAMEWORK 추가
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
    // `DILO_PLUS_ONLY`로 광고요청할 경우 필수값
    adManager.setCompanionSlot(companionView)
    
    // (optional) 광고 스킵버튼을 사용하는경우
    // 스킵버튼을 받는 목적 -> 스킵가능 여부에 따른 버튼 표시/숨김 자동 
    // 설정하지 않고 직접구현 가능 @see AdManager#onSkipEnabled
    adManager.setSkipButton(companionView)
    
    // (optional) 컴패니언 닫기버튼구현
    adManager.setCloseButton(closeButton)
    
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
    
    // 컴패니언이 사용자에의해 닫혔을경우 호출
    adManager.onCompanionClosed {  
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
      // 예) test.com/audio/episode.mp3 
      // URL일경우 http|https의 스키마는 빼고 광고요청하는것을 권장
      epiCode: epiCode,
      
      // 광고요청한 에피소드가 속해있는 채널명
      // 추후 리포트에 반영되는 값
      // 채널명이 없을경우 리포트에 보여질 임의값 설정
      chName: "",
      
      // 광고요청한 에피소드의 타이틀
      // 추후 리포트에 반영되는 값
      // 에피소드명이 없을경우 리포트에 보여질 임의값 설정
      epName: "",
      
      // 광고요청한 에피소드의 창작자명
      // 추후 리포트에 반영되는 값
      // 창작자명이 없을경우 리포트에 보여질 임의값 설정
      creatorName: "",
      
      // 광고요청한 에피소드의 창작자 중복되지 않는 고유식별값
      // DILO가 아닌 구현하는 쪽에서 식별할수있는 값
      // 예) `user#0001`, `12391`, `user@dilo.co.kr` ...
      // 중복될 경우 같은 창작자로 인식하여 리포트 집계
      creatorIdentifier: "",
      
      // 광고시간 설정
      // RequestParam.FillType.SINGLE_ANY일 경우 nil설정 
      drs: drs,

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

      // 광고를 호출한 시점에서의 위치
      // 에피소드를 시작하기전에 호출할경우 .PRE
      // 에피소드 중간에 광고를 호출할경우 .MID
      // 에피소드가 끝난후 호출할경우 .POST
      adPosition: adPosition
    )

    adManager.requestAd(adRequestParam)
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
|requestAd|\_: `RequestParam`<br />\_: (`Bool`) -> `Void`|`Void`|광고요청<br />콜백 리스너를 통해서 요청결과값 리턴|
|start| |`Void`|광고 재생|
|playOrPause| |`Void`|광고 중지/재생|
|skip| |`Bool`|현재 재생중인 광고 스킵요청<br />스킵이 불가능할 경우 `false` 리턴<br />\*스킵이 불가능한 경우<br />1. 스킵이 n초이고 광고 프로세스가 n보다 작을때<br />2. 스킵자체가 불가능한 광고일 경우<br /><br />\*setSkipButton(UIView)을 이용하여 스킵버튼을 지정한경우 직저적으로 구현하지 않아도 됩니다.|
|stop| |`Void`|광고 종료<br />받아온 광고를 모두 종료한다.<br />\*사용자가 직접호출할 수 없도록 권장함.|
|reloadCompanion|\_: `UIView`|`Void`|현재 광고에 포함되어있는 컴패니언을 다시 불러온다|
|getAdState| |`Float`|현재 광고상태 확인.<br />-1: 광고를 요청하지 않은 상태, 광고를 요청했으나 노필인 상태, 혹은 그외에 광고를 재생할 수 없는 상태<br />0: 광고를 받아왔고 재생할 수 있는 상태 혹은 광고가 일시중지 상태<br />1: 광고가 재생중인 상태|
|onAdReady|\_: () -> `Void`|`Void`|광고요청 후 응답된 광고가 1개 이상일 경우 호출|
|onAdStart|\_: (`AdInfo`) -> `Void`|`Void`|광고가 시작되었을 경우 리스너 호출.<br />\*광고가 n개일 경우 n번 호출|
|onTimeUpadate|\_: (`DiloSDK.Progress`) -> `Void`|`Void`|광고가 재생중일때 0.3초마다 리스너 호출|
|onPause|\_: () -> `Void`|`Void`|재생중인 광고가 일시중지 되었을 경우 호출<br />\*playOrpause() 함수를 직접 호출 했을경우에만 발생하는 이벤트.|
|onResume|\_: () -> `Void`|`Void`|일시중지 되었던 광고가 재개되었을 경우 호출<br />\*playOrPuase() 함수를 직접 호출 했을경 우에만 발생하는 이벤트.|
|onSkipEnabled|\_: (`int`) -> `Void`|`Void`|현재 재생중인 광고가 스킵이 가능해졌을 경 우 리스너 호출|
|onAdCompleted|\_: () -> `Void`|`Void`|재생중인 광고가 완료되었을때 호출.<br />광고개 n개있을경우 n번 호출|
|onAllAdsCompleted|\_: () -> `Void`|`Void`|응답받은 모든 광고가 재생완료 되었을 경우 호출|
|onError|\_: () -> `Void`|`Void`|광고 실행중 에러가 발생했을 경우 호출|

Class RequestParam
---

Property
|Name|Type|Description|
|---|:---:|:---|
|bundleId|`String`|번들 식별값<br />Bundle.main.bundleIdentifier|
|epiCode|`String`|DILO와 광고하기로 약속된 컨텐츠 식별값<br />\*보통은 컨텐츠 URL|
|drs|`NSNumber?`|drs(초)만큼 길이의 광고 응답<br />\*fillType이 .SINGLE_ANY일 경우 nil값 셋팅|
|productType|`RequestParam.ProductType`|.DILO: 오디오 광고<br />.DILO_PLUS: [오디오, 오디오 + 컴패니언] 랜덤<br />.DILO_PLUS_ONLY: 오디오 + 컴패니언 광고
|fillType|`RequestParam.FillType`|.SINGLE: drs(초) 길이의 단일 광고 요청<br />.MULTI: drs(초) 길이의 한개 이상의 광고 요청<br />.SINGLE_ANY: 6, 10, 15초 랜덤 단일광고 요청

Method
|Name|Parameter|Type|Description|
|---|:---:|:---:|:---|
|init|bundleId: `String`<br />epiCode: `String`<br />drs: `NSNumber?`<br />productType: `ProductType`<br />fillType: `FillType`|`RequestParam`|`RequestParam`인스턴스 생성자|

Enum RequestParam.ProductType
---

Property
|Name|Type|Description|
|---|:---:|:---|
|DILO| |오디오광고만 요청|
|DILO_PLUS| |오디오, (오디오 + 컴패니언) 광고 랜덤 요청|
|DILO_PLUS_ONLY| |오디오 + 컴패니언 광고 요청|


Enum RequestParam.FillType
---

Property
|Name|Type|Description|
|---|:---:|:---|
|SINGLE| |drs(초) 길이의 단일 광고 요청|
|MULTI| |총 drs(초) 길이의 한개 이상의 광고요청|
|SINGLE_ANY| |drs(초)와 관계없이 6, 10, 15초 단일 광고 랜덤 요청|


Enum RequestParam.AdPositionType
---

Property
|Name|Type|Description|
|---|:---:|:---|
|PRE| |에피소드 시작전에 광고 호출한경우|
|MID| |에피소드 플레이중에 광고를 호출한 경우|
|POST| |에피소드가 끝난후 광고를 호출한 경우|


Enum AdRequeastError
---

Property
|Name|Type|Description|
|---|:---:|:---|
|InvalidADServerURL| |SDK에 설정된 광고 서버 URL이 잘못되었을경우 (해당 경우에는 DILO 담당자에게 연락)|
|InvalidRequestOption| |광고 요청에 필요한 필수값을 잘못 입력하였을 경우|



Class Progress
---

Property
|Name|Type|Description|
|---|:---:|:---|
|current|`int`|현재 재생중인 광고의 순서<br />1 Base Index|
|total|`int`|요청에 응답된 광고 총 갯수|
|duration|`Double`|현재 재생중인 광고의 총 길이(초)|
|seconds|`Double`|현재 재생중인 광고의 현재시간(초)|


Class AdInfo
---

Property
|Name|Type|Description|
|---|:---:|:---|
|adType|`String`|광고 타입<br />- audio: 오디오 광고<br />- hybrid: 오디오 + 컴패니언 광고|
|advertiser|`String`|광고주명|
|title|`String`|광고명|
|current|`Int`|광고의 순서<br />1 Base Index|
|total|`Int`|요청에 응답된 광고 총 갯수|
|duration|`Double`|광고의 총 길이(초)|
|skipTime|`Double`|광고 스킵가능한 시간(초)<br />\*0일경우 스킵불가|
|hasCompanion|`Bool`|컴패니언 광고여부|





