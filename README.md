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










