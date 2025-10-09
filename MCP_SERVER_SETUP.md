# UGOTOFFICE DocSpace MCP Server Setup

## 📌 개요

ONLYOFFICE DocSpace MCP Server는 Model Context Protocol (MCP)을 통해 AI 클라이언트(Claude, ChatGPT 등)가 DocSpace와 상호작용할 수 있게 해주는 서버입니다.

**주요 기능**:
- 파일 및 폴더 관리 (생성, 읽기, 업데이트, 삭제)
- 룸(Room) 관리
- 사용자 관리
- AI 에이전트와의 실시간 통합

## 🏗️ 아키텍처

```
┌─────────────────────────────────────────┐
│  AI Client (Claude Desktop, etc.)       │
└───────────────┬─────────────────────────┘
                │ MCP Protocol
                ↓
┌─────────────────────────────────────────┐
│  DocSpace MCP Server (Node.js)          │
│  - @onlyoffice/docspace-mcp             │
└───────────────┬─────────────────────────┘
                │ REST API
                ↓
┌─────────────────────────────────────────┐
│  UGOTOFFICE DocSpace Server             │
│  - Custom branded                       │
│  - Mobile editing enabled               │
└─────────────────────────────────────────┘
```

## 🚀 설치 방법

### 방법 1: NPM 패키지 사용 (권장)

```bash
# NPM 전역 설치
npm install -g @onlyoffice/docspace-mcp

# 또는 npx로 직접 실행
npx @onlyoffice/docspace-mcp
```

### 방법 2: Docker로 실행

```bash
# Docker Hub에서 pull
docker pull onlyoffice/docspace-mcp:latest

# 실행
docker run -d \
  -e DOCSPACE_BASE_URL=https://docspace.ugot.uk \
  -e DOCSPACE_API_KEY=your-api-key \
  -p 3000:3000 \
  onlyoffice/docspace-mcp:latest
```

### 방법 3: 소스에서 빌드

```bash
# DocSpace MCP 소스 클론
git clone https://github.com/ONLYOFFICE/docspace-mcp.git
cd docspace-mcp

# 의존성 설치
npm install

# 빌드
npm run build

# 실행
npm start
```

## ⚙️ 설정

### 환경변수

MCP 서버는 **환경변수로만** 설정됩니다:

```bash
# 필수 설정
export DOCSPACE_BASE_URL="https://docspace.ugot.uk"
export DOCSPACE_API_KEY="your-api-key-here"

# 선택적 설정
export DOCSPACE_TOOLSETS="files,folders,rooms,people"  # 사용할 toolset
export DOCSPACE_ENABLED_TOOLS="create_file,get_file,create_folder"  # 특정 tool만 활성화
export DOCSPACE_DISABLED_TOOLS="delete_file"  # 특정 tool 비활성화
export DOCSPACE_DYNAMIC="true"  # Dynamic meta-tools 사용
```

### API Key 생성

DocSpace에서 API Key를 생성해야 합니다:

```bash
# DocSpace 관리자 페이지 접속
https://docspace.ugot.uk/settings/developer-tools

# 1. "Developer Tools" 메뉴
# 2. "Create API Key" 클릭
# 3. Name: "MCP Server"
# 4. Scopes: 필요한 권한 선택
#    - rooms:read, rooms:write
#    - files:read, files:write
#    - users:read
# 5. Copy API Key
```

## 🔧 Claude Desktop 연동

### 설정 파일 위치

**macOS**:
```
~/Library/Application Support/Claude/claude_desktop_config.json
```

**Windows**:
```
%APPDATA%\Claude\claude_desktop_config.json
```

**Linux**:
```
~/.config/Claude/claude_desktop_config.json
```

### 설정 예시

```json
{
  "mcpServers": {
    "ugotoffice-docspace": {
      "command": "npx",
      "args": [
        "-y",
        "@onlyoffice/docspace-mcp"
      ],
      "env": {
        "DOCSPACE_BASE_URL": "https://docspace.ugot.uk",
        "DOCSPACE_API_KEY": "your-api-key-here",
        "DOCSPACE_TOOLSETS": "files,folders,rooms",
        "DOCSPACE_DYNAMIC": "true"
      }
    }
  }
}
```

### Claude Desktop 재시작

설정 파일 저장 후:
1. Claude Desktop 완전 종료
2. 재시작
3. 새 대화에서 MCP 서버 사용 가능

## 🛠️ 사용 가능한 Toolsets

### 1. Files Toolset
- `create_file`: 파일 생성
- `get_file`: 파일 읽기
- `update_file`: 파일 수정
- `delete_file`: 파일 삭제
- `list_files`: 파일 목록

### 2. Folders Toolset
- `create_folder`: 폴더 생성
- `get_folder`: 폴더 정보
- `delete_folder`: 폴더 삭제
- `list_folders`: 폴더 목록

### 3. Rooms Toolset
- `create_room`: 룸 생성
- `get_room`: 룸 정보
- `update_room`: 룸 수정
- `delete_room`: 룸 삭제
- `list_rooms`: 룸 목록

