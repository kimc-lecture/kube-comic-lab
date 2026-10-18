# 만화로 배우는 실습 쿠버네티스

쿠베냥냥마켓에 첫 출근한 개발자가 되어, 서비스 런칭까지 동료들과 함께하는 쿠버네티스 실습 강의입니다. 만화로 상황을 이해하고 브라우저 터미널에서 직접 해결하며, 과제별 채점과 힌트로 학습합니다.

이 저장소는 **수강생을 위한 공개 안내 저장소**입니다. 소스 코드를 빌드할 필요 없이 Docker Hub의 이미지를 받아 실행합니다.

- Docker 이미지: `kimc1992/kube-comic-lab:0.0.1`
- 전체 실행·종료·정리 안내: [USER.md](USER.md)
- 기본 언어: 한국어. 상단에서 주요 UI를 영어로 전환할 수 있습니다.

## 준비 사항

Docker가 실행 중이어야 합니다. Docker에 CPU 2코어 이상(권장 4코어), 메모리 6GB 이상(권장 8GB), 디스크 여유 25GB 이상을 준비하세요. 최초 준비에는 인터넷이 필요합니다.

아래 명령은 Linux 또는 Windows WSL2의 Bash 터미널 기준입니다. Docker Desktop 등에서 `sudo`가 필요 없다면 빼고 실행하세요. 현재 배포 검증 대상은 Linux amd64이며 다른 환경의 검증 결과는 추후 안내합니다.

## 시작하기

```bash
sudo docker pull kimc1992/kube-comic-lab:0.0.1
sudo docker run -d --name kcna-lab --privileged --restart unless-stopped \
  -v kcna-lab-docker:/var/lib/docker \
  -v kcna-lab-home:/root \
  -p 127.0.0.1:7680:7680 \
  -p 127.0.0.1:7681:7681 \
  -p 127.0.0.1:30080-30083:30080-30083 \
  kimc1992/kube-comic-lab:0.0.1
```

브라우저에서 **http://localhost:7680**을 엽니다. 처음에는 클러스터와 공용 환경 준비에 몇 분 걸립니다. `클러스터 Ready`가 표시되면 첫 챕터부터 시작하세요. 챕터별 과제는 자동으로 준비됩니다.

내부 실습 클러스터 실행을 위해 `--privileged`가 필요합니다. 웹 터미널에는 별도 인증이 없으므로 예제처럼 포트를 `127.0.0.1`에 바인딩해 개인 PC에서 사용하세요.

## 종료와 다시 실행

```bash
sudo docker stop -t 30 kcna-lab    # 학습 데이터 보존 후 종료
sudo docker start kcna-lab         # 이어서 학습
sudo docker restart -t 30 kcna-lab # 재시작
```

같은 컨테이너가 있으면 `docker run`을 반복하지 않고 `docker start`를 사용합니다. 준비 완료를 기다린 뒤 같은 주소로 접속하세요.

## 완전히 정리하기

아래 명령은 **답안·작업 파일·진행 기록·실습 이미지 캐시를 모두 삭제**합니다.

```bash
sudo docker stop -t 30 kcna-lab
sudo docker rm kcna-lab
sudo docker volume rm kcna-lab-home kcna-lab-docker
# 배포 이미지도 삭제하려면
sudo docker image rm kimc1992/kube-comic-lab:0.0.1
```

컨테이너만 삭제하거나 버전을 업데이트하며 데이터를 보존하는 방법은 [USER.md](USER.md)를 확인하세요.

## 배우는 내용

| 파트 | 내용 |
|---|---|
| 입사 첫 주 | 클러스터, 컨테이너와 이미지, Pod |
| 런칭 준비 | Deployment, Service, Ingress, ConfigMap·Secret, Storage |
| 운영의 맛 | Scheduling, Probe·HPA, RBAC·보안, 트러블슈팅 |
| 배포 자동화 | Kustomize, Helm, GitOps·Argo CD |
| 수료식 | 종합 평가 |

## 접속 주소

| 서비스 | 주소 |
|---|---|
| 강의 포털 | http://localhost:7680 |
| Argo CD | http://localhost:30081 |
| Gitea | http://localhost:30082 |

실습 앱과 Ingress 주소는 해당 챕터에서 안내합니다. 사내 인증서 설정과 오류 진단은 [USER.md](USER.md)를 확인하세요.

쿠베냥냥마켓은 강의의 이해를 돕기 위한 가상의 회사입니다. 별도 라이선스가 명시되지 않은 강의 콘텐츠·만화 이미지의 이용·재배포는 제작자에게 문의해 주세요.
# kube-comic-lab
