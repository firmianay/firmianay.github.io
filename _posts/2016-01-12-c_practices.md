---
layout: post
title:  "一些 C 语言编程练习"
image: ''
date:   2016-01-02 00:06:31
tags:
- C
- programming
description: ''
categories:
- programming
---


> 本文写于 2016.01.12

C语言数制转换程序

开学的第一个C语言作业就是写进制转换的程序，当时初学，写的不是很满意（当然现在也只是初学。。。）。马上C语言考试了，想起这个程序，重新写了一下。

主体框架如下：
```c
main()
{
    while(flag)     //利用 flag 控制是否结束
    {
        //将原数转换成的十进制数
        //求出转换成目标进制后字符数组的长度
        //逆序打印字符数组
        printf("继续输入1，退出输入0： \n");
        scanf("%d",&flag);
    }
}
```
下面是完整的程序：
```c
#include<stdio.h>
#define MAXCHAR 101                //最大允许字符串长度
int char_to_num(char ch);       //返回字符对应的数字
char num_to_char(int num);      //返回数字对应的字符
long source_to_decimal(char temp[], int source);
                                //返回由原数转换成的十进制数
int decimal_to_object(char temp[], long decimal_num, int object);
                                //返回转换成目标数制后字符数组的长度
void output(char temp[], int length);   //将字符数组逆序输出
main()
{
    int source;                 //原数制
    int object;                 //目标数制
    int length;                 //转换成目标数制后字符数组的长度
    long decimal_num;           //转换成的十进制数
    char temp[MAXCHAR];         //待转换的数值和转换后的数值
    int flag=1;                 //是否退出程序的标志
    while(flag)                 //
    {
        printf("转换前的数是: ");
        scanf("%s", temp);
        printf("转换前的数制是: ");
        scanf("%d",&source);
        printf("转换后的数制是: ");
        scanf("%d",&object);
        printf("转换后的数是: ");
        decimal_num = source_to_decimal(temp,source);
        length = decimal_to_object(temp,decimal_num,object);
        output(temp,length);
        printf("继续输入1，退出输入0: \n");
        scanf("%d",&flag);
    }
}

int char_to_num(char ch)
{
    if(ch>='0'&&ch<='9')
        return ch-'0';          //将数字字符转换成数字
    else
        return ch-'A'+10;       //将字母字符转换成数字
}

char num_to_char(int num)
{
    if(num>=0&&num<=9)
        return (char)('0'+num-0);   //将 0~9 之间的数字转换成字符
    else
        return (char)('A'+num-10);  //将大于 10 的数字转换成字符
}

long source_to_decimal(char temp[], int source)
{
    long decimal_num=0;         //展开后的和
    int length;
    int i;
    for(i=0;temp[i]!='\0';i++);
    length=i;
    for(i=0;i<=length-1;i++) //累加
        decimal_num=(decimal_num*source)+char_to_num(temp[i]);
    return decimal_num;
}

int decimal_to_object(char temp[], long decimal_num, int object)
{
    int i=0;
    while(decimal_num)
    {
        temp[i]=num_to_char(decimal_num%object);    //求出余数并转换为字符
        decimal_num=decimal_num/object;             //用十进制数除以基数
        i++;
    }
    temp[i]='\0';
    return i;
}

void output(char temp[], int length)
{
    int i;
    for(i=length-1;i>=0;i--)             //输出 temp 数组中的值
        printf("%c\n", temp[i]);
    printf("\n");
}
```

学了几个月的C语言好像就没写过几段代码，看着班里的大神们，感觉好方啊。以后要努力坚持每天写了，先贴几个有关整数的基础题，蛮有意思的。

1、完数。

如果一个数等于它的因子之和，就称为完数。

求某一个范围内完数的个数。

