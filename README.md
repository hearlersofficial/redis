# hearlers-redis

이 저장소는 "Hearlers" 프로젝트를 위한 Redis Sentinel 클러스터를 Helm 차트를 사용하여 관리합니다.

## 개요

이 Helm 차트는 Redis Sentinel을 사용한 고가용성 Redis 클러스터를 배포합니다. Master-Replica 구조에 Redis Sentinel을 추가하여 자동 장애 조치(failover) 기능을 제공합니다. 개발 환경에서는 외부 접근을 위한 NodePort 서비스를 제공하며, 운영 환경에서는 보안을 강화한 ClusterIP 서비스를 사용합니다.

- **Redis Version:** 7.2.5
- **Helm Chart:** `hearlers-redis` (version: 0.1.0)
- **Dependencies:**
  - Bitnami Redis (version: 18.1.5)
- **Architecture:** Redis Master + Replica + Sentinel (3개)

---

## 프로젝트 구조

```
hearlers-redis/
├── .github/
│   └── workflows/
│       └── deploy-dev.yml  # 개발 환경 자동 배포 워크플로우
├── helm/
│   └── redis/
│       ├── Chart.yaml          # Helm 차트 정보 및 의존성 정의
│       ├── values.yaml         # 기본 설정값
│       ├── values-development.yaml # 개발 환경용 설정값
│       └── values-production.yaml  # 운영 환경용 설정값
└── README.md
```

- **`.github/workflows/`**: `develop` 브랜치에 코드가 푸시되면 GitHub Actions를 통해 개발 서버에 자동으로 Redis 클러스터를 배포하는 CI/CD 파이프라인이 설정되어 있습니다.
- **`helm/redis/`**: Redis 배포를 위한 Helm 차트와 환경별 설정 파일이 위치합니다.

---

## 설정 (Configuration)

Helm 차트의 설정은 `values.yaml` 파일을 통해 관리되며, 각 환경에 따라 특정 값을 덮어쓸 수 있습니다.

- **`values.yaml`**: 모든 환경에 적용되는 기본 설정 파일입니다. Redis Sentinel이 활성화되어 있으며 ClusterIP 서비스 타입이 기본으로 설정되어 있습니다.
- **`values-development.yaml`**: 개발 환경을 위한 설정 파일입니다.
  - 인증을 비활성화하여 개발 편의성을 높입니다.
  - Master 및 Replica replica 수를 `1`로 설정합니다.
  - Redis Sentinel 3개를 활성화하여 고가용성을 제공합니다.
  - 외부 접근을 위해 `NodePort` 타입의 서비스를 사용하며, Redis 포트는 `30379`, Sentinel 포트는 `30380`으로 설정됩니다.
  - 데이터 영속성을 비활성화하여 메모리 기반으로 동작합니다.
- **`values-production.yaml`**: 운영 환경을 위한 설정 파일입니다.
  - 보안을 위해 인증을 활성화합니다.
  - Master replica 수를 `1`, Replica replica 수를 `2`로 설정합니다.
  - Redis Sentinel 3개를 활성화하여 고가용성을 제공합니다.
  - 개발 환경보다 높은 CPU 및 메모리 리소스를 할당합니다.
  - 데이터 영속성을 비활성화하여 메모리 기반으로 동작합니다.

---

## 배포 (Deployment)

### 자동 배포 (CI/CD)

`develop` 브랜치에 변경 사항이 푸시되면, `.github/workflows/deploy-dev.yml` 워크플로우가 자동으로 실행되어 개발 환경에 최신 버전의 Helm 차트를 배포합니다.

### 수동 배포

로컬 환경이나 다른 환경에 수동으로 배포하려면 다음 명령어를 사용할 수 있습니다.

**1. Helm 의존성 업데이트**

```bash
helm dependency update ./helm/redis
```

**2. Helm 차트 배포**

- **개발 환경 (Development)**
  ```bash
  helm upgrade --install redis-dev ./helm/redis \
    -f ./helm/redis/values-development.yaml \
    --namespace development \
    --create-namespace
  ```

- **운영 환경 (Production)**
  ```bash
  helm upgrade --install redis-prod ./helm/redis \
    -f ./helm/redis/values-production.yaml \
    --namespace production \
    --create-namespace
  ```

---

## 접속 정보

### Redis Master (Sentinel을 통한 접근)

- **개발 환경 (Development)**:
  - `NodePort`를 통해 외부에 노출됩니다.
  - Redis 접속 주소: `<Node_IP>:30379`
  - Sentinel 접속 주소: `<Node_IP>:30380`
  - 인증: 비활성화
  - Sentinel을 통한 자동 장애 조치 지원

- **운영 환경 (Production)**:
  - `ClusterIP`로 내부 접근만 가능합니다.
  - Redis 접속 주소: `redis-prod-master:6379`
  - Sentinel 접속 주소: `redis-prod-sentinel:26379`
  - 인증: 활성화 (비밀번호 필요)
  - Sentinel을 통한 자동 장애 조치 지원

### Redis Replica

- **개발 환경 (Development)**:
  - 접속 주소: `redis-dev-replica:6379`

- **운영 환경 (Production)**:
  - 접속 주소: `redis-prod-replica:6379`

### Redis Sentinel

- **개발 환경 (Development)**:
  - 접속 주소: `redis-dev-sentinel:26379`

- **운영 환경 (Production)**:
  - 접속 주소: `redis-prod-sentinel:26379`

---

## 주요 특징

- **고가용성**: Redis Sentinel을 통한 자동 장애 조치(failover) 기능 제공
- **메모리 기반 저장**: 데이터 영속성을 비활성화하여 빠른 성능을 제공합니다.
- **환경별 최적화**: 개발과 운영 환경에 맞는 리소스 할당 및 보안 설정을 제공합니다.
- **자동 배포**: GitHub Actions를 통한 CI/CD 파이프라인으로 자동 배포가 가능합니다.
- **확장성**: Master-Replica 구조로 읽기 성능을 향상시킵니다.
- **모니터링**: Sentinel을 통한 Redis 인스턴스 상태 모니터링 및 알림

---

## 트러블슈팅

### 일반적인 문제

1. **Pod이 Pending 상태에서 멈춤**:
   - 리소스 가용성을 확인하세요.
   - PersistentVolumeClaim 설정을 확인하세요.

2. **연결 거부 오류**:
   - 서비스 설정을 확인하세요.
   - NodePort 접근을 위한 방화벽 규칙을 확인하세요.

3. **인증 실패**:
   - 운영 환경에서 비밀번호 설정을 확인하세요.
   - Redis 인증 설정을 확인하세요.

### 유용한 명령어

```bash
# Pod 상태 확인
kubectl get pods -n development

# 로그 확인
kubectl logs -f deployment/redis-dev-master -n development

# Redis 연결 테스트
kubectl exec -it redis-dev-master-0 -n development -- redis-cli ping

# Sentinel 상태 확인
kubectl exec -it redis-dev-sentinel-0 -n development -- redis-cli -p 26379 sentinel masters

# 로컬 접근을 위한 포트 포워딩
kubectl port-forward svc/redis-dev-master 6379:6379 -n development
kubectl port-forward svc/redis-dev-sentinel 26379:26379 -n development
```
