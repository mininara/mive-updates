# mive-updates

mininara 개발 앱들의 **자체(인앱) 업데이트** manifest/APK 호스팅 저장소.
공개(public) 저장소여야 앱이 `raw.githubusercontent.com` 에서 익명으로 받을 수 있다.
소스 코드는 각 앱의 private 저장소에 그대로 두고, 여기엔 배포 산출물만 올린다.

## 구조 — 앱별 독립 폴더 (Mive 접두어)
앱마다 폴더 하나. 각 폴더는 자기 `manifest.json` 과 APK 들을 가지며, 다른 앱과 독립적으로 릴리즈한다.

```
mive-updates/
  MiveFiles/                   # com.alt.files (Mive Files)
    manifest.json
    minifiles_<versionCode>.apk
  Mive<App2>/                  # 다음 앱 추가 시
    manifest.json
    <app2>_<versionCode>.apk
  README.md
```

각 앱의 `UPDATE_MANIFEST_URL` (BuildConfig) 은 자기 폴더의 manifest 를 가리킨다:
```
https://raw.githubusercontent.com/mininara/mive-updates/main/MiveFiles/manifest.json
https://raw.githubusercontent.com/mininara/mive-updates/main/Mive<App2>/manifest.json
```

## manifest.json 스키마
`apps` 는 packageName 으로 키잉(앱이 자기 packageName 으로 조회). 폴더당 보통 1개 항목.
```json
{
  "apps": {
    "<packageName>": {
      "versionCode": 0,
      "versionName": "1.0.x",
      "apkUrl": "https://raw.githubusercontent.com/mininara/mive-updates/main/<MiveApp>/<file>.apk",
      "sha256": "<APK sha256 소문자 hex>",
      "minSupportedVersion": 0,
      "rolloutPercent": 100,
      "mandatory": false,
      "releaseNotes": "변경 내용"
    }
  },
  "flags": { "<app>.someToggle": true }
}
```

## 릴리즈 절차 (앱 공통)
1. 해당 앱을 **release 서명**(동일 키스토어)으로 빌드. versionCode 는 설치본보다 커야 한다.
2. APK 를 그 앱 폴더에 올린다(raw 호스팅). 트래픽/용량이 커지면 GitHub Releases 자산으로 전환 권장.
3. `sha256sum <apk>` 로 해시 계산.
4. 그 앱 폴더의 `manifest.json` 의 versionCode/versionName/apkUrl/sha256 갱신 후 push.
5. 설치된 구버전 앱을 실행하면 업데이트 안내가 뜬다.

> ⚠️ 자체 업데이트는 기존 앱 위에 덮어 설치하므로 **새 APK 가 기존 설치본과 동일 키로 서명**돼야 한다.
>   서명이 다르면 signature mismatch 로 거부된다.
