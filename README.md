## 재난문자 푸시 알림 구현

### 1) 구현 기능

### 2) 기본 개념

#### (1) Remote Notification
- 불시에 발생한 업데이트 
- 불특정 시간 
- 예측 불가하므로 local static 하게 작성할 수 없다
- 따라서 원격 알림을 보낼 Provider(Server) 필요

<img src=“https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/remote_notif_simple_2x.png”>
- Provider (Server)
- APNS

#### (2) APNS
> Apple Push Notification Service

원격 알림을 사용할 때에 꼭 거쳐야 하는 핵심 서비스
서버에서 바로 기계로 보내지 않고 반드시 APNS를 거친다

- 저장 외 전달 기능을 수행하는 QOS 포함
전달받을 기기가 오프라인 상태인 경우 일정 기간동안 메세지를 보관하고
기기가 Online 상태가 되었을 때 전달하고 기기, 앱 별로 가장 최근의 알림만 저장
각 서버에서 보내는 알림을 최신 상태로 저장 하다가 장비의 오프라인 시간이 길어지면 모든 알림을 삭제

=> 기기 상태에 따라 최신의 알람을 보내도록 관리

- 보안 
특정 사용자에게만 전달되어야 하는 정보가 제3자에 보내지지 않게 
메세지가 오염되지 않게 
자체 보안 아키텍처를 통해 보안 강화
    - *Connection trust*
        - 연결 신뢰
        - provider-to-APNs connection trust (애플과 계약을 맺은 서버만)
            - token-based: valid authentication key certificate
            - certificate-based: SSL certificate
        - APNs-to-device connection trust
    - *Device token*
        - 각 원격 알림에서 End to End (제공자와 디바이스) 
        - 디바이스가 APNs 에게 받은 고유 token을 provider에 전달
        - 고유 식별자 NSData
        - APNs 만 해석 가능 
        - provider는 각각의 push 알림에 token 을 포함시켜 전송한다.
        - 상황에 따라 새롭게 발급 가능 (새 장치, 백업 복원, iOS 업데이트)

<참고 자료>
https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1

##### APN 구성하기
1. Xcode에서 Capability 추가 : push message
2. Apple Developer 에서 key 생성: APNs
3. Firebase 프로젝트 설정 -> APN 인증 키 업로드 

#### (3) Firebase Cloud Messaging
> remote notification 관리 위한 서버 

- 원격 알림 메세지 전송 : 사용자에게 표시되는 알림 메세지를 실시간 또는 예약 전송
- 다양한 메시지 타겟팅 : 단일 기기, 기기 그룹, 주제를 구독한 기기
- 발송 메시지 저장, 관리 : 알림 내용, 상태, 플랫폼, 최종 전송 시간, 열람율 관리

A/B testing 을 위한 방법 
1. Remote config
2. Cloud messaging

##### registration tokens
The FCM SDK generates a registration token for the client app instance on app launch.
Similar to the APNs device token, this token allow you to send targeted notifications to any particular instance of your app

In the same way that Apple platforms typically deliver an APNs device token on app start,
FCM provides a registration token via *FIRMessagingDelegate*’s *messaging:didReceiveRegistrationToken*
- FCM의 현재 등록 토큰, 갱신되는 시점에 대해 알고
- 그에 따른 적절한 액션을 취할 수 있다.
- 이 토큰을 등록하여 Cloud Message 를 보낸다.

https://firebase.google.com/docs/cloud-messaging/ios/client

### 3) 새롭게 알게 된 것

- UNUserNotificationCenterDelegate
    - willPresent
    - 전달받은 Notification를 어떻게 display할 것인지 알림 형태를 지정할 수 있다.
    - 기본값 지정

```swift
extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.list, .banner, .sound])
    }
}

```

- UIApplication.shared
> applicationIconBadgeNumber 을 컨트롤하는 데에 사용했음

*UIApplication*
- The centralized point of control and coordination for apps running in iOS
https://developer.apple.com/documentation/uikit/uiapplication

*shared*
- The singleton app instance
- return the app instance is created in the UIApplicationMain(_:_:_:_:) function
- The UIApplicationMain function creates the shared app instance at launch time



