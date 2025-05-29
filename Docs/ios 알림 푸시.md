>[!question]
>GQ1. 푸시 알림 기능을 구현하려면 무엇을 알아야 하는가?
>GQ2. 각각의 개념들을 알고 싶다.

## Description
### **iOS 원격 푸시 알림 공부 로드맵**
1. **APNs(Apple Push Notification service) 개념**
    - 애플의 공식 푸시 알림 전달 시스템. 모든 원격 푸시 알림은 APNs를 통해 전달됨.
2. **푸시 알림 권한 요청**
    - 사용자가 앱에서 알림 수신을 허용할 수 있도록 권한을 요청해야 함.
3. **Device Token 발급 및 관리**
    - 앱이 APNs에 등록하고, APNs가 디바이스 토큰(알림 수신을 위한 주소)을 발급해줌.
4. **푸시 알림 Capabilities 설정 (Xcode)**
    - Xcode에서 프로젝트의 Capabilities에서 “Push Notifications” 활성화, App ID/프로비저닝 프로파일 설정.
5. **서버(백엔드) 준비**
    - APNs에 푸시 메시지를 보낼 서버 필요.
    - Firebase Cloud Messaging(FCM), Node.js, Python 등 다양한 백엔드 사용 가능.
6. **인증서/키 등록 (APNs 인증)**
    - 서버가 APNs에 연결하려면 인증서(.p8 키 등) 등록 필요.
7. **푸시 메시지 전송 구현**
    - 서버가 디바이스 토큰을 사용해 APNs로 메시지 전송.
8. **iOS 앱에서 푸시 알림 수신 처리**
    - 앱이 포그라운드/백그라운드/종료 상태에서 알림을 어떻게 처리하는지 코드로 구현.
9. **알림 페이로드(내용) 이해**
    - 푸시 알림은 JSON 형태로 보내지며, 타이틀, 메시지, 배지, 사운드, 커스텀 데이터 등 포함 가능.
10. **알림 권한/설정 변경 처리**
    - 사용자가 알림을 거부했을 때, 설정으로 유도하거나 안내 메시지 보여주기.

## 주요 기능
## **1. APNs(Apple Push Notification service) 개념**
### **● APNs란?**
- **Apple Push Notification service**의 약자.
- 애플이 직접 운영하는 **공식 푸시 알림 전달 서비스**.
- **iOS, iPadOS, watchOS, macOS** 등 애플 생태계에서 푸시 알림을 보낼 때 반드시 APNs를 거쳐야 함.
### **● 동작 구조**
1. **앱**이 기기에 설치되어 있고, 푸시 알림을 사용하도록 등록함.
2. **기기**가 APNs에 등록 → APNs가 **Device Token**(디바이스 식별용 토큰) 발급.
3. **앱 서버(백엔드)**가 이 토큰을 저장.
4. 알림이 필요할 때 **앱 서버가 APNs에 요청**(디바이스 토큰 + 메시지).
5. **APNs가 해당 기기로 알림 전달**.
6. **기기에서 알림을 수신**하고, 시스템 또는 앱이 사용자에게 표시.

### **● 왜 APNs를 써야 할까?**
- **보안:** 애플이 중간에서 인증/암호화, 전달을 모두 보장.
- **신뢰성:** 전 세계 모든 애플 기기에 빠르고 안정적으로 알림 전달.
- **에너지 절약:** 백그라운드에서 비효율적으로 네트워크 연결을 유지하지 않아도 됨(시스템 차원 관리).

### **● 주요 특징**
- 푸시 알림은 **인터넷이 연결된 상태에서만** 전달됨.
- 알림 종류:
    - **알림(Notification):** 배너/알림 센터에 표시
    - **사운드(Sound):** 소리 재생
    - **배지(Badge):** 앱 아이콘 우측에 뱃지 숫자
    - **커스텀 데이터:** 앱에 전달되는 JSON 데이터 등
- 사용자는 알림 권한을 직접 허용/거부할 수 있음.

### **● 용어 요약**
- **Device Token:** 기기마다 APNs에서 발급해주는 유일한 식별자. 서버에서 이 토큰을 사용해 특정 기기에 알림을 보냄.
- **Payload:** 실제로 전달되는 알림의 내용(메시지, 사운드, 뱃지, 데이터 등).