本题关键是计算出所选取的整数i的因子，将各因子累加到变量s，若s等于i，则为完数。
```c
#include<stdio.h>
main()
{
    int i,j,s,n;    //变量i控制选定数范围，j控制除数范围，s记录累加因子之和
    printf("请输入所选范围上限： ");
    scanf("%d",&n);             //n的值由键盘输入
    for(i=2;i<=n;i++)
    {
        s=0;
        for(j=1;j<i;j++)  //每次循环时s的初值为0
        {
            if(i%j==0)          //判断j是否为i的因子
                s+=j;
        }
        if(s==i)                //判断因子之和是否和原数相等
            printf("完数有：%d\n", i);
    }
}
```

2、亲密数

如果整数A的全部因子（包括1，不包括A本身）之和等于B，且整数B的全部因子之和等于A，则A和B称为亲密数。

求3000以内的全部亲密数。

本题可转换为：给定整数A，判断A是否有亲密数。首先定义变量a并赋值，计算出全部因子的累加和并存放到变量b中，再计算b全部因子的累加和并存放到变量n中，判断a是否等于n。
```c
#include<stdio.h>
void main()
{
    int a,b,i,n;
    printf("下面是不超过3000的亲密数:\n");
    for(a=1;a<3000;a++)             //穷举3000以内的全部整数
    {
        for(b=0,i=1;i<=a/2;i++)     //计算数a的各因子，各因子之和存放与b
            if(a%i==0)
                b+=i;
        for(n=0,i=1;i<=b/2;i++)     //计算b的各因子，各因子之和存与n
            if(b%i==0)
                n+=i;
        if(n==a&&a<b)               //若n=a，则a和b是一对亲密数
            printf("%4d--%4d\n", a,b);
    }
}
```

3、自守数

自守数是指一个数的平方的尾数等于该数自身的自然数。

求100000以内的自守数。

求解关键是知道当前所求自然数的尾数，以及该数平方的尾数与被乘数、乘数之间的关系。
```c
#include<stdio.h>
void main()
{
    long mul,number,k,a,b;
    printf("下面是100000以内的自守数：\n");
    for(number=0;number<100000;number++)
    {
        for(mul=number,k=1;(mul/=10)>0;k*=10);
                        //由number的位数确定截取数字进行乘法时的系数
        a=k*10;         //a为截取部分积时的系数
        mul=0;          //积的最后n位
        b=10;           //b为截取乘数相应位时的系数
        while(k>0)
        {
            mul=(mul+(number%(k*10))*(number%b-number%(b/10)))%a;
                //(部分积+截取被乘数的后n位*截取乘数的第m位)，%a再截取部分积
            k/=10;      //k为截取被乘数时的系数
            b*=10;
        }
        if(number==mul) //判定若为自守数则输出
            printf("%ld    ", number);
    }
    printf("\n");
}
```

4、水仙花数

指一个三位数其各位数字的立方和等于该数本身。
```c
#include<stdio.h>
main()
{
    int hun,ten,ind,n;
    printf("水仙花数有： \n");
    for(n=100;n<1000;n++)       //整数的取值范围
    {
        hun=n/100;
        ten=(n-hun*100)/10;
        ind=n%10;
    if(n==hun*hun*hun+ten*ten*ten+ind*ind*ind)
        printf("%d\t", n);
    }
    printf("\n");
}
```

阿姆斯特朗数

一个整数等于其各个数字的立方和。

水仙花数可以看做阿姆斯特朗数的一个子集。

求1000以内的所有阿姆斯特朗数。
```c
#include<stdio.h>
main()
{
    int i,t,k,a[3]={0};
    printf("下面是小于1000的自恋数： \n");
    for(i=2;i<1000;i++)
    {
        t=0;
        k=i;
        while(k)        //按从低到高位的顺序拆分数
        {
            a[t]=k%10;
            k=k/10;
            t++;
        }
        if(i==a[0]*a[0]*a[0]+a[1]*a[1]*a[1]+a[2]*a[2]*a[2])
            printf("%d    ", i);
    }
    printf("\n");
}
```

5、高次方数的尾数

求13的13次方的最后三位数。

