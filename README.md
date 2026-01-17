## 1. Network (네트워크)

---

### 웹 통신의 큰 흐름: https://www.google.com/ 을 접속할 때 일어나는 일

**1. URL 파싱 및 DNS 조회**:
- 브라우저가 URL을 파싱하여 프로토콜(HTTPS), 도메인(www.google.com), 경로(/) 분리
- DNS Lookup 수행: 브라우저 캐시 -> OS 캐시 -> hosts 파일 -> DNS 서버 순서
- DNS 서버에서 도메인에 해당하는 IP 주소 반환

**2. TCP 연결 수립** (3-Way Handshake):
```
클라이언트 --[SYN]--------> 서버
클라이언트 <-[SYN+ACK]---- 서버
클라이언트 --[ACK]--------> 서버
```

**3. TLS/SSL Handshake** (HTTPS):
- Client Hello: 지원하는 암호화 방식 전송
- Server Hello: 선택한 암호화 방식 + 인증서 전송
- 클라이언트가 CA로 인증서 검증
- Pre-Master Secret 교환으로 대칭키 생성

**4. HTTP 요청 전송**:
```http
GET / HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0
Accept: text/html
```

**5. 서버 처리 및 응답**:
- 웹 서버(Nginx/Apache)가 요청 수신
- 정적 파일이면 바로 반환, 동적이면 WAS로 전달
- HTTP 응답 생성 및 전송

**6. 브라우저 렌더링**:
- HTML 파싱 -> DOM 트리 생성
- CSS 파싱 -> CSSOM 트리 생성
- DOM + CSSOM -> 렌더 트리 생성
- 레이아웃 계산 -> 페인팅 -> 화면 표시

**7. TCP 연결 종료** (4-Way Handshake)

---

### OSI 7계층과 그 존재 이유, TCP/IP 4계층에 대해 설명해주세요

**OSI 7계층이 존재하는 이유**:
- 네트워크 통신 과정을 단계별로 파악 가능
- 특정 계층에 문제가 생기면 해당 계층만 수정 가능
- 표준화를 통한 상호 운용성 보장

**OSI 7계층**:

| 계층 | 이름         | 역할                                   | 프로토콜/장비            | 전송 단위 |
| ---- | ------------ | -------------------------------------- | ------------------------ | --------- |
| 7    | Application  | 사용자 인터페이스, 애플리케이션 서비스 | HTTP, FTP, SMTP, DNS     | Data      |
| 6    | Presentation | 데이터 변환, 암호화, 압축              | SSL/TLS, JPEG, MPEG      | Data      |
| 5    | Session      | 세션 수립/유지/종료 관리               | NetBIOS, RPC             | Data      |
| 4    | Transport    | 종단 간 신뢰성 있는 전송               | TCP, UDP                 | Segment   |
| 3    | Network      | 라우팅, 논리 주소 지정                 | IP, ICMP, 라우터         | Packet    |
| 2    | Data Link    | 물리 주소 지정, 오류/흐름 제어         | Ethernet, 스위치, 브리지 | Frame     |
| 1    | Physical     | 비트 전송, 전기 신호 변환              | 케이블, 허브, 리피터     | Bit       |

**TCP/IP 4계층**:

| TCP/IP 계층    | OSI 계층 | 역할                  |
| -------------- | -------- | --------------------- |
| Application    | 5, 6, 7  | 애플리케이션 프로토콜 |
| Transport      | 4        | 종단 간 통신          |
| Internet       | 3        | IP 라우팅             |
| Network Access | 1, 2     | 물리적 네트워크 접근  |

---

### 소켓이 열리고 나서 서버-클라이언트가 요청되는 과정을 OSI 계층과 연관지어 설명해주세요

**송신 측** (데이터 전송):

```
[Application Layer - 7계층]
애플리케이션이 HTTP 요청 데이터 생성
"GET /api/users HTTP/1.1\r\nHost: example.com\r\n\r\n"

[Presentation Layer - 6계층]
데이터 인코딩, 암호화 (HTTPS인 경우 TLS 암호화)

[Session Layer - 5계층]
세션 관리, 연결 상태 유지

[Transport Layer - 4계층]
TCP 헤더 추가 (출발지/목적지 포트, 시퀀스 번호, ACK 번호)
데이터를 세그먼트로 분할

[Network Layer - 3계층]
IP 헤더 추가 (출발지/목적지 IP 주소, TTL)

[Data Link Layer - 2계층]
이더넷 헤더/트레일러 추가 (MAC 주소, CRC)

[Physical Layer - 1계층]
비트 스트림으로 변환하여 물리적 매체로 전송
```