### **● 용어 요약**
- **Device Token:** 기기마다 APNs에서 발급해주는 유일한 식별자. 서버에서 이 토큰을 사용해 특정 기기에 알림을 보냄.
- **Payload:** 실제로 전달되는 알림의 내용(메시지, 사운드, 뱃지, 데이터 등).

### **● 기타**
**1.** **APNs는 ‘웹사이트’가 아니라 ‘클라우드 서비스’**
- APNs는 **Apple Push Notification service**의 약자로, 애플이 운영하는 ‘클라우드 서버’(인터넷 상의 서비스)야.
- 웹사이트처럼 접속해서 보는 화면이 있는 게 아니라, **앱이나 서버가 ‘네트워크 통신’을 통해 접속**하는 서비스라고 생각하면 돼.
- 때문에 따로 해야할 작업이 있는 것은 아니고 앱단에서 APNs에 접근해서 토큰을 얻어오는 식.
**2. 실제로 개발 코드에서는**
 iOS 푸시 알림 권한 및 Device Token 받기 예시
```
// 권한 요청
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
}
// 여기서 iOS가 APNs에 등록 요청!
UIApplication.shared.registerForRemoteNotifications()

// 여기서 deviceToken을 받음!
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
// 서버로 전송!
}
```


### **2. 푸시 알림 권한 요청**
### ● 개념
• iOS에서는 사용자 동의 없이 알림을 보낼 수 없음.
• 앱이 처음 실행될 때(또는 필요할 때), 알림을 허용할지 묻는 팝업을 띄워야 함.
• 사용자가 ‘허용’해야만 푸시 알림 수신 가능!

● 동작 흐름
1. 앱에서 권한 요청 코드를 실행
2. 팝업이 뜸
• “이 앱이 알림을 보내려고 합니다. 허용하시겠습니까?”
3. 사용자가 허용 or 거부 선택
4. 결과(허용/거부)를 앱에서 확인 가능

● 예시 코드 (Swift)
```
import UserNotifications
UNUserNotificationCenter.current().requestAuthorization(
    options: [.alert, .sound, .badge]
) { granted, error in
    if granted {
        print("알림 권한 허용됨")
    } else {
        print("알림 권한 거부됨")
    }
}
```

옵션 설명
• .alert: 배너, 알림센터 표시 허용
• .sound: 소리 알림 허용
• .badge: 앱 아이콘 우측 배지(숫자) 표시 허용


● 팁 & 주의사항
• 권한 요청은 보통 앱 최초 실행 시 한 번만!
• 사용자가 ‘거부’하면, 나중에 시스템 설정에서만 바꿀 수 있음(앱 내에서 팝업 다시 띄울 수 없음).
• 권한 요청 전에 왜 필요한지 안내 문구를 먼저 보여주는 게 유저 경험에 좋음.
• 알림은 info를 건들지 않아도 됨


● 권한 상태 확인 및 설정 이동
사용자가 알림을 꺼뒀을 때 설정으로 이동하도록 안내할 수도 있어:
```
if let appSettings = URL(string: UIApplication.openSettingsURLString) {
    UIApplication.shared.open(appSettings)
}
```

● 한 줄 요약
iOS 앱은 반드시 사용자의 동의를 받아야만 푸시 알림을 보낼 수 있으며,
이를 위해 requestAuthorization 메서드를 사용한다!

### **3. Device Token 발급 및 관리**
● Device Token이란?
• Device Token은
iOS 기기마다 APNs가 발급해주는 고유 식별자(주소)야.
• 이 토큰은
“이 기기로 푸시 알림을 보내주세요!”
라고 APNs에 요청할 때 사용하는 주소 역할을 해.

● 발급 과정
1. 앱이 APNs에 등록 요청
(UIApplication.shared.registerForRemoteNotifications() 실행)
2. APNs가 Device Token 발급
(네트워크를 통해 기기-APNs 간에 자동 처리)
3. iOS가 Device Token을 앱에게 전달
(AppDelegate의 didRegisterForRemoteNotificationsWithDeviceToken 메서드에서 수신)

