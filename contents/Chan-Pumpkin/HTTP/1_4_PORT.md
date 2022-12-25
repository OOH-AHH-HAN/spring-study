# PORT

## 한번에 둘 이상 연결해야 하면?

클라이언트 PC 한번에 여러개 연결 해야함

게임 패킷인지, 화상통화 패킷인지, 웹 브라우저 요청 패킷인지 알 수가 없음.

TCP/IP 패킷 정보에 보면 출발지 PORT와 목적지 PORT가 있음

패킷 정보에 IP, PORT가 있음.

같은 IP 내에서 프로세스 구분하는 것이 PORT

A PC에 게임 포트 8090 화상통화 포트 21100 웹브라우저 10009

ip 100.100.100.1

B PC 게임 포트 8090 화상통화 포트 21100 웹브라우저 10009

ip 200.200.200.2

ip : 포트

0~65535 : 할당 가능

0~1023 : 잘 알려진 포트, 사용하지 않는 것이 좋음

FTP : 20, 21

TELNET : 23

HTTP : 80

HTTPS : 443