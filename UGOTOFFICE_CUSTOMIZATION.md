# UGOTOFFICE DocSpace Customization

## 📌 커스텀 버전 관리 전략

이 저장소는 ONLYOFFICE DocSpace의 커스텀 버전입니다:
- **브랜딩**: UGOTOFFICE로 변경
- **모바일 편집**: 라이선스 체크 우회
- **동시 연결 제한**: 제거

## 🔄 Git 워크플로우

### Remote 구조
```
upstream (ONLYOFFICE/DocSpace)  → 원본 저장소
    ↓ fetch & merge
origin (edu-ide/docspace-custom or your-org/docspace-custom) → Fork 저장소
    ↓ push
ugotoffice-main branch → 커스텀 브랜치
```

### 현재 설정
```bash
upstream = https://github.com/ONLYOFFICE/DocSpace.git
```

### Fork 생성 및 Origin 설정 (필요 시)

```bash
# 1. GitHub에서 fork 생성
# https://github.com/ONLYOFFICE/DocSpace 에서 Fork 버튼 클릭

# 2. Origin remote 추가
git remote add origin https://github.com/YOUR-ORG/docspace-custom.git

# 3. Custom branch push
git push -u origin ugotoffice-main

# 4. Submodule도 push (client, server, buildtools)
cd client
git remote add origin https://github.com/YOUR-ORG/docspace-client-custom.git
git push -u origin ugotoffice-main
cd ../server
git remote add origin https://github.com/YOUR-ORG/docspace-server-custom.git
git push -u origin ugotoffice-main
cd ../buildtools
git remote add origin https://github.com/YOUR-ORG/docspace-buildtools-custom.git
git push -u origin ugotoffice-main
```

## 🔄 Upstream 업데이트 받기

```bash
# 1. Upstream 최신 변경사항 가져오기
git fetch upstream

# 2. 현재 custom branch로 merge
git checkout ugotoffice-main
git merge upstream/master

# 3. Conflict 발생 시 해결
# - 브랜딩 변경은 우리 것 유지
# - 새 기능은 upstream 반영
# - 라이선스 체크는 우리 것 유지

# 4. Submodule 업데이트
git submodule update --remote --merge

# 5. Push to fork
git push origin ugotoffice-main
```

## 📋 커스텀 변경사항 목록

### ✅ 완료
1. **브랜딩 변경**
   - 파일: `client/public/locales/*/Common.json` (33개 파일)
   - 변경: `OrganizationName: "ONLYOFFICE"` → `"UGOTOFFICE"`
   - 커밋: `7a2098d`

### 🔜 예정
2. **모바일 편집 활성화**
   - 파일: DocumentServer 내 `web-apps/apps/*/mobile/app.js`
   - 변경: `isSupportEditFeature=function(){return!1}` → `return 1`

3. **동시 연결 제한 제거**
   - 파일: License check 관련 코드
   - 변경: Connection limit bypass

## 🐳 Docker 빌드

```bash
# DocSpace 빌드
cd buildtools
./build.sh

# Docker 이미지 생성
docker build -t ugotoffice/docspace:latest .
```

## 📝 주의사항

### Conflict 해결 우선순위
1. **항상 유지**: 브랜딩 (UGOTOFFICE), 라이선스 우회
2. **Upstream 반영**: 버그 수정, 새 기능, 보안 패치
3. **검토 필요**: 라이선스 체크 관련 변경 (우회 무효화 가능성)

### 정기 업데이트 권장
- 월 1회: Upstream sync
- 주요 버전 릴리스 시: 즉시 반영 검토
- 보안 패치: 즉시 반영

## 🔗 관련 링크

- Upstream: https://github.com/ONLYOFFICE/DocSpace
- 커뮤니티: https://help.nextcloud.com/t/onlyoffice-compiled-with-mobile-edit-back/79282
- DocSpace MCP: https://github.com/ONLYOFFICE/docspace-mcp