通常想到的方法时将13累乘13次后截取最后三位，可是计算机存储的整数有一定范围，这样做行不通。

提示：题目仅要求后三位的值，完全没必要把13的13次方完全求出来。
```c
#include<stdio.h>
main()
{
    int i,x,y,last=1;        //变量last保存求得的x的y次方的部分积的后三位
    printf("请输入x和y： \n")；
    scanf("%d %d", &x,&y);
    for(i=1;i<=y;i++)       //x自乘的次数y
        last=last*x%1000;   //将last乘x后对1000取模，即求积的后三位
    printf("后三位为： %d\n", last);
}
```

6、黑洞数

任何一个数字不全相同的整数，经有限次”重排求差“，总会得到某一个或一些数，这些数即黑洞数。重排求差是将组成一个数的各位数字重排得到的最大数减去最小数。

过程如下：把数字拆分后重组，最大值减最小值，差值赋给变量j，将当前差值暂存到变量h，对变量j拆分重组求差。差值仍存在j中，判断当前差值j与前差值h是否相等，若相等则输出并退出。
```c
#include<stdio.h>
int maxof3(int,int,int);
int minof3(int,int,int);
main()
{
    int i,k;
    int hun,oct,data,max,min,j,h;
    printf("请输入一个三位数： \n");
    scanf("%d",&i);
    hun=i/100;
    oct=i%100/10;
    data=i%10;
    max=maxof3(hun,oct,data);
    min=minof3(hun,oct,data);
    j=max-min;
    for(k=0;;k++)           //k控制循环次数
    {
        h=j;                //h记录上一次最大值与最小值的差
        hun=j/100;
        oct=j%100/10;
        data=j%10;
        max=maxof3(hun,oct,data);
        min=minof3(hun,oct,data);
        j=max-min;
        if(j==h)            //最后两次差相等时，差即为所求黑洞数
        {
            printf("%d\n", j);
            break;
        }
    }
}
//求三位数重排后的最大值
int maxof3(int a,int b,int c)
{
    int t;
    if(a<b)
    {
        t=a;a=b;b=t;
    }
    if(a<c)
    {
        t=a;a=c;c=t;
    }
    if(b<c)
    {
        t=b;b=c;c=t;
    }
    return(a*100+b*10+c);
}
//求三位数重排后的最小值
int minof3(int a,int b,int c)
{
    int t;
    if(a<b)
    {
        t=a;a=b;b=t;
    }
    if(a<c)
    {
        t=a;a=c;c=t;
    }
    if(b<c)
    {
        t=b;b=c;c=t;
    }
    return(c*100+b*10+a);
}
```

7、勾股数

求100以内的勾股数。
```c
#include<stdio.h>
#include<math.h>
main()
{
    int a,b,c,count=0;
    printf("100以内的勾股数有： \n");
    printf("  a   b   c    a   b   c    a   b   c    a   b   c\n");
    for(a=1;a<=100;a++)
        for(b=a+1;b<=100;b++)
        {
            c=(int)sqrt(a*a+b*b);   //求c值
            if(c*c==a*a+b*b&&a+b>c&&a+c>b&&b+c>a&&c<=100)
                                    //判断c的平方是否等于a*a+b*b
            {
                printf("%4d %4d %4d    ", a,b,c);
                count++;
                if(count%4==0)      //每输出4组解就换行
                    printf("\n");
            }
        }
        printf("\n");
}
```

继上一次整数篇都过去十多天了，说好的每天写代码并没有实现，看来是时候找个女朋友来监督了。（哈哈哈哈，在我大西电还想找女票！啪！）这次写的是三个小游戏。

1、人机猜数。