**수신 측** (데이터 수신):
역순으로 각 계층의 헤더를 제거하며 상위 계층으로 전달합니다.

---

### 소켓을 통해 서버-클라이언트가 연결되기 위한 시스템콜을 설명해주세요

**서버 측**:
```c
// 1. 소켓 생성 (전화기 구입)
socket(AF_INET, SOCK_STREAM, 0);
// 2. 소켓에 주소 바인딩 (전화번호 부여)
bind(server_fd, ...); // "나는 80번 포트에서 기다릴게"라고 OS에 등록
// 3. 연결 대기 상태로 전환 (벨소리 들을 준비)
listen(server_fd, backlog);
// 4. 클라이언트 연결 수락 (수화기 들기)
accept(server_fd, ...); // 연결된 클라이언트 전용의 새로운 소켓 생성
// 5. 데이터 송수신
read/write...
```

**클라이언트 측**:
```c
// 1. 소켓 생성
socket(AF_INET, SOCK_STREAM, 0);
// 2. 서버에 연결 요청
connect(sock_fd, ...); // 이때 OS가 클라이언트에게 남는 포트(Random Port)를 자동으로 할당 (Implicit Bind)
// 3. 데이터 송수신
write/read...
```

---

### 서버에서 accept() 시 소켓을 새로 만드는데, 그럼 서버 소켓은 2개인가요?

네, 역할이 다른 **두 종류의 소켓**이 존재합니다.

1. **Listening Socket** (리스닝 소켓):
   - 처음 `socket()`, `bind()`, `listen()` 한 소켓. (`server_fd`)
   - **역할**: 오직 **새로운 연결 요청(SYN)을 받아들이는(Accept) 문지기 역할**만 합니다.
   - 실제로 데이터를 주고받지는 않습니다.
2. **Connected Socket** (연결된 소켓):
   - `accept()`가 호출될 때마다 새로 생성되는 소켓. (`client_fd`)
   - **역할**: 연결된 **특정 클라이언트와 실제 데이터(read/write)를 주고받는 역할**입니다.
   - **1:N 구조**: 리스닝 소켓은 1개지만, 연결된 소켓은 클라이언트 수만큼(N개) 생성됩니다.

---

### 클라이언트가 1000명 연결되면 TCP 버퍼도 1000개 생기나요?

네, 맞습니다. **연결(Connected Socket)당 버퍼 1쌍**이 생성됩니다.

**구분 및 개수** (클라이언트 1000명 기준):
1. **Listen Socket**: 1개
   - **버퍼 없음**: 데이터를 주고받지 않으므로 Send/Recv 버퍼가 필요 없습니다.
   - **대기열 큐**(Queue): 대신 연결 요청을 잠시 담아둘 `SYN Queue`와 `Accept Queue`를 딱 1개씩 가집니다.
2. **Connected Socket**: 1000개
   - **버퍼 존재**: 각 소켓마다 독립적인 **Send Buffer + Revc Buffer** (1쌍)이 할당됩니다.
   - **메모리**: 따라서 연결이 많아지면 소켓 파일 자체보다, 이 **TCP 버퍼들이 차지하는 메모리**가 서버 리소스의 주범이 됩니다.

---

### 포트는 0~65535번 구멍밖에 없는데, 7만 명이 동시 접속하면 어떻게 되나요?

**결론부터 말하면: 접속 가능합니다.** (서버의 포트가 부족해서 못 받는 일은 없습니다.)

이유는 **소켓을 식별하는 기준**(Unique Key)이 단순히 "도착지 포트" 하나가 아니기 때문입니다.
TCP 연결은 아래 5가지 정보(5-Tuple)로 식별됩니다.

```
{ 프로토콜, 출발지 IP, 출발지 Port, 도착지 IP, 도착지 Port }
```

서버 입장에서는:
- **도착지 Port**: 80 (고정)
- **도착지 IP**: 서버 IP (고정)
- 하지만 **출발지 IP**와 **출발지 Port**가 클라이언트마다 다릅니다.

즉, **서버의 포트는 80번 하나만 사용**하면서도, 클라이언트의 IP와 포트 조합이 다르기 때문에 서로 다른 연결로 구분할 수 있습니다.

---

### 그럼 accept()로 만든 새로운 소켓은 포트를 새로 할당받나요? (포트 소모)

**아니요! 포트를 새로 할당받지 않습니다.**
이 부분이 가장 많이 오해하는 부분입니다.