● 예시 코드 (Swift)
```
// AppDelegate.swift

// 1. APNs 등록 요청 후, 토큰 수신 콜백
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    // 2. deviceToken은 Data 타입, 보통 서버 전송 전에 String으로 변환
    let tokenParts = deviceToken.map { data in String(format: "%02.2hhx", data) }
    let tokenString = tokenParts.joined()
    print("Device Token: \(tokenString)")

    // 3. 서버로 deviceToken 전송 (API 호출 등)
}
```

● 관리 방법
• 앱 서버에 Device Token 저장
• 보통 앱 최초 실행 시, 또는 토큰이 갱신될 때 서버에 전송
• 서버는 이 토큰을 이용해 푸시 알림 발송 대상 관리
• 토큰 갱신 감지
• iOS는 가끔 deviceToken을 자동으로 갱신(보안, 시스템 사정)
• 토큰이 변경될 때마다 항상 서버에 최신 토큰으로 갱신 필요

● 주의할 점
• 디바이스마다 토큰이 다름 (앱/기기/설치/빌드마다 다를 수 있음)
• Simulator의 토큰은 실제 푸시 수신에 쓸 수 없음
(테스트는 실제 기기에서 진행!)
• 사용자가 알림 권한을 거부해도, 토큰은 발급 가능
(단, 실제 알림 수신은 불가)

● 기타
• APNs에서 주는 토큰을 서버(백엔드)에 직접 적어주는 것이 아니라
앱 내에 서버 API로 토큰을 주는 방식

📢 한 줄 요약
Device Token은 “내 기기로 푸시를 보내주세요!”라는 주소표이고,
항상 서버에 저장/관리해야 푸시 알림을 보낼 수 있다!


### **4. 푸시 알림 Capabilities 설정 (Xcode)**
**1. 왜 해야 할까?**
• iOS 앱이 푸시 알림 기능을 사용하려면,
Xcode(프로젝트 설정)에서 “이 앱은 푸시 알림을 쓸 거예요!”라고 명시해야
실제 기기/프로비저닝 프로파일에서 알림 기능이 활성화됨.
• 앱의 번들 ID, 인증서, 프로비저닝 프로파일 등도 이 설정을 기준으로 맞춰짐.

**2. 설정 방법**
① Xcode에서 Capabilities 켜기
1. Xcode에서 프로젝트 열기
2. 왼쪽에서 앱 Target 선택
(보통 맨 위에 있는 프로젝트명을 클릭 → Targets 중 앱 선택)
3. 탭 메뉴 중 ‘Signing & Capabilities’ 선택
4. + Capability 버튼 클릭
(또는, “Capability 추가”라고 보일 수도 있음)
5. Push Notifications 검색/선택
(목록에서 바로 찾을 수 있음)
6. 추가되면 Push Notifications 항목이 리스트에 들어감
(별다른 옵션 없이 켜지기만 하면 됨!)

② 앱 아이디(Apple Developer) 설정도 확인
• Xcode에서 Capabilities를 켜면,
Apple Developer Center
App ID에 자동으로 “Push Notifications” 기능이 활성화됨.
• 가끔 자동 연동이 안 될 때는,
Apple Developer Center에서 App ID를 찾아
직접 “Push Notifications”을 켜야 할 때도 있음.

③ 프로비저닝 프로파일 재생성(필요할 때만)
• Capabilities를 추가한 뒤,
프로비저닝 프로파일을 새로고침(재생성)해서 내려받아야 할 때도 있음.
• (앱 배포/실기기 테스트할 때, 이 부분이 제대로 안 되어 있으면
푸시 알림 등록이 실패할 수 있음!)

**3. 정리**
• Xcode에서 Capabilities(기능)로 Push Notifications를 활성화한다.
• 이 설정은 앱의 인증서, 배포, 실제 푸시 등록에 반드시 필요하다.
• 이 과정이 끝나면 앱이 iOS 푸시 알림 기능을 사용할 “자격”을 얻게 됨!

**📌 실습 TIP**
• 꼭 무료 계정이 아닌 유료 계정이어야만 사용할 수 있다.
• 설정 후 바로 실기기에서 알림 권한 요청/토큰 발급까지 잘 되는지 확인해보면 좋아!
• 프로필/앱ID 문제로 등록이 안 되면
“권한 거부”나 “푸시 등록 실패” 에러가 뜨기도 하니
Capabilities와 Developer Center 상태를 꼭 점검해.

