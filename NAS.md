# 연구용 NAS 운영 문서 (Industrial AI Lab)

> 최종 갱신: 2026-10-16 · 관리자: KWEON (prof-kweon)

연구용 NAS는 학생용 NAS와 **완전히 분리**되어 있으며, Linux 숫자 UID/GID와 Unix 권한으로 접근을 통제한다. Synology ACL(DSM 권한 탭)은 사용하지 않는다.

---

## 1. 현재 구성

| 항목 | 값 |
|---|---|
| NAS 공유 폴더 | `61.81.98.90:/volume1/Industrial-AI-Lab` |
| 마운트 경로 | `/mnt/nas-industrial-ai` (DGX Spark .85) |
| NFS | v4.0, `sec=sys`, Squash: 매핑 없음, 비동기: 아니요 |
| NFS 허용 클라이언트 | **61.81.98.85만** (.86은 의도적으로 제외) |
| 최상위 폴더 | `root:iai(5000)`, 모드 `2770` |
| 하위 폴더 | `prof-kweon(5001):iai(5000)`, 모드 `2770` |

```text
Industrial-AI-Lab/
├── datasets/
├── models/
├── checkpoints/
├── embeddings/
├── experiments/
└── shared/
```

- `2770`의 `2`는 setgid 비트로, 새로 만든 파일과 폴더가 자동으로 그룹 `iai`를 상속한다.
- 연구자는 `umask 0007`을 사용한다. 그래야 다른 구성원도 파일을 수정할 수 있고, 기타 사용자는 접근할 수 없다.

---

## 2. 계정 체계

### 2-1. ID 규칙

| 범위 | 용도 |
|---|---|
| 1000–4999 | 학생·교수 등 일반 계정 (스크립트 자동 할당) |
| **5000–5099** | **연구용 예약 (수동 지정만)** |

- `iai` 그룹 = **GID 5000**
- `prof-kweon` = **UID 5001**, 기본 그룹 `iai`
- 신규 연구자는 **UID 5002부터** 순서대로 사용
- 모든 서버(.82–.89)의 `/etc/login.defs`에서 `UID_MAX`·`GID_MAX`를 **4999**로 설정하여 자동 할당이 예약 범위를 침범하지 않도록 함 (백업: `/etc/login.defs.bak-20261016`)

**왜 필요한가:** `sec=sys`에서 NAS는 이름이 아니라 숫자만 본다. 학생 계정 번호는 서버마다 순차 할당되므로 10xx대 번호는 서버마다 다른 사람을 가리킨다. 실제로 .86의 UID 1009는 학생 계정이었다. 연구용 번호를 별도 범위로 분리해야 다른 서버의 학생과 겹치지 않는다.

### 2-2. 학생 계정과 연구 계정의 차이

| 구분 | 학생 계정 | 연구 계정 |
|---|---|---|
| 대상 | 캡스톤 등 수업 참여 학생 | `iai` 연구실 구성원 (교수·연구자) |
| 계정명 | `s<학번>` | 개인 계정명 (예: `prof-kweon`) |
| UID/GID 범위 | 1000–4999 | 5000–5099 |
| 번호 할당 | 스크립트가 자동 할당 (`create_team.sh`, `add_team_member.sh`, `03_create_accounts_samepw.sh`) | 관리자가 수동 지정 (`useradd -u`, `groupadd -g`) |
| 서버 간 번호 | 서버마다 다를 수 있음 | **모든 서버에서 동일** |
| 기본 그룹 | 개인 그룹 (`useradd` 기본 동작) | `iai` (GID 5000) |
| 팀·연구 그룹 | 팀 그룹을 보조 그룹으로 소속 | 기본 그룹 `iai`, 필요 시 팀 그룹을 보조 그룹으로 추가 |
| 사용 NAS | 학생용 (`course_2026_fall`, `course_2026_spring`) | 연구용 (`Industrial-AI-Lab`) |
| NAS NFS 허용 | `61.81.98.0/24` 전체 | .85만 |
| NAS 폴더 권한 | 팀 `root:<팀GID> 777` / 개인 `<UID>:<팀GID> 700` / 공유 `<관리자UID>:<팀GID> 770` | 전체 `2770` (setgid, 그룹 `iai`) |
| umask | 시스템 기본 (022) | `0007` (`.bashrc`에 설정) |
| 접속 방식 | Docker 컨테이너 SSH (`STUDENT_SSH_PORTS`) 또는 JupyterHub. 호스트 SSH 불가 | 호스트 SSH (`AllowUsers`에 추가 필요) |
| 수명 | 학기 단위 (봄·가을 코호트), 과제 종료 후 정리·재배정 | 연구실 소속 기간 동안 유지 |

