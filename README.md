# 기본과제

# 프론트엔드 배포 파이프라인

## 개요

GitHub Actions를 활용한 CI/CD 파이프라인을 구축하여 Next.js 프로젝트를 AWS S3와 CloudFront에 자동 배포합니다.  
코드 푸시 시 다음 순서로 배포가 진행됩니다.

![배포 파이프라인](https://github.com/user-attachments/assets/906ebf55-efb2-451d-97b5-e22fae38fb96)

### 배포 순서 요약

1. GitHub Actions에서 저장소 코드를 체크아웃합니다.
2. `npm ci`를 사용하여 의존성을 설치합니다.
3. `npm run build`로 프로젝트를 빌드합니다.
4. IAM 인증 정보를 바탕으로 AWS CLI를 설정합니다.
5. `aws s3 sync` 명령어로 S3 버킷에 빌드 파일을 업로드합니다.
6. `aws cloudfront create-invalidation`으로 캐시를 무효화하여 변경사항을 반영합니다.

## 주요 링크

- **S3 웹사이트 엔드포인트**: [S3 웹사이트 엔드포인트](http://jinsoulawsbucket.s3-website-ap-southeast-2.amazonaws.com)
- **CloudFront 도메인**: [CloudFront 도메인](https://dw4otricr6sk2.cloudfront.net)

## 주요 개념

### GitHub Actions과 CI/CD 도구

GitHub Actions는 코드 변경 사항을 자동으로 감지하여 지정된 작업(빌드, 테스트, 배포 등)을 실행합니다. 이 프로젝트에서는 `main` 브랜치에 push되면 배포가 자동화됩니다.

### S3와 스토리지

Amazon S3는 정적 웹사이트 호스팅이 가능한 객체 스토리지입니다. `next export`를 통해 생성된 정적 파일을 업로드하여 웹사이트를 호스팅합니다.

### CloudFront와 CDN

CloudFront는 전 세계에 분산된 엣지 로케이션을 통해 빠르게 정적 파일을 전달하는 CDN 서비스입니다. S3와 연동하여 웹 자산을 캐시하고 전달 속도를 향상시킵니다.

### 캐시 무효화(Cache Invalidation)

CloudFront는 성능을 위해 자산을 캐싱합니다. 변경된 파일을 즉시 반영하기 위해서는 무효화 작업이 필요합니다. GitHub Actions에서 배포 직후 `invalidate` 명령어를 실행해 최신 파일을 즉시 배포합니다.

### Repository Secret과 환경 변수

GitHub 저장소의 `Secrets` 기능을 활용해 IAM 액세스 키, 리전, 버킷 이름 등의 민감한 정보를 안전하게 관리합니다. 워크플로우에서는 `${{ secrets.XXX }}` 형식으로 참조합니다.

## 사용한 Secrets 변수 목록

| 이름                         | 설명                                |
| ---------------------------- | ----------------------------------- |
| `AWS_ACCESS_KEY_ID`          | IAM 사용자의 액세스 키              |
| `AWS_SECRET_ACCESS_KEY`      | IAM 사용자의 비밀 키                |
| `AWS_REGION`                 | S3 및 CloudFront 리전 코드          |
| `S3_BUCKET_NAME`             | 정적 파일이 업로드되는 S3 버킷 이름 |
| `CLOUDFRONT_DISTRIBUTION_ID` | CloudFront 배포 ID                  |

---

## 로컬에서 테스트해보고 싶은 경우

```bash
npm ci
npm run build
npx serve out
```

---

# 심화과제

## CDN과 성능 최적화

CloudFront와 같은 CDN(Content Delivery Network)을 도입함으로써 정적 자산이 전 세계에 분산된 엣지 서버에 캐싱되어, 사용자와 가장 가까운 위치에서 빠르게 파일을 전달할 수 있게 됩니다.  
이로 인해 초기 페이지 로드 속도와 전체적인 사용자 경험이 크게 개선됩니다.

### 네트워크 성능 비교 (S3 단독 vs CloudFront)

![Image](https://github.com/user-attachments/assets/5a339176-b069-4309-971a-601f48b276d0)
![Image](https://github.com/user-attachments/assets/46a2bc71-1b6b-4c3b-81b5-9465d34812a3)

> Chrome DevTools의 Network 탭에서 측정 (Same network 조건)

### 주요 개선 효과

- ✅ **지연 시간 감소**: 정적 자산이 글로벌 엣지 로케이션에서 제공되어, 물리적 거리에 따른 지연을 줄일 수 있습니다.
- ✅ **캐싱에 의한 리소스 절약**: 자주 변경되지 않는 파일은 CloudFront에서 캐싱되어 S3에 대한 직접 요청을 줄입니다.
- ✅ **트래픽 부하 분산**: S3 버킷에 직접 트래픽이 몰리는 것을 방지하여 서버 부하를 줄입니다.
- ✅ **장애 대응 향상**: 엣지 서버를 통해 일부 S3 이슈 발생 시에도 서비스 지속 가능성이 향상됩니다.

### 도입 요약

- 정적 웹 자산을 S3에 업로드한 후, CloudFront를 통해 전 세계 사용자에게 빠르게 전달
- GitHub Actions를 통해 자동화된 빌드 및 배포 파이프라인 구축
- `aws cloudfront create-invalidation` 명령을 통해 변경된 리소스의 최신 상태 반영

### 향후 고도화 방향

- 페이지 단위 캐시 TTL 조정으로 더 섬세한 캐시 정책 설정
- 정적 리소스에 `Cache-Control` 헤더 명시로 캐시 전략 최적화
- `Route53`과 `ACM`을 통한 HTTPS 및 커스텀 도메인 연결

---
### 과제 회고
현재 회사에서는 AWS를 사용하지 않고 있어 실제로 배포 자동화나 CDN을 활용한 최적화 경험이 없었는데, 이번 과제를 통해 그 흐름과 원리를 익힐 수 있어 좋았습니다.

사이드 프로젝트에서는 라이트하우스 점수를 높이기 위해 이미지, 폰트 등 정적 자산의 최적화는 물론, 렌더링 최적화까지 며칠씩 들여 작업했던 기억이 있습니다.
그에 비해 S3 + CloudFront 조합은 서버 단에서 간단한 설정만으로 전역 캐싱과 빠른 전달을 가능하게 해준다는 점이 매우 인상 깊었습니다.

GitHub Actions를 활용한 자동화 파이프라인도 특히 유익했습니다. 반복적인 배포 작업을 줄이고 사람의 실수를 방지할 수 있으며, Secrets 설정을 통해 민감한 정보를 안전하게 관리할 수 있다는 점에서 실무 적용 가능성이 높다고 느꼈습니다.

이번 경험을 바탕으로 이후 프로젝트에서는 배포 자동화뿐 아니라, 서버-클라이언트 협업 구조에서의 성능까지 고려한 프론트엔드 개발을 목표로 삼게 되었습니다.
