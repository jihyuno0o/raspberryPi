# raspberryPi

### 2021-05-18

hello.c
```
#include <stdio.h>
#include <malloc.h>
#include <string.h>
#include "myHeader.h"

int main()
{
	char buf[256];
	int a,b,c;
	
	char *s1 = "abcdefghijklmnop";
	char *tt = "1,2,3,4,5,6";
	
	while(1)
	{
		printf("Input search num : ");
		scanf("%d", &a);
		strcpy(buf,GetToken(a,tt,','));
		printf("%d 번째 아이템은 %s 입니다\n", a, buf);
	}

	return 0;
}
```

myHeader.h
```
int chrFind(char *str, char chr);
int strLen(char *str);
char **Split(char *str, char chr);
int chrCount(char *str, char chr);
char *GetToken(int index, char *str, char chr);
```
myLib.c
```#include <stdio.h>
#include <malloc.h>
#include <string.h>
#include "myHeader.h"

char *GetToken(int index, char *str, char chr)// 2, "123,456,789" , deli ','  ===>"789"
{
	char** ss = Split(str,chr);
	return *(ss + index); 
	/*char buf[1024]; //stack 재사용영역 
	int i,j,k,n=0;
	for(i=0;i<index-1;i++)
	{
		k = chrFind(str+n,chr);
		if(k == -1) return NULL;
		n += k + 1;
	}
	j = chrFind(str+n,chr);
	if(j == -1) j = strlen(str);
	else j += n;
	strncpy(buf,str+n,j-n); buf[j-n]=0; //strNcpy : NULL 미처리 
	//printf("Input String :%s\n[%d] Item : %s\n",str,index,buf);
	return buf;*/
}

char **Split(char *str, char chr)	
{
	char *s1 = malloc(strLen(str));
	char **s2 = malloc((chrCount(str,chr)+1)*4);
	strcpy(s1, str);
	
	int i = 1;
	*(s2+0) = s1;
	while(*s1)
	{
		if(*s1 == chr) 
		{
			*s1 = 0;
			*(s2+i++) = s1 + 1; 
		}
		s1++;
	}
	return s2;
}

int chrCount(char *str, char chr)
{
	int i = 0;
	while(*str)
	{
		if(*str++ == chr) i++;
	}
	return i;
}

int strLen(char *str)
{
	int i = 0;
	while(*str++) i++; return i;
}

int chrFind(char *str, char chr)
{
	int i = 0;
	while(*str)
	{
		if(*str++ ==  chr) return i;
		i++;
	}
	return -1;
}

int chrFindEx(char *str, char chr,int start)
{
	int i = 0;
	while(*str)
	{
		if(*str++ ==  chr) return i;
		i++;
	}
	return -1;
}
```




그리고 LEDcontrol
엔터키로 불 껐다 켰다 하기

```
#include <stdio.h>
#include <wiringPi.h>

int main()
{
	wiringPiSetup();
	pinMode(8,OUTPUT);

	int i=0;
	while(1)
	{
		getchar();
		if(i%2 == 0) digitalWrite(8, HIGH);
		else digitalWrite(8, LOW);
		i++;
	}
	return 0;
}
```


### 2021-05-20

pwd 현재위치 ls 파일 리스트 ls -al 자세히

mkdir ** 디렉토리 만들기 **디렉토리이름

rmdir 디렉토리 삭제

cd ** 디렉토리 들어가기

find . *.h | grep -r strcmp 찾기 strcmp다 찾아줌 

->라이브러리 찾을 때 extern 붙어있는 거 찾으면 됨 

(대신 cd /usr/include 로 가야함)

