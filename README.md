# shixi

最近在实习的空闲时间把c语言，跟java又重新复习了一遍发现了一些以前从没注意到的细节，对这两门语言也有了更深入的理解，就写一点经常会用到的东西吧，以及一些小技巧。以后再忘记了，就直接看看自己写的就行。

说到为什么复习c语言，是因为实习的任务是完成一个stm32的单片机编程，用来分析无人机飞行时所记录的参数 
作为一个计科专业的人来说，硬件方面我还是有些欠缺，于是我就负责软件方面的编写；

大概的任务是把一个无人机记录的一连串以16进制表示的字符串按照惯导协议分析成对应的飞机状态。 
而无人机记录的数据大概是这样的

EB 90 74 D1 4D 0D 00 00 00 EE F9 17 BD 84 82 8F 3D BC DF 20 3F 00 00 00 00 C3 B2 0E 3C 3F C3 6E 3C 55 17 13 3C 00 00 00 00 00 00 00 00 7F FF FF FF 5C F9 FF FF 7F FF FF FF 55 49 84 3D CD A8 34 3D 93 30 A9 3B 00 00 00 00 1B 55 ED 3C 5E 04 E2 3A 62 54 1D C1 00 0F 27 0F 27 01 00 01 02 26 F6 FF 1E FF 15 00 FF 00 00 00 00 00 00 00 00 00 00 00 00 65 26 EB 90 74 D1 4E 0D 00 00 00 D1 C4 17 BD 7F A6 8F 3D 3C E6 20 3F 00 00 00 00 E8 39 24 3C 91 DD 61 3C 96 5E 0D 3C 00 00 00 00 00 00 00 00 7F FF FF FF 5C F9 FF FF 7F FF FF FF 11 03 84 3D A6 6F 35 3D FB 9A B1 3B 00 00 00 00 18 A4 AE 3C 9A 77 C5 3C 18 24 1D C1 00 0F 27 0F 27 01 00 01 02 26 F6 FF 1E FF 15 00 FF 00 00 00 00 00 00 00 00 00 00 00 00 75 26

经过这次项目我学到了一些技巧： 
1. 读取文件并记录，这里我用到了fscanf(fp, “%s”, v); 这个函数 // 从一个流中执行格式化输入,fscanf遇到空格和换行时结束，注意空格时也结束。把每8位的一个16进制数存到了一个*v所指向的数组中，方便后续的操作。 
2. 读取4个字节的16进制数并转换成无符号数，这里用了result = (d << 24) | (c << 16) | (b << 8) | a;移位操作，比较方便。
3. 读取4个字节的16进制数并转化成浮点类型，这里用到了 result = ((float)ss);指针的这个特性。没有用移位操作是因为符号位的原因。 
4. 发现浮点数并没有以前想的那么简单，可以参考这位博主的文章https://blog.csdn.net/tercel_zhang/article/details/52537726【关于浮点数】 
5. 把一个8位的16进制数写成2进制的形式并按位来分析状态，用到了一个简单的算法

int a[8]; 
int i，b; 
for (i = 0; i != 8; ++i) 
{ 
a[7 - i] = b % 2; 
b /= 2; 
}
以下是完整的c代码（最初版本，尚未改进，也是今天这个项目刚做完）

#include <stdio.h>
#include<time.h>
#include<stdlib.h>
#include<string.h>
#include<math.h>
int i = 0;
char *q = (char *)malloc(sizeof(int) * 100);
char *v = q;
char *s = (char *)malloc(sizeof(int) * 100);
char *s1 = s;
/*int findanswer()
{
	//int GetS1Value(void);
	int cha = &s1 + 1 - &s1;
	//printf("结果是：%d", cha);
	printf("结果是：%d", s1);
	return 0;
}

char* find()
{
	const char *b = "EB";
	//char *s= (char *)malloc(1048576);
	//char *s1 = s;
	if (strstr(v, b) != NULL)
	{
		printf("\n\n");
		findanswer();
	}
	return s1;
}*/

int open()
{
	FILE *fp;
	char *v = q;
	fp = fopen("C:/Users/hp/Desktop/test.txt", "r");
	if (fp == NULL)
		printf("cantfind the file!");
	while (!feof(fp))   // feof  如果文件结束，则返回非0值，否则返回0，文件结束符只能被clearerr()清除。
	{
		fscanf(fp, "%s", v);   // 从一个流中执行格式化输入,fscanf遇到空格和换行时结束，注意空格时也结束。
		printf("%s ", v);
		v = v + 3;
	}
	fclose(fp);                // 关闭文件
	return 0;
}

int framesize()					//帧长
{
	int a;
	sscanf(q + 6, "%X", &a);
	printf("\n\n帧长=%d", a);
	printf("\n帧识别字：D1");
	return 0;

}