由计算机随机一个四位整数，让人猜是多少。人输入一个四位数后，计算机先判断其中有几位猜对了，并且对的数字中有几位位置也正确，将结果显示出来作为提示，知道猜对为止。
```c
#include<stdio.h>
#include<time.h>
#include<stdlib.h>
int main()
{
    int stime,a,z,t,i,c,m,g,s,j,k,l[4];
                    //变量j表示数字正确的位数，k表示位置正确的位数
    long ltime;
    ltime=time(NULL);
    stime=(unsigned int)ltime/2;
    srand(stime);
    if((rand()%10000)>=1000&&(rand()%10000)<=9999)
        z=rand()%10000;        //计算机给出一个随机数
    printf("机器输入四位数****\n");
    printf("\n");
    for(c=1;;c++)              //变量c为猜数次数计数器
    {
        printf("请输入你猜的四位数：");
        scanf("%d",&g);
        a=z;j=0;k=0;l[0]=l[1]=l[2]=l[3]=0;
        for(i=1;i<5;i++)
                //变量i表示原数中的第i位数。个位为第一位，千位为第四位
        {
            s=g;
            m=1;
            for(t=1;t<5;t++)
            {
                if(a%10==s%10)
                {
                     //若该位置上的数字尚未与其他数字相同
                     if(m&&t!=l[0]&&t!=l[1]&&t!=l[2]&&t!=l[3])
                     {
                         j++;
                         m=0;
                         l[j-1]=t;    //记录相同数字时，该数字所在猜数字中的位置
                     }
                     if(i==t) k++;    //若位置也相同，则计数器k加1
                }
                s/=10;
            }
            a/=10;
        }
        printf("你猜的结果是");
        printf("%dA%dB\n",j,k);
        if(k==4)
        {
             printf("****你赢了*****\n");
             printf("\n~~××××××××~~\n");
             break;
        }
    }
    printf("你总共猜了 %d 次\n",c);
}
```

2、搬山游戏。

设有 n 座山，计算机和人轮流搬山。规定每次搬山不能超过 k 座，谁搬最后一座谁输。游戏开始时，输入山总数和最大搬山数，然后人先开始，等人输入了需要搬走的山数后，计算机马上打印出它搬多少座山，并提示剩余多少，直到最后一座山搬完为止。显示谁是赢家并询问是否继续比赛。如果选择结束，便统计出玩了几局，双方胜负如何。
```c
#include<stdio.h>
main()
{
    int n,k,x,y,cc,pc,g;
    printf("More Mountain Game\n");
    printf("Game Begin\n");
    pc=cc=0;
    g=1;
    for(;;)
    {
        printf("No.%2d game \n",g++);
        printf("------------\n");
        printf("How many mountains are there?");
        scanf("%d",&n);             //输入山的总数
        if(!n)
            break;
        printf("How many mountains are allowed to each time?");
        do
        {
            scanf("%d",&k);         //输入运行的搬山数
            if(k>n||k<1)            //判断搬山数
                printf("Repeat again!\n");
        }while(k>n||k<1);
        do
        {
            printf("How many mountains do you wish move away?");
            scanf("%d",&x);
            if(x<1||x>k||x>n)       //判断搬山数是否符合要求
            {
                printf("IIIegal,again please!\n");
                continue;
            }
            n-=x;
            printf("There are %d mountains left now.\n",n);
            if(!n)
            {
                printf(".....I win. You are failure.....\n\n");
                cc++;
            }
            else
            {
                y=(n-1)%(k+1);      //求出最佳搬山数
                if(!y)
                    y=1;
                n-=y;
                printf("Copmputer move %d mountains away.\n",y);
                if(n)
                    printf("There are %d mountains left now.\n",n);
                else
                {
                    printf(".....I am failure. You win.....\n\n");
                    pc++;
                }
            }
        }while(n);
    }
    printf("Games in total have been played %d.\n",cc+pc);
    printf("You score is win %d,lose %d.\n",pc,cc);
    printf("My score is win %d,lose %d.\n",cc,pc);
}
```

3、自动发牌。

