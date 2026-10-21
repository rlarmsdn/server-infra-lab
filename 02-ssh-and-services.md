# 02. 호스트 이름, 서비스 상태 확인, SSH 원격 접속

## 목표
호스트 이름을 설정하고, systemd와 firewalld 상태를 확인한 뒤, Windows PC에서 VM 서버로 SSH 원격 접속한다.

## 환경
- 서버: Rocky Linux 9.8 (VirtualBox VM `rocky01`, NAT 네트워크)
- 접속 클라이언트: Windows PowerShell (내장 OpenSSH 클라이언트)

## 한 일

### 1. 호스트 이름 변경
```
sudo hostnamectl set-hostname rocky01
hostnamectl
```
`Static hostname: rocky01`로 변경된 것을 확인했다.

### 2. 서비스와 방화벽 상태 확인 (systemd, firewalld)
```
systemctl status sshd
systemctl is-enabled sshd
sudo systemctl status firewalld
sudo firewall-cmd --list-all
```

| 확인 항목 | 결과 |
|---|---|
| sshd | `enabled`, `active (running)`, 22번 포트 listening |
| firewalld | `enabled`, `active (running)` |
| 방화벽 허용 서비스 | `cockpit dhcpv6-client ssh` (ssh는 설치 시 기본 허용) |

### 3. VirtualBox 포트 포워딩
NAT 네트워크의 VM은 호스트 PC에서 직접 접근할 수 없어서 포트 포워딩 규칙을 추가했다.

| 이름 | 프로토콜 | 호스트 IP | 호스트 포트 | 게스트 포트 |
|---|---|---|---|---|
| ssh | TCP | 127.0.0.1 | 2222 | 22 |

호스트 포트를 22가 아닌 2222로 쓴 것은 호스트의 22번 포트와 겹치지 않게 하기 위해서다.

### 4. PowerShell에서 SSH 접속
```
ssh -p 2222 duru@127.0.0.1
```
- 최초 접속 시 서버의 호스트 키 지문(fingerprint)을 확인하고 `yes`로 등록했다.
- 접속 후 `hostname`, `whoami`, `ip a`로 VM 서버에서 명령이 실행되는 것을 확인했다 (`rocky01`, `duru`, `10.0.2.15`).

- ### 5. SSH 키 인증 설정
```
ssh-keygen -t ed25519
```
PC에서 키 쌍(개인 키/공개 키)을 생성하고, 공개 키를 서버의 `~/.ssh/authorized_keys`에 등록했다. 서버는 이 파일의 권한이 느슨하면 키 인증을 거부하므로 `~/.ssh`는 700, `authorized_keys`는 600으로 설정했다.
이후 비밀번호 입력 없이 접속되는 것을 확인했다.

## 알게 된 점
- `systemctl status`는 출력이 화면에 다 들어가면 바로 프롬프트로 돌아오고, 길 때만 페이저(less)가 열려 `q`로 빠져나온다.
- 방화벽의 `ssh` 허용은 직접 설정한 것이 아니라 설치 시 기본값이었다.
- 처음 접속하는 서버에서는 호스트 키 지문을 확인하는데, 이는 접속 대상 서버가 바뀌었는지 검증하는 보안 장치다.
- 실제 서버는 모니터 없이 SSH로 관리하므로, 이 구성으로 실무와 같은 방식으로 접속해 볼 수 있었다.

## SSH 접속이 안 될 때 점검 순서 (정리)
1. 서버에 IP가 있고 네트워크가 연결돼 있는가 → `ip a`
2. sshd 서비스가 실행 중인가 → `systemctl status sshd`
3. 방화벽이 22번 포트를 허용하는가 → `firewall-cmd --list-all`
4. (VM 환경) 포트 포워딩 규칙이 맞는가

## 다음 단계
- SSH 키 인증 설정 (비밀번호 없이 접속)
- 사용자 추가와 권한 관리