int data4()					//读32位无符号
{
	int a, b, c, d, result;
	char *str1 = q + 12;
	char *str2 = q + 15;
	char *str3 = q + 18;
	char *str4 = q + 21;
	sscanf(q + 12, "%X", &a);//%x为读入16进制数，大小写均可。sscanf是从c字符串中读入变量。
	sscanf(str2, "%X", &b);
	sscanf(str3, "%X", &c);
	sscanf(str4, "%X", &d);
	result = (d << 24) | (c << 16) | (b << 8) | a;
	printf("帧计数器\t= %d", result);
	return 0;
}

int data5()
{
	int b;
	sscanf(q + 3 * 8, "%X", &b);
	char *n = q + 3 * 8;
	int a[8];
	int i;
	for (i = 0; i != 8; ++i)
	{
		a[7 - i] = b % 2;
		b /= 2;
	}
	printf("\n");
	if (a[0] == 0)
		printf("初始化\n");
	if (a[1] == 1)
		printf("正常\n");
	if (a[2] == 2)
		printf("错误\n");
	printf("罗盘是否需要校准:");
	if (a[3 == 0])
		printf("正常\n");
	else
		printf("需要校准\n");
	if (a[4] == 0)
		printf("罗盘正常\n");
	else
		printf("罗盘故障\n");
	if (a[5] == 0)
		printf("陀螺正常\n");
	else
		printf("陀螺故障\n");
	if (a[6] == 0)
		printf("加计正常\n");
	else
		printf("加计故障\n");
	if (a[7] == 0)
		printf("气压计正常\n");
	else
		printf("气压计故障\n");
	return 0;
}

int data5redundancy(int l)
{
	int b;
	sscanf(q + 3 * l, "%X", &b);
	char *n = q + 3 * l;
	int a[8];
	int i;
	for (i = 0; i != 8; ++i)
	{
		a[7 - i] = b % 2;
		b /= 2;
	}
	printf("\n");
	if (a[1] == 1||a[0]==1)
		printf("加计 内部\n");
	else
		printf("加计 外部\n");

	if (a[3] == 1|| a[2] == 1)
		printf("陀螺 内部\n");
	else
		printf("陀螺 外部\n");

	if (a[4] == 0)
		printf("罗盘 外部\n");
	else
		printf("罗盘 内部\n");

	if (a[6] == 0)
		printf("GPS  内部\n");
	else
		printf("GPS  外部\n");
	return 0;
}

int data6()			//读32位浮点数
{
	unsigned int a, b, c, d;
	float result;
	char *str1 = q + 27 + 12;
	char *str2 = q + 30 + 12;
	char *str3 = q + 33 + 12;
	char *str4 = q + 36 + 12;
	unsigned char ss[4] = { 0 };
	sscanf(str1, "%X", &a);//%x为读入16进制数，大小写均可。sscanf是从c字符串中读入变量。
	sscanf(str2, "%X", &b);
	sscanf(str3, "%X", &c);
	sscanf(str4, "%X", &d);
	printf("  a=%X", a);
	printf("  b=%X", b);
	printf("  c=%X", c);
	printf("  d=%X", d);
	ss[0] = a;
	ss[1] = b;
	ss[2] = c;
	ss[3] = d;
	result = *((float*)ss);
	printf("帧计数器=%f", result);
	return 0;
}

int intdataloop(int i)			//读32位整形数
{
	int a, b, c, d, result;	
	char *str1 = q + 3 * i;
	char *str2 = q + 3 * (i + 1);
	char *str3 = q + 3 * (i + 2);
	char *str4 = q + 3 * (i + 3);
	sscanf(str1, "%X", &a);//%x为读入16进制数，大小写均可。sscanf是从c字符串中读入变量。
	sscanf(str2, "%X", &b);
	sscanf(str3, "%X", &c);
	sscanf(str4, "%X", &d);
	//printf("a=%X", a);
	//printf("\tb=%X", b);
	//printf("\tc=%X", c);
	//printf("\td=%X", d);
	result = (d << 24) | (c << 16) | (b << 8) | a;
	printf("\t = %d", result);
	return 0;
}

int fdataloop(int i)			//读32位浮点数
{
	int a, b, c, d;
	float result;
	char *str1 = q + 3 * i;
	char *str2 = q + 3 * (i + 1);
	char *str3 = q + 3 * (i + 2);
	char *str4 = q + 3 * (i + 3);
	unsigned char ss[4] = { 0 };
	sscanf(str1, "%X", &a);//%x为读入16进制数，大小写均可。sscanf是从c字符串中读入变量。
	sscanf(str2, "%X", &b);
	sscanf(str3, "%X", &c);
	sscanf(str4, "%X", &d);
	//printf("  \na=%X", a);
	//printf(" \tb=%X", b);
	//printf(" \tc=%X", c);
	//printf(" \td=%X", d);
	ss[0] = a;
	ss[1] = b;
	ss[2] = c;
	ss[3] = d;
	result = *((float*)ss);
	printf("\t= %e", result);
	return 0;
}