1. **포트 공유**: `accept()`로 생성된 '연결된 소켓(Connected Socket)'은 리스닝 소켓과 **동일한 로컬 포트(예: 80번)를 그대로 사용**합니다. 다른 포트 번호(예: 81, 82...)를 새로 파는 것이 아닙니다.
2. **이유**: 만약 서버가 응답할 때마다 포트를 바꾼다면, 클라이언트(브라우저) 입장에서는 80번으로 보냈는데 갑자기 뜬금없는 포트에서 응답이 오는 셈이 되어 연결이 성립되지 않습니다.
3. **구분 방법**: "어? 그럼 80번 포트 하나에 소켓이 수만 개인데 안 섞이나요?" 
   -> 네, 아까 말씀드린 **5-Tuple**(특히 클라이언트의 IP와 포트)을 보고 커널이 정확히 배달해 줍니다.

---

### 소켓을 생성하는데 포트를 안 쓴다는 게 무슨 말인가요? 소켓은 포트가 있어야 하잖아요?

여기서 중요한 개념은 **소켓은 포트 그 자체가 아니라, '파일(객체)'이다** 라는 점입니다.

**소켓의 실체**:
- 컴퓨터(OS) 내부에서 소켓은 그냥 **파일 디스크립터**(File Descriptor, FD)라는 **정수**(숫자)일 뿐입니다.
- 예를 들어, `int client_fd = accept(...)`를 실행하면, OS는 메모리에 새로운 소켓 구조체를 만들고 `3`, `4`, `5` 같은 단순한 파일 번호(ID)를 리턴해줍니다.

**구조 비교**:
1. **소켓**(FD 3번): `{ 내 IP: 서버, 내 포트: 80 }` (리스닝용)
   - "누구든지 80번으로 오면 나한테 말해."
2. **소켓**(FD 4번): `{ 내 IP: 서버, 내 포트: 80, 상대 IP: 철수, 상대 포트: 1234 }` (철수 전담)
   - "난 80번인데, 철수(1234번)랑만 얘기해."
3. **소켓**(FD 5번): `{ 내 IP: 서버, 내 포트: 80, 상대 IP: 영희, 상대 포트: 5678 }` (영희 전담)
   - "나도 80번인데, 영희(5678번)랑만 얘기해."

보시다시피 **내 포트: 80**은 모두 같지만, **파일 번호(FD)가 다르고 상대방 정보가 다르기 때문에** 엄연히 다른 소켓(파일)들이 생성된 것입니다.
따라서 포트 번호 고갈 걱정 없이 소켓(파일)만 계속 만들어낼 수 있는 것입니다.

---

### 웹 서버 소프트웨어(Apache, Nginx)는 OSI 7계층 중 어디서 작동하나요?

Apache와 Nginx는 **7계층** (Application Layer)에서 동작합니다.

**이유**:
- HTTP 프로토콜을 기반으로 동작
- 요청 URL, 헤더, 쿠키 등 애플리케이션 레벨 데이터 처리

**로드밸런싱**:
- **L4 로드밸런싱** (Transport): IP+Port 기반 라우팅. 빠름.
- **L7 로드밸런싱** (Application): URL, 헤더, 쿠키 기반 라우팅. 정교한 분산 가능.

---

### TCP와 UDP의 차이점에 대해서 설명해주세요

| 구분           | TCP                            | UDP                       |
| -------------- | ------------------------------ | ------------------------- |
| 연결 방식      | 연결 지향 (3-way handshake)    | 비연결성                  |
| 신뢰성         | 신뢰성 있는 전송 (ACK, 재전송) | 신뢰성 없음               |
| 순서 보장      | 순서 보장 (Sequence Number)    | 순서 보장 안 함           |
| 속도           | 느림 (오버헤드)                | 빠름                      |
| 흐름/혼잡 제어 | 있음                           | 없음                      |
| 사용 예        | 웹(HTTP), 이메일, 파일 전송    | 스트리밍, 게임, DNS, VoIP |

---

### UDP는 그럼 신뢰성이 없나요?

UDP 자체는 신뢰성을 보장하지 않지만, **애플리케이션 레벨에서 신뢰성을 구현**할 수 있습니다.

**UDP를 사용하면서 신뢰성을 확보하는 방법**:
- **QUIC 프로토콜**: HTTP/3의 기반. UDP 위에서 신뢰성, 멀티플렉싱, 암호화 제공

---

### TCP에서 신뢰성을 제어하기 위해 어떤 것을 하나요?

