# GitHub Actions Secrets 설정 가이드

## Repository: https://github.com/jungfrau70/github-actions-demo-day2

### Settings > Secrets and variables > Actions에서 다음 Secrets 설정:

#### Docker Hub 인증
- `DOCKER_USERNAME`: your-docker-username
- `DOCKER_PASSWORD`: your-docker-password

#### AWS EKS 배포 (PROD 환경)
- `AWS_ACCESS_KEY_ID`: [AWS Access Key ID]
- `AWS_SECRET_ACCESS_KEY`: [AWS Secret Access Key]
- `AWS_REGION`: ap-northeast-2
- `EKS_CLUSTER_NAME`: cloud-intermediate-cluster
- `EKS_NAMESPACE`: production

#### GCP GKE 배포 (STAGING 환경)
- `GCP_PROJECT_ID`: [GCP Project ID]
- `GCP_SA_KEY`: [GCP Service Account Key JSON]
- `GCP_REGION`: asia-northeast1
- `GKE_CLUSTER_NAME`: cloud-intermediate-gke
- `GKE_NAMESPACE`: staging

#### 모니터링 허브 연결
- `MONITORING_HUB_URL`: http://3.37.234.110:9091
- `GRAFANA_URL`: http://3.37.234.110:3005
- `GRAFANA_USERNAME`: admin
- `GRAFANA_PASSWORD`: admin

#### 애플리케이션 설정
- `APP_ENV`: production
- `APP_PORT`: 3000
- `LOG_LEVEL`: info
