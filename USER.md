# 참가자: Docker Hub에서 받아 실행하고 정리하기

소스 코드나 빌드 도구를 설치할 필요가 없습니다. 강사가 Docker Hub에 배포한 이미지를 다운로드해 실행합니다. 아래 명령은 Linux 터미널 기준입니다. Docker Desktop 등에서 `sudo`가 필요하지 않으면 빼고 실행하세요. Windows에서는 WSL2의 Linux 터미널을 사용하면 아래 명령을 그대로 따라갈 수 있습니다.

Windows에서 처음 시작한다면 [WSL + Docker Engine 설치 안내](WSL.md)를 먼저 따라 하세요. Docker Desktop은 필수가 아닙니다. 사내망의 인증서 오류·프록시 설정은 [사내 CA 안내](CORPORATE-CA.md)를 확인하세요.

## 1. 준비

- Docker가 실행 중이어야 합니다.
- Docker에 메모리 6GB 이상을 할당하고 디스크 여유 공간 25GB 이상을 준비합니다.
- 강의 이미지: `kimc1992/kube-comic-lab:0.0.3`. 강사가 새 버전을 안내하면 태그를 변경합니다.

```bash
IMAGE='kimc1992/kube-comic-lab:0.0.3'
sudo docker pull "$IMAGE"
```

비공개 저장소를 안내받았다면 먼저 `sudo docker login`을 실행하고 접근 권한이 있는 계정으로 로그인합니다. 새 터미널에서 이미지 관련 명령을 실행할 때는 `IMAGE`를 다시 설정하세요.

## 2. 처음 실행

```bash
sudo docker run -d --name kcna-lab --privileged --restart unless-stopped \
  -v kcna-lab-docker:/var/lib/docker \
  -v kcna-lab-home:/root \
  -p 127.0.0.1:8085:7680 \
  -p 127.0.0.1:7681:7681 \
  -p 127.0.0.1:30080:30080 \
  -p 127.0.0.1:30083:30083 \
  "$IMAGE"
```

브라우저에서 **http://127.0.0.1:8085**을 엽니다. 최초 실행에서는 실습 이미지와 Ingress·metrics-server를 한 번에 준비합니다. 인터넷이 필요하고 다운로드와 설치에 몇 분 걸릴 수 있습니다. 준비 완료 후에는 챕터를 열면 해당 과제의 파일·리소스가 자동으로 준비되므로 세팅 버튼을 따로 누르지 않습니다. 다음 실행부터는 저장된 캐시와 정상 공용 구성요소를 재사용합니다.

`--privileged`는 내부 실습 클러스터 실행에 필요합니다. 웹 터미널은 인증 없는 root 셸이므로 포트는 위 예제처럼 **127.0.0.1에 바인딩**해서 사용하세요.

사내 CA는 **호스트의 이미지 다운로드와 컨테이너 내부 다운로드 양쪽**에 필요할 수 있습니다. [사내 CA 설정 전체 안내](CORPORATE-CA.md)를 먼저 확인하세요.

사내 인증서가 필요하면 위 `docker run` 명령의 `"$IMAGE"` 앞에 다음 옵션을 추가합니다.

```bash
-v /실제/경로/corp-ca.crt:/certs/corp-ca.crt:ro
```

이미 같은 이름의 컨테이너가 있으면 `docker run`을 반복하지 말고 아래 다시 시작 명령을 사용하세요.

### 기존 7680 포트를 8085로 바꾸기

