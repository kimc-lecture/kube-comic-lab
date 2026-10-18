# Windows에서 WSL 설치하기

WSL은 Windows에서 Ubuntu 같은 Linux를 실행하는 기능입니다. 이 강의에서는 **WSL 2의 Ubuntu 터미널 + Docker Desktop**으로 실습합니다.

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

## 3. Docker Desktop 연결

1. [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/)를 설치하고 실행합니다.
2. **Settings → General → Use WSL 2 based engine**을 확인합니다. 지원 환경에서는 기본 적용되어 항목이 보이지 않을 수도 있습니다.
3. **Settings → Resources → WSL Integration**에서 **Ubuntu**를 활성화하고 적용합니다.

이제 **Ubuntu 터미널**에서 확인합니다.

```bash
docker version
docker run --rm hello-world
```

클라이언트와 서버 정보가 나오고 `Hello from Docker!`가 표시되면 준비 완료입니다. 이 구성에서는 Ubuntu 안에 Docker Engine을 별도로 설치하지 않습니다. 연결 오류가 나면 Docker Desktop이 실행 중인지와 Ubuntu 통합 설정을 확인하세요. [Docker 공식 WSL 2 안내](https://docs.docker.com/desktop/features/wsl/)

## 4. 강의 시작

[README의 시작하기](README.md#시작하기) 또는 [전체 실행 안내](USER.md#2-처음-실행)로 돌아가 **Ubuntu 터미널**에서 강의 이미지를 실행하세요. Docker Desktop 통합 환경에서는 안내 명령의 `sudo`를 빼고 실행할 수 있습니다.

브라우저는 Windows에서 열고 **http://localhost:7680**으로 접속합니다. 강의 실습에는 Docker가 사용할 메모리 6GB 이상과 디스크 여유 25GB 이상을 준비하세요.
