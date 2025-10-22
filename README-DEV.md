# DocSpace Custom - 개발 가이드

## 🚀 빠른 시작

### Option 1: Tilt (통합 개발)
```bash
cd /mnt/sdc1/ws/workspace/monorepo/external/docspace-custom
tilt up
```
- ✅ Kubernetes 통합 테스트
- ✅ 전체 스택 모니터링
- ❌ 느린 빌드 (15-20분)
- 📝 수동 트리거 모드 (UI에서 "Trigger Update" 클릭)

### Option 2: 로컬 개발 (빠른 반복)
```bash
cd client
yarn install
yarn dev
```
- ✅ Hot reload (초 단위)
- ✅ 빠른 피드백
- ❌ 백엔드와 분리됨
- 💡 프론트엔드만 수정 시 추천

### Option 3: 스크립트 (프로덕션 빌드)
```bash
cd /mnt/sdc1/ws/workspace/monorepo/scripts
./build-and-deploy-docspace-router.sh  # 생성 예정
```
- ✅ 전체 자동화
- ✅ 빌드 시간 표시
- ✅ 배포 검증
- 💡 최종 배포 전 사용

---

## 📝 라이선스 우회 코드

### 위치
```
client/packages/client/src/pages/PortalSettings/categories/common/Branding/whitelabel.js
```

### 수정 내용 (131-133라인)
```javascript
// CUSTOM: License bypass - always enable white label customization
const isSettingPaid = true;
const showNotAvailable = false;
```

---

## 🔧 개발 워크플로우

### 1. 프론트엔드만 수정하는 경우
```bash
# 1. 로컬 개발 서버 시작
cd client
yarn dev

# 2. 브라우저에서 http://localhost:5001 접속
# 3. 코드 수정 → 자동 새로고침
# 4. 확인 후 Dockerfile.app 빌드
```

### 2. 백엔드도 수정하는 경우
```bash
# Tilt 사용 (수동 트리거)
tilt up

# Tilt UI에서:
# 1. 'router' 리소스 선택
# 2. 'Trigger Update' 클릭
# 3. 빌드 로그 모니터링 (15-20분)
```

### 3. 최종 프로덕션 빌드
```bash
# 빌드 + 배포 + 테스트 전체 자동화
/mnt/sdc1/ws/workspace/monorepo/scripts/build-and-deploy-docspace-router.sh
```

---

## ⚡ BuildKit 캐시 최적화

### 적용된 최적화
- APT cache (`/var/cache/apt`)
- Yarn cache (`/usr/local/share/.cache/yarn`)
- NuGet cache (`/root/.nuget/packages`)
- Maven cache (`/root/.m2`)

### 효과
- 첫 빌드: 30-40분
- 두 번째+ 빌드: 15-20분 (30-40% 개선)

---

## 🐛 디버깅

### 빌드 로그 확인
```bash
# Tilt 사용 시
tilt logs router

# 수동 빌드 시
tail -f /tmp/docspace-router-build.log
```

### Pod 로그 확인
```bash
kubectl logs -n docspace -l app=router --tail=100 -f
```

### 브랜딩 기능 테스트
```bash
# Chrome DevTools로 테스트
open https://docspace.ugot.uk/portal-settings/customization/branding

# 스크립트로 테스트
/mnt/sdc1/ws/workspace/monorepo/scripts/test-docspace-whitelabel.sh
```

---

## 📊 개발 모드 비교

| 특징 | Tilt | 로컬 yarn dev | 빌드 스크립트 |
|------|------|---------------|--------------|
| 빌드 시간 | 15-20분 | 초 단위 | 15-20분 |
| Hot Reload | ❌ | ✅ | ❌ |
| K8s 통합 | ✅ | ❌ | ✅ |
| 백엔드 테스트 | ✅ | ❌ | ✅ |
| 추천 용도 | 통합 테스트 | 빠른 UI 개발 | 최종 배포 |

---

## 🎯 자주 수정하는 파일들

### 브랜딩/라이선스
- `client/packages/client/src/pages/PortalSettings/categories/common/Branding/whitelabel.js`
- `server/web/ASC.Web.Core/WhiteLabel/TenantLogoManager.cs` (백엔드)

### 프론트엔드 UI
- `client/packages/client/src/` (React 컴포넌트)
- `client/packages/shared/` (공통 컴포넌트)

### 빌드 설정
- `buildtools/install/docker/Dockerfile.app`
- `client/package.json`

---

## 💡 팁

1. **프론트엔드만 수정**: `yarn dev` 사용 (가장 빠름)
2. **통합 테스트 필요**: Tilt 수동 트리거 사용
3. **최종 배포**: 빌드 스크립트 사용
4. **BuildKit 캐시**: 첫 빌드 후 두 번째부터 빠름
5. **Tilt UI**: http://localhost:10350
