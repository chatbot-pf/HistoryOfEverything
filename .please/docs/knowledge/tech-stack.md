# Tech Stack: The History of Everything

## 언어
- **Dart** (SDK >=2.0.0 <3.0.0)

## 프레임워크
- **Flutter** — 크로스 플랫폼 모바일 UI 프레임워크

## 애니메이션
- **Flare** (flare_flutter) — 2D 벡터 애니메이션 렌더링
- **Nima** (nima) — 2D 스켈레탈 애니메이션 렌더링
- 커스텀 `LeafRenderObjectWidget` / `RenderBox` 기반 렌더링

## 상태 관리
- **BLoC 패턴** + **RxDart** (^0.19.0) — 반응형 상태 관리

## 로컬 저장
- **SharedPreferences** (^0.4.3) — 즐겨찾기 등 사용자 데이터 저장

## UI / UX 라이브러리
- **flutter_markdown** (^0.2.0) — Markdown 렌더링 (아티클)
- **intl** (>=0.14.0) — 국제화/날짜 포매팅
- **cupertino_icons** (^0.1.2) — iOS 스타일 아이콘

## 유틸리티
- **url_launcher** (^4.0.1) — 외부 URL 열기
- **share** (^0.5.3) — 앱 공유 기능

## 플랫폼
- Android
- iOS

## 빌드
- `flutter run` (app/ 디렉토리에서 실행)
- Git submodules: Flare-Flutter, Nima-Flutter (dependencies/ 디렉토리)
