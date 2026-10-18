# 사내망에서 실행하기 — CA 인증서와 네트워크

회사에서 HTTPS 트래픽을 검사해 `x509: certificate signed by unknown authority` 또는 `SSL certificate problem`이 발생한다면 **회사가 제공한 CA 인증서**를 등록해 실행할 수 있습니다. 이 강의 이미지는 실행 시 `/certs`에 넣은 CA를 등록하는 기능을 제공합니다.

다만 CA는 인증서 신뢰 오류를 해결합니다. 외부 다운로드 차단, 필수 프록시, Docker 실행 제한까지 해제하지는 않습니다. 아래 조건을 확인하세요.

| 환경 | 필요한 준비 |
|---|---|
| HTTPS 검사로 인증서 오류 발생 | WSL 호스트 CA 등록 + 강의 컨테이너에 CA 마운트 |
| 명시적 HTTP/HTTPS 프록시 필수 | 호스트 Docker와 강의 컨테이너의 프록시·내부 통신 예외 설정 |
| 외부 레지스트리 차단 | 회사에서 허용한 미러 또는 강사가 제공하는 오프라인 배포본 |
| WSL·Docker·privileged 실행 금지 | 회사 정책에 맞는 별도 실습 환경 |

## 1. 회사 CA 파일 준비

IT·보안 담당자에게 HTTPS 검사에 사용하는 루트 CA와 필요한 중간 CA를 받습니다. 개인키가 포함된 `.pfx`·`.key` 파일은 필요하지 않습니다. CA 파일은 공개 저장소에 올리지 않습니다.

Ubuntu 터미널에서 파일을 준비합니다. 아래 Windows 사용자 이름과 경로는 실제 값으로 바꾸세요.

```bash
mkdir -p "$HOME/certs"
cp "/mnt/c/Users/내이름/Downloads/corp-ca.crt" "$HOME/certs/corp-ca.crt"
openssl x509 -in "$HOME/certs/corp-ca.crt" -noout -subject -issuer -dates
```

위 확인 명령은 PEM 형식 기준입니다. `BEGIN CERTIFICATE`로 시작하지 않는 DER 형식 `.cer` 파일은 변환합니다.

```bash
openssl x509 -inform DER -in "/mnt/c/Users/내이름/Downloads/corp-ca.cer" \
  -out "$HOME/certs/corp-ca.crt"
```

CA가 여러 개면 **인증서 하나당 파일 하나**로 `$HOME/certs` 안에 둡니다. 아래 호스트 등록 예제는 PEM 형식의 `.crt` 파일 기준입니다.

## 2. WSL 호스트에 등록 — 강의 이미지 다운로드용

Windows의 인증서 저장소와 WSL Ubuntu의 저장소는 별개입니다. **Docker Engine을 실행하는 Ubuntu**에서 등록합니다. `ca-certificates`와 `openssl`이 없다면 회사 패키지 미러나 허용된 경로로 먼저 설치하세요.

```bash
sudo install -m 0644 "$HOME/certs/corp-ca.crt" /usr/local/share/ca-certificates/corp-ca.crt
sudo update-ca-certificates
# Docker 설치 전이라면 재시작은 생략하고 WSL 안내의 설치 단계로 돌아갑니다.
sudo systemctl restart docker
sudo docker pull kimc1992/kube-comic-lab:0.0.1
```

다른 CA 파일도 같은 방법으로 등록합니다. Docker 재시작은 실행 중인 다른 컨테이너에도 영향을 줄 수 있습니다.