외부 접속은 **8085**, 컨테이너 내부 포털은 **7680**을 사용합니다. 이미지 재빌드는 필요 없습니다. Docker 포트 연결 방식은 [공식 포트 게시 안내](https://docs.docker.com/engine/network/port-publishing/)를 참고하세요.

기존 컨테이너의 포트 변경에는 컨테이너 교체가 필요합니다. 먼저 기존 볼륨·CA·프록시 설정을 확인합니다.

```bash
sudo docker inspect kcna-lab --format '{{json .Mounts}}'
```

위 처음 실행 예제처럼 `kcna-lab-home`이 `/root`에, `kcna-lab-docker`가 `/var/lib/docker`에 연결된 경우:

```bash
sudo docker stop -t 30 kcna-lab
sudo docker rm kcna-lab
```

이후 위 **2. 처음 실행**의 `docker run`을 새 포트 설정으로 실행합니다. 기존 사내 CA 마운트와 프록시·미러 옵션도 그대로 추가합니다. 다른 볼륨 이름이나 경로를 썼다면 기존 설정을 유지하세요. 볼륨은 삭제하지 않습니다.

Windows 브라우저에서 **http://127.0.0.1:8085/**를 엽니다. PowerShell에서 확인하려면:

```powershell
curl.exe --noproxy "*" -v --max-time 5 http://127.0.0.1:8085/
```

포트 변경은 기존 7680 포트의 충돌·전달 문제를 구분하기 위한 방법입니다. 포털 서버 자체가 멈췄거나 클러스터가 실패한 경우에는 별도 진단이 필요합니다.

## 3. 종료·다시 실행·재시작

```bash
sudo docker stop -t 30 kcna-lab    # 종료: 답안·진행 상황·이미지 캐시 보존
sudo docker start kcna-lab         # 다시 실행
sudo docker restart -t 30 kcna-lab # 실행 중인 환경 재시작
```

다시 실행한 후에도 같은 브라우저 주소를 사용하고 클러스터 Ready를 기다립니다.

## 4. 실습 하나만 정리 / 초기화

포털의 **⏹ 실습 종료**로 해당 실습의 자원·답안·작업 파일을 정리할 수 있습니다. 완료 기록·이미지 캐시·공용 애드온은 유지됩니다.

터미널에서 실행할 수도 있습니다.

```bash
sudo docker exec kcna-lab lab stop 03     # 03번 실습 정리, 완료 기록 유지
sudo docker exec kcna-lab lab reset 03 4  # 03번의 과제 4만 초기화
sudo docker exec kcna-lab lab reset 03    # 03번 전체 초기화, 완료 기록도 초기화
```

기본적으로 새 챕터의 실습을 시작하면 이전 실습의 리소스·답안·작업 파일을 정리합니다. 포털은 실행 전에 확인하며 완료 기록은 남습니다. 여러 챕터를 동시에 유지하려면 최초 `docker run` 명령에 `-e LAB_AUTO_CLEANUP=0`을 추가합니다. 실행 중인 컨테이너의 설정을 바꾸려면 아래처럼 볼륨을 보존해 컨테이너를 교체하세요.

## 5. 컨테이너만 삭제 — 학습 데이터 보존

```bash
sudo docker stop -t 30 kcna-lab
sudo docker rm kcna-lab
```

두 볼륨은 남습니다. 2번의 `docker run`을 다시 실행하면 기존 답안·진행 상황·캐시를 재사용합니다. 이 단계에서는 `docker volume rm`을 실행하지 않습니다.

## 6. 완전히 정리 — 학습 데이터 삭제

**다음 명령은 답안·작업 파일·완료 기록·내려받은 실습 이미지를 모두 삭제합니다.** 다음 실행은 처음부터 시작합니다. 기본 이름으로 실행한 경우에만 그대로 사용하세요.

```bash
sudo docker stop -t 30 kcna-lab
sudo docker rm kcna-lab
sudo docker volume rm kcna-lab-home kcna-lab-docker
```

배포 이미지까지 디스크에서 지우려면, 1번의 `IMAGE`를 실제 사용한 값으로 설정한 뒤 실행합니다.

```bash
sudo docker image rm "$IMAGE"
```

다른 프로젝트의 데이터까지 지우는 `docker system prune`은 필요하지 않습니다.

## 7. 새 버전으로 업데이트

```bash
IMAGE='kimc1992/kube-comic-lab:0.0.3' # 강사가 안내한 새 버전으로 변경
sudo docker pull "$IMAGE"
sudo docker stop -t 30 kcna-lab
sudo docker rm kcna-lab
```

이후 2번의 `docker run`을 다시 실행합니다. 같은 두 볼륨을 사용하면 답안·진행 상황·캐시를 보존합니다.

## 8. 준비가 안 되거나 오류가 날 때

### 진단 로그

```bash
sudo docker ps -a --filter name=kcna-lab
sudo docker logs --tail 100 kcna-lab
sudo docker exec kcna-lab tail -50 /var/log/kcna/provision.log
sudo docker stats --no-stream kcna-lab
```

클러스터 준비 오류는 로그를 강사에게 전달하세요. 문의할 때는 사용한 이미지 태그, 운영체제, 위 진단 로그를 함께 전달해 주세요. 비밀번호·토큰 등은 지운 뒤 공유하세요.

## 구버전 GitOps 서버 정리 (0.0.1 → 0.0.3)

0.0.3는 Argo CD·Redis·Gitea를 받거나 설치하지 않습니다. 다만 구버전 볼륨을 재사용하면 기존 서버가 클러스터에 남아 실행될 수 있습니다. 새 이미지로 컨테이너를 교체한 뒤 아래 명령을 **원할 때 한 번** 실행합니다.

```bash
sudo docker exec kcna-lab kcna-addon retire-gitops
```

이 명령은 구버전 `argocd`, `gitea`, `nyang-dev`, `nyang-prod` 네임스페이스를 삭제합니다. Gitea 서버 안의 저장소도 삭제되므로 필요한 기록은 먼저 보관하세요. 로컬 작업 폴더·다른 챕터·두 볼륨은 유지합니다. 예전에 다운로드한 이미지는 캐시에 남으며 자동으로 지우지 않습니다. 14화의 새 작업 폴더는 `~/nyang-release`입니다.
