## VPN
* VPN이란?
   - 전용 회선이 아니라 공중망을 이용하여 사설 네트워크를 가상으로 연결하는 기술
   - 공중망의 저렴한 비용과 사설 IP 통신을 자유롭게 하는 특징

* IPSec VPN
   1) 개념 및 특징
   - Site-to-Site VPN 뿐 아니라 개별 사용자를 연결하는 원격 접속 VPN까지 가능한 기술
   - 터널을 경유하는 트래픽을 암호화해 전송 중에 유출되더라도 정보를 보호
   - IKE(Internet Key Exchange)라는 키 정보 교환 절차가 필요 
   2) 동작 과정
   - IPSec 통신은 물리적으로 사용자 정보를 특정 키 정보로 암호화해 암호문 형태로 전달하고 수신자는 키를 사용해 복호화해 정보를 수신하는 과정
   - 첫 번째 터널은 암호화에 필요한 키 정보를 교환하기 위한 보안 터널, 두 번째 터널은 실제 암호화된 사용자 트래픽이 경유하는 보안 터널(IPSec 동작에 두 개의 터널 필요)
   - IKE Phase 1
     - 트래픽을 암호화하는 데 사용되는 인증 키, 열쇠 전달을 위한 보안 터널을 생성하는 단계
     - IPSec Peer 인증 및 키 정보 교환을 위한 보안 채널 생성
     *) IPSec Peer 간에 생성되는 보안 터널을 SA(Security Association)
     - Main Mode : IPSec Peer 대화를 위한 보안 채널 형성을 위해 단계별 협상 진행(6개의 메시지 교환)
        - Peer 신분 정보 암호화되어 Peer의 신분이 외부에 노출되지 않음
     - Aggressive Mode : 보안 세션 연결을 시도하는 Peer가 모든 정보에 대해 일괄적인 협상을 시도(3개의 메시지 교환)
        - 첫 번째 메시지에 신분 정보를 포함하여 외부에 노출될 가능성이 있지만 속도와 자원 사용량 우수
      *) ISAKMP(Internet Security Association and Key Management Protocol) : 보안 채널을 생성함으로써 인터넷 환경에서 키 정보를 암호문 형태로 교환할 수 있게 개발된 키 보안 프로토콜(1단계의 결과물)
   - IKE Phase 2 
      - 사용자 트래픽의 암호화를 위한 터널 생성(IPSec SA)
      -  ISAKMP SA를 기반으로 인증 정책을 협상하므로 ISAKMP 터널 필수
      - IPSec SA는 단방향 통신이라 송신 터널과 수신 터널이 요구(보안성 향상)
      - 단방향 터널로 사용자 트래픽이 암호화 될 때 암호화 패킷에 AH, ESP 보안 프로토콜 정보가 삽입
      - AH(Authentication Header) : 인증을 위한 헤더를 제공하는 프로토콜, 암호화를 제공하지 않고 인증을 통해 패킷 조작 여부 작업을 수행
      - ESP(Encapsulating Security Payload) : 보안을 위해 페이로드를 캡슐화 하는 프로토콜, 상위 계층 정보인 페이로드의 캡슐화(암호화) 제공
   3) 모드
    - 터널 모드(Tunnel Mode) : 
    - 전송 모드(Transport Mode) : 
