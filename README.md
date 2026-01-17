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
1. CPU가 가상 주소 접근 -> **MMU** 확인 -> Valid bit 0 (없음)
2. **Page Fault** 발생 -> OS로 트랩 전송
3. OS가 디스크(Swap Area)에서 해당 페이지 탐색
4. 물리 메모리의 빈 프레임에 로드 (없다면 페이지 교체 알고리즘 동작)
5. 페이지 테이블 업데이트 후 프로세스 재개
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

트랜잭션은 데이터베이스의 논리적 작업 단위입니다.

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

- **RDBMS**: 정해진 스키마, 관계형 데이터, 데이터 무결성 중시 (금융, 결제). scale-up 위주.
- **NoSQL**: 유연한 스키마, 대용량 분산 처리, 읽기/쓰기 성능 중시 (로그, 소셜 피드). scale-out 용이.

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
- **Snapshot Isolation**: 트랜잭션 시작 시점의 스냅샷을 보는 효과를 줍니다.
- 이를 통해 `READ_COMMITTED`나 `REPEATABLE_READ` 격리 수준에서 **Non-Blocking Read**를 가능하게 합니다.

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

- **역인덱스**: 키워드를 통해 문서를 찾는 방식 (책의 맨 뒤 '색인'과 유사). RDBMS의 `LIKE` 검색보다 훨씬 빠릅니다.
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

<br>

1. `.java` 소스코드 -> `javac` 컴파일 -> `.class` 바이트코드
2. **JVM**(Class Loader)이 바이트코드 로드
3. **Execution Engine** (Interpreter + **JIT Compiler**)이 기계어로 변환 실행

**JIT Compiler**: 반복되는 코드를 네이티브 코드로 미리 컴파일해두어 인터프리터의 느린 속도 개선.

</details>

<details>
<summary>GC (Garbage Collection) 란?</summary>

<br>

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

<br>

- **Synchronized**: 락(Lock)을 걸어 한 번에 하나의 스레드만 접근 허용 (가시성 + 원자성 보장).
- **Volatile**: CPU 캐시가 아닌 **메인 메모리에서 직접 읽기/쓰기** (가시성 보장, 원자성 보장 X). 상태 플래그 용도로 적합.
- **Atomic Class**: CAS 알고리즘으로 락 없이 스레드 안전한 연산 제공.

</details>

<details>
<summary>동시성 로직에 쓰이는 자료구조 (ConcurrentHashMap)</summary>

<br>

<br>

**HashMap**은 동기화되지 않음. **Hashtable**은 전체에 락을 걸어 느림.
**ConcurrentHashMap**:
- **Java 7 (Segment Lock)**: 맵을 여러 세그먼트로 나누어 부분적으로 락을 걺.
- **Java 8+ (Bucket Lock / CAS)**:
  - 각 버킷(노드)의 첫 번째 노드에만 `synchronized`를 걸거나,
  - **CAS (Compare-And-Swap)** 알고리즘을 사용하여 락 없이 원자적 연산 수행.

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
- **계약**(Contract):
  1. `equals()`가 `true`이면, 두 객체의 `hashCode()`는 반드시 같아야 함.
  2. `hashCode()`가 같다고 해서 `equals()`가 반드시 `true`일 필요는 없음 (해시 충돌).
  - equals를 재정의할 때 hashCode를 재정의하지 않으면, 같은 값을 가진 객체가 해시 컬렉션(HashMap, HashSet)에서 다른 객체로 취급되어 값을 찾지 못하는 오동작 발생.

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

- **기본형**(Primitive): 값이 복사되어 전달됨. 메서드에서 변경해도 원본 영향 없음.
- **참조형**(Reference): **주소값의 복사본**이 전달됨.
  - 메서드 내부에서 객체의 멤버 변수(`obj.name = "xx"`)를 바꾸면 **원본도 바뀜** (같은 힙 메모리를 가리키므로).
  - 하지만 변수 자체(`obj = newObj`)를 바꾸면 **원본은 바뀌지 않음** (복사된 주소값만 바뀌므로).

</details>

<details>
<summary>Reflection(리플렉션) 이란?</summary>

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

Client -> **DispatcherServlet** -> **HandlerMapping**(Controller 찾기) -> **HandlerAdapter** -> **Controller**(비즈니스 로직) -> **ViewResolver** -> View/Data Response

</details>

<details>
<summary>IoC (Inversion of Control) 와 DI (Dependency Injection)</summary>

<br>

- **IoC**(제어의 역전): 객체의 생명주기 관리를 개발자가 아닌 프레임워크(Container)가 담당.
- **DI**(의존성 주입): 필요한 의존 객체를 직접 생성하지 않고 외부(Container)에서 주입받음. 결합도를 낮춤.
- 생성자 주입(Constructor Injection)을 권장 (불변성, 순환참조 방지).

</details>

<details>
<summary>Spring Bean 생명주기</summary>

<br>

생성 -> 의존성 주입 -> 초기화(@PostConstruct) -> 사용 -> 소멸(@PreDestroy)

</details>

<details>
<summary>AOP (Aspect Oriented Programming)</summary>

<br>

핵심 로직과 공통 관심사(로깅, 트랜잭션, 보안 등)를 분리하여 모듈화하는 것.
Spring은 프록시 패턴을 사용하여 AOP를 구현합니다.

</details>

<details>
<summary>@Transactional 동작 원리</summary>

<br>

AOP 프록시를 통해 동작합니다.
1. 메서드 호출 시 프록시가 가로챔.
2. 트랜잭션 시작 (AutoCommit off).
3. 실제 메서드 실행.
4. 성공 시 Commit, 예외 발생(Runtime Exception) 시 Rollback.

