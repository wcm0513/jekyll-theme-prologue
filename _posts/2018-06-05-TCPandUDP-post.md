---
title: TCP와 UDP의 차이점
author: wcm
layout: post
---

●TCP와 UDP의 차이점

※TCP(Transmission Control Protocol)
연결형 서비스를 지원하는 전송계층 프로토콜. 인터넷 환경에서 기본으로 사용한다. 호스트간 신뢰성 있는 데이터 전달과 흐름제어 및 혼잡제어 등을 제공하는 전송계층이다.

(1)특징
-가상 회선 연결 방식, 연결형 서비스를 제공(연결 후 통신).
-높은 신뢰성(Sequence Number, AckNumber를 통한 신뢰성 보장)
-연결의 설정(3-way handshaking)과 해제(4-way handshaking)
-데이터 흐름 제어(수신자 버퍼 오버플로우 방지)및 혼잡 제어(네트워크 내 패킷 수가 과도하게 증가하는 현상 방지)
-전이중(Full-Duplex), 점대점(Point to Point) 서비스

(2)소켓 통신 과정
-서버 : 소켓을 생성, 주소 할당, 연결 요청 기다림, 요청에 대한 응답
-클라이언트 : 소켓을 생성, 주소 할당, 연결 요청

(3)관련 클래스
Socket, ServerSocket

※UDP(User Datagram Protocol)
비연결형 서비스를 지원하는 전송 계층 프로토콜. 사용자 데이터그램형 프로토콜. 인터넷상에서 서로 정보를 주고받을 때 정보를 보낸다는 신호나 받는다는 신호 절차를 거치지 않고, 보내는 쪽에서 일방적으로 데이터를 전달하는 통신 프로토콜. 보내는 쪽에서 받는 쪽이 데이터를 받았는지 받지 않았는지 확인할 수 업속, 또 확인할 필요도 없도록 만들어진 프로토콜.

(1)특징
-비연결형(port만 확인하여 소켓을 식별하고 송수신)
-패킷 오버헤드가 적어 네트워크 부하 감소
-비신뢰성
-오류 검출(헤더에 오류 검출 필드를 포함하여 무결성 검사)
-TCP의 handshaking같은 연결 설정이 없다
-DNS, NFS, SNMP, RIP 등 사용

(2)소켓 통신 과정
-서버 : 소켓을 생성, 주소 할당, 데이터를 송수신
-클라이언트 : 소켓 생성 후 데이터 수신

(3)관련 클래스
DatagramSocket, DatagramPacket, MulticastSocket

※비교
UDP는 TCP와 달리 데이터의 수신에 대한 책임을 지지 않는다. 이는 송신자는 정보를 보냈지만, 정보가 수신자에게 제때에 도착했는지 또는 정보 내용이 서로 뒤바뀌었는지에 관해서 송신자는 상관할 필요가 없다.
TCP보다 안정성 면에서는 떨어지지만, 속도는 훨씬 빠르다.
TCP는 1:1 통신 방식이지만, UDP는 1:1, 1:N, N:N 통신 방식이다.
TPC는 패킷을 관리할 필요가 없지만 UDP는 패킷을 관리해야 한다.

※예제

(1)TCP 통신 방식

-서버

#include "server.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <signal.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/time.h>
#include <memory.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <stdarg.h>

#include <unistd.h>
#include <assert.h>

struct packet {
        char data[BUFSIZE];
        int dataSize;
        int dataSeq;
        bool isFinish;
};

void serverOpen(const char* port) {
        int serv_sock, clnt_sock;
        struct sockaddr_in servAddr, clntAddr;
        socklen_t clntAddrSize;

        assert(-1 != (serv_sock = socket(PF_INET, SOCK_STREAM, 0)));
        printf("Socket has been created.\n");

        usleep(500000);
        memset(&servAddr, 0, sizeof(servAddr));
        servAddr.sin_family = AF_INET;
        servAddr.sin_addr.s_addr = htonl(INADDR_ANY);
        servAddr.sin_port = htons(atoi(port));

        assert(-1 != bind(serv_sock, (const struct sockaddr*)&servAddr, sizeof(servAddr)));
        printf("The server address information (ANYIP.%s) is bound.\n\n\n", port);

        usleep(500000);

        assert(-1 != listen(serv_sock, 5));
        printf("The listen queue has been created.\n");

        usleep(500000);

        printf("Ready to receive TCP connection request\n");

        clntAddrSize = sizeof(clntAddr);
        assert(-1 != (clnt_sock = accept(serv_sock, (struct sockaddr*)&clntAddr, &clntAddrSize)));

        printf("a client (%s:%i) has been connected...\n\n\n", inet_ntoa(clntAddr.sin_addr), ntohs(clntAddr.sin_port));

        rcvPacket(clnt_sock);

        close(clnt_sock);
        close(serv_sock);
}

