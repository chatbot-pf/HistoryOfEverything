# Spec: Flutter 3.x 마이그레이션 및 현대화

## 개요
The History of Everything 앱을 현재 Dart SDK 2.x / Flutter 1.x 기반에서 최신 Flutter 3.x / Dart 3.x 환경으로 마이그레이션합니다.

## 목표
1. **Dart 3.x 호환**: null safety 적용, deprecated API 교체
2. **Flutter 3.x 업그레이드**: 최신 Flutter SDK에서 빌드 및 실행 가능
3. **의존성 업데이트**: Flare-Flutter, Nima-Flutter 등 로컬 의존성 호환성 확보
4. **패키지 업데이트**: flutter_markdown, rxdart, shared_preferences 등 최신 버전으로 업그레이드
5. **빌드 검증**: Android/iOS 양 플랫폼에서 빌드 및 기본 동작 확인

## 범위
- `app/` 디렉토리의 모든 Dart 소스 코드
- `dependencies/` 디렉토리의 Flare-Flutter, Nima-Flutter 라이브러리
- pubspec.yaml 의존성 버전
- Android/iOS 플랫폼 설정 파일

## 범위 외
- 새로운 기능 추가
- UI/UX 변경
- 콘텐츠(아티클/애니메이션) 변경

## 성공 기준
- [ ] `flutter analyze` 오류 없음
- [ ] `flutter test` 모든 테스트 통과
- [ ] `flutter build apk` 성공
- [ ] `flutter build ios` 성공 (macOS 환경)
- [ ] 기존 기능 동작 유지 (타임라인 탐색, 검색, 즐겨찾기)
