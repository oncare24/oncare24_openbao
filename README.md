# oncare24_openbao

OpenBao Docker 실행 설정입니다.
이 설정은 개발 및 팀 테스트용 설정입니다.

---

## 1. OpenBao 컨테이너 실행

```powershell
docker compose up -d
```

확인:

```powershell
docker ps
```

정상 결과 예시:

```text
oncare-openbao   openbao/openbao:2.5.3   0.0.0.0:8200->8200/tcp
```

---

## 2. OpenBao 컨테이너 접속

```powershell
docker exec -it oncare-openbao sh
```

컨테이너 내부에서 실행:

```sh
export BAO_ADDR=http://127.0.0.1:8200
bao status
```

처음 실행 시 결과 예시:

```text
Initialized    false
Sealed         true
```

---

## 3. 최초 1회 초기화

```sh
bao operator init
```

결과 예시:

```text
Unseal Key 1: ...
Unseal Key 2: ...
Unseal Key 3: ...
Unseal Key 4: ...
Unseal Key 5: ...

Initial Root Token: ...
```

> Unseal Key와 Root Token은 따로 보관하고 GitHub에 올리지 않습니다.

---

## 4. Unseal 실행

```sh
bao operator unseal
bao operator unseal
bao operator unseal
```

각 명령어 실행 시 `Unseal Key`를 하나씩 입력합니다.

상태 확인:

```sh
bao status
```

정상 결과:

```text
Initialized    true
Sealed         false
```

---

## 5. Root Token 설정

```sh
export BAO_TOKEN=초기화때_나온_Root_Token
```

예시:

```sh
export BAO_TOKEN=xxx.xxxxxxxx
```

---

## 6. KV 저장소 활성화

```sh
bao secrets enable -path=secret kv-v2
```

이미 활성화되어 있으면 그대로 사용하면 됩니다.

---

## 7. 저장/조회 테스트

저장:

```sh
bao kv put secret/cap2/test hello=world
```

조회:

```sh
bao kv get secret/cap2/test
```

정상 결과 예시:

```text
hello    world
```

---

## 8. 브라우저 접속

```text
http://localhost:8200
```

로그인 토큰:

```text
초기화 때 나온 Root Token
```

---

## 9. 백엔드 환경변수 설정

백엔드를 로컬 Windows에서 실행하는 경우:

```powershell
$env:BAO_ADDR="http://localhost:8200"
$env:BAO_TOKEN="초기화때_나온_Root_Token"
$env:BAO_KV_MOUNT="secret"
```
---

## 9-1. 같은 와이파이에서 다른 PC가 OpenBao 서버에 접속하는 방법

현재 README의 `localhost` 접속 방식은 OpenBao를 실행하는 PC 내부에서만 사용하는 로컬 테스트 방식입니다.

실제 팀 테스트에서는 OpenBao를 실행하는 PC를 KMS 서버처럼 사용하고, 백엔드가 실행되는 다른 PC에서 같은 와이파이를 통해 OpenBao 서버에 접속합니다.

예시 구조:

```text
[OpenBao 서버 PC]
- Docker로 OpenBao 실행
- IP 예시: 192.168.0.15
- OpenBao 포트: 8200

[백엔드 실행 PC]
- Spring Boot 백엔드 실행
- BAO_ADDR=http://192.168.0.15:8200 으로 접속
```

OpenBao 설정 파일 확인(openbao/config/openbao.hcl):
```text
ui = true

api_addr = "http://192.168.0.15:8200"

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = true
}
```

백엔드 PC 환경변수 설정:

```text
$env:BAO_ADDR="http://192.168.0.15:8200"
$env:BAO_TOKEN="초기화때_나온_Root_Token"
$env:BAO_KV_MOUNT="secret"
```

---

## 10. 재시작

```powershell
docker compose down
docker compose up -d
docker exec -it oncare-openbao sh
```

컨테이너 내부에서:

```sh
export BAO_ADDR=http://127.0.0.1:8200

bao operator unseal
bao operator unseal
bao operator unseal

bao status
```

정상 결과:

```text
Initialized    true
Sealed         false
```
---

## License Notice / 라이선스 안내

### English

This repository does not contain or modify the OpenBao source code.  
It only provides Docker Compose and configuration files to run OpenBao for the OnCare24 project.

OpenBao is licensed under the Mozilla Public License 2.0 (MPL-2.0).

- OpenBao official site: https://openbao.org/
- OpenBao GitHub: https://github.com/openbao/openbao
- OpenBao license: MPL-2.0

The OnCare24 project uses OpenBao as a separate KMS server for storing and managing encryption-related key materials.

### 한국어

이 저장소는 OpenBao의 소스 코드를 포함하거나 수정하지 않습니다.  
OnCare24 프로젝트에서 OpenBao를 Docker로 실행하기 위한 Docker Compose 및 설정 파일만 제공합니다.

OpenBao는 Mozilla Public License 2.0(MPL-2.0) 라이선스를 따릅니다.

- OpenBao 공식 사이트: https://openbao.org/
- OpenBao GitHub: https://github.com/openbao/openbao
- OpenBao 라이선스: MPL-2.0

OnCare24 프로젝트는 OpenBao를 별도 KMS 서버로 사용하며, 암호화 관련 키 정보를 저장하고 관리하기 위해 OpenBao API를 호출합니다.