int calu8(int i)//处理8位
{
	int a;
	char *str1 = q + 3 * i;
	sscanf(str1, "%X", &a);
	//printf("  \na=%X", a);
	printf("\t%d", a);
	return 0;
}

int calu8gps_status(int i)//gps分析
{
	int a;
	char *str1 = q + 3 * i;
	sscanf(str1, "%X", &a);
	//printf("  \na=%X", a);
	printf("\n");
	switch (a)
	{
	case 0:
		printf("无 GPS 数据");
		break;
	case 1:
		printf("GPS 信号失锁");
		break;
	case 2:
		printf("2D 定位");
		break;
	case 3:
		printf("3D 定位");
		break;
	case 4:
		printf("3D_DGPS");
		break;
	case 5:
		printf("3D RTK Float");
		break;
	case 6:
		printf("3D RTK Fixed");
		break;
	}
	//printf("\t%d", a);
	return 0;
}

int calu16(int i)//处理16位
{
	int  a, b, result;
	char *str1 = q + 3 * i;
	char *str2 = q + 3 * (i + 1);
	sscanf(str1, "%X", &a);
	sscanf(str2, "%X", &b);
	//printf("\na=%X", a);
	//printf("\tb=%X", b);
	result = (b << 8) | a;
	printf("\t%d", result);
	return 0;
}

int calu16ofcycle(int i)//处理16位的分析数据
{
	int  a, b, result;
	char *str1 = q + 3 * i;
	char *str2 = q + 3 * (i + 1);
	sscanf(str1, "%X", &a);
	sscanf(str2, "%X", &b);
	//printf("\na=%X", a);
	//printf("\tb=%X", b);
	result = (b << 8) | a;
	printf("\t%d", result%36000);
	return 0;
}

int main()
{
	open();					//读入文件
	framesize();			//帧长
	data5();				//U8 state
	data4();	//帧计数器
	printf("\n抬头正\t");
	fdataloop(9);		//data[9]---data[16]
	printf("\n右滚正\t");
	fdataloop(13);
	printf("\nN 0d E 90d W -90d S +-180d");
	fdataloop(17);
	printf("\nGPS 航迹向");
	fdataloop(21);
	printf("\n抬头正\t");
	fdataloop(25);
	printf("\n右滚正\t");
	fdataloop(29);
	printf("\n顺时针为正");
	fdataloop(33);
	printf("\nlon INS");
	intdataloop(37);		//data[17]---data[22]
	printf("\nlat INS"); 
	intdataloop(41);
	printf("\n原始气压高度  单位0.01m");
	intdataloop(45);
	printf("\n原始 GPS 高度 单位0.01m");
	intdataloop(49);
	printf("\nalt INS");
	intdataloop(53);
	printf("\n正 北向速度");
	fdataloop(57);		//data[23]---data[30]
	printf("\n正 东向速度"); 
	fdataloop(61);
	printf("\n正 地向速度");
	fdataloop(65);
	printf("\n空速  无效");
	fdataloop(69);
	printf("\n正 北向加速度");
	fdataloop(73);
	printf("\n正 东向加速度");
	fdataloop(77);
	printf("\n正 地向加速度");
	fdataloop(81);
	printf("\n卫星数目");
	calu8(85);				//satellite_num 卫星数目
	printf("\n水平精度因子");
	calu16(86);				// hdop 
	printf("\n垂直精度因子");
	calu16(88);				//vdop
	calu8gps_status(90);			//逐位分析
	printf("\n时");
	calu8(91);			//gps时
	printf("\n分");
	calu8(92);			//gps分
	printf("\n秒");
	calu8(93);			//gps秒
	printf("\n摄氏度");
	calu8(94);			//摄氏度
	printf("\n双天线航向 单位 0.1 度    ");
	calu16ofcycle(95);			//双天线航向
	printf("\n天线航向标准差 单位 0.1 度");
	calu16ofcycle(97);			// 天线航向标准差
	printf("\n\n");
	printf("各传感器使用状态");
	data5redundancy(99);			//各传感器使用状态
	printf("\n内部 GPS 采样间隔 单位 100ms");
	calu8(100);			//内部 GPS 采样间隔
	printf("\n外部 GPS 采样间隔 单位 100ms");
	calu8(101);			//外部 GPS 采样间隔
	printf("\n北向速度");
	fdataloop(102);
	printf("\n东向速度");
	fdataloop(106);
	printf("\n地向速度");
	fdataloop(110); 
	printf("\n");
	free(q);
	free(s);
}
这是连上串口后xcom串口助手跑的程序： 
这是连上串口后xcom串口助手跑的程序
这也是我的第一个正式的项目，也就今儿刚完成，比较激动，emmmm就写篇博客纪念一下吧。