![image](https://user-images.githubusercontent.com/79901413/119457265-744e2100-bd76-11eb-93cc-524cc77b02df.png)

(inet_addr 찾는거)

.

p4 온도 p5 조도 p6 가변
채널번호 0: 조도센서 1: 온도센서 3:가변

test
```
#include <stdio.h>
#include <wiringPi.h>
#include <wiringPiI2C.h>

int main()
{
	float val;
	int hndl = wiringPiI2CSetup(0x48); // I2C 모듈의 Handle 반환,0x48: 모듈자체주소
	
	wiringPiI2CWrite(hndl, 3 ); // 채널번호 0:조도 1:온도 3:가변
	wiringPiI2CRead(hndl);
	
	while(1)
	{
		val = wiringPiI2CRead(hndl);
		printf("read from I2C address [3] : %f\n",val);
		delay(500);
	}
}
```

채널 세개 한번에 코드에 넣고 [ ./i2ctest 채널번호 ] 로 실행하기
```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <wiringPi.h>
#include <wiringPiI2C.h>

int main(int argc, char *argv[]) // int main(int argc, char *argv[])
{		   // ...$ i2ctest 1 2 3
		   // ---> argc : 4 , argv = {"i2ctest", "1", "2", "3"}
	float val;
	int hndl = wiringPiI2CSetup(0x48); // I2C 모듈의 Handle 반환,0x48: 모듈자체주소
	
	if(argc < 2)
	{
		printf("\nUsage : %s AIN_No \n\n", argv[0]);
		return 0;
	}
	
	int ch =atoi(argv[1]); // atoi: ascii to int
	
	/*if(strcmp(argv[1] ,"0")==0) 
	{
		ch =0;
		printf("조도 센서를 측정합니다.\n\n");
	}
	if(strcmp(argv[1] ,"1")==0) ch =1;
	if(strcmp(argv[1] ,"3")==0) ch =3; */
	
	wiringPiI2CWrite(hndl, ch ); // 채널번호 0:조도 1:온도 3:가변
	wiringPiI2CRead(hndl); // 채널 enable, 확인 응답
	
	while(1)
	{
		val = wiringPiI2CRead(hndl);
		printf("read from I2C address [%d] : %f\n",ch, val);
		delay(500);
	}
}
```

조도센서를 이용해서
200 이상(어두울때)은 led On 이하(밝을때)는 led Off
```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <wiringPi.h>
#include <wiringPiI2C.h>

int main(int argc, char *argv[]) // int main(int argc, char *argv[])
{		   // ...$ i2ctest 1 2 3
		   // ---> argc : 4 , argv = {"i2ctest", "1", "2", "3"}
	float val;
	int hndl = wiringPiI2CSetup(0x48); // I2C 모듈의 Handle 반환,0x48: 모듈자체주소
	wiringPiSetup();
	pinMode(25, OUTPUT);
	
	if(argc < 2)
	{
		printf("\nUsage : %s AIN_No \n\n", argv[0]);
		return 0;
	}
	
	int ch =atoi(argv[1]); // atoi: ascii to int
	
	/*if(strcmp(argv[1] ,"0")==0) 
	{
		ch =0;
		printf("조도 센서를 측정합니다.\n\n");
	}
	if(strcmp(argv[1] ,"1")==0) ch =1;
	if(strcmp(argv[1] ,"3")==0) ch =3; */
	
	wiringPiI2CWrite(hndl, ch ); // 채널번호 0:조도 1:온도 3:가변
	wiringPiI2CRead(hndl); // 채널 enable, 확인 응답
	
	while(1)
	{
		val = wiringPiI2CRead(hndl);
		printf("read from I2C address [%d] : %f\n",ch, val);
		if(val > 200) digitalWrite(25,LOW);
		else digitalWrite(25,HIGH);
		delay(500);
	}
}
```

![KakaoTalk_20210520_164006480](https://user-images.githubusercontent.com/79901413/118938960-27d79f80-b98a-11eb-9419-7d72eb3924b8.jpg)


### 2021-05-21

초음파 센서 사용하기 

가청 주파수 (20-20K Hz), 
음속 340m/sec

1. trig 발사
2. 발사와 동시 timer 가동
3. Echo 감지와 동시 timer 종료
4. 3.-2.의 dt(Ms) 계산
5. dt*1.7mm 

초음파 센서로 거리 측정하기
```
#include <stdio.h>
#include <stdlib.h>
#include <wiringPi.h>

int main()
{
	int wTrig = 15;
	int wEcho = 16;
	
	wiringPiSetup();
	pinMode(wTrig, OUTPUT); // 측정 신호 발사
	pinMode(wEcho, INPUT); // 반사 신호 검출 
	
	while(1)
	{
		digitalWrite(wTrig, LOW); 
		delayMicroseconds(100); // 트리거 신호를 위한 초기화 
		
		digitalWrite(wTrig, HIGH);
		delayMicroseconds(10); 
		digitalWrite(wTrig, LOW); // 10us 의 트리거 신호
		delayMicroseconds(200); // 실제 신호발사까지 지연시간 
		
		while(digitalRead(wEcho) == LOW); // until high
		long start = micros(); // micros() : 현재 시간의 마이크로초 단위 count
		while(digitalRead(wEcho) == HIGH); // until low
		long end = micros(); 
		
		double dist = (end - start) * 0.17; // 음속으로 계산하기
		printf("Distance : %f \n" , dist);
		delay(1000);		
	}

}
```

### 2021-05-24

MPU6050 : Gyroscope(각속도) + Accelerometer(가속도) + Temperature 센서 모듈

MPU6050 센서 모듈은 통합 6 축 모션 추적 장치입니다.

3 축 자이로 스코프, 3 축 가속도계, 디지털 모션 프로세서 및 온도 센서가 모두 단일 IC에 있습니다.

.

i2cdetect -y 1 : i2c Address 확인

.

가속도=측정값/16384(g), 각속도=측정값/131(°/s)

(16384? 2^14 = 16384 로 14비트를 사용한다)

.

![image](https://user-images.githubusercontent.com/79901413/119425094-b27d1d80-bd41-11eb-8cac-2f7bd75abda3.png)
 
.


```
#include <stdio.h>
#include <wiringPi.h>
#include <wiringPiI2C.h>
short i2cInt16(int hndl, int addr);

int main()
{
	int i2cAddr = 0x68; // i2cdetect -y 1 로 확인
	int bufAddr = 0x3b; // Memory Block
	int pwrAddr = 0x6b; // 0x가 16진수 
	
	wiringPiSetup();
	int hndl = wiringPiI2CSetup(i2cAddr); // i2c 주소 넣어야함
	
	// wiringPiI2CWriteReg8(fd, addr, value): 해당 메모리 번지에 값을 쓸 수 있다.
	wiringPiI2CWriteReg8(hndl, pwrAddr, 0);  // 0 값으로 초기화
	
	double x1,y1,z1,x2,y2,z2;

	while(1)
	{
		x1 = i2cInt16(hndl, bufAddr)/16384.0;
		y1 = i2cInt16(hndl, bufAddr+2)/16384.0;
		z1 = i2cInt16(hndl, bufAddr+4)/16384.0;
		x2 = i2cInt16(hndl, bufAddr+8)/131.0; // 온도 데이터 건너뜀
		y2 = i2cInt16(hndl, bufAddr+10)/131.0;
		z2 = i2cInt16(hndl, bufAddr+12)/131.0;
		
		printf("[x1=%f y1=%f z1=%f], [x2=%f y2=%f z2=%f]\n", x1,y1,z1,x2,y2,z2);
		// [가속도](측정값/16384)g, [각속도](측정값/131)°/s
	}
}

short i2cInt16(int hndl, int addr)
{
	short d1 = wiringPiI2CReadReg8(hndl, addr);
	short d2 = wiringPiI2CReadReg8(hndl, addr+1);
	short d3 = (d1 << 8 | d2); // 비트 쉬프트 레프트? 상위로 옮김 
	return d3;
}
```


### 2021-05-25

초음파 센서를 이용해서 LED제어하기

거리에 따라서 RED, YELLOW, GREEN 색을 키도록 한다

.

![KakaoTalk_20210525_110235725](https://user-images.githubusercontent.com/79901413/119428978-e1e35880-bd48-11eb-831c-efc37c76300c.jpg)

.

```
#include <stdio.h>
#include <stdlib.h>
#include <wiringPi.h>

int main()
{
	int wTrig = 15;
	int wEcho = 16;
	int wRed = 25;
	int wGreen = 24;
	int wYellow = 23;
	
	wiringPiSetup();
	pinMode(wTrig, OUTPUT); // 측정 신호 발사
	pinMode(wEcho, INPUT); // 반사 신호 검출
	pinMode(wRed, OUTPUT);
	pinMode(wGreen, OUTPUT);
	pinMode(wYellow, OUTPUT);
	
	while(1)
	{
		digitalWrite(wTrig, LOW); 
		delayMicroseconds(100); // 트리거 신호를 위한 초기화 
		
		digitalWrite(wTrig, HIGH);
		delayMicroseconds(10); 
		digitalWrite(wTrig, LOW); // 10us 의 트리거 신호
		delayMicroseconds(200); // 실제 신호발사까지 지연시간 
		
		while(digitalRead(wEcho) == LOW); // until high
		long start = micros(); // micros() : 현재 시간의 마이크로초 단위 count
		while(digitalRead(wEcho) == HIGH); // until low
		long end = micros(); 
		
		double dist = (end - start) * 0.17;
		printf("Distance : %f \n" , dist);	
			
		if(dist <= 100)
		{
			digitalWrite(wRed, HIGH);
			digitalWrite(wYellow, LOW);
			digitalWrite(wGreen, LOW);
		}
		else if(dist > 100 && dist <= 300)
		{
			digitalWrite(wRed, LOW);
			digitalWrite(wYellow, HIGH);
			digitalWrite(wGreen, LOW);
		}
		else
		{
			digitalWrite(wRed, LOW);
			digitalWrite(wYellow, LOW);
			digitalWrite(wGreen, HIGH);
		}
			

		delay(1000);		
	}

}
```

.

.

라즈베리파이 socket 통신

.



<Client>

1. Socket 선언

2. 초기화/생성/Open 구성

3. Connection to Server

<Server>

1. Socket 선언

2. 초기화/생성/Open 구성 
//구성 단계까지는 Client와 같음

3. Bind - local port

4. Listen

5. Accept (Client의 Connetion 요청을 받아들임 -> Session 연결)


### 2021-05-26

printf : console 출력
	
fprintf : file 출력
	
sprintf : buffer 출력

. 

초음파센서에서 읽은 값을 Sever form 창에서 볼 수 있다.
	
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <fcntl.h>
#include <wiringPi.h>

char *IP = "192.168.0.35";
int PORT = 9001;

int main()
{
	int wTrig = 15;
	int wEcho = 16;
	
	wiringPiSetup();
	pinMode(wTrig, OUTPUT); // 측정 신호 발사
	pinMode(wEcho, INPUT); // 반사 신호 검출 
	
	int sock; // socket handle
	struct sockaddr_in sockinfo;
	char buf[1024];
	int i,j,k;
	
	sock = socket(AF_INET, SOCK_STREAM, 0);
	sockinfo.sin_family = AF_INET;
	inet_pton(AF_INET, IP, &sockinfo.sin_addr.s_addr); // sin_addr: 32bit=long
	sockinfo.sin_port = htons(PORT);
	
	connect(sock, (struct sockaddr*)&sockinfo, sizeof(sockinfo));	
	k = fcntl(sock, F_SETFL, 0);
	fcntl(sock, F_SETFL, k | O_NONBLOCK);
	 
	while(1)
	{	
		digitalWrite(wTrig, LOW); 
		delayMicroseconds(100); // 트리거 신호를 위한 초기화 
		
		digitalWrite(wTrig, HIGH);
		delayMicroseconds(10); 
		digitalWrite(wTrig, LOW); // 10us 의 트리거 신호
		delayMicroseconds(200); // 실제 신호발사까지 지연시간 
		
		while(digitalRead(wEcho) == LOW); // until high
		long start = micros(); // micros() : 현재 시간의 마이크로초 단위 count
		while(digitalRead(wEcho) == HIGH); // until low
		long end = micros(); 
		
		double dist = (end - start) * 0.17;
		sprintf(buf,"Distance : %f \n" , dist);
		if(buf[0] == 'q') break;
		send(sock, buf, strlen(buf), 0);
		delay(1000);		
		
		
/*		scanf("%s",buf);
		if(buf[0] == 'q') break;
		send(sock, buf, strlen(buf), 0); //strlen: 해당 어레이의 길이를 반환
		i = recv(sock, buf, 1024, 0);
		if(i > 0) buf[i] = 0;
		if(buf[0] == 'q') break;
		printf("%s\n", buf);
		*/
	}
	close(sock);
}
```

	
### 2021-05-27

tcp 통신 client

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <fcntl.h>
#include <wiringPi.h>
#include <pthread.h>

void * readProc();
int PORT = 9001;
int sock, sock_cli;
struct sockaddr_in sockinfo, sockinfo_cli;
char buf[1024]; // 1k

int main()
{
   pthread_t readThread;

   sock = socket(AF_INET, SOCK_STREAM, 0);
   sockinfo.sin_family = AF_INET;
   sockinfo.sin_addr.s_addr = htonl(INADDR_ANY);
   sockinfo.sin_port = htons(PORT);
   
   bind(sock, (struct sockaddr*)&sockinfo, sizeof(sockinfo)); 
   listen(sock, 100);
   int n = sizeof(sockinfo_cli);
   sock_cli = accept(sock, (struct sockaddr*)&sockinfo_cli, &n);
   
   pthread_create(&readThread, NULL, readProc, NULL);


   while(1)
   {
      printf("Input text > ");
      scanf("%s",buf);
      if(buf[0] == 'q') break;
      send(sock_cli, buf, strlen(buf), 0);
   }
   close(sock);
   pthread_join(readThread, NULL);

}

void * readProc()
{
   int i;
   char buf[1024];
   while(1)
   {
      i = recv(sock_cli, buf, 1024, 0);
      if(i > 0) 
      {
         buf[i] = 0;
         printf("%s\n", buf); // console 출력
      }
      delay(500);
   }
   return NULL;
}
```
	
.
	
tcp 통신 Server

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <fcntl.h>
#include <wiringPi.h>
#include <pthread.h>

void * readProc();
int PORT = 9001;
int sock, sock_cli; // socket handle
struct sockaddr_in sockinfo, sockinfo_cli;
char buf[1024]; // 1k

int main()
{
   pthread_t readThread;

   sock = socket(AF_INET, SOCK_STREAM, 0);
   sockinfo.sin_family = AF_INET;
   sockinfo.sin_addr.s_addr = htonl(INADDR_ANY);
   sockinfo.sin_port = htons(PORT);
   
   bind(sock, (struct sockaddr*)&sockinfo, sizeof(sockinfo)); 
   listen(sock, 100);
   int n = sizeof(sockinfo_cli);
   sock_cli = accept(sock, (struct sockaddr*)&sockinfo_cli, &n);
   
   pthread_create(&readThread, NULL, readProc, NULL);


   while(1)
   {
      printf("Input text > ");
      scanf("%s",buf);
      if(buf[0] == 'q') break;
      send(sock_cli, buf, strlen(buf), 0);
   }
   close(sock);
   pthread_join(readThread, NULL);

}

void * readProc()
{
   int i;
   char buf1[1024];
   while(1)
   {
      i = recv(sock_cli, buf1, 1024, 0);
      if(i > 0) 
      {
         buf1[i] = 0;
         printf("%s\n", buf1); // console 출력
      }
      delay(500);
   }
   return NULL;
}
```
	
.
	
udp 통신 client
	
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

char *IP = "192.168.0.35";
int PORT = 9200;
struct sockaddr_in udp_info;
int sock;

int main()
{
	char buf[512];
	// udp info setting
	sock = socket(AF_INET, SOCK_DGRAM, 0); // SOCK_DGRAM : tcp랑 차이점 
	udp_info.sin_family = AF_INET;
	inet_pton(AF_INET, IP, &udp_info.sin_addr.s_addr);
	udp_info.sin_port = htons(PORT);
	// connection 필요없음 : udp 특징
	
	while(1)
	{
		printf("Input Text > ");
		scanf("%s",buf);
		if(buf[0] == 'q') break;
		sendto(sock,buf,strlen(buf),0, (struct sockaddr*)&udp_info,sizeof(udp_info));
	}
	close(sock);
}
```

	