void rcvPacket(int sock) {
        Packet rcvPkt;
        rcvPkt.dataSeq = 0;
        rcvPkt.isFinish = false;

        int rcvedSize = 0;

        const char eof = 0x1A;

        while(1) {
                assert(-1 != (rcvPkt.dataSize = read(sock, rcvPkt.data, BUFSIZE)));
                if(*(rcvPkt.data) == eof) {
                        break;
                }

                usleep(250000);

                printf("%i Bytes data (seq: %i) received.\n", rcvPkt.dataSize, rcvPkt.dataSeq);
                rcvedSize += rcvPkt.dataSize;

                toApp(rcvPkt);
                rcvPkt.dataSeq++;
        }

        usleep(500000);
        printf("The file (%i Bytes) has been received.\n", rcvedSize);
}

void toApp(Packet pkt) {
        FILE* fp = NULL;
        static bool isFirst = true;
        long int offset;

        if(isFirst == true) {
                assert(fp = fopen(FILENAME, "w+"));
                isFirst = false;
        } else
        assert(fp = fopen(FILENAME, "a"));

        offset = BUFSIZE * pkt.dataSeq;

        fseek(fp, offset, SEEK_SET);

        fwrite(pkt.data, pkt.dataSize, 1, fp);

        fclose(fp);
}

-클라이언트

#include "client.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <signal.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/time.h>
#include <memory.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <stdarg.h>

#include <unistd.h>
#include <assert.h>

struct packet {
  char data[BUFSIZE];
  int dataSize;
  int dataSeq;
  bool isFinish;
};

void clientOpen(const char* ip, const char* port) {
  int clnt_sock;
  struct sockaddr_in servAddr;

  FILE* fp = NULL;
  size_t fSize;

  Packet sndPkt;
  int sentSize = 0;

  const char eof = 0x1A;

  assert(-1 != (clnt_sock = socket(PF_INET, SOCK_DGRAM, 0)));
  printf("Socket has been created.\n\n\n");

  usleep(500000);
  memset(&servAddr, 0, sizeof(servAddr));
  servAddr.sin_family = AF_INET;
  servAddr.sin_addr.s_addr = inet_addr(ip);
  servAddr.sin_port = htons(atoi(port));

  usleep(500000);

  assert(fp = fopen(FILENAME, "r"));

  fseek(fp, 0, SEEK_END);
  fSize = ftell(fp);
  fseek(fp, 0, SEEK_SET);

  sndPkt.dataSeq = 0;
  sndPkt.isFinish = false;

  while(fSize > 0) {
    sndPkt.dataSize = fSize > BUFSIZE ? BUFSIZE : fSize;

    sndPkt.isFinish = fSize > BUFSIZE ? false : true;

    fread(sndPkt.data, sndPkt.dataSize, 1, fp);

    sendto(clnt_sock, sndPkt.data, sndPkt.dataSize, 0, (const struct sockaddr*)&servAddr, sizeof(servAddr));
    printf("%i Bytes data (seq: %i) sent.\n", sndPkt.dataSize, sndPkt.dataSeq);

    fSize -= sndPkt.dataSize;
    sentSize += sndPkt.dataSize;
    sndPkt.dataSeq++;
    usleep(500000);
  }

  sendto(clnt_sock, &eof, sizeof(eof), 0, (const struct sockaddr*)&servAddr, sizeof(servAddr));
  printf("The file (%i Bytes) has been sent.\n", sentSize);
  fclose(fp);

  close(clnt_sock);
}


(2)UDP 통신 방식

-서버

#include "server.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <signal.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/time.h>
#include <memory.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <stdarg.h>

#include <unistd.h>
#include <assert.h>

struct packet {
        char data[BUFSIZE];
        int dataSize;
        int dataSeq;
        bool isFinish;
};

