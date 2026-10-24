# 04. WSL2 GPU 환경에서 LLM 서빙과 GPU 컨테이너

## 목표
Windows PC의 GPU(RTX 5060 Ti)를 WSL2 Ubuntu에서 사용해 LLM을 서빙하고, Docker 컨테이너에서도 GPU를 사용하도록 구성한다.

## 환경
- 호스트: Windows PC (RTX 5060 Ti, VRAM 8GB), NVIDIA 드라이버 616.64
- WSL2 + Ubuntu 24.04
- Docker 29.x, Ollama

## 한 일

### 1. WSL2 설치와 GPU 인식
```
wsl --install -d Ubuntu-24.04
nvidia-smi
```
Windows의 NVIDIA 드라이버를 WSL이 공유하므로 Ubuntu 안에는 드라이버를 따로 설치하지 않았다. WSL에서 `nvidia-smi`로 GPU가 인식되는 것을 확인했다.

### 2. Ollama로 LLM 서빙
```
curl -fsSL https://ollama.com/install.sh | sh
ollama run llama3.2:3b
ollama ps
```
`ollama ps`의 PROCESSOR 항목이 `100% GPU`로 표시되어 GPU 추론을 확인했다. 모델 실행 중 Windows의 `nvidia-smi`에서 VRAM 사용량이 증가하는 것도 확인했다(약 0.6GB → 3.3GB).

### 3. Docker 설치
```
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
docker run --rm hello-world
```
일반 사용자가 `sudo` 없이 Docker를 쓰도록 `docker` 그룹에 추가했다.

### 4. NVIDIA Container Toolkit으로 컨테이너 GPU 사용
```
sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
docker run --rm --gpus all ubuntu nvidia-smi
```
Docker에 NVIDIA 런타임을 등록한 뒤, `--gpus all` 옵션으로 컨테이너 안에서 GPU(RTX 5060 Ti)가 인식되는 것을 확인했다.

## 막혔던 점과 해결
- **Ubuntu가 시작 메뉴에 보이지 않음**: `wsl -l -v`로 설치된 배포판이 없음을 확인하고 `wsl --install -d Ubuntu-24.04`로 설치했다.
- **ollama ps가 모델에게 전달됨**: 대화 모드(`>>>`)에서 입력해 모델에 대한 질문으로 처리되었다. `/bye`로 나온 뒤 터미널에서 다시 실행했다.

## 알게 된 점
- WSL2는 Windows 안에서 리눅스 커널을 실행하는 기능이며, GPU는 Windows 드라이버를 공유한다. 실제 GPU 서버는 리눅스에 드라이버와 컨테이너 툴킷을 직접 설치해야 한다.
- 컨테이너에서 GPU를 쓰려면 호스트 드라이버와 NVIDIA Container Toolkit이 필요하고, 실행 시 `--gpus` 옵션으로 연결한다.
- VRAM 8GB로는 3B급 모델이 적합하다.

## 한계와 다음 단계
- 단일 GPU, 개인 PC 환경이다. 멀티 GPU, 스케줄링, 모니터링은 다루지 않았다.
- Kubernetes(k3s)에 GPU 워크로드 배포
- GPU 사용량 모니터링 구성
