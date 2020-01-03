# GoogleSigninForUnity_Pod5.0.0
---
## 1. 개요
iOS에서 GoogleSignin Pod 4.x.x 버전 사용시 ITMS-90809: Deprecated API Usage 이슈가 발생한다. 원인은 4.x.x 버전에서 UIWebView API를 사용하기 때문인데 GoogleSignin Pod 5.0.0 이상 버전에서는 해당 이슈가 발생하지 않는다. 그러나 GoogleSignin Unity 1.0.4에서는 4.x.x 버전에 맞추어 iOS 네이티브 코드가 작성되어 있어 GoogleSignin Unity 패키지의 네이티브 코드를 Pod 5.0.0에 맞도록 수정하였다.
## 2. 참고자료
Google Sign-In SDK 5.0.0 quick migration guide를 참고하여 수정하였다.  
https://developers.google.com/identity/sign-in/ios/quick-migration-guide#migrating_from_versions_prior_to_v500
## 3. 테스트 환경
GoogleSignin Unity 1.0.4 ( https://github.com/googlesamples/google-signin-unity/releases )  
Xcode 11.3
## 4. 수정내용
### 4.1 Assets\GoogleSignIn\Editor\GoogleSignInDependencies.xml 수정
#### 4.1.1 pod version 5.0.0 이상으로 설정
#### origin
~~~ xml
<iosPod name="GoogleSignIn" version=">= 4.0.2" bitcodeEnabled="false" minTargetSdk="6.0">
</iosPod>
~~~
#### change
~~~ xml
<iosPod name="GoogleSignIn" version=">= 5.0.0" bitcodeEnabled="false" minTargetSdk="6.0">
</iosPod>
~~~
### 4.2 Assets\Plugins\iOS\GoogleSignIn\GoogleSignIn.h 수정
#### 4.2.1 GIDSignInUIDelegate 삭제
#### origin
~~~ objc
@interface GoogleSignInHandler
    : NSObject <GIDSignInDelegate, GIDSignInUIDelegate>
@end
~~~
#### change
~~~ objc
@interface GoogleSignInHandler
    : NSObject <GIDSignInDelegate>
@end
~~~
### 4.3 Assets\Plugins\iOS\GoogleSignIn\GoogleSignIn.mm 수정
#### 4.3.1 에러코드 주석처리
#### origin
~~~ objc
case kGIDSignInErrorCodeNoSignInHandlersInstalled:
        currentResult_->result_code = kStatusCodeDeveloperError;
        break;
~~~
#### change
~~~ objc
//case kGIDSignInErrorCodeNoSignInHandlersInstalled:
//        currentResult_->result_code = kStatusCodeDeveloperError;
//        break;
~~~
#### 4.3.2 GoogleSignIn_Configure 에 presentingViewController 설정코드 추가
#### origin
~~~ objc
[GIDSignIn sharedInstance].shouldFetchBasicProfile = true;
~~~
#### change
~~~ objc
[GIDSignIn sharedInstance].shouldFetchBasicProfile = true;
[GIDSignIn sharedInstance].presentingViewController = UnityGetGLViewController();
~~~
#### 4.3.3 signInSilently 변경
#### origin
~~~ objc
[[GIDSignIn sharedInstance] signInSilently];
~~~
#### change
~~~ objc
[[GIDSignIn sharedInstance] restorePreviousSignIn];
~~~
### 4.4 Assets\Plugins\iOS\GoogleSignIn\GoogleSignInAppController.mm 수정
#### 4.4.1 uiDelegate 삭제
#### origin
~~~ objc
GIDSignIn *signIn = [GIDSignIn sharedInstance];
signIn.clientID = clientId;
signIn.uiDelegate = gsiHandler;
signIn.delegate = gsiHandler;
~~~
#### change
~~~ objc
GIDSignIn *signIn = [GIDSignIn sharedInstance];
signIn.clientID = clientId;
//signIn.uiDelegate = gsiHandler;
signIn.delegate = gsiHandler;
~~~
#### 4.4.2 hadleURL 수정
#### origin
~~~ objc
/**
 * Handle the auth URL
 */
- (BOOL)GoogleSignInAppController:(UIApplication *)application
                          openURL:(NSURL *)url
                sourceApplication:(NSString *)sourceApplication
                       annotation:(id)annotation {
  BOOL handled = [self GoogleSignInAppController:application
                                         openURL:url
                               sourceApplication:sourceApplication
                                      annotation:annotation];

  return [[GIDSignIn sharedInstance] handleURL:url
                             sourceApplication:sourceApplication
                                    annotation:annotation] ||
         handled;
}

/**
 * Handle the auth URL.
 */
- (BOOL)GoogleSignInAppController:(UIApplication *)app
                          openURL:(NSURL *)url
                          options:(NSDictionary *)options {

  BOOL handled =
      [self GoogleSignInAppController:app openURL:url options:options];

  return [[GIDSignIn sharedInstance]
                     handleURL:url
             sourceApplication:
                 options[UIApplicationOpenURLOptionsSourceApplicationKey]
                    annotation:
                        options[UIApplicationOpenURLOptionsAnnotationKey]] ||
         handled;
}
~~~
#### change
~~~ objc
/**
 * Handle the auth URL
 */
- (BOOL)GoogleSignInAppController:(UIApplication *)application
                          openURL:(NSURL *)url
                sourceApplication:(NSString *)sourceApplication
                       annotation:(id)annotation {
  BOOL handled = [self GoogleSignInAppController:application
                                         openURL:url
                               sourceApplication:sourceApplication
                                      annotation:annotation];

  return [[GIDSignIn sharedInstance] handleURL:url] || handled;
}

/**
 * Handle the auth URL.
 */
- (BOOL)GoogleSignInAppController:(UIApplication *)app
                          openURL:(NSURL *)url
                          options:(NSDictionary *)options {

  BOOL handled =
      [self GoogleSignInAppController:app openURL:url options:options];

  return [[GIDSignIn sharedInstance] handleURL:url] || handled;
}
~~~