void serverOpen(const char* port) {
        int serv_sock;
        struct sockaddr_in servAddr;

        assert(-1 != (serv_sock = socket(PF_INET, SOCK_DGRAM, 0)));
        printf("Socket has been created.\n");

        usleep(500000);
        memset(&servAddr, 0, sizeof(servAddr));
        servAddr.sin_family = AF_INET;
        servAddr.sin_addr.s_addr = htonl(INADDR_ANY);
        servAddr.sin_port = htons(atoi(port));

        assert(-1 != bind(serv_sock, (const struct sockaddr*)&servAddr, sizeof(servAddr)));
        printf("The server address information (ANYIP.%s) is bound.\n\n\n", port);

        usleep(500000);

        rcvPacket(serv_sock);

        close(serv_sock);
}

void rcvPacket(int sock) {
        struct sockaddr_in fromAddr;
        socklen_t fromAddrSize;

        Packet rcvPkt;
        rcvPkt.dataSeq = 0;
        rcvPkt.isFinish = false;

        int rcvedSize = 0;

        const char eof = 0x1A;

        while(1) {
                fromAddrSize = sizeof(fromAddr);
                assert(-1 != (rcvPkt.dataSize = recvfrom(sock, rcvPkt.data, BUFSIZE, 0, (struct sockaddr*)&fromAddr, &fromAddrSize)));

                if(*(rcvPkt.data) == eof) {
                        break;
                }

                usleep(250000);

                printf("%i Bytes data (seq: %i) received.\n", rcvPkt.dataSize, rcvPkt.dataSeq);
                rcvedSize += rcvPkt.dataSize;

                toApp(rcvPkt);
                rcvPkt.dataSeq++;
        }

        usleep(500000);
        printf("The file (%i Bytes) has been received.\n", rcvedSize);
}

void toApp(Packet pkt) {
        FILE* fp = NULL;
        static bool isFirst = true;
        long int offset;

        if(isFirst == true) {
                assert(fp = fopen(FILENAME, "w+"));
                isFirst = false;
        } else
        assert(fp = fopen(FILENAME, "a"));

        offset = BUFSIZE * pkt.dataSeq;

        fseek(fp, offset, SEEK_SET);

        fwrite(pkt.data, pkt.dataSize, 1, fp);

        fclose(fp);
}

-클라이언트

#include "client.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <signal.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/time.h>
#include <memory.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <stdarg.h>

#include <unistd.h>
#include <assert.h>

struct packet {
  char data[BUFSIZE];
  int dataSize;
  int dataSeq;
  bool isFinish;
};

void clientOpen(const char* ip, const char* port) {
  int clnt_sock;
  struct sockaddr_in servAddr;

  FILE* fp = NULL;
  size_t fSize;

  Packet sndPkt;
  int sentSize = 0;

  const char eof = 0x1A;

  assert(-1 != (clnt_sock = socket(PF_INET, SOCK_DGRAM, 0)));
  printf("Socket has been created.\n\n\n");

  usleep(500000);
  memset(&servAddr, 0, sizeof(servAddr));
  servAddr.sin_family = AF_INET;
  servAddr.sin_addr.s_addr = inet_addr(ip);
  servAddr.sin_port = htons(atoi(port));

  usleep(500000);

  assert(fp = fopen(FILENAME, "r"));

  fseek(fp, 0, SEEK_END);
  fSize = ftell(fp);
  fseek(fp, 0, SEEK_SET);

  sndPkt.dataSeq = 0;
  sndPkt.isFinish = false;

  while(fSize > 0) {
    sndPkt.dataSize = fSize > BUFSIZE ? BUFSIZE : fSize;

    sndPkt.isFinish = fSize > BUFSIZE ? false : true;

    fread(sndPkt.data, sndPkt.dataSize, 1, fp);

    sendto(clnt_sock, sndPkt.data, sndPkt.dataSize, 0, (const struct sockaddr*)&servAddr, sizeof(servAddr));
    printf("%i Bytes data (seq: %i) sent.\n", sndPkt.dataSize, sndPkt.dataSeq);

    fSize -= sndPkt.dataSize;
    sentSize += sndPkt.dataSize;
    sndPkt.dataSeq++;
    usleep(500000);
  }

  sendto(clnt_sock, &eof, sizeof(eof), 0, (const struct sockaddr*)&servAddr, sizeof(servAddr));
  printf("The file (%i Bytes) has been sent.\n", sentSize);
  fclose(fp);

  close(clnt_sock);
}


