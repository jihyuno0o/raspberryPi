# raspberryPi

#2021-05-18

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



