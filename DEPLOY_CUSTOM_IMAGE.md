# DocSpace 커스텀 이미지 배포 가이드

## 🎯 목적
Tenant resolution bypass가 포함된 커스텀 DocSpace API 이미지를 영구 배포합니다.

## 📦 커스텀 이미지 정보
- **이미지**: `localhost:5000/docspace-api:production`
- **기반 이미지**: `onlyoffice/docspace-api:3.2.1`
- **변경 내용**: Wizard API를 위한 tenant resolution bypass 추가 (TenantManager.cs)

## 🔧 빌드 및 배포 프로세스

### 1. DLL 빌드
```bash
cd /mnt/sdc1/ws/workspace/monorepo/external/docspace-custom/server

# Clean build
/home/tripleyoung/.dotnet/dotnet clean common/ASC.Core.Common/ASC.Core.Common.csproj
/home/tripleyoung/.dotnet/dotnet build common/ASC.Core.Common/ASC.Core.Common.csproj -c Debug --no-incremental
```

### 2. Docker 이미지 빌드
```bash
cd /mnt/sdc1/ws/workspace/monorepo/external/docspace-custom

# Build image
docker build --no-cache -t ugotoffice/docspace-api:production -f Dockerfile.quick .

# Push to local registry
docker tag ugotoffice/docspace-api:production localhost:5000/docspace-api:production
docker push localhost:5000/docspace-api:production
```

### 3. K8s 배포
```bash
# 방법 A: kubectl set image (빠름, 임시)
kubectl set image deployment/api api=localhost:5000/docspace-api:production -n docspace
kubectl rollout status deployment/api -n docspace

# 방법 B: kubectl patch (영구)
kubectl patch deployment api -n docspace -p '{
  "spec": {
    "template": {
      "spec": {
        "containers": [{
          "name": "api",
          "image": "localhost:5000/docspace-api:production"
        }]
      }
    }
  }
}'
```

## ✅ 검증
```bash
# 이미지 확인
kubectl get deployment api -n docspace -o jsonpath='{.spec.template.spec.containers[0].image}'
# 출력: localhost:5000/docspace-api:production

# API 테스트
curl -s https://docspace.ugot.uk/api/2.0/settings | jq '.response | {wizardToken, tenantStatus, tenantAlias}'
# 모든 필드가 null이 아니어야 함
```

## 🔄 재배포 시나리오

### Namespace 재생성 후
1. Helm으로 DocSpace 재설치
2. 위 "빌드 및 배포 프로세스" 3단계 실행

### Tilt 재시작 후
- Deployment만 kubectl patch하면 됨 (빌드 불필요)

### 코드 수정 후
- 전체 "빌드 및 배포 프로세스" 1-3단계 실행

## 📝 수정된 파일
- `server/common/ASC.Core.Common/Context/Impl/TenantManager.cs` (L271-283)
  - Wizard/Settings 요청 시 tenant ID 1 사용하는 bypass 로직 추가

## ⚠️ 주의사항
- 로컬 레지스트리(`localhost:5000`) 이미지는 노드 재시작 시 사라질 수 있음
- 프로덕션 환경에서는 외부 레지스트리(Docker Hub, Harbor 등) 사용 권장
