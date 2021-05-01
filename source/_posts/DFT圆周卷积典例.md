---
title: DFT圆周卷积典例
date: 2019-09-27 12:01:05
tags: [原创,数字信号处理]
declare: true
---

## 题目

已知x(n)={1,0,2,1,3}，求x(n)&lowast;x(n)，x(n)⑤x(n)，x(n)⑩x(n)

<!--more-->

## 解答过程

### （1）x(n)&lowast;x(n)

列表：

![]( https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/%E5%BA%8F%E5%88%971.png)

**把划线中间的区域各自加起来**

得到：
$$
y(n)=
\left[ \begin{matrix} 1 \\ 0 \\ 4 \\ 2 \\ 10 \\ 4 \\ 13 \\ 6 \\ 9 \end{matrix} \right]
$$


待续...

## C语言实现


### 线性卷积序列

~~~c++
void conv(int x[],int y[]){
    int n,s=0;
    for(n=1;n<=9;n++){
        y[n]=0;
        for(int m=0;m<=n-1;m++){
            s=n-m-1;
            if(m<0||m>=5){
               y[n]+=0;}
            else if(s<0||s>=5){
                y[n]+=0;}
            else
                y[n]+=x[m]*x[s];
        }
    }
}
~~~

---

### 5点圆周卷积

~~~c++
void R_5(int x[],int y[]){
    int t=0;
    for(int n=0;n<5;n++){
        y[n]=0;
        for(int m=0;m<5;m++){
            t=(n-m);
            if(t<0){t=t+5;}
            y[n]+=x[t]*x[m];
        }
    }   
}
~~~

---

### 10点圆周卷积

~~~c++
void R_10(int x[],int y[]){
    int t=0;
    for(int n=0;n<10;n++){
        y[n]=0;
        for(int m=0;m<10;m++){
            t=(n-m);
            if(t<0){t=t+10;}
            y[n]+=x[t]*x[m];
        }
    }   
}
~~~

---

### 运行结果

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/DS/dft.png)

---

### 完整代码(实际用的是C++)

~~~c++
#include<iostream>
using namespace std;

/*线性卷积*/
void conv(int x[],int y[]){
    int n,s=0;
    for(n=1;n<=9;n++){
        y[n]=0;
        for(int m=0;m<=n-1;m++){
            s=n-m-1;
            if(m<0||m>=5){
               y[n]+=0;}
            else if(s<0||s>=5){
                y[n]+=0;}
            else
                y[n]+=x[m]*x[s];
        }
    }
}

/*5点圆周卷积*/
void R_5(int x[],int y[]){
    int t=0;
    for(int n=0;n<5;n++){
        y[n]=0;
        for(int m=0;m<5;m++){
            t=(n-m);
            if(t<0){t=t+5;}
            y[n]+=x[t]*x[m];
        }
    }   
}

/*10点圆周卷积*/
void R_10(int x[],int y[]){
    int t=0;
    for(int n=0;n<10;n++){
        y[n]=0;
        for(int m=0;m<10;m++){
            t=(n-m);
            if(t<0){t=t+10;}
            y[n]+=x[t]*x[m];
        }
    }   
}

int main(int argc, const char** argv) {
    int x[10]={1,0,2,1,3};
    int y_conv[10]={0},y_5[10]={0},y_10[10];
    int y_conv_temp[15]={0};
    conv(x,y_conv_temp);
    /* 
    for(int i=1;i<=9;i++){
        cout<<"y_conv["<<i<<"]="<<y_conv[i]<<endl;
    } 
    */
   for(int k=0;k<10;k++){
       y_conv[k]=y_conv_temp[k+1];
   }
    R_5(x,y_5);
    /*
    for(int i=0;i<=4;i++){
        cout<<"y_5["<<i<<"]="<<y_5[i]<<endl;
    }
    */
    R_10(x,y_10);
    /*
    for(int i=0;i<=9;i++){
        cout<<"y_10["<<i+1<<"]="<<y_10[i]<<endl;
    }
    */  

   for(int i=0;i<=9;i++){
        cout<<"y_conv["<<i+1<<"]="<<y_conv[i]<<"    "<<"y_5["<<i+1<<"]="<<y_5[i]<<"    "<<"y_10["<<i+1<<"]="<<y_10[i]<<endl;
    }
    return 0;
}

~~~