</details>

<details>
<summary>JPA N+1 문제와 해결 방법</summary>

<br>

**N+1 문제**: 1번의 쿼리로 조회한 엔티티와 연관된 엔티티를 조회하기 위해 N번의 추가 쿼리가 발생하는 성능 이슈.

**해결**:
1. **Fetch Join**: `select m from Member m join fetch m.team` (한 방 쿼리).
2. **@EntityGraph**: 어노테이션으로 fetch join 효과.
3. **Batch Size**: `IN` 절을 사용하여 일정 개수만큼 묶어서 조회.

</details>

<details>
<summary>Filter vs Interceptor</summary>

<br>

- **Filter**: Servlet Container 레벨 (DispatcherServlet 앞). 보안, 인코딩 처리.
- **Interceptor**: Spring MVC 레벨 (DispatcherServlet 뒤, Controller 앞). 인증, 로깅, 공통 로직 처리.

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

- **대칭키**: 암/복호화 키가 같음. 빠르지만 키 전달 문제. (AES)
- **비대칭키**: 공개키/개인키. 느리지만 키 전달 안전. (RSA)

</details>

<details>
<summary>단방향 암호화 (Hashing)</summary>

<br>

복호화가 불가능한 암호화. 비밀번호 저장에 사용.
- **Salt**: 같은 비밀번호라도 다른 해시값이 나오도록 랜덤 문자열 추가.
- **Rainbow Table** 공격 방어.

</details>

<details>
<summary>JWT (JSON Web Token)</summary>

<br>

Stateless한 인증 방식. 서버 저장소 없이 토큰 자체에 정보를 담아 검증.
- 구성: Header . Payload(Claim) . Signature
- Access Token의 만료 기간을 짧게 잡고 Refresh Token을 병행 사용하여 보안성을 높임.

</details>

<details>
<summary>OAuth 2.0 흐름</summary>

<br>

제3자 애플리케이션이 사용자 비밀번호 없이 서비스 접근 권한을 얻는 표준.
1. Client가 Resource Owner(사용자)에게 권한 요청
2. 사용자가 승인하면 Auth Server가 **Authorization Code** 발급
3. Client가 Code로 **Access Token** 교환
4. Token으로 Resource Server 이용

</details>

<details>
<summary>SQL Injection & XSS</summary>

<br>

- **SQL Injection**: 입력값에 SQL 구문을 삽입해 DB 조작. -> **Prepared Statement**로 방어.
- **XSS** (Cross Site Scripting): 악성 스크립트 실행 공격. -> **입력값 검증 및 Escaping**으로 방어.

</details>

<details>
<summary>CI/CD, Docker, Load Balancer</summary>

<br>

- **CI/CD**: 지속적 통합 및 배포 자동화 파이프라인.
- **Docker**: 환경에 구애받지 않고 애플리케이션을 실행하는 컨테이너 기술.
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
<summary>분산 락 (Distributed Lock) 구현 방법</summary>

<br>

여러 서버에서 공유 자원에 동시 접근할 때 정합성을 맞추기 위해 사용합니다.

1. **Redis (Spin Lock / Pub-Sub)**:
   - `SETNX` (Set if Not Exists) 명령어로 락 획득 시도.
   - **Redisson** 라이브러리: Pub-Sub 방식으로 재시도 부하를 줄이고, 타임아웃/자동 만료 기능 제공.
2. **DB (Named Lock / Pessimistic Lock)**:
   - `GET_LOCK()` 함수 등을 사용하여 DB 차원의 락 사용.
3. **ZooKeeper**: 순차 노드를 생성하여 가장 낮은 번호의 노드가 락 획득.

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

**Base64**: 바이너리 데이터를 **ASCII 문자열로 변환**하는 인코딩 방식입니다.

**원리**:
- 64개의 안전한 문자(A-Z, a-z, 0-9, +, /)만 사용
- 데이터 크기가 약 33% 증가

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

| 포맷 | 압축 방식 | 투명도 | 용도 |
| ---- | --------- | ------ | ---- |
| **JPEG** | 손실 압축 | X | 사진, 복잡한 이미지 |
| **PNG** | 무손실 압축 | O | 로고, 아이콘, 스크린샷 |
| **WebP** | 손실/무손실 | O | 웹 최적화 (JPEG/PNG 대체) |

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

| 구분 | JSON | XML |
| ---- | ---- | --- |
| 문법 | 간결함 (`{}`, `[]`) | 태그 기반 (`<tag>`) |
| 용량 | 작음 | 큼 |
| 파싱 속도 | 빠름 | 느림 |
| 주 사용처 | REST API | SOAP, 레거시 시스템 |

**현재 트렌드**: REST API에서는 **JSON**이 표준.

</details>

## 8. CS & Etc (자료구조, 디자인패턴, Git, 일반)

<details>
<summary>Array와 LinkedList의 차이점</summary>

<br>

| 구분          | Array (배열)            | LinkedList (연결 리스트)                 |
| ------------- | ----------------------- | ---------------------------------------- |
| 메모리        | 연속된 공간 할당        | 불연속적 공간 (노드와 포인터)            |
| 접근 (Access) | 인덱스로 즉시 접근 O(1) | 순차 접근 O(N)                           |
| 삽입/삭제     | 데이터 이동 필요 O(N)   | 포인터 변경만 필요 O(1) (탐색 시간 제외) |
| 크기          | 고정                    | 가변                                     |

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
