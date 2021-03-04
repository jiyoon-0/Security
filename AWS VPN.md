! Amazon Web Services  
! Virtual Private Cloud  

! AWS utilizes unique identifiers to manipulate the configuration of   
! a VPN Connection. Each VPN Connection is assigned an identifier and is   
! associated with two other identifiers, namely the     
! Customer Gateway Identifier and Virtual Private Gateway Identifier.  
!
! Your VPN Connection ID 		         : vpn-0777e44f5c3f8a391  
! Your Virtual Private Gateway ID    : vgw-0b1275be0f46582fd  
! Your Customer Gateway ID		       : cgw-0e69d43fc950c188d  
!  
!  
! This configuration consists of two tunnels. Both tunnels must be   
! configured on your Customer Gateway.  
!  
! --------------------------------------------------------------------------------   
! IPSec Tunnel #1  
! --------------------------------------------------------------------------------  
! #1: Internet Key Exchange (IKE) Configuration  
!  
! A policy is established for the supported ISAKMP encryption,   
! authentication, Diffie-Hellman, lifetime, and key parameters.
> 인증, 디피-헬만 암호화 등 설정을 기반으로 한 ISAKMP 보안 터널 생성 
! Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
> 암호화 AES 128, 해시 SHA 1, 디피-헬프만 2가 최소 스펙, 아래 보면 AES256-SHA256으로 변경 가능
! Category "VPN" connections in the GovCloud region have a minimum requirement of AES128, SHA2, and DH Group  
 14. 
! You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! NOTE: If you customized tunnel options when creating or modifying your VPN connection, you may need to modify these sample configurations to match the custom settings for your tunnels.
> 설정하는 상대방 VPN도 동일한 스펙으로 설정 필요
!
! Higher parameters are only available for VPNs of category "VPN," and not for "VPN-Classic".
!
! The address of the external interface for your customer gateway must be a static address. 
! Your customer gateway may reside behind a device performing network address translation (NAT). 
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T. 
!
! Configuration begins in root VDOM.


Go to VPN-->IPSec--> AutoKey, create Phase 1
> Phase 1은 ISAKMP 터널을 생성하기 위한 과정
 a. Name: vpn-0777e44f5c3f8a391-0 ! Name must be shorter than 15 chars, best if shorter than 12
 b. Remote Gateway: Static IP Address
 c. IP address: 13.209.59.252
 d. Local Interface: wan1
 e. Mode: Main
 > Peer 신분 정보 암호화되어 Peer의 신분이 외부에 노출되지 않음   
 f. Authentication Method: Pre-shared Key
 > 사전에 인증 키를 상호간에 물리적으로 공유(관리자가 인증 패스워드를 미리 정해놓고 사용, 인증키 직접 설정 필요)   
 g. Pre-Shared Key: QzlTUSOKfg3hXAlrfaqYC_uAjxvygmXl

 h. Select "Enable IPsec Interface Mode"
 i. Ike Version: 1
 > 암호화, 해시, 디피-헬만 그룹 정의
 j. Local-gateway: Select Specify and enter 165.246.204.254
 k. Encryption: aes128 
 l. Authentication: sha1
 m. DH group: 2     ! and deselect 5
 n. Keylife: 28800 seconds
 > ISAKMP SA가 활성화 하는 시간(외부에 키가 노출되는 것을 막기 위함)
    
! NAT Traversal is enabled by default but if your FortiGate device is not behind a NAT/PAT device, please deselect NAT Traversal.

 o. Select  Dead Peer Detection Enable
 p. Click ok


! #2: IPSec Configuration
! 
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
!
! Category "VPN" connections in the GovCloud region have a minimum requirement of AES128, SHA2, and DH Group 14.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
! Higher parameters are only available for VPNs of category "VPN," and not for "VPN-Classic".

Go to VPN-->IPSec--> AutoKey, create Phase 2
> 실제 트래픽을 암호화할 터널을 생성하는 Phase 2
 a. Name   :  vpn-0777e44f5c3f8a391-0 ! Name must be shorter than 15 chars, best if shorter than 12
 b. Phase 1:  vpn-0777e44f5c3f8a391-0

Select Advanced

 c. Encryption: aes128 
 d. Authentication: sha1
 #e. Select Enable Replay Detection
 #f. Select Perfect Forward Secrecy
 g. DH Group: 2 ! and deselect 5
 #h. Keylife: 3600 seconds 
 #i. Enable Autokey Keep Alive
 j. Click Ok      


! --------------------------------------------------------------------------------
! #3: Tunnel Interface Configuration
!  
! A tunnel interface is configured to be the logical interface associated  
! with the tunnel. All traffic routed to the tunnel interface will be 
! encrypted and transmitted to the VPC. Similarly, traffic from the VPC
! will be logically received on this interface.
!
!
! The address of the interface is configured with the setup for your 
! Customer Gateway.  If the address changes, the Customer Gateway and VPN 
! Connection must be recreated with Amazon VPC.
!
! This is required in order for tunnel failover via gwdetect to function
!
! Perform this from the Global VDOM.

