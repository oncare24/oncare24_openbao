# oncare24_openbao

OpenBao Docker 실행 설정입니다.

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