# UGOTOFFICE DocSpace Docker Build Guide

## 📦 빌드 구성

### Custom DocumentServer
- **Base**: `onlyoffice/documentserver:latest`
- **Customization**:
  - ✅ 모바일 웹 편집 활성화
  - ✅ 동시 연결 제한 우회 (예정)
- **이미지**: `ugotoffice/documentserver:latest`

### DocSpace
- **Base**: ONLYOFFICE DocSpace
- **Customization**:
  - ✅ UGOTOFFICE 브랜딩
- **이미지**: `ugotoffice/docspace:latest`

## 🚀 빌드 방법

### 1. DocumentServer 커스텀 이미지 빌드

```bash
cd buildtools/install/docker

# Build custom DocumentServer
docker build -f Dockerfile.documentserver -t ugotoffice/documentserver:latest .

# Verify build
docker images | grep ugotoffice/documentserver
```

### 2. DocSpace 빌드 (선택사항)

```bash
# DocSpace client build
cd client
yarn install
yarn build

# DocSpace server build
cd ../server
dotnet build

# Create Docker image (if needed)
cd ../buildtools
./build.sh
```

## 🐳 Docker Compose 실행

### 환경변수 설정

```bash
# .env 파일 생성
cat > .env <<EOF
DOCUMENT_SERVER_IMAGE_NAME=ugotoffice/documentserver:latest
REGISTRY=
DOCUMENT_CONTAINER_NAME=onlyoffice-documentserver
DOCUMENT_SERVER_JWT_SECRET=your-secret-here
DOCUMENT_SERVER_JWT_HEADER=AuthorizationJwt
NETWORK_NAME=onlyoffice
VOLUMES_DIR=./volumes
EOF
```

### 실행

```bash
# Network 생성 (최초 1회)
docker network create onlyoffice

# DocumentServer 시작
docker-compose -f buildtools/install/docker/ds.ugotoffice.yml up -d

# 로그 확인
docker logs -f onlyoffice-documentserver
```

### 예상 로그 출력
```
🚀 UGOTOFFICE DocumentServer starting...
🔧 UGOTOFFICE: Enabling mobile editing...
  ✓ Patching: /var/www/onlyoffice/documentserver/web-apps/apps/documenteditor/mobile/app.js
  ✓ Patching: /var/www/onlyoffice/documentserver/web-apps/apps/spreadsheeteditor/mobile/app.js
  ✓ Patching: /var/www/onlyoffice/documentserver/web-apps/apps/presentationeditor/mobile/app.js
✅ Mobile editing enabled successfully!
```

## 🧪 테스트

### 모바일 편집 테스트

1. **모바일 브라우저 또는 Chrome DevTools Mobile Mode에서 접속**
2. **문서 열기**
3. **편집 가능 확인** (이전: "mobile editing is not possible" 메시지)

### Health Check

```bash
# DocumentServer health
curl http://localhost:8000/healthcheck

# Info endpoint
curl http://localhost:8000/info/info.json
```

## 🔄 업데이트 프로세스

### Upstream DocumentServer 업데이트

```bash
# 1. 최신 base image pull
docker pull onlyoffice/documentserver:latest

# 2. Custom image rebuild
docker build -f Dockerfile.documentserver -t ugotoffice/documentserver:latest .

# 3. Container restart
docker-compose -f buildtools/install/docker/ds.ugotoffice.yml up -d --force-recreate
```

## 🛠️ 트러블슈팅

### 문제: Mobile editing not enabled

**증상**: 모바일에서 여전히 "not possible" 메시지

**해결**:
```bash
# Container 내부 확인
docker exec -it onlyoffice-documentserver bash

# 패치 적용 여부 확인
grep -r "isSupportEditFeature" /var/www/onlyoffice/documentserver/web-apps/apps/*/mobile/app.js

# 수동 실행
/usr/local/bin/enable-mobile-edit.sh

# Container 재시작
docker restart onlyoffice-documentserver
```

### 문제: Build fails

**증상**: Docker build error

**해결**:
```bash
# 캐시 없이 rebuild
docker build --no-cache -f Dockerfile.documentserver -t ugotoffice/documentserver:latest .

# 이전 이미지/컨테이너 정리
docker system prune -a
```

## 📋 파일 구조

```
buildtools/install/docker/
├── Dockerfile.documentserver          # Custom DocumentServer image
├── documentserver-entrypoint.sh      # Custom entrypoint
├── enable-mobile-edit.sh             # Mobile edit enabler
├── ds.ugotoffice.yml                 # UGOTOFFICE compose file
└── ds.yml                            # Original compose file
```

## 🔐 보안 고려사항

1. **JWT Secret**: 프로덕션에서는 강력한 random secret 사용
2. **Network Isolation**: Docker network로 서비스 격리
3. **Volume Permissions**: 적절한 파일 권한 설정

## 📝 변경 이력

### Version 1.0 (2025-01-09)
- ✅ Mobile editing enabler 스크립트
- ✅ Custom DocumentServer Dockerfile
- ✅ Docker Compose 설정