이미지 다운로드부터 실패한다면 강의 컨테이너의 `/certs` 마운트만으로 해결되지 않습니다. 호스트 CA·프록시·레지스트리 접근부터 확인하세요. Docker Desktop을 사용하는 경우에는 대신 [Docker의 호스트 CA 등록 안내](https://docs.docker.com/engine/network/ca-certs/)를 따릅니다.

## 3. 강의 컨테이너에도 등록 — 실습 이미지 다운로드용

호스트에서 이미지를 받을 수 있어도, 강의 컨테이너 안에서는 별도의 Docker가 실습 이미지를 받습니다. **같은 CA 폴더를 `/certs`에 읽기 전용으로 연결**합니다.

```bash
sudo docker run -d --name kcna-lab --privileged --restart unless-stopped \
  --mount type=bind,src="$HOME/certs",dst=/certs,readonly \
  -v kcna-lab-docker:/var/lib/docker \
  -v kcna-lab-home:/root \
  -p 127.0.0.1:7680:7680 \
  -p 127.0.0.1:7681:7681 \
  -p 127.0.0.1:30080-30083:30080-30083 \
  kimc1992/kube-comic-lab:0.0.1
```

기동 시 `.crt`·`.pem`·`.cer` 파일을 읽고 내부 Docker가 다운로드하기 전에 CA를 등록합니다. 인증서는 로컬 컨테이너에 적용되며 Docker Hub 원본 이미지에 업로드되지 않습니다. CA를 변경하면 컨테이너를 재시작해 다시 읽게 합니다.

**이미 `kcna-lab`이 있다면** 먼저 종료하고 컨테이너만 교체합니다. 아래 두 명령은 볼륨을 삭제하지 않습니다. 이후 위 CA 마운트가 포함된 `docker run`을 실행하세요.

```bash
sudo docker stop -t 30 kcna-lab
sudo docker rm kcna-lab
```

기존 두 볼륨을 그대로 연결하면 학습 데이터와 이미지 캐시를 재사용합니다. 이때 `docker volume rm`은 실행하지 않습니다.

## 4. 적용 확인

```bash
sudo docker logs kcna-lab 2>&1 | grep '사내 CA'
sudo docker exec kcna-lab tail -50 /var/log/kcna/provision.log
```

`사내 CA N개 등록` 메시지와 최종 준비 완료를 확인합니다. 브라우저에서 **http://localhost:7680**을 엽니다.

## 5. CA를 넣어도 다운로드가 안 되면

- `x509`·`certificate`: CA 누락·만료·잘못된 파일·중간 CA 누락을 확인합니다.
- `timeout`·`connection refused`·DNS 오류: 회사 VPN, WSL 네트워크, 방화벽과 프록시를 확인합니다.
- `407 Proxy Authentication Required`: 회사 프록시 인증 설정이 필요합니다.
- `429`·`toomanyrequests`: 레지스트리 요청 제한입니다. 인증서와 별도로 로그인이나 미러 설정이 필요합니다.

명시적 프록시가 필수라면 Ubuntu의 패키지 다운로드·호스트 Docker 데몬·강의 컨테이너 내부 Docker에 각각 설정이 필요합니다. 호스트 Docker는 [공식 프록시 설정](https://docs.docker.com/engine/daemon/proxy/)을 따릅니다. 강의 컨테이너는 `docker run`에 `-e HTTP_PROXY=...`, `-e HTTPS_PROXY=...`, `-e NO_PROXY=...`를 추가할 수 있습니다. 내부 Docker와 kind 노드가 직접 통신하도록 localhost·내부 서비스 이름·실제 Docker/kind/Pod/Service 주소 범위를 프록시 예외에 포함해야 합니다. 주소와 인증 방식은 회사 담당자에게 확인하세요. CA 마운트만으로 모든 프록시 환경에서 동작한다고 보장하지는 않습니다.

준비 중 접근하는 주요 레지스트리는 `docker.io`(Docker Hub), `registry.k8s.io`, `quay.io`입니다. 인증·이미지 저장소의 리다이렉트 주소도 허용이 필요할 수 있습니다. Docker 설치에는 Ubuntu 패키지 저장소와 `download.docker.com` 접근도 필요합니다.

회사 Docker Hub 미러가 있다면 `docker run`에 `-e KCNA_REGISTRY_MIRROR=https://미러주소`를 추가할 수 있습니다. 이 옵션은 **강의 컨테이너 내부 Docker의 Docker Hub 다운로드용**이며, 호스트의 강의 이미지 다운로드나 다른 레지스트리를 모두 대체하지 않습니다.

외부 인터넷이 완전히 차단되면 강사에게 **오프라인 번들 이미지**를 요청하세요. 현재 기본 태그 `0.0.1`은 첫 기동 때 인터넷에서 실습 이미지를 다운로드합니다. 오프라인 배포본의 제공 여부와 해당 환경의 검증 결과는 별도로 확인해야 합니다.

참고: [Docker CA 인증서 안내](https://docs.docker.com/engine/network/ca-certs/)
