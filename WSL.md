# Windows에서 WSL 설치하기

WSL은 Windows에서 Ubuntu 같은 Linux를 실행하는 기능입니다. 이 강의에서는 **WSL 2의 Ubuntu 안에 Docker Engine을 설치**해 실습합니다. Docker Desktop은 필수가 아닙니다.

## 1. WSL과 Ubuntu 설치

Windows 시작 메뉴에서 **PowerShell**을 검색하고 **관리자 권한으로 실행**한 뒤 입력합니다.

```powershell
wsl --install -d Ubuntu
```

설치 후 PC를 재부팅합니다. 시작 메뉴에서 **Ubuntu**를 열고 Linux 사용자 이름과 비밀번호를 만드세요. 비밀번호 입력 중에는 화면에 글자가 표시되지 않습니다.

설치 명령은 Windows 11 또는 Windows 10 버전 2004(빌드 19041) 이상에서 사용할 수 있습니다. 자세한 조건과 설치 문제 해결은 [Microsoft WSL 설치 안내](https://learn.microsoft.com/ko-kr/windows/wsl/install)를 참고하세요.

## 2. WSL 2 확인

**PowerShell**에서 실행합니다.

```powershell
wsl --update
wsl -l -v
```

`Ubuntu`의 `VERSION`이 **2**이면 준비됐습니다. **1**이면 다음 명령으로 변환합니다. 배포판 이름이 다르면 목록에 표시된 이름을 사용하세요.

```powershell
wsl --set-version Ubuntu 2
```

## 3. Ubuntu에 Docker Engine 설치

아래는 새로 설치한 Ubuntu 기준입니다. 기존 Docker가 정상 동작하면 설치를 반복하지 말고 확인 단계로 넘어갑니다. Docker Desktop을 이미 사용한다면 기존 WSL 통합을 이용할 수도 있습니다. 같은 Ubuntu에서 두 엔진을 함께 설정하지 마세요.

사내망에서 인증서 오류가 난다면 먼저 [사내 CA 설정](CORPORATE-CA.md)을 확인하세요. Windows에 등록된 CA가 Ubuntu에 자동으로 등록되는 것은 아닙니다.

**Ubuntu 터미널**에서 Docker 공식 패키지 저장소를 추가합니다.

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
sudo systemctl enable --now docker
```

기존에 `docker.io` 등 다른 패키지로 설치했다면 [Docker 공식 Ubuntu 설치 안내](https://docs.docker.com/engine/install/ubuntu/)의 충돌 패키지 정리 절차를 먼저 확인하세요. 수강생은 이미지 빌드가 필요 없어 Buildx와 Compose를 설치할 필요가 없습니다.

### systemd 오류가 나면

`System has not been booted with systemd`가 나오면 Ubuntu에서 `sudo nano /etc/wsl.conf`를 실행하고 다음 항목을 추가합니다. 기존 설정은 유지하고, 이미 `[boot]` 구역이 있다면 그 안의 값을 수정합니다.

```ini
[boot]
systemd=true
```

작업을 저장한 뒤 **PowerShell**에서 실행합니다. 이 명령은 실행 중인 모든 WSL 배포판을 종료합니다.

```powershell
wsl --shutdown
```

Ubuntu를 다시 열고 `sudo systemctl enable --now docker`를 실행합니다. [Microsoft systemd 안내](https://learn.microsoft.com/ko-kr/windows/wsl/systemd)

## 4. 동작 확인과 강의 시작

**Ubuntu 터미널**에서 실행합니다.

```bash
sudo docker version
sudo docker run --rm hello-world
```

서버 정보와 `Hello from Docker!`가 표시되면 준비됐습니다. 사내 CA가 필요한 경우에는 [CA 설정 안내](CORPORATE-CA.md)를 먼저 적용하세요.

[README의 시작하기](README.md#시작하기) 또는 [전체 실행 안내](USER.md#2-처음-실행)로 돌아가 같은 Ubuntu 터미널에서 강의 이미지를 실행합니다. 브라우저는 Windows에서 열고 **http://localhost:7680**으로 접속합니다.

실습에는 Docker가 사용할 메모리 6GB 이상과 디스크 여유 25GB 이상이 필요합니다. WSL의 메모리 제한은 Windows 사용자 폴더의 `.wslconfig`로 조정할 수 있습니다. [Microsoft WSL 고급 설정](https://learn.microsoft.com/ko-kr/windows/wsl/wsl-config#configuration-settings-for-wslconfig)