**운영상 주의:**

- 학생 계정은 서버마다 번호가 다르므로, 학생 NAS의 접근 통제는 "그 팀이 쓰는 서버" 기준으로만 의미가 있다. 연구 계정은 어느 서버에서든 같은 사람을 가리키도록 번호를 고정한다.
- 연구 계정을 학생 스크립트로 만들지 않는다. 자동 할당은 4999까지만 배정하므로 연구용 번호가 나오지 않는다.
- 학생 계정을 `iai` 그룹에 넣지 않는다. 학생이 연구에 참여할 때는 5-4의 프로젝트 폴더 방식을 쓴다.

---

## 3. 알려진 정상 동작

- **File Station에 `Industrial-AI-Lab`이 보이지 않는다.** 정상이다. DSM 사용자(`admin123` 등)는 NAS 내부 UID를 쓰므로 root도 GID 5000도 아니어서 접근 불가로 판단된다. 폴더는 제어판 → 공유 폴더에는 항상 보이며, 설명란에 안내 메모가 있다.
- **`#recycle` 폴더가 `d---------`로 보인다.** DSM 휴지통 폴더로, 그대로 둔다.
- **NFS에서 삭제한 파일은 휴지통에 가지 않는다.** 즉시 삭제되므로 중요 데이터는 별도 백업한다.

---

## 4. 금지 사항

- **DSM 공유 폴더의 "권한" 탭에서 저장하지 않는다.** Synology ACL이 재적용되어 일반 사용자 접근이 다시 거부될 수 있다. 일반 탭(설명 등)은 수정해도 무방함을 확인했다.
- 연구용 공유 폴더에 `Everyone` 권한을 추가하지 않는다.
- NFS 규칙에서 "모든 사용자를 admin으로 매핑"하지 않는다.
- 하위 폴더 전체에 `chmod 777`이나 재귀적 권한 변경을 하지 않는다.
- NAS에 SSH로 접속해 그룹 번호를 변경하지 않는다. (DSM 내부 DB와 어긋날 위험)
- 학생용 NAS(`course_2026_fall`, `course_2026_spring`)와 그 마운트는 이 작업과 무관하므로 건드리지 않는다.

---

## 5. 운영 절차

### 5-1. 연구자 추가

NFS 클라이언트가 되는 **모든 서버에서 같은 UID**로 만든다.

```bash
# [해당 서버] 관리자 계정으로 실행
getent passwd 5002                       # 출력이 없어야 함
sudo useradd -m -s /bin/bash -u 5002 -g iai <계정명>
sudo passwd <계정명>
sudo chmod 700 /home/<계정명>
echo 'umask 0007' | sudo -u <계정명> tee -a /home/<계정명>/.bashrc
id <계정명>                              # uid=5002 gid=5000(iai)
```

호스트 SSH로 직접 접속해야 한다면 `/etc/ssh/sshd_config`의 `AllowUsers`에 추가하고 `sudo sshd -t`로 확인한 뒤 sshd를 reload한다.

### 5-2. 다른 서버(.86 등)를 연구용 NAS에 연결

1. 해당 서버에서 번호가 비어 있는지 확인한다.
   ```bash
   getent group 5000; getent passwd 5001
   ```
2. 같은 번호로 그룹과 계정을 만든다.
   ```bash
   sudo groupadd -g 5000 iai
   sudo useradd -m -s /bin/bash -u 5001 -g iai prof-kweon
   ```
3. 그 서버의 학생 컨테이너에서 학생이 root·sudo 권한을 갖는지 확인한다. 갖는다면 연결하지 않는다. (`sec=sys`는 허용된 IP의 root가 임의의 UID를 주장할 수 있다.)
4. DSM → 공유 폴더 → `Industrial-AI-Lab` → 편집 → **NFS 권한** 탭에서 해당 IP를 추가한다. (Squash: 매핑 없음, 권한 없는 포트: 거부됨)
5. 마운트한다.
   ```bash
   sudo mkdir -p /mnt/nas-industrial-ai
   sudo mount -t nfs -o vers=4.0,hard,proto=tcp 61.81.98.90:/volume1/Industrial-AI-Lab /mnt/nas-industrial-ai
   ```