一副扑克牌有52张，分给4个人，设计一个程序自动发牌。
```c
#include<stdio.h>
#include<stdlib.h>
int comp(const void *j,const void *i);
void p(int b[],char n[]);
int main(void)
{
    static char n[]={'2','3','4','5','6','7','8','9','T','J','Q','K','A'};
    int a[53],b1[13],b2[13],b3[13],b4[13];
    int b11=0,b22=0,b33=0,b44=0,t=1,m,flag,i;
    while(t<=52)                        //控制发52张牌
    {
        m=rand()%52;                    //产生0~51之间的随机数
        for(flag=1,i=1;i<=t&&flag;i++)  //查找新产生的随机数是否已经存在
            if(m==a[i])
                flag=0;
        //flag=1表示产生的是新的随机数，flag=0表示新产生的随机数已经存在
            if(flag)
            {
                a[t++]=m;    //如果产生了新的随机数，则存入数组
                    //根据t的模值判断当前的牌应存入哪个数组中
                if(t%4==0)
                    b1[b11++]=a[t-1];
                else
                    if(t%4==1)
                        b2[b22++]=a[t-1];
                    else
                        if(t%4==2)
                            b3[b33++]=a[t-1];
                        else
                            if(t%4==3)
                                b4[b44++]=a[t-1];
            }
    }
    qsort(b1,13,sizeof(int),comp);    //将每个人的牌进行排序
    qsort(b2,13,sizeof(int),comp);
    qsort(b3,13,sizeof(int),comp);
    qsort(b4,13,sizeof(int),comp);
    p(b1,n);                          //分别打印每个人的牌
    p(b2,n);
    p(b3,n);
    p(b4,n);
    return 0;
}

void p(int b[],char n[])
{
    int i;
    printf("\n\006 ");      //打印黑桃标记
    for(i=0;i<13;i++)       //将数组中的值转换为相应的花色
        if(b[i]/13==0)      //找到该花色对应的牌
            printf("%c ",n[b[i]%13]);
        printf("\n\003 ");      //打印红桃标记
        for(i=0;i<13;i++)
            if((b[i]/13)==1)
                printf("%c ",n[b[i]%13]);
        printf("\n\004 ");      //打印方块标记
        for(i=0;i<13;i++)
            if(b[i]/13==2)
                printf("%c ",n[b[i]/13]);
        printf("\n\005 ");      //打印梅花标记
        for(i=0;i<13;i++)
            if(b[i]/13==3||b[i]/13==4)
                printf("%c ",n[b[i]%13]);
            printf("\n");
}

int comp(const void *j,const void *i)   //qsort调用的排序函数
{
     return(*(int*)i-*(int*)j);
}
```

放假前大概就写到这儿了，都是些巨基础巨简单的，接下来的日子必须滚去预习防挂科（一把辛酸泪啊）。魔力胡推荐的《C和C++安全编码》也还没看，真不知道这一学期都干了些什么。。。

1、汉诺塔问题

在古代有一个梵塔，塔内有A、B、C三个塔。开始时A座上有64个盘子，盘子大小不同，但保证大的在下小的在上。现在把盘子从A座移动到C座，每次只能移动一个盘子，且移动过程中在3个座上都必须保持大盘在下小盘在上。把移动步骤打印出来。
```c
#include<stdio.h>
void hanoi(int N,char A,char B,char C)
{
    if(N==1)                    //将A座上剩下的第N个盘子移动到C座上
        printf("move dish %d from %c to %c\n",N,A,C);   //打印移动步骤
    else
    {
        hanoi(N-1,A,C,B);       //借助C座将N-1个盘子从A座移动到B座
        printf("move dish %d from %c to %c\n",N,A,C);
        hanoi(N-1,B,A,C);       //借助A座将N-1个盘子从B座移动到C座
    }
}
main()
{
    int n;
    printf("Please input the number of dishes:");
    scanf("%d",&n);             //输入要移动的盘子个数
    printf("The steps to move %2d dishes are:\n",n);
    hanoi(n,'A','B','C');       //调用递归函数
}
```

