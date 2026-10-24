# 03. 디스크 추가와 rsync/cron 자동 백업

## 목표
VM에 디스크를 추가해 파티션·파일시스템·마운트를 구성하고, rsync로 백업과 복구를 하고, cron으로 자동화한다.

## 환경
- 서버: Rocky Linux 9.8 (VirtualBox VM `rocky01`)
- 추가 디스크: 5GB (VDI, 동적 할당)

## 한 일

### 1. 디스크 추가와 인식 확인
VM을 종료하고 VirtualBox에서 5GB 디스크를 추가한 뒤 `lsblk`로 `sdb`를 확인했다.

### 2. 파티션, 파일시스템, 마운트
```
sudo parted /dev/sdb --script mklabel gpt mkpart primary xfs 1MiB 100%
sudo mkfs.xfs /dev/sdb1
sudo mkdir /data
sudo mount /dev/sdb1 /data
df -h /data
```
| 단계 | 하는 일 |
|---|---|
| 파티션 | 디스크에 사용 영역(`sdb1`)을 정의 |
| 파일시스템 | `sdb1`에 xfs 파일 저장 규칙을 생성 |
| 마운트 | `sdb1`을 `/data` 폴더에 연결 |

`/data`에 약 5GB가 보이고 파일이 정상적으로 읽고 써지는 것을 확인했다.

### 3. rsync 백업과 복구
```
rsync -av ~/important/ /data/backup/important/
rm ~/important/resume.txt
rsync -av /data/backup/important/ ~/important/
```
테스트 파일을 일부러 삭제한 뒤 백업본에서 복구되는 것을 확인했다.
rsync는 변경된 파일만 전송하므로 복구 시 삭제된 파일만 전송되었다.

### 4. cron 자동 백업
`crontab -e`에 1분 주기 작업을 등록해 파일을 만든 뒤 백업 폴더에 자동으로 복사되는 것을 확인했다.
이후 실제 운영 주기인 매일 새벽 2시(`0 2 * * *`)로 변경했다.

### 5. 재부팅 후 자동 마운트 (/etc/fstab)
```
echo "UUID=$(sudo blkid -s UUID -o value /dev/sdb1) /data xfs defaults,nofail 0 0" | sudo tee -a /etc/fstab
sudo umount /data
sudo mount -a
```
디스크 이름(`sdb`)은 바뀔 수 있어서 UUID로 등록했고, 디스크가 없어도 부팅이 멈추지 않도록 `nofail` 옵션을 넣었다. 먼저 `mount -a`로 설정을 검증한 뒤 재부팅해서 `/data`가 자동으로 마운트되고 백업 파일이 유지되는 것을 확인했다.

## 막혔던 점과 해결
- **parted 명령 실패**: `parted /dev /sdb`처럼 경로 사이에 공백이 들어가 `/dev`(디렉터리)를 디스크로 인식했다. 에러 메시지의 `Is a directory`를 읽고 원인을 찾아 `/dev/sdb`로 수정했다. 디스크에는 변경이 없었다.

## 알게 된 점
- 리눅스는 드라이브 문자 대신 디스크를 폴더에 마운트해서 사용한다.
- 파티션 명령은 대상 디스크를 잘못 고르면 데이터가 지워지므로 `lsblk`로 먼저 확인한다.
- 같은 서버 안의 다른 디스크 백업은 서버 장애나 랜섬웨어에 취약하다. 3-2-1 원칙(복사본 3개, 매체 2종류, 외부 보관 1개)이 필요하다.

## 한계와 다음 단계
- RAID/LVM 구성과 장애 시나리오 실습
- NFS 공유와 백업 복구 절차서 작성