6. 확인 후 `/etc/fstab`에 등록한다.

### 5-3. 다른 팀 프로젝트에 연구자로 참여

```bash
sudo usermod -aG <팀그룹명> prof-kweon    # -a 필수
```

- 팀 GID는 서버마다 다르므로 그 팀이 있는 서버에서 추가한다.
- 학생용 NAS의 팀 폴더에는 setgid가 없어서, 새 파일이 기본 그룹 `iai`로 생성된다. 팀 폴더에서는 `sg <팀그룹명> -c '명령'` 또는 `newgrp <팀그룹명>`으로 작업하거나, 만든 뒤 `chgrp <팀그룹명>`으로 고친다.
- NFS `sec=sys`는 그룹을 최대 16개까지만 전달한다. 학기가 끝난 팀 그룹에서는 `sudo gpasswd -d prof-kweon <팀그룹명>`으로 제외한다.

### 5-4. 연구 프로젝트에 학생 참여

학생을 `iai`에 넣지 않는다. 대신 연구용 NAS 아래에 별도 GID를 가진 프로젝트 폴더를 만든다. (GID는 5000–5099 범위에서 지정)

```bash
sudo mkdir /mnt/nas-industrial-ai/shared/<프로젝트명>
sudo chown 0:<프로젝트GID> /mnt/nas-industrial-ai/shared/<프로젝트명>
sudo chmod 2770 /mnt/nas-industrial-ai/shared/<프로젝트명>
```

---

## 6. 점검 및 복구

### 접근 점검

```bash
# [DGX Spark .85] admin으로 실행
sudo -u prof-kweon bash -c 'touch /mnt/nas-industrial-ai/ok && rm /mnt/nas-industrial-ai/ok && echo OK'
sudo -u nobody ls /mnt/nas-industrial-ai     # "허가 거부"가 나와야 정상
sudo stat -c '%A %a %u %g' /mnt/nas-industrial-ai   # drwxrws--- 2770 0 5000
```

### 일반 사용자 접근이 다시 거부될 때

대부분 DSM 권한 탭 저장으로 ACL이 재적용된 경우다. 최상위 폴더만 다시 전환하면 된다.

```bash
# [DGX Spark .85] admin으로 실행
sudo chown 0:5000 /mnt/nas-industrial-ai
sudo chmod 2770 /mnt/nas-industrial-ai
```

하위 폴더도 거부되면 해당 폴더에 `chown 5001:5000`, `chmod 2770`을 개별 적용한다. (재귀 변경은 하지 않는다.)

### login.defs 되돌리기

```bash
sudo cp -a /etc/login.defs.bak-20261016 /etc/login.defs
```

---

## 7. 원인 분석 기록 (2026-10-16)

**증상:** root(`sudo`)로는 접근되지만 `prof-kweon`은 `Permission denied`.

**원인:** 공유 폴더가 Synology ACL로 통제되고 있었다. DSM 권한 탭의 `iai` 허용 항목은 NAS 내부 `iai` 그룹의 숫자 GID를 가리키는데, 이는 Linux의 GID와 무관하고 구성원도 0명이었다. `sec=sys`에서 NAS는 클라이언트가 보낸 숫자 UID/GID만 보므로 어떤 ACL 항목과도 일치하지 않았다. root는 no-squash 설정으로 NAS에서도 root로 취급되어 ACL을 우회했다. 모드 비트가 `000`/`777`로 다르게 보인 것은 ACL에서 근사 계산된 표시값과 NFS 속성 캐시 때문이며, 실제 판정과는 무관했다.

**해결:** NFS 쪽에서 최상위 폴더에 `chown`/`chmod`를 실행해 Synology가 ACL 대신 Unix 권한으로 다루도록 전환했다. 학생용 NAS와 동일한 방식이다.

**작업 이력:**

1. .85: `iai` GID 1070 → 5000, `prof-kweon` UID 1009 → 5001, 기본 그룹 `iai`, 개인 그룹 삭제, 홈 `700`
2. NAS: 최상위 `root:5000 2770`, 하위 폴더 6개 생성
3. DSM: NFS 규칙에서 .86 삭제, 공유 폴더 설명란 메모 추가
4. .82–.89: `/etc/login.defs`의 `UID_MAX`·`GID_MAX`를 4999로 변경