1. **3-Way Handshake**: 연결 수립 시 양방향 통신 가능 여부 확인
2. **흐름 제어** (Flow Control): Sliding Window로 수신자 버퍼 상태에 맞춰 전송량 조절
3. **혼잡 제어** (Congestion Control): Slow Start, Congestion Avoidance로 네트워크 혼잡 방지
4. **오류 제어**: Checksum, Sequence Number, ACK, Timeout & Retransmission

---

### TCP의 흐름 제어(Flow Control)와 혼잡 제어(Congestion Control) 기법들

**1. 흐름 제어 (Flow Control)**: 송신 측과 **수신 측**의 속도 차이 해결
- **Stop and Wait**: 매번 응답(ACK)을 받고나서 다음 패킷 전송. (비효율)
- **Sliding Window**: 수신 측의 버퍼 크기(Window Size)만큼 확인 응답 없이 연속 전송. ACK를 받으면 윈도우를 옆으로 이동(Sliding)시킴.

**2. 혼잡 제어 (Congestion Control)**: 송신 측과 **네트워크**의 혼잡도 해결
- **기본 동작 원리**:
  - **Slow Start**: 패킷을 1개, 2개, 4개... 지수적으로 늘려가며 전송 속도를 올림.
  - **Congestion Avoidance**: 임계치(Threshold) 도달 시 선형적으로 1씩 증가.
- **대표적인 알고리즘**:
  - **TCP Tahoe**: 패킷 유실 시 윈도우 크기를 무조건 **1로 초기화**. (처음부터 다시 Slow Start)
  - **TCP Reno**: 패킷 유실(3 Duplicate ACKs) 시 윈도우 크기를 **절반으로 줄임**. (Fast Recovery)
  - **TCP CUBIC**: (현재 리눅스 기본값) 3차 함수 곡선을 이용해 윈도우 크기를 유연하게 조절.

---

### TCP 3-Way Handshake, 4-Way Handshake에 대해서 설명해주세요

**3-Way Handshake** (연결 수립):
1. Client -> Server: **SYN** (접속 요청)
2. Server -> Client: **SYN + ACK** (요청 수락)
3. Client -> Server: **ACK** (응답 확인) -> 연결 성립

**4-Way Handshake** (연결 종료):
1. Client -> Server: **FIN** (종료 요청)
2. Server -> Client: **ACK** (확인, 잠시만 기다려) -> Server는 남은 데이터 전송
3. Server -> Client: **FIN** (나도 끝났어)
4. Client -> Server: **ACK** (알겠어) -> Server 연결 종료
5. Client: **TIME_WAIT** 후 종료 (지연 패킷 처리 보장)

---

### HTTP와 HTTPS의 차이점에 대해서 설명해주세요

| 구분 | HTTP             | HTTPS                 |
| ---- | ---------------- | --------------------- |
| 포트 | 80               | 443                   |
| 보안 | 취약 (평문 전송) | 안전 (SSL/TLS 암호화) |

**HTTPS 특징**:
- **기밀성**: 암호화로 도청 방지
- **무결성**: 데이터 변조 탐지
- **인증**: CA 인증서로 서버 신원 확인

---

### HTTPS의 SSL Handshake 과정을 설명해주세요

1. **Client Hello**: 지원 암호화 방식, 랜덤값 전송
2. **Server Hello**: 선택한 암호화 방식, 랜덤값, **인증서**(공개키 포함) 전송
3. **인증서 검증**: 클라이언트가 CA 공개키로 서버 인증서 검증
4. **Pre-Master Secret**: 수신한 서버의 **공개키**로 생성한 pre-Master Secret을 암호화하여 전송 (비대칭키)
5. **Session Key 생성**: 양측이 서버측 랜덤값 + 클라이언트측 랜덤값 + Pre-Master Secret으로 동일한 **대칭키**(Session Key) 생성
6. **통신 시작**: 이후 데이터는 **대칭키**로 암호화하여 통신

---

### 왜 처음엔 비대칭키를 쓰고 나중엔 대칭키로 이루어지나요?

**Hybrid 방식의 이유**:
1. **비대칭키(공개키)**: 키 교환에 안전하지만, **복잡한 수학적 연산(소인수분해, 타원곡선 등)이 필요해 CPU 부하가 크고 매우 느림**.
2. **대칭키**: **단순한 비트 연산(XOR, Shift)이나 CPU 하드웨어 가속(AES-NI)**을 사용하므로 **매우 빠름**.