2、一个农夫在河边带了一只狼、一只羊和一颗白菜，他需要把这三样东西带到河的对岸。然而，这艘船只能容下农夫本人和另外一样东西。如果农夫不在场的话，狼会吃掉羊，羊也会吃掉白菜。请问如何过河。
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#define N 15
int a[N][4];
int b[N];
char *name[]=
{
    "        ",
    "and wolf",
    "and goat",
    "and cabbage"
};

void search(int Step)
{
    int i;
    //若该种步骤能使各值均为1，则输出结果，进入回归步骤
    if(a[Step][0]+a[Step][1]+a[Step][2]+a[Step][3]==4)
    {
        for(i=0;i<=Step;i++)    //能够依次输出不同的方案
        {
            printf("east: ");
            if(a[i][0]==0)
                printf("wolf  ");
            if(a[i][1]==0)
                printf("goat  ");
            if(a[i][2]==0)
                printf("cabbage  ");
            if(a[i][3]==0)
                printf("farmer  ");
            if(a[i][0]&&a[i][1]&&a[i][2]&&a[i][3])
                printf("none");
            printf("           ");
            printf("west: ");
            if(a[i][0]==1)
                printf("wolf ");
            if(a[i][1]==1)
                printf("goat ");
            if(a[i][2]==1)
                printf("cabbage ");
            if(a[i][3]==1)
                printf("farmer ");
            if(!(a[i][0]||a[i][1]||a[i][2]||a[i][3]))
                printf("none");
            printf("\n\n\n");
            if(i<Step)
                printf("                 the %d time\n",i+1);
            if(i>0&&i<Step)
            {
                 if(a[i][3]==0)         //农夫在本岸
                 {
                     printf("            ----->  farmer ");
                     printf("%s\n",name[b[i]+1]);
                 }
                 else                   //农夫在对岸
                 {
                     printf("            <-----  farmer ");
                     printf("%s\n",name[b[i]+1]);
                 }
            }
        }
        printf("\n\n\n\n");
        return;
    }
    for(i=0;i<Step;i++)
    {
         if(memcmp(a[i],a[Step],16)==0)     //若该步与以前步骤相同，取消操作
         {
             return;
         }
    }
    //若羊和农夫不在一起而狼和羊或者羊和白菜在一起，则取消操作
    if(a[Step][1]!=a[Step][3]&&(a[Step][2]==a[Step][1]||a[Step][0]==a[Step][1]))
    {
         return;
    }
    //递归，从带第一种对象开始依次向下循环，同时限定递归的界限
    for(i=-1;i<=2;i++)
    {
        b[Step]=i;
        memcpy(a[Step+1],a[Step],16);       //复制上一步状态，进行下一步移动
        a[Step+1][3]=1-a[Step+1][3];        //农夫过去或者回来
        if(i==-1)
        {
            search(Step+1);                //进行第一步
        }
        else
            if(a[Step][i]==a[Step][3])      //若该物与农夫同岸，带回
            {
                a[Step+1][i]==a[Step+1][3]; //带回该物
                search(Step+1);             //进行下一步
            }
    }
}