Go to Network Tab -—> Interface --> wan1 and edit vpn-0777e44f5c3f8a391-0
> 암호화 맵을 적용할 인터페이스 설정
 a. IP : 169.254.191.238
 b. Remote IP: 169.254.191.237
 c. Select Ping
 d. Administrative Status: Up
 e. Select Ok.

!You can set MTU and MSS on the tunnel by performing this from the CLI:
 config global
 config system interface
  edit "vpn-0777e44f5c3f8a391-0"
    set mtu 1427 
    set tcp-mss 1379
   next 


! ----------------------------------------------------------------------------
! #4 Static Route Configuration
!
! Your Customer Gateway needs to set a static route for the prefix corresponding to your 
! VPC to send traffic over the tunnel interface.
! An example for a VPC with the prefix 10.0.0.0/16 is provided below:
! 
! This is configured from the root VDOM
!
Go to Router Tab --> Static --> Static Route, and click on Create New

 a. Destination IP/Mask: 10.0.0.0/16
 b. Device: vpn-0777e44f5c3f8a391-0
 c. Select Ok

! Static routing does not allow for failover of traffic between tunnels. If there is a problem with one of the 
! tunnels, we would want to failover the traffic to the second tunnel. This is done by using "gwdetect" in fortigate. 
! The gwdetect command will ping the other end of the tunnel, and check if the tunnel is up. If the pings fail, it will
! remove the static route from the routing table, and the second route in the table will become active. 
! 
! This can be done only using the CLI.
!
! The following config will tell the Fortigate device, what IP it should ping to test the tunnel. This IP should be 
! the inside IP address of the virtual private gateway.
! This is required in order for tunnel failover via gwtect to function. Additionally, this is required to keep the tunnel up, since
! traffic must be sent from your side of the VPN tunnel to prevent the tunnel from being taken down.

config vdom
    edit root
        config router gwdetect
        edit 1
        set interface "vpn-0777e44f5c3f8a391-0"
        set server "169.254.191.237"

! Using the following command, you can set the interval and failtime for gwdetect. Interval is number of seconds 
! between pings. Failtime is the number of lost consecutive pings.Using the respective values of 2 and 5, your tunnel 
! will failover in 10 seconds.

        set interval 2
        set failtime 5
    next
end

! --------------------------------------------------------------------------------
! #5: Firewall Policy Configuration
!
! Create a firewall policy permitting traffic from your local subnet to the VPC subnet and vice versa
! This example policy permits all traffic from the local subnet to the VPC.
! 
!This is configured from the root VDOM

Go to Firewall --> Policy --> Policy
1) Create New
   a. Incoming Interface/Zone = internal ! This is the interface out which your local LAN resides
   b. Source Address = all
   c. Outgoing Interface/Zone = "vpn-0777e44f5c3f8a391-0"
   d. Destination Address = all
   e. Schedule = always
   f. Service = ALL
   g. Action = ACCEPT
   h. Click OK

! Now create a policy to permit traffic going the other way

2) Create New
   a. Incoming Interface/Zone = "vpn-0777e44f5c3f8a391-0"
   b. Source Address = all
   c. Outgoing Interface/Zone = internal  ! This is the interface out which your local LAN resides
   d. Destination Address = all
   e. Schedule = always
   f. Service = ALL
   g. Action = ACCEPT
   h. Click OK

! --------------------------------------------------------------------------------
! IPSec Tunnel #2
! --------------------------------------------------------------------------------
! #1: Internet Key Exchange (IKE) Configuration   
Go to VPN-->IPSec--> AutoKey, create Phase 1

 a. Name: vpn-0777e44f5c3f8a391-1 ! Name must be shorter than 15 chars, best if shorter than 12
 b. Remote Gateway: Static IP Address
 c. IP address: 15.164.57.115
 d. Local Interface: wan1
 e. Mode: Main
 f. Authentication Method: Pre-shared Key
 g. Pre-Shared Key: EkHJhJKBshYAcZjZRjhjwrOLMvDZ5FtF

 h. Select "Enable IPsec Interface Mode"
 i. Ike Version: 1
 j. Local-gateway: Select Specify and enter 165.246.204.254
 k. Encryption: aes128 
 l. Authentication: sha1
 m. DH group: 2     ! and deselect 5
 n. Keylife: 28800 seconds
    
! NAT Traversal is enabled by default but if your FortiGate device is not behind a NAT/PAT device, please deselect NAT Traversal.

 o. Select  Dead Peer Detection Enable
 p. Click ok


! #2: IPSec Configuration
! 
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
!
! Category "VPN" connections in the GovCloud region have a minimum requirement of AES128, SHA2, and DH Group 14.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
! Higher parameters are only available for VPNs of category "VPN," and not for "VPN-Classic".

