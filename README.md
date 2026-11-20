# 개발 팁

## Flutter에서 백그라운드 복귀 후 화면 새로고침하기

권한 팝업이나 알림창이 잠깐 포커스를 가져가도 Flutter 앱은 다시 `resumed` 상태가 될 수 있습니다. 백그라운드에서 돌아올 때만 데이터를 갱신하려면 앱이 숨겨진 적 있는지 기록해 두세요.

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
    onResume: () {
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

- `onShow`는 화면이 다시 보일 때, `onResume`은 입력 포커스까지 돌아왔을 때 호출됩니다. 화면이 보이는 즉시 데이터 갱신을 시작하려면 위 예제에서 `onResume`을 `onShow`로 바꿔도 됩니다.
- 전화나 앱 전환기도 앱을 실제로 숨기면 `hidden`까지 갈 수 있습니다. 어떤 시스템 UI가 떴는지보다 Flutter가 관찰한 상태 전환을 기준으로 판단하세요.
- 앱이 종료됐다가 새로 실행될 때 필요한 데이터는 화면의 초기 로딩에서 가져오세요. 앱이 갑자기 종료되면 생명주기 알림이 생략될 수 있습니다.

[Flutter AppLifecycleState 공식 문서](https://api.flutter.dev/flutter/dart-ui/AppLifecycleState.html) · [AppLifecycleListener 콜백 구현](https://api.flutter.dev/flutter/widgets/AppLifecycleListener/didChangeAppLifecycleState.html)