### 4. People Toolset
- `get_user`: 사용자 정보
- `list_users`: 사용자 목록
- `invite_user`: 사용자 초대

## 🧪 테스트

### Claude Desktop에서 테스트

대화창에서:

```
Can you list all my rooms in UGOTOFFICE DocSpace?
```

```
Create a new file called "meeting-notes.md" in the "Projects" folder
```

```
Show me all users in the "Engineering" room
```

### API 직접 테스트 (curl)

```bash
# API Key 테스트
curl -X GET "https://docspace.ugot.uk/api/2.0/people/@self" \
  -H "Authorization: Bearer YOUR_API_KEY"

# 파일 목록
curl -X GET "https://docspace.ugot.uk/api/2.0/files/@my" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## 🔒 보안 고려사항

### API Key 보안

1. **환경변수 사용**: 설정 파일에 직접 하드코딩 금지
2. **최소 권한 원칙**: 필요한 scope만 부여
3. **정기 갱신**: API Key 주기적 교체
4. **접근 제어**: IP 제한 또는 VPN 사용

### Production 설정

```json
{
  "mcpServers": {
    "ugotoffice-docspace": {
      "command": "npx",
      "args": ["-y", "@onlyoffice/docspace-mcp"],
      "env": {
        "DOCSPACE_BASE_URL": "${DOCSPACE_URL}",  # 환경변수 참조
        "DOCSPACE_API_KEY": "${DOCSPACE_API_KEY}",
        "DOCSPACE_TOOLSETS": "files,folders",  # 최소 권한
        "DOCSPACE_DISABLED_TOOLS": "delete_file,delete_folder"  # 위험 작업 비활성화
      }
    }
  }
}
```

## 🐛 트러블슈팅

### 문제: MCP 서버가 시작되지 않음

**증상**: Claude Desktop에서 "Server not available"

**해결**:
```bash
# 1. NPM 캐시 정리
npm cache clean --force

# 2. 패키지 재설치
npm uninstall -g @onlyoffice/docspace-mcp
npm install -g @onlyoffice/docspace-mcp

# 3. 로그 확인
npx @onlyoffice/docspace-mcp --verbose
```

### 문제: API Key 인증 실패

**증상**: "401 Unauthorized"

**해결**:
```bash
# 1. API Key 유효성 확인
curl -X GET "https://docspace.ugot.uk/api/2.0/people/@self" \
  -H "Authorization: Bearer YOUR_API_KEY"

# 2. API Key 재생성
# DocSpace 관리 페이지에서 새 Key 생성

# 3. 환경변수 업데이트
export DOCSPACE_API_KEY="new-api-key"
```

### 문제: CORS 오류

**증상**: "CORS policy blocked"

**해결**:
```bash
# DocSpace 서버 CORS 설정 확인
# config/appsettings.json:

{
  "Cors": {
    "AllowedOrigins": [
      "http://localhost:3000",
      "https://claude.ai"
    ]
  }
}
```

## 📊 모니터링

### MCP 서버 상태 확인

```bash
# Health check
curl http://localhost:3000/health

# Metrics
curl http://localhost:3000/metrics

# Active tools
curl http://localhost:3000/tools
```

### 로그 수준 설정

```bash
export LOG_LEVEL="debug"  # debug, info, warn, error
npx @onlyoffice/docspace-mcp
```

## 🔄 업데이트

### NPM 패키지 업데이트

```bash
# 최신 버전 확인
npm outdated @onlyoffice/docspace-mcp

# 업데이트
npm update -g @onlyoffice/docspace-mcp

# 특정 버전 설치
npm install -g @onlyoffice/docspace-mcp@1.2.3
```

### Breaking Changes 대응

```bash
# 버전 고정 (package.json)
{
  "dependencies": {
    "@onlyoffice/docspace-mcp": "^1.0.0"
  }
}
```

## 📝 사용 예시

### Claude와의 대화 예시

**사용자**: "UGOTOFFICE DocSpace에 있는 모든 프로젝트 룸을 보여줘"

**Claude**: "DocSpace에서 룸 목록을 조회하겠습니다..."
[MCP call: list_rooms]

**Claude**: "다음 3개의 프로젝트 룸이 있습니다:
1. Frontend Development (5명)
2. Backend Architecture (3명)
3. DevOps Pipeline (4명)"

---

**사용자**: "Frontend Development 룸에 meeting-notes.md 파일을 만들어줘"

**Claude**: "파일을 생성하겠습니다..."
[MCP call: create_file(room="Frontend Development", name="meeting-notes.md")]

**Claude**: "✅ meeting-notes.md 파일이 Frontend Development 룸에 생성되었습니다."

## 🔗 참고 자료

- **공식 저장소**: https://github.com/ONLYOFFICE/docspace-mcp
- **MCP 프로토콜**: https://modelcontextprotocol.io
- **DocSpace API**: https://api.onlyoffice.com/docspace
- **Claude Desktop MCP**: https://modelcontextprotocol.io/clients/claude-desktop

## ⚖️ 라이선스

DocSpace MCP Server는 Apache-2.0 라이선스로 배포됩니다.