Go to VPN-->IPSec--> AutoKey, create Phase 2

 a. Name   :  vpn-0777e44f5c3f8a391-1 ! Name must be shorter than 15 chars, best if shorter than 12
 b. Phase 1:  vpn-0777e44f5c3f8a391-1

Select Advanced

 c. Encryption: aes128 
 d. Authentication: sha1
 e. Select Enable Replay Detection
 f. Select Perfect Forward Secrecy
 g. DH Group: 2 ! and deselect 5
 h. Keylife: 3600 seconds 
 i. Enable Autokey Keep Alive
 j. Click Ok      


! --------------------------------------------------------------------------------
! #3: Tunnel Interface Configuration
!  
! A tunnel interface is configured to be the logical interface associated  
! with the tunnel. All traffic routed to the tunnel interface will be 
! encrypted and transmitted to the VPC. Similarly, traffic from the VPC
! will be logically received on this interface.
!
!
! The address of the interface is configured with the setup for your 
! Customer Gateway.  If the address changes, the Customer Gateway and VPN 
! Connection must be recreated with Amazon VPC.
!
! This is required in order for tunnel failover via gwdetect to function
!
! Perform this from the Global VDOM.

Go to Network Tab -—> Interface --> wan1 and edit vpn-0777e44f5c3f8a391-1

 a. IP : 169.254.164.242
 b. Remote IP: 169.254.164.241
 c. Select Ping
 d. Administrative Status: Up
 e. Select Ok.

!You can set MTU and MSS on the tunnel by performing this from the CLI:
 config global
 config system interface
  edit "vpn-0777e44f5c3f8a391-1"
    set mtu 1427 
    set tcp-mss 1379
   next 


! ----------------------------------------------------------------------------
! #4 Static Route Configuration
!
! Your Customer Gateway needs to set a static route for the prefix corresponding to your 
! VPC to send traffic over the tunnel interface.
! An example for a VPC with the prefix 10.0.0.0/16 is provided below:
! 
! This is configured from the root VDOM
!
Go to Router Tab --> Static --> Static Route, and click on Create New

 a. Destination IP/Mask: 10.0.0.0/16
 b. Device: vpn-0777e44f5c3f8a391-1
 c. Select Ok

! Static routing does not allow for failover of traffic between tunnels. If there is a problem with one of the 
! tunnels, we would want to failover the traffic to the second tunnel. This is done by using "gwdetect" in fortigate. 
! The gwdetect command will ping the other end of the tunnel, and check if the tunnel is up. If the pings fail, it will
! remove the static route from the routing table, and the second route in the table will become active. 
! 
! This can be done only using the CLI.
!
! The following config will tell the Fortigate device, what IP it should ping to test the tunnel. This IP should be 
! the inside IP address of the virtual private gateway.
! This is required in order for tunnel failover via gwtect to function. Additionally, this is required to keep the tunnel up, since
! traffic must be sent from your side of the VPN tunnel to prevent the tunnel from being taken down.

config vdom
    edit root
        config router gwdetect
        edit 1
        set interface "vpn-0777e44f5c3f8a391-1"
        set server "169.254.164.241"

! Using the following command, you can set the interval and failtime for gwdetect. Interval is number of seconds 
! between pings. Failtime is the number of lost consecutive pings.Using the respective values of 2 and 5, your tunnel 
! will failover in 10 seconds.

        set interval 2
        set failtime 5
    next
end

! --------------------------------------------------------------------------------
! #5: Firewall Policy Configuration
!
! Create a firewall policy permitting traffic from your local subnet to the VPC subnet and vice versa
! This example policy permits all traffic from the local subnet to the VPC.
! 
!This is configured from the root VDOM

Go to Firewall --> Policy --> Policy
1) Create New
   a. Incoming Interface/Zone = internal ! This is the interface out which your local LAN resides
   b. Source Address = all
   c. Outgoing Interface/Zone = "vpn-0777e44f5c3f8a391-1"
   d. Destination Address = all
   e. Schedule = always
   f. Service = ALL
   g. Action = ACCEPT
   h. Click OK

! Now create a policy to permit traffic going the other way

2) Create New
   a. Incoming Interface/Zone = "vpn-0777e44f5c3f8a391-1"
   b. Source Address = all
   c. Outgoing Interface/Zone = internal  ! This is the interface out which your local LAN resides
   d. Destination Address = all
   e. Schedule = always
   f. Service = ALL
   g. Action = ACCEPT
   h. Click OK



! Additional Notes and Questions
!  - Amazon Virtual Private Cloud Getting Started Guide: 
!       http://docs.amazonwebservices.com/AmazonVPC/latest/GettingStartedGuide
!  - Amazon Virtual Private Cloud Network Administrator Guide: 
!       http://docs.amazonwebservices.com/AmazonVPC/latest/NetworkAdminGuide
!  - XSL Version: 2009-07-15-1119716

