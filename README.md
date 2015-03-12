# Swiftでプッシュ通知を試す

## 前提

- mobile backendのアカウント登録/アプリ作成は済んでいる
- プッシュ通知を送るための証明書(.p12形式)の用意はできている
 - [APNsとの連携に必要な準備](http://mb.cloud.nifty.com/doc/tutorial/push_setup_ios.html)を参照

## SDKの読み込み

- NCMB.frameworkをプロジェクトに追加(ファイルをコピーする)
- Objective-Cで書かれたライブラリをswiftで使用するにはbridge-headerを作成する必要がある
- [Qiita](http://qiita.com/skatata/items/1facd024d239b9545031)を参考にした
- NCMB-bridge-header.hを追加

```objective-c
#ifndef PushTestInSwift_NCMB_bridge_header_h
#define PushTestInSwift_NCMB_bridge_header_h

#import <NCMB/NCMB.h>

#endif
```

- Build SettingsのSwift Compilerの項目を開く
- Objective-C Bridging Headerに$(SRCROOT)/$(PROJECT)/GameFeatKit-Bridging-Header.hを指定
 - ヘッダーファイルの場所は自分のプロジェクトにあわせて適宜変更してください。


## AppDelegateの編集

- APIキーの設定とデバイストークン要求(プッシュ通知の許可画面がでるところ)をdidFinishLaunchingWithOptionsメソッドに書く

```swift
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    // Override point for customization after application launch.
    NCMB.setApplicationKey("YOUR_APP_KEY",
        clientKey: "YOUR_CLIENT_KEY")
    
    
    if NSFoundationVersionNumber > NSFoundationVersionNumber_iOS_7_1 {
        //iOS8でのデバイストークン取得
        let userNotificationTypes = (UIUserNotificationType.Alert |
            UIUserNotificationType.Badge |
            UIUserNotificationType.Sound);
        
        let settings = UIUserNotificationSettings(forTypes: userNotificationTypes, categories: nil)
        application.registerUserNotificationSettings(settings)
        application.registerForRemoteNotifications()
    } else {
        //iOS7以前でのデバイストークン取得
        var types:UIRemoteNotificationType = UIRemoteNotificationType.Badge | UIRemoteNotificationType.Alert | UIRemoteNotificationType.Sound
        UIApplication.sharedApplication().registerForRemoteNotificationTypes(types)
    }
    
    
    
    return true
}
```

- デバイストークン取得後に端末情報をmobile backendに保存する
 - プッシュ通知を送る端末としてmobile backendに登録する必要がある

```swift
    func application(application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData) {
        let installation = NCMBInstallation.currentInstallation()
        installation.setDeviceTokenFromData(deviceToken)
        installation.saveInBackgroundWithBlock(nil) //必要があればBlock内でエラー処理を行う
    }
```

- アプリを実行するとプッシュ通知の許可画面が出てくる
- 許可してあげるとmobile backendのデータストアにあるinstallationクラスに端末情報が登録される
- プッシュ通知の送信がダッシュボードからできるようになる
