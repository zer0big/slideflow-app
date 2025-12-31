# 🚀 SlideFlow 배포 가이드

> Azure Static Web Apps + GitHub Actions를 활용한 자동 배포 완벽 가이드

---

## 📋 목차

1. [개요](#-개요)
2. [사전 준비](#-사전-준비)
3. [초기 배포 설정](#-초기-배포-설정)
4. [일반 개발 워크플로우](#-일반-개발-워크플로우)
5. [배포 확인 및 모니터링](#-배포-확인-및-모니터링)
6. [트러블슈팅](#-트러블슈팅)
7. [고급 설정](#-고급-설정)

---

## 🎯 개요

SlideFlow는 **Azure Static Web Apps**에 호스팅되며, **GitHub Actions**를 통해 자동으로 배포됩니다.

### CI/CD 파이프라인 구조

```
코드 변경 (main 브랜치)
    ↓
GitHub Actions 자동 트리거
    ↓
빌드 및 검증
    ↓
Azure Static Web Apps 배포
    ↓
자동 배포 완료 (30-60초)
```

### 주요 특징

- ✅ **자동 배포**: `main` 브랜치 푸시 시 자동 실행
- ✅ **빠른 배포**: 평균 30-60초 내 완료
- ✅ **무중단 배포**: 롤링 업데이트 방식
- ✅ **자동 롤백**: 실패 시 이전 버전 유지

---

## 🛠️ 사전 준비

### 필수 도구

| 도구 | 버전 | 설치 확인 |
|---|---|---|
| Git | 2.x 이상 | `git --version` |
| GitHub 계정 | - | https://github.com |
| Azure 계정 | - | https://portal.azure.com |
| VS Code (권장) | 최신 | - |

### 리포지토리 클론

```bash
# 리포지토리 클론
git clone https://github.com/zer0big/slideflow-app.git
cd slideflow-app

# 원격 저장소 확인
git remote -v
```

---

## 🏗️ 초기 배포 설정

> ⚠️ 이 섹션은 최초 1회만 수행합니다. 이미 설정된 경우 [일반 개발 워크플로우](#-일반-개발-워크플로우)로 이동하세요.

### 1️⃣ Azure Static Web App 생성

#### Azure Portal에서 생성

1. [Azure Portal](https://portal.azure.com) 접속
2. **리소스 만들기** → **Static Web App** 검색
3. 기본 정보 입력:
   - **구독**: 사용 중인 구독 선택
   - **리소스 그룹**: `rg-slideflow` (신규 생성)
   - **이름**: `stapp-slideflow`
   - **요금제**: `Free`
   - **지역**: `East Asia`

4. 배포 세부 정보:
   - **원본**: `GitHub` 선택
   - GitHub 계정 연동 및 인증
   - **조직**: `zer0big`
   - **리포지토리**: `slideflow-app`
   - **분기**: `main`

5. 빌드 세부 정보:
   - **빌드 사전 설정**: `Custom`
   - **앱 위치**: `/`
   - **API 위치**: (비워둠)
   - **출력 위치**: `/`

6. **검토 + 만들기** → **만들기**

#### CLI로 생성 (선택사항)

```bash
# Azure CLI 로그인
az login

# 리소스 그룹 생성
az group create --name rg-slideflow --location eastasia

# Static Web App 생성
az staticwebapp create \
  --name stapp-slideflow \
  --resource-group rg-slideflow \
  --location eastasia \
  --sku Free
```

### 2️⃣ GitHub Actions Workflow 생성

Azure Portal에서 Static Web App을 생성하면 자동으로 GitHub Actions workflow가 생성됩니다.

#### Workflow 파일 위치
`.github/workflows/azure-static-web-apps-ashy-field-0d8c27200.yml`

#### Workflow 구조

```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - main  # main 브랜치 푸시 시 자동 실행

jobs:
  build_and_deploy_job:
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v3
      - name: Build And Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_* }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "/"
          output_location: "/"
```

### 3️⃣ 환경 변수 설정

#### Gemini API 키 설정

1. Azure Portal → Static Web App → **구성**
2. **애플리케이션 설정** 탭 선택
3. **+ 추가** 클릭
4. 설정 추가:
   - **이름**: `GEMINI_API_KEY`
   - **값**: `your-gemini-api-key`
5. **저장** 클릭

#### CLI로 설정 (선택사항)

```bash
az staticwebapp appsettings set \
  --name stapp-slideflow \
  --resource-group rg-slideflow \
  --setting-names GEMINI_API_KEY=your-api-key
```

---

## 💻 일반 개발 워크플로우

### 📝 코드 수정 및 배포 프로세스

#### 1단계: 로컬 개발 환경 설정

```bash
# 최신 코드 받기
git pull origin main

# 새 기능 브랜치 생성 (권장)
git checkout -b feature/new-feature

# VS Code로 열기
code .
```

#### 2단계: 코드 수정

- `index.html` 파일 수정
- 로컬에서 테스트 (웹 서버 실행)

```bash
# Python으로 로컬 서버 실행
python -m http.server 8000

# 브라우저에서 http://localhost:8000 접속하여 테스트
```

#### 3단계: 변경사항 커밋

```bash
# 변경된 파일 확인
git status

# 모든 변경사항 스테이징
git add .

# 커밋 (의미있는 메시지 작성)
git commit -m "feat: 새로운 기능 추가"
```

**커밋 메시지 컨벤션**:
- `feat:` - 새로운 기능
- `fix:` - 버그 수정
- `docs:` - 문서 수정
- `style:` - 코드 포맷팅
- `refactor:` - 코드 리팩토링
- `perf:` - 성능 개선

#### 4단계: GitHub에 푸시

```bash
# main 브랜치에 직접 푸시 (간단한 변경)
git checkout main
git merge feature/new-feature
git push origin main

# 또는 Pull Request 생성 (중요한 변경)
git push origin feature/new-feature
# GitHub에서 Pull Request 생성 후 병합
```

#### 5단계: 자동 배포 확인

푸시 후 자동으로 배포가 시작됩니다:

1. GitHub → **Actions** 탭 접속
2. 최근 워크플로우 실행 상태 확인
3. 녹색 체크 ✅ = 배포 성공
4. 빨간색 X ❌ = 배포 실패 (로그 확인)

---

## 📊 배포 확인 및 모니터링

### GitHub Actions에서 확인

```bash
# 브라우저에서 확인
https://github.com/zer0big/slideflow-app/actions

# 또는 CLI로 확인 (GitHub CLI 필요)
gh run list
gh run view
```

### 배포 로그 확인

1. GitHub Actions 페이지에서 워크플로우 실행 클릭
2. **Build and Deploy Job** 확장
3. **Build And Deploy** 단계의 로그 확인

**성공 로그 예시**:
```
Status: Succeeded. Time: 30.8573575(s)
Deployment Complete :)
Visit your site at: https://ashy-field-0d8c27200.1.azurestaticapps.net
```

### Azure Portal에서 확인

1. Azure Portal → Static Web App → **환경**
2. 배포 상태 및 히스토리 확인
3. **개요** → URL 클릭하여 사이트 접속

### 배포된 사이트 테스트

```bash
# 브라우저에서 접속
https://ashy-field-0d8c27200.1.azurestaticapps.net

# curl로 확인
curl -I https://ashy-field-0d8c27200.1.azurestaticapps.net
```

---

## 🔧 트러블슈팅

### 문제 1: 배포가 실패합니다

**증상**: GitHub Actions에서 빨간색 X 표시

**해결 방법**:

1. **로그 확인**:
   ```
   GitHub → Actions → 실패한 워크플로우 → 로그 확인
   ```

2. **일반적인 원인**:
   - Workflow 파일 문법 오류
   - Azure API 토큰 만료
   - 파일 권한 문제

3. **해결**:
   ```bash
   # Workflow 재실행
   GitHub Actions → 실패한 워크플로우 → Re-run jobs
   
   # 또는 빈 커밋으로 재트리거
   git commit --allow-empty -m "chore: retrigger deployment"
   git push origin main
   ```

### 문제 2: 변경사항이 사이트에 반영되지 않습니다

**원인**: 브라우저 캐시 또는 CDN 캐시

**해결 방법**:

1. **브라우저 캐시 삭제**: `Ctrl + Shift + R` (하드 리프레시)
2. **시크릿 모드로 테스트**
3. **배포 완료 후 2-3분 대기** (CDN 전파 시간)

### 문제 3: API 키가 작동하지 않습니다

**확인 사항**:

1. Azure Portal → Static Web App → **구성** → `GEMINI_API_KEY` 확인
2. API 키 형식 확인
3. Google Cloud Console에서 API 키 활성화 상태 확인

**재설정**:
```bash
az staticwebapp appsettings set \
  --name stapp-slideflow \
  --resource-group rg-slideflow \
  --setting-names GEMINI_API_KEY=new-api-key
```

### 문제 4: GitHub Actions Workflow 파일을 푸시할 수 없습니다

**오류 메시지**: 
```
refusing to allow a Personal Access Token to create or update workflow
```

**원인**: GitHub 토큰에 `workflow` 권한 없음

**해결**:
1. GitHub → Settings → Developer settings → Personal access tokens
2. 토큰 재생성 시 `workflow` 스코프 체크
3. 또는 Azure Portal에서 자동 생성된 workflow 사용

---

## ⚙️ 고급 설정

### 환경별 배포 (Preview vs Production)

**Preview 배포** (Pull Request):
```yaml
on:
  pull_request:
    types: [opened, synchronize]
```

**Production 배포** (Main 브랜치):
```yaml
on:
  push:
    branches:
      - main
```

### 커스텀 도메인 설정

1. Azure Portal → Static Web App → **사용자 지정 도메인**
2. **+ 추가** 클릭
3. 도메인 입력 및 DNS 레코드 설정
4. 검증 후 SSL 자동 적용

### 롤백 절차

**최근 배포로 롤백**:

```bash
# 이전 커밋으로 되돌리기
git revert HEAD
git push origin main

# 또는 특정 커밋으로 리셋
git reset --hard <commit-hash>
git push origin main --force
```

**Azure Portal에서 이전 버전 복원**:
1. Static Web App → **환경**
2. 이전 빌드 선택 → **승격**

### 성능 모니터링

**Application Insights 연동**:

1. Azure Portal → Static Web App → **Application Insights**
2. 새 리소스 만들기 또는 기존 리소스 연결
3. 메트릭 및 로그 확인

---

## 📚 추가 리소스

### 공식 문서

- [Azure Static Web Apps 공식 문서](https://learn.microsoft.com/azure/static-web-apps/)
- [GitHub Actions 문서](https://docs.github.com/actions)
- [Azure CLI 참조](https://learn.microsoft.com/cli/azure/)

### 유용한 명령어 모음

```bash
# Azure 리소스 상태 확인
az staticwebapp show \
  --name stapp-slideflow \
  --resource-group rg-slideflow

# 환경 목록 조회
az staticwebapp environment list \
  --name stapp-slideflow \
  --resource-group rg-slideflow

# 배포 토큰 확인
az staticwebapp secrets list \
  --name stapp-slideflow \
  --resource-group rg-slideflow

# 로그 스트리밍
az staticwebapp logs show \
  --name stapp-slideflow \
  --resource-group rg-slideflow
```

---

## 🎓 요약: 일반적인 개발 사이클

```bash
# 1. 최신 코드 받기
git pull origin main

# 2. 코드 수정
# (index.html 편집)

# 3. 로컬 테스트
python -m http.server 8000

# 4. 커밋
git add .
git commit -m "feat: 기능 개선"

# 5. 푸시 (자동 배포 트리거)
git push origin main

# 6. GitHub Actions에서 배포 확인
# https://github.com/zer0big/slideflow-app/actions

# 7. 배포된 사이트 확인
# https://ashy-field-0d8c27200.1.azurestaticapps.net
```

---

## ❓ FAQ

**Q: 배포는 얼마나 자주 할 수 있나요?**  
A: 제한 없습니다. `main` 브랜치에 푸시할 때마다 자동으로 배포됩니다.

**Q: 배포 비용은 얼마인가요?**  
A: Free 플랜 사용 시 무료입니다. (월 100GB 대역폭, 0.5GB 스토리지)

**Q: 여러 환경(개발/스테이징/프로덕션)을 관리할 수 있나요?**  
A: 네, Pull Request마다 자동으로 Preview 환경이 생성됩니다.

**Q: 배포 실패 시 이전 버전이 유지되나요?**  
A: 네, 배포 실패 시 이전 버전이 계속 서비스됩니다.

---

<div align="center">

**문제가 해결되지 않으면 [GitHub Issues](https://github.com/zer0big/slideflow-app/issues)에 문의하세요!**

</div>
