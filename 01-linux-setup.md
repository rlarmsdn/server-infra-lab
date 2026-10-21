# 01. Rocky Linux 9 서버 구축

## 목표
VirtualBox 가상 머신에 Rocky Linux 9.8 Minimal을 직접 설치하고, 기본 네트워크와 패키지 저장소 연결을 확인한다.

## 환경
- 호스트: Windows PC (Ryzen 5 7500X3D, RAM 32GB, NVMe SSD)
- 가상화: VirtualBox 7.2.22
- VM 사양: RAM 2GB / CPU 2코어 / 디스크 20GB / 네트워크 NAT
- OS: Rocky Linux 9.8 Minimal Install

## 한 일
1. VirtualBox에서 VM(`rocky01`)을 만들고 Rocky Linux 9.8 Minimal ISO를 연결했다.
   자동 설치(Unattended Installation)는 끄고 설치 과정을 직접 진행했다.
2. 설치 프로그램에서 시간대(Asia/Seoul), 자동 파티셔닝, 네트워크 활성화, root 계정, 관리자 권한(sudo)을 가진 일반 사용자를 설정했다.
3. 재부팅 후 일반 사용자로 로그인해서 아래 항목을 확인했다.

| 확인 항목 | 명령어 | 결과 |
|---|---|---|
| 현재 사용자 | `whoami` | 일반 사용자로 로그인된 것 확인 |
| 네트워크 | `ip a` | `enp0s3`에 `10.0.2.15/24` 할당 (NAT) |
| 외부 저장소 연결 | `sudo dnf check-update` | 패키지 업데이트 목록 조회 성공 |

## 막혔던 점과 해결
- **VM 창에서 마우스가 빠져나오지 않음**: 호스트 키(기본 Right Ctrl)가 키보드에서 동작하지 않았다.
  VirtualBox 환경 설정 → 입력 → 가상 머신에서 호스트 키를 `Ctrl + Alt`로 변경해 해결했다.
- **설치 방식 선택**: 자동 설치를 켜 두면 설치 과정을 학습할 수 없어서 해제하고 수동으로 진행했다.

## 배운 점
- 설치 단계에서 파티션, 네트워크, 계정 권한이 어떻게 구성되는지 직접 확인했다.
- Minimal Install은 기본 패키지가 최소라서 필요한 구성 요소를 직접 설치해야 한다.

## 다음 단계
- 호스트 이름 변경, SSH 접속 설정, 방화벽(firewalld) 확인, systemd 서비스 관리
