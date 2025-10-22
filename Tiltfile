# DocSpace Custom Tiltfile
# 라이선스 우회가 적용된 DocSpace router 빌드 및 배포

# BuildKit 활성화 (캐시 최적화)
docker_prune_settings(disable=False, max_age_mins=60)

custom_build(
    ref='localhost:5000/docspace-router:bazel',
    command='bazel build //external/docspace-custom:docspace_router_image',
    deps=[
        'buildtools',
        'client',
        'server',
        'Tiltfile',
        'buildtools/install/docker',
    ],
    ignore=[
        'client/node_modules',
        'client/.cache',
        'client/dist',
        'client/build',
        'Logs',
        '.git',
    ],
    disable_push=True,
)

# Kubernetes 배포
k8s_yaml('../kubernetes-docspace/values-custom.yaml')

# Router 리소스 설정
k8s_resource(
    'router',
    port_forwards=[
        '8080:8080',  # Nginx
    ],
    labels=['docspace'],
    # 자동 재배포 활성화
    auto_init=True,
    # 빌드 완료까지 대기 (15-20분)
    trigger_mode=TRIGGER_MODE_MANUAL,  # 수동 트리거 (빌드 시간 고려)
)

# DocSpace 네임스페이스
allow_k8s_contexts('docker-desktop')  # 로컬 개발 환경만 허용

# 빌드 완료 알림
print("""
╔══════════════════════════════════════════════════════════╗
║  DocSpace Custom Tiltfile                                ║
╠══════════════════════════════════════════════════════════╣
║  🔧 수동 트리거 모드 활성화                                 ║
║     이유: 빌드 시간 15-20분으로 자동 재빌드 비효율적         ║
║                                                          ║
║  📝 사용법 (Bazel 연동):                                   ║
║     1. 코드 수정                                          ║
║     2. Tilt UI에서 'router' 리소스를 선택                 ║
║     3. 'Trigger Update' → Bazel이 이미지 빌드             ║
║     4. 빌드 완료 대기 (15-20분)                           ║
║                                                          ║
║  ⚡ 빠른 프론트엔드 개발:                                   ║
║     cd client && yarn dev                                ║
║     → Hot reload 사용 (초 단위)                           ║
║                                                          ║
║  🎯 브랜딩 수정 파일:                                      ║
║     client/packages/client/src/pages/                    ║
║       PortalSettings/categories/common/Branding/         ║
║       whitelabel.js:131-133                              ║
╚══════════════════════════════════════════════════════════╝
""")
