# hearlers-redis

Hearlers 프로젝트를 위한 Redis 배포를 위한 Helm 차트입니다.

## 개요

- **Redis Version:** 8.2.1
- **Helm Chart:** `hearlers-redis` (version: 0.1.0)
- **Dependencies:** Bitnami Redis (version: 22.0.7)
- **Architecture:**
  - dev: Standalone Redis (Master only)
  - prod: TBD

## 프로젝트 구조

```
hearlers-redis/
├── .github/workflows/deploy-dev.yml  # 개발 환경 자동 배포
├── helm/redis/
│   ├── Chart.yaml                    # Helm 차트 정의
│   ├── values-development.yaml       # 개발 환경 설정
│   └── values.schema.json           # 설정 스키마
└── README.md
```

## 개발 환경 설정

`values-development.yaml`에서 다음 설정을 사용합니다:

- **Architecture:** `standalone` (단일 Redis 인스턴스)
- **Authentication:** 비활성화
- **Persistence:** 비활성화 (메모리 기반)
- **Service:** NodePort (외부 접근 가능)
- **Resources:** 
  - CPU: 100m (request) / 200m (limit)
  - Memory: 128Mi (request) / 256Mi (limit)

## 배포

### 자동 배포 (CI/CD)

`develop` 브랜치에 푸시하면 GitHub Actions가 자동으로 개발 서버에 배포합니다.

### 수동 배포

```bash
# Helm 의존성 업데이트
helm dependency update ./helm/redis

# 개발 환경 배포
helm upgrade --install redis-dev ./helm/redis \
  -f ./helm/redis/values-development.yaml \
  --namespace development \
  --create-namespace
```

## 접속 정보

### 개발 환경

- **Redis 접속:** `<Node_IP>:30379`
- **인증:** 없음 (컨테이너 호스팅 VM 허용 IP정책으로 인증)
- **데이터:** 메모리 기반 (재시작 시 초기화)



## 참고 문서

- [Bitnami Redis Chart](https://github.com/bitnami/charts/tree/main/bitnami/redis)
- [Redis Documentation](https://redis.io/documentation)
