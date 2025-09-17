


# BloodHound 활용 가이드

BloodHound는 Active Directory 환경에서 복잡한 공격 경로를 시각적으로 분석하고 식별하는 데 사용되는 강력한 그래프 이론 도구입니다. 사용자, 컴퓨터, 그룹, GPO 등 AD 객체 간의 관계를 매핑하여 권한 상승 및 측면 이동 기회를 찾아냅니다.

---

### **1. BloodHound CE 설치 및 실행**

BloodHound Community Edition(CE)은 Docker를 통해 쉽게 설치하고 실행할 수 있습니다.

#### **설치 전 준비**
- `docker` 및 `docker-compose`가 설치되어 있어야 합니다.
```bash title="Make sure you have"
apt install docker.io
```

#### **설치 및 실행**
```bash(title="BloodHound CE 설치 및 실행")
# 1. docker-compose.yml 파일 다운로드
curl -L https://ghst.ly/getbhce -o docker-compose.yml

# 2. Docker 컨테이너 빌드 및 실행
docker-compose pull && docker-compose up -d

# 3. 초기 비밀번호 확인 (로그에서 'Password' 검색)
docker-compose logs bloodhound | grep -i passw
```

- 웹 브라우저에서 `http://127.0.0.1:8080` (또는 Kali VM의 IP)으로 접속하여 `admin` 계정과 확인된 비밀번호로 로그인합니다.

#### VM에서 Bloodhound 열고 로컬 브라우저에서 접속
```bash title="kali"
export BLOODHOUND_HOST=0.0.0.0
docker-compose up -d
```


#### **초기화**
```bash(title="BloodHound 도커 초기화")
# 모든 데이터 및 컨테이너 삭제
docker-compose down -v
```

---

### **2. 데이터 수집 (Ingestion)**

BloodHound에 데이터를 로드하려면, 대상 AD 환경에서 `SharpHound` (Windows) 또는 `bloodhound.py` (Linux/macOS) 인제스터(Ingestor)를 실행하여 데이터를 수집해야 합니다.

#### **`netexec`을 이용한 데이터 수집 (권장)**
- `netexec`은 자격증명을 통해 원격으로 `SharpHound`를 실행하고 데이터를 수집하는 가장 편리한 방법입니다.

```bash(title="netexec으로 BloodHound 데이터 수집")
# --bloodhound: BloodHound 데이터 수집 모듈 활성화
# --collection All: 모든 종류의 데이터 수집 (기본값)
# --dns-server: DNS 서버 지정 (선택 사항)
netexec ldap <DC_IP> -u <username> -p <password> --bloodhound --collection All --dns-server <DC_IP>
```

- 수집이 완료되면 `netexec`이 생성한 `.zip` 파일을 BloodHound GUI의 왼쪽 하단에 있는 업로드 버튼을 통해 업로드합니다.

---

### **3. 공격 경로 분석 (Attack Path Analysis)**

BloodHound는 AD 객체 간의 관계를 그래프 형태로 시각화하여, 도메인 관리자(Domain Admin) 권한을 획득하거나 특정 중요 자산에 접근할 수 있는 최단 경로를 찾아줍니다.

#### **자주 사용되는 Cypher 쿼리 예시**
- **도메인 관리자로 가는 최단 경로 찾기:**
  `MATCH p=shortestPath((n)-[*1..]->(m)) WHERE m.name = 'DOMAIN ADMINS@<DOMAIN.LOCAL>' RETURN p`
- **Kerberoastable 사용자 찾기:**
  `MATCH (u:User) WHERE u.hasspn = true AND u.dontreqpreauth = false RETURN u`
- **DCSync 권한을 가진 사용자 찾기:**
  `MATCH (u:User) WHERE u.hasspn = true AND u.dontreqpreauth = false RETURN u`

#### **그래프 시각화 해석**
- **노드 (Nodes):** 사용자, 컴퓨터, 그룹, GPO, OU 등 AD의 객체들을 나타냅니다.
- **엣지 (Edges):** 객체들 간의 관계(예: `MemberOf`, `AdminTo`, `HasSession`, `CanRDP`)를 나타냅니다.
- **공격 태그 (Attack Tags):** `Owned` (공격자가 장악한 객체), `HighValue` (도메인 관리자 등 중요 객체) 등으로 공격의 진행 상황을 표시할 수 있습니다.

---

### **4. 방어 (Mitigation)**

BloodHound는 공격뿐만 아니라 방어에도 매우 유용합니다. 공격 경로를 미리 파악하여 보안 취약점을 선제적으로 제거할 수 있습니다.

- **공격 경로 제거:** BloodHound로 식별된 위험한 관계(예: `AdminTo` 관계의 불필요한 계정)를 제거하거나 수정합니다.
- **권한 최소화:** 모든 사용자 및 서비스 계정에 최소 권한의 원칙을 적용합니다.
- **정기적인 감사:** 주기적으로 BloodHound를 실행하여 새로운 공격 경로가 생성되지 않았는지 확인합니다.


---




Bloodhund CE

```bash title="Make sure you have"
apt install docker.io
```


```bash
curl -L https://ghst.ly/getbhce -o docker-compose.yml
docker-compose pull && docker-compose up -d
docker-compose logs bloodhound | grep -i passw
```


then go to http://127.0.0.1:8080 with user admin and password u got

```bash title="ingest data you get from netexec"
netexec ldap $target -u 'TABATHA_BRITT' -p 'marlboro(1985)' --bloodhound --collection All --dns-server $target
```

Reference for cypher queries
https://support.bloodhoundenterprise.io/hc/en-us/articles/16721164740251-Searching-with-Cypher


### VM에서 Bloodhound 열고 로컬 브라우저에서 접속
```bash title="kali"
export BLOODHOUND_HOST=0.0.0.0
docker-compose up -d
```


### BloodHound 도커 초기화
```bash
docker-compose down -v
```


---

![[Pasted image 20250804233114.png]]

Bloodhound에서 각 대상 사이의 권한은 '**AD 개체 권한**'