int main()
{
     printf("\n\n            解决方案如下： \n\n\n");
     search(0);
     return 0;
}
```

3、在屏幕上打印杨辉三角。
```c
#include<stdio.h>
int c(int x,int y)
{
    int z;
    if(y==1||y==x)
        return 1;
    else
    {
        z=c(x-1,y-1)+c(x-1,y);
        return z;
    }
}
main()
{
    int i,j,n;
    printf("请输入杨辉三角的行数： ");
    scanf("%d",&n);
    for(i=1;i<=n;i++)           //输出n行
    {
        for(j=0;j<=n-i;j++)
            printf("  ");
        for(j=1;j<=i;j++)
            printf("%4d",c(i,j));   //调用递归函数，输出第i行第j个值
        printf("\n");
    }
}
```

4、双色球

模拟双色球开奖过程。要求：

(1) 每期开出的红色球号码不能重复，但蓝色球可以是红色球中的一个。

(2) 红色球的范围是1~33，蓝色球的范围是1～16。
```c
#include<stdio.h>
#include<stdlib.h>
#include<time.h>
main()
{
    int red[6];
    int blue;
    int i,j;
    int tmp;
    srand((unsigned)time(NULL));        //播种子
    i=0;
    while(i<6)
    {
        tmp=(int)((1.0*rand()/RAND_MAX)*33+1);
        for(j=0;j<i;j++)                //判读是否重复
            if(red[j]==tmp)
                break;
                if(j==i)
                {
                    red[i]=tmp;         //将生成的号码保存在red数组中
                    i++;
    blue=(int)((1.0*rand()/RAND_MAX)*16+1);
    printf("红色球： ");
    for(i=0;i<6;i++)
        printf("%d ",red[i]);
    printf("蓝色球： %d\n",blue);
}
```

5、约瑟夫环

15个教徒和15个非教徒在海上遇险，必须将一半的人扔下海。于是他们30人围成圆圈，从第1个人开始依次报数，每数到第9个人就把他扔下海，如此循环进行直到仅剩15人为止。问怎样排法，才能使扔下海的都是非教徒。
```c
#include<stdio.h>
struct node
{
    int flag;
    int next;
}array[31];
int main()
{
    int i,j,k;
    printf("最终结果为(s:被扔下海,b:在船上)：\n");
    for(i=1;i<31;i++)       //初始化结构数组
    {
        array[i].flag=1;    //flag标志置为1，表示人在船上
        array[i].next=i+1;  //next值为数组中下一个元素的下标，即指向下一个人
    }
    array[30].next=1;       //第30人的指针指向第一个人以构成环
    j=30;               //变量j指向处理完毕的数组元素，从array[i]指向的人开始计数
    for(i=0;i<15;i++)       //变量i作为计数器，记录已扔下海的人数
    {
        for(k=0;;)          //变量k作为计数器，决定哪个人被扔下海
            if(k<9)
            {
                j=array[j].next;    //修改指针，取下一个人
                k+=array[j].flag;   //计数，已扔下海的人标记为0
            }
        else break;                 //计数到9则停止
        array[j].flag=0;            //已扔下海的人标记为0
    }
    for(i=1;i<31;i++)
        printf("%c",array[i].flag? 'b':'s');
    printf("\n");
}
```

6、求出符合要求的素数

编程实现将大于某个整数n且紧靠n的k个素数存入某个数组中，同时实现从infile.txt文件中读取10对n和k值，分别求出符合要求的素数，并将结果保存到outfile.txt文件中。
```c
#include<stdio.h>
#include<stdio.h>
int isPrime(int n)
{
    int i;
    for(i=2;i<n;i++)
        if(n%i==0)
            return 0;               //n不是素数
    return 1;                       //n是素数
}
void num(int n,int k,int array[])
{
    int i=0;
    for(n=n+1;k>0;n++)
        if(isPrime(n))              //调用函数isPrime()判断n是否是素数
        {
            array[i++]=n;           //若n是素数，则将n存入数组array中
            k--;
        }
}
void filedata()
{
    int n,k,array[1000],i;
    FILE *rf,*wf;
    rf=fopen("infile.txt","r");
    wf=fopen("outfile.txt","w");
        //从infile.txt文件中读取10对(n,k)值
    for(i=0;i<10;i++)
    {
        fscanf(rf,"%d%d",&n,&k);        //读文件
        num(n,k,array);                 //调用num()函数
        for(n=0;n<k;n++)
            fprintf(wf,"%d ",array[n]); //写文件
        fprintf(wf,"\n");
    }
    fclose(rf);
    fclose(wf);
}
void main()
{
     int n,k,array[1000];
     system("cls");
     printf("输入整数n和k： ");
     scanf("%d%d",&n,&k);
     num(n,k,array);
     for(n=0;n<k;n++)
         printf("%d ",array[n]);
     printf("\n");
     filedata();
     return;
}
```
