---
layout:     post
title:      "AtCoder Beginner Contest 189 题解" #标题
subtitle:   一些奇怪的细节 #副标题
date:       2021-01-24 #日期
author:     Lin_Xuan
header-img: img/post-Note-Head-002.jpg #图片
catalog: true
tags:
   - ACM
   - 题解
   - AtCoder
---

[传送门](https://atcoder.jp/contests/abc189)

## [A：Slot](https://atcoder.jp/contests/abc189/tasks/abc189_a)

&emsp;&emsp;签到题

```c++
//核心
    string s;
    while(cin>>s)
    {
        if(s[0]==s[1] and s[1]==s[2]) cout<<"Won"<<endl;
        else cout<<"Lost"<<endl;
    }
```

## [B：Alcoholic](https://atcoder.jp/contests/abc189/tasks/abc189_b)

&emsp;&emsp;签到题，模拟即可。

&emsp;&emsp;但是有坑，要注意不能使用多组输入的模式，否则会WA(*鬼知道为什么*)

```c++
#include<bits/stdc++.h>

using namespace std;
using LL=long long;
using ULL=unsigned long long;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;

int main()
{
    LL n,x;
    // while(cin>>n>>x)
    // CNM 多组输入会WA！！
    cin>>n>>x;
    {
        x*=100ll;
        int ans=-1;
        LL v,p;
        for(int i=1;i<=n;i++)
        {
            cin>>v>>p;
            x-=v*p;
            if(x<0 and ans==-1){
                ans=i;
                break;
            }
        }
        cout<<ans<<endl;
    }
    return 0;
}

```



## [C：Mandarin Orange](https://atcoder.jp/contests/abc189/tasks/abc189_c)

### 解题思路：

&emsp;&emsp;暴力即可……原来$1E5$是可以暴力过的啊，才300ms。

&emsp;&emsp;一共1E4个数，尝试每个数当作选择的x，然后$O(n)$的遍历去求在该x下的最大区间，算出橘子的数量即可，最后去最大值。还可以用vis数组标记，优化一下。

### 代码：

```c++
#include<bits/stdc++.h>

using namespace std;
using LL=long long;
LL get(int x,int n,vector<int>& a)
{
        int cnt=0;
        int maxn=0;
        for(int i=1;i<=n;i++)
        {
            if(a[i]>=x) cnt++;
            else
            {
                maxn=max(cnt,maxn);
                cnt=0;
            }
        }
        maxn=max(maxn,cnt);
        return maxn*x;

}
int main()
{
    // try n^2
    // 果然暴力出奇迹
    int n;
    cin>>n;
    vector<int> a(n+1);
    vector<int> vis(1e5+5);
    for(int i=1;i<=n;i++) cin>>a[i];
    
    LL ans=0;
    for(int i=1;i<=n;i++)
    {
        if(vis[a[i]]==true) continue;
        else ans=max(ans,get(a[i],n,a));
        vis[a[i]]=true;
    }
    cout<<ans<<endl;
    return 0;
}

```



## [D：Logical Expression](https://atcoder.jp/contests/abc189/tasks/abc189_d)

### 解题思路：

&emsp;&emsp;可以用一维DP求解。设`a[i]`为前`i`个运算符和`i+1`个数字整体结果为`true`的可能个数。对于前`i-1`和前`i`个数字共有$2^{i}$种数字组合，如果结果为`true`的可能有`a[i-1]`个，则：若第`i`个操作是`AND`，则后面的数字只能填`true`才能保持整体为`true`，更新`a[i]=a[i-1]`;如果操作为`OR`，则数字取`true`，前面任意取(共`2<<i`种可能)或者数字去`false`，前面必须取`true`，两种情况加起来，更新`a[i]=a[i-1]+(1<<i)`;

### 代码：

&emsp;&emsp;由于`a[i]`只会使用1次，因此使用一个数字就可以了。

```c++
#include<bits/stdc++.h>

using namespace std;
using LL=long long;
int main()
{
    LL cnt=1;//第i为为真的可能性
    int n;
    cin>>n;
    char s[10];
    for(int t=1;t<=n;t++)
    {
        cin>>s;
        if(s[0]=='O')//或操作
            cnt+=(1ll<<(t));
    }
    cout<<cnt<<endl;
    return 0;
}

```



## [E：Rotate and Flip](https://atcoder.jp/contests/abc189/tasks/abc189_e)

~~已补题~~

### 解题思路：

&emsp;&emsp;运用线性代数的坐标变换。每次变换可以看作右乘(左乘)一个矩阵。由方阵相乘的可结合性，可以计算右乘的矩阵，最后统一相乘即可。四种操作类型可以等价于右乘下面的四个矩阵。(由于需要引入一个常数P，把二位的坐标扩充成三维)
$$
\begin{bmatrix}
   0 &  -1 &  0 \\
   1 &  0 &  0 \\
   0 &  0 &  1
 \end{bmatrix} \tag{1}
$$
$$
\begin{bmatrix}
   0 &  1 &  0 \\
   -1 &  0 &  0 \\
   0 &  0 &  1
 \end{bmatrix} \tag{2}
$$
$$
\begin{bmatrix}
   -1 &  0 &  0 \\
   0 &  1 &  0 \\
   2p &  0 &  1
 \end{bmatrix} \tag{3}
$$
$$
\begin{bmatrix}
   1 &  0 &  0 \\
   0 &  -1 &  0 \\
   0 &  2p &  1
 \end{bmatrix} \tag{4}
$$

### 代码：

```c++
#include<bits/stdc++.h>

using namespace std;
using LL=long long;
using ULL=unsigned long long;
using P=pair<LL,LL>;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;


const vector<vector<LL>> multi(const vector<vector<LL>>&a,const vector<vector<LL>>&b,const LL n=3)//矩阵相乘
{
    vector<vector<LL>> ret(n,vector<LL>(n,0));
    // k,i,j,最优locality~
    for(LL k=0;k<n;k++)
        for(LL i=0;i<n;i++)
        {
            LL tmp=a[i][k];
            for(LL j=0;j<n;j++)
                ret[i][j]+=tmp*b[k][j];
        }
    return ret;
};
void prLL(vector<LL> dot,vector<vector<LL>> Q){
    LL x=dot[0]*Q[0][0]+dot[1]*Q[1][0]+dot[2]*Q[2][0];
    LL y=dot[0]*Q[0][1]+dot[1]*Q[1][1]+dot[2]*Q[2][1];
    cout<<x<<" "<<y<<endl;
}
int main()
{
    LL n;
    cin>>n;
    vector<vector<LL> > a(n+1,vector<LL>(3,1));
    for(LL i=1;i<=n;i++){
        cin>>a[i][0]>>a[i][1];
    }

    /*q次操作*/
    LL q;
    cin>>q;
    //初始化方阵 
    vector< vector<vector<LL>> > Q(q+1,vector<vector<LL>>(3,vector<LL>(3,0)));//累计的右乘
    vector<vector<LL>> ope1(3,vector<LL>(3,0)),ope2(3,vector<LL>(3,0)),\
    ope3(3,vector<LL>(3,0)),ope4(3,vector<LL>(3,0));//四种操作
    Q[0][0][0]=Q[0][1][1]=Q[0][2][2]=1;
    ope1[1][0]=ope1[2][2]=1,ope1[0][1]=-1;//顺时针
    ope2[0][1]=ope2[2][2]=1,ope2[1][0]=-1;//逆时针

    ope3[1][1]=ope3[2][2]=1,ope3[0][0]=-1;//x=p对称
    ope4[0][0]=ope4[2][2]=1,ope4[1][1]=-1;//y=p对称
    for(LL i=1;i<=q;i++)
    {
        LL lab,p;
        cin>>lab;
        if(lab==1){
            Q[i]=multi(Q[i-1],ope1);
        }
        else if(lab==2){
            Q[i]=multi(Q[i-1],ope2);
        }
        else if(lab==3){
            cin>>p;
            ope3[2][0]=2*p;
            Q[i]=multi(Q[i-1],ope3);
        }
        else if(lab==4){
            cin>>p;
            ope4[2][1]=2*p;
            Q[i]=multi(Q[i-1],ope4);
        }
    }

    /*m 次 询问*/
    LL m;
    cin>>m;
    while(m--){
        LL A,B;
        cin>>A>>B;
        prLL(a[B],Q[A]);
    }
    return 0;
}

```

