# GitHub SSH 인증 운영 가이드

## 1. 목적과 적용 범위

Industrial AI Lab의 DGX Spark 서버에서 GitHub 저장소에 접근할 때 HTTPS Personal Access Token(PAT)을 반복 입력하지 않고 전용 SSH 키로 인증한다.

| 항목 | 설정 |
| --- | --- |
| 실행 서버 | DGX Spark `61.81.98.85` |
| Linux 사용자 | `prof-kweon` |
| 로컬 저장소 | `/home/prof-kweon/Industrial-AI-Lab` |
| GitHub 저장소 | `prof-kweon/Industrial-AI-Lab` |
| Git 원격 주소 | `git@github.com:prof-kweon/Industrial-AI-Lab.git` |
| SSH 개인 키 | `~/.ssh/id_ed25519_github` |
| SSH 공개 키 | `~/.ssh/id_ed25519_github.pub` |
| SSH 키 주석 | `Industrial-AI-Lab-DGX-Spark` |

이 설정은 연구용 NAS 마운트, GPU 드라이버 및 운영 중인 Docker 컨테이너를 변경하지 않는다.

## 2. 설정 완료 내역

1. `prof-kweon` 계정에 `~/.ssh` 디렉터리와 Ed25519 키 쌍 생성.
2. GitHub 계정 **Settings → SSH and GPG keys**에 공개 키를 **Authentication Key**로 등록.
3. `ssh -T` 인증 테스트 성공.
4. 저장소 `origin`을 HTTPS에서 SSH 주소로 변경.
5. 현재 저장소의 `core.sshCommand`에 GitHub 전용 개인 키를 지정.
6. `git ls-remote origin HEAD`로 PAT 입력 없는 원격 조회 성공.

현재 SSH 키는 편의를 위해 **passphrase 없이 생성**했다. 따라서 개인 키 파일과 DGX 계정의 접근 권한 보호가 중요하다.

## 3. GitHub 인증 확인

**실행 서버: DGX Spark (`61.81.98.85`)**

```bash
cd /home/prof-kweon/Industrial-AI-Lab

git remote -v
git ls-remote origin HEAD
```

`git remote -v`에서 `git@github.com:prof-kweon/Industrial-AI-Lab.git` 주소가 보이고, `git ls-remote`가 사용자 이름이나 PAT를 요구하지 않고 커밋 해시를 출력하면 정상이다.

SSH 인증만 별도로 확인할 때:

```bash
ssh -T -i ~/.ssh/id_ed25519_github -o IdentitiesOnly=yes git@github.com
```

정상적인 GitHub 인증 메시지는 `Hi prof-kweon! You've successfully authenticated, but GitHub does not provide shell access.`이다. GitHub는 SSH 셸을 제공하지 않으므로 이 명령이 인증 메시지와 함께 비영(非零) 종료 코드를 반환할 수 있다.

## 4. 일상적인 Git 작업

**실행 서버: DGX Spark (`61.81.98.85`)**

```bash
cd /home/prof-kweon/Industrial-AI-Lab

git status
git add <변경한-파일>
git commit -m "Describe the change"
git push origin main
```

- `git add`, `git commit`은 로컬 작업이므로 GitHub 인증이 필요 없다.
- `git push`, `git pull`, `git fetch`는 SSH 키를 통해 인증한다.
- 다른 환경에서 원격 저장소가 갱신되었다면 변경 이력을 확인한 후 안전하게 통합한다.
- `git push --force`, `git reset --hard`는 기존 이력을 손상시킬 수 있으므로 임의로 사용하지 않는다.

## 5. SSH 설정 명령 기록

아래 명령은 **설정 과정의 기록**이다. 이미 키가 생성되어 있으므로 **재실행하지 않는다**. 특히 `ssh-keygen`을 같은 경로로 다시 실행하면 기존 개인 키를 덮어쓸 위험이 있다.

```bash
# 실행 서버: DGX Spark (61.81.98.85)
mkdir -p -m 700 ~/.ssh
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_github -C "Industrial-AI-Lab-DGX-Spark" -N ""

cd /home/prof-kweon/Industrial-AI-Lab
git remote set-url origin git@github.com:prof-kweon/Industrial-AI-Lab.git
git config --local core.sshCommand "ssh -i ~/.ssh/id_ed25519_github -o IdentitiesOnly=yes"
```

## 6. 보안 및 운영 주의사항

- **개인 키** `~/.ssh/id_ed25519_github`를 GitHub 저장소, 채팅, 메일, NAS 공유 폴더 등에 복사하거나 공개하지 않는다.
- **공개 키** `~/.ssh/id_ed25519_github.pub`만 GitHub에 등록한다.
- 개인 키 권한은 `600`, `~/.ssh` 디렉터리 권한은 `700`으로 유지한다.
- SSH 키는 현재 GitHub **계정 수준 인증 키**다. 해당 계정이 접근 가능한 저장소에 대한 권한을 가질 수 있으므로, 저장소 하나만으로 권한을 제한하는 배포 키(deploy key)와는 다르다.
- 서버 분실·침해 또는 키 노출이 의심되면 GitHub **Settings → SSH and GPG keys**에서 해당 공개 키를 삭제하고 새 키를 발급한다.
- SSH 인증이 정상 작동하므로 이전에 발급한 HTTPS용 PAT가 더 이상 필요 없다면 GitHub 설정에서 폐기할 수 있다.
- GitHub에는 소스 코드·문서·설정을 보관하고, 대용량 데이터셋·모델 가중치·실험 산출물은 연구용 NAS에 보관한다. 실제 추적 여부는 `.gitignore`와 `git status`로 확인한다.

## 7. 문제 해결

| 증상 | 확인 사항 |
| --- | --- |
| `Permission denied (publickey)` | GitHub 공개 키 등록 여부, 개인 키 경로, `core.sshCommand` 확인 |
| GitHub 사용자 이름 또는 PAT를 다시 요청함 | `git remote -v`가 `https://`로 되어 있지 않은지 확인 |
| `Host key verification failed` | GitHub 공식 SSH 호스트 키 지문과 서버의 호스트 키 기록을 검증. 검증 없이 승인하지 않음 |
| `push rejected (fetch first)` | 원격에 새 커밋이 있는 상태. `git fetch origin` 후 커밋 이력을 확인하여 통합 |

**안전한 읽기 전용 점검 명령 — DGX Spark (`61.81.98.85`)**

```bash
cd /home/prof-kweon/Industrial-AI-Lab
git remote -v
git config --local --get core.sshCommand
GIT_PAGER=cat git log --oneline --graph --decorate --all -10
git status --short
```

## 8. 운영 상태

- GitHub SSH 인증 테스트: **성공**
- SSH 원격 저장소 조회: **성공**
- 기존 HTTPS → SSH 원격 주소 변경: **완료**
- 향후 변경 문서의 GitHub 커밋·푸시는 별도 수행 필요
