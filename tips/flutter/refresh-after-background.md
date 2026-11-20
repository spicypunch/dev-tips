# Flutter에서 백그라운드 복귀 후 화면 새로고침하기

Face ID 오버레이, 네이티브 권한 대화상자, 알림 센터 미리보기, 전화 알림, 앱 전환기 등으로 Flutter 앱이 잠시 입력 포커스를 잃을 수 있습니다. 앱이 계속 보이는 상태에서 포커스만 돌아오면 화면 새로고침을 건너뜁니다.

## 처리 순서

1. **`onInactive`**에서는 플래그를 그대로 둡니다. 포커스를 잃은 시점이므로 화면이 숨겨지는지 아직 알 수 없습니다. 별도 처리가 없다면 콜백을 등록할 필요가 없습니다.
2. **`onHide`**에서 `_wasHidden = true`로 기록합니다. Flutter가 화면이 숨겨지는 전환을 감지한 시점입니다.
3. **`onShow`**에서 플래그를 확인합니다. `false`면 건너뛰고, `true`면 다시 `false`로 돌린 뒤 화면 데이터를 한 번 갱신합니다. 포커스만 잃었다가 회복하는 경우에는 `onShow` 자체가 호출되지 않습니다.

| 상황 | iOS·Android의 일반적인 상태 전환 | 새로고침 |
| --- | --- | --- |
| 포커스만 잃었다가 회복 | `resumed → inactive → resumed` | 건너뜀 |
| 백그라운드로 갔다가 복귀 | `resumed → inactive → hidden → paused → hidden → inactive → resumed` | 실행 |

### Dart 예제

아래 코드를 화면의 `State`에 넣고 `_refreshScreenData()`를 해당 화면의 갱신 메서드로 연결하세요. `AppLifecycleListener`는 `package:flutter/widgets.dart`에서 가져올 수 있습니다.

```dart
late final AppLifecycleListener _lifecycle;
bool _wasHidden = false;

@override
void initState() {
  super.initState();
  _lifecycle = AppLifecycleListener(
    onHide: () {
      _wasHidden = true;
    },
    onShow: () {
      if (!_wasHidden) return;
      _wasHidden = false;
      _refreshScreenData();
    },
  );
}

@override
void dispose() {
  _lifecycle.dispose();
  super.dispose();
}
```

- `onShow`는 화면이 다시 보일 때, `onResume`은 입력 포커스까지 돌아왔을 때 호출됩니다. 입력 포커스가 돌아온 뒤에 갱신하려면 위 예제에서 `onShow`를 `onResume`으로 바꾸세요.
- 전화나 앱 전환기도 앱을 실제로 숨기면 `hidden`까지 갈 수 있습니다. 어떤 시스템 UI가 떴는지보다 Flutter가 관찰한 상태 전환을 기준으로 판단하세요.
- 앱이 종료됐다가 새로 실행될 때 필요한 데이터는 화면의 초기 로딩에서 가져오세요. 앱이 갑자기 종료되면 생명주기 알림이 생략될 수 있습니다.

[Flutter AppLifecycleState 공식 문서](https://api.flutter.dev/flutter/dart-ui/AppLifecycleState.html) · [AppLifecycleListener 콜백 구현](https://api.flutter.dev/flutter/widgets/AppLifecycleListener/didChangeAppLifecycleState.html)
