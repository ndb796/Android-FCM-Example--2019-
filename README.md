## [부록] FCM 연동 예제
```
* 안드로이드 스튜디오 프로젝트 생성
* [Firebase Console](https://console.firebase.google.com)
* Android [앱 추가] - 패키지 이름, SHA1 인증서 설정 - [앱 등록]
* [구성 파일 다운로드] - 루트 디렉토리(app)에 놓기 - 안드로이드 스튜디오에서 [Sync with File System] - [다음]
* build.gradle 설정 내용 확인하기
* build.gradle (Project)에 classpath 'com.google.gms:google-services:4.0.1'
* build.gradle (Module: App)에 implementation 'com.google.firebase:firebase-messaging:17.3.4'
* build.gradle (Module: App)의 맨 아랫줄에 apply plugin: 'com.google.gms.google-services'
* USB로 연결하여 앱 실행 테스트 - 앱 연동 [확인]
* </activity> 아래에 다음 내용 삽입하기
<service android:name=".MyFireBaseMessagingService">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>
* MyFireBaseMessagingService.java
package com.example.fcm_test;

import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.app.PendingIntent;
import android.content.Context;
import android.content.Intent;
import android.media.RingtoneManager;
import android.net.Uri;
import android.os.Build;
import android.support.v4.app.NotificationCompat;
import android.util.Log;
import com.google.firebase.messaging.FirebaseMessagingService;
import com.google.firebase.messaging.RemoteMessage;

public class MyFireBaseMessagingService extends FirebaseMessagingService {
    @Override
    public void onNewToken(String token) {
        Log.d("FCM Log", "Refreshed token: " + token);
    }
    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        if (remoteMessage.getNotification() != null) {
            Log.d("FCM Log", "알림 메시지: " + remoteMessage.getNotification().getBody());
            String messageBody = remoteMessage.getNotification().getBody();
            String messageTitle = remoteMessage.getNotification().getTitle();
            Intent intent = new Intent(this, MainActivity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
            PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_ONE_SHOT);
            String channelId = "Channel ID";
            Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
            NotificationCompat.Builder notificationBuilder =
                    new NotificationCompat.Builder(this, channelId)
                            .setSmallIcon(R.mipmap.ic_launcher)
                            .setContentTitle(messageTitle)
                            .setContentText(messageBody)
                            .setAutoCancel(true)
                            .setSound(defaultSoundUri)
                            .setContentIntent(pendingIntent);
            NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                String channelName = "Channel Name";
                NotificationChannel channel = new NotificationChannel(channelId, channelName, NotificationManager.IMPORTANCE_HIGH);
                notificationManager.createNotificationChannel(channel);
            }
            notificationManager.notify(0, notificationBuilder.build());
        }
    }
}
* MainActivity.java
package com.example.fcm_test;

import android.support.annotation.NonNull;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.widget.Toast;

import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.iid.FirebaseInstanceId;
import com.google.firebase.iid.InstanceIdResult;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        FirebaseInstanceId.getInstance().getInstanceId()
                .addOnCompleteListener(new OnCompleteListener<InstanceIdResult>() {
                    @Override
                    public void onComplete(@NonNull Task<InstanceIdResult> task) {
                        if (!task.isSuccessful()) {
                            Log.w("FCM Log", "getInstanceId failed", task.getException());
                            return;
                        }
                        String token = task.getResult().getToken();
                        Log.d("FCM Log", "FCM 토큰: " + token);
                        Toast.makeText(MainActivity.this, token, Toast.LENGTH_SHORT).show();
                    }
                });
    }
}
* 앱 실행 - [Firebase 콘솔] - [Cloud Messaging] - [새 알림] - 알림 보내기
```
## [부록] 서버 FCM API
```
# PHP 5.6 curl 설치하기
sudo apt-get install php5.6-curl
# Apache 재시작하기
/etc/init.d/apache2 restart
# FIrebase 콘솔에서 서버 Key 확인하기
$token = $this->input_check('token');
$title = $this->input_check('title');
$message = $this->input_check('message');
if($token == "" || $title == "" || $message == "") {
        $this->output->set_status_header(400); // 입력 값 형식이 올바르지 않습니다.
        exit;
}
// 보내고자 하는 데이터를 배열에 담기
$notification = array();
$notification['title'] = $title;
$notification['body'] = $message;
$tokens = array();
$tokens[0] = $token;
// 전송 진행하기
$url = 'https://fcm.googleapis.com/fcm/send';
$apiKey = "서버 Key";
$fields = array('registration_ids' => $tokens,'notification' => $notification);
$headers = array('Authorization:key='.$apiKey,'Content-Type: application/json');
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($fields));
$result = curl_exec($ch);
if ($result === FALSE) {
        $this->output->set_status_header(500); // FCM 푸시 메시지 전송에 실패했습니다.
}
curl_close($ch);
echo $result;
```