-> 따라서, **처음에만 비대칭키로 대칭키를 안전하게 공유**하고, 이후 **대용량 데이터는 빠른 대칭키로 통신**합니다.

---

### HTTP 버전별 차이를 설명해주세요

- **HTTP/1.0**: 매 요청마다 연결 수립/종료 (비효율)
- **HTTP/1.1**: **Keep-Alive** (연결 재사용), **Pipelining** (순차적 요청, Head-of-Line Blocking 문제 존재)
- **HTTP/2**: **Multiplexing** (한 연결로 동시 다발적 요청 처리), **Header Compression**, **Server Push**
- **HTTP/3**: **QUIC** (UDP 기반), TCP의 Head-of-Line Blocking 완전 해결, 초기 연결 속도 비약적 향상

---

### HTTP 상태 코드(Status Code)에 대해 설명해주세요

- **2xx** (Success): 요청 성공 (200 OK, 201 Created)
- **3xx** (Redirection): 페이지 이동 필요 (301 Moved Permanently, 302 Found)
- **4xx** (Client Error): 클라이언트 잘못 (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found)
- **5xx** (Server Error): 서버 잘못 (500 Internal Server Error, 502 Bad Gateway)

---

### GET과 POST의 차이점에 대해서 설명해주세요

| 구분        | GET                   | POST                 |
| ----------- | --------------------- | -------------------- |
| 목적        | 데이터 조회 (Read)    | 데이터 생성 (Create) |
| 데이터 위치 | URL Query String      | HTTP Body            |
| 멱등성      | O (여러 번 해도 같음) | X (할 때마다 생성됨) |
| 캐싱        | 가능                  | 불가능               |

---

### RESTful이란 무엇이며 특징을 설명해주세요

**REST** (Representational State Transfer):
HTTP URI로 자원을 명시하고, HTTP Method(GET, POST, PUT, DELETE)로 행위를 정하는 아키텍처 스타일입니다.

**RESTful 원칙**:
1. **URI는 자원을 표현** (동사 제외, 명사 위주, 계층 구조) -> `GET /users/1`
2. **Method는 행위를 표현** -> 조회(GET), 생성(POST), 수정(PUT/PATCH), 삭제(DELETE)
3. **Stateless**: 서버는 클라이언트 상태를 저장하지 않음 (JWT, LocalStorage)

---

### CORS란 무엇이며 설명해주세요

**CORS** (Cross-Origin Resource Sharing):
보안상 브라우저는 다른 출처(도메인, 포트 등)의 리소스 요청을 차단합니다(SOP). 이를 허용해주기 위해 서버가 응답 헤더에 "이 출처는 허용한다"라고 알려주는 메커니즘입니다.

- **Preflight Request**: 실제 요청 전 `OPTIONS` 메서드로 서버의 허용 여부를 먼저 확인함.
- **해결**: 서버에서 `Access-Control-Allow-Origin` 헤더 설정.

---

### Cookie와 Session의 차이점을 설명해주세요

| 구분    | Cookie           | Session             |
| ----- | ---------------- | ------------------- |
| 저장 위치 | **클라이언트** (브라우저) | **서버** (메모리/DB)     |
| 보안    | 취약 (탈취/변조 용이)    | 안전 (Session ID만 노출) |
| 용량    | 작음 (4KB)         | 서버 허용 범위 내          |
| 만료    | 설정 가능            | 브라우저 종료 시 삭제        |

**동작 원리**:
1. 로그인 성공 시 서버가 Session ID 생성 및 저장
2. Response Header(`Set-Cookie`)에 Session ID를 담아 전송
3. 이후 클라이언트는 요청마다 Cookie에 Session ID를 담아 보냄
4. 서버는 Session ID로 사용자 식별

---

### LocalStorage, SessionStorage, Cookie의 차이점

| 구분    | Cookie              | LocalStorage    | SessionStorage  |
| ----- | ------------------- | --------------- | --------------- |
| 저장 위치 | 클라이언트 (전송 시 헤더 포함)  | 클라이언트 (브라우저 내부) | 클라이언트 (브라우저 내부) |
| 용량    | 4KB                 | 5MB 이상          | 5MB 이상          |
| 서버 전송 | 매 요청마다 전송됨          | 전송되지 않음         | 전송되지 않음         |
| 수명    | 만료 기한 설정 가능         | 영구 저장 (지울 때까지)  | 탭/브라우저 종료 시 삭제  |
| 용도    | 인증 토큰(JWT), 오늘 그만보기 | 자동 로그인 저장, 장바구니 | 일회성 폼 데이터       |
