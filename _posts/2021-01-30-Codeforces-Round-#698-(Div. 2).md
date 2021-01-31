---
layout:     post
title:      Codeforces Round \#698 (Div. 2) 题解 #标题
subtitle:   有点挑战的感觉真好 #副标题
date:       2021-01-30 #日期
author:     Lin_Xuan
header-img: img/post-Note-Head-河灯-Lifeline.jpg #图片
catalog: true
tags:
   - ACM
   - 题解
   - Codeforces
---

&emsp;&emsp;传送门：[Codeforces Round #698 (Div. 2)](https://codeforces.com/contest/1478) 

&emsp;&emsp;CF有史以来的最好成绩！！开森

&emsp;&emsp;鸽了三天才写题解……桑心

&emsp;&emsp;咕掉了D题……好累了\~ 

## [A: Nezzar and Colorful Balls](https://codeforces.com/contest/1478/problem/A) 

&emsp;&emsp;签到,老CF特色了，花里胡哨一打碎，最后意思是数据重复出现的最大次数……

```c++
    int t;
    cin>>t;
    while(t--)
    {
        int n;cin>>n;
        vector<int> a(n+1);
        for(int i=1;i<=n;i++){
            int x;cin>>x;
            a[x]++;
        }
        cout<<*max_element(a.begin(),a.end())<<endl;
    }
```

## [B: Nezzar and Lucky Number](https://codeforces.com/contest/1478/problem/B) 

### 解题方案：

&emsp;&emsp;当时卡了好久代码老长了，赛后优化了一下代码后发现思路其实很简单……

&emsp;&emsp;规定十进制数里有d出现的数字为幸运数，要求验证n能否有任意个幸运数累加得到。

&emsp;&emsp;分两种情况，一种是$n<10*d$。那样的话组成n的幸运数十进制的第二位就不可能是d，只能考虑构成10+d,20+d……。求出n%d和n/d。重点是用n%d和其他数组成幸运数(剩下的以单个的d存在即可)。不断尝试n%d+d,直到用完所有的d，判断是否能得到个位为d的数。不能的话答案极为"No"。

&emsp;&emsp;当$n>=10\times d$是，一定可以由幸运数累加得到。让我们把n分成a1+a2。其中a1的十位是d，个位是0，高位与n一致。a2为n-a1。那么，n可以由a1+a2%d和a2/d个d相加得到。这些数都是幸运数。

### 代码：

```c++
#include<bits/stdc++.h>

using namespace std;
using LL=long long;
using ULL=unsigned long long;
using LD=long double;
using P=pair<int,bool>;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;

int main()
{
    int t;
    cin>>t;
    while(t--)
    {
        int q,d;
        cin>>q>>d;
        for(int i=1;i<=q;i++)
        {
            int val; cin>>val;
            if(val>=d*10 or val%d==0){
                cout<<"Yes"<<endl;
                continue;
            }
            int cnt=val/d;
            int target=val%d;
            bool flag=false;
            for(int i=1;i<=cnt;i++)
            {
                if(i*d%10+target==d){
                    flag=true;
                    break;
                }    
            }
            if(flag) cout<<"Yes"<<endl;
            else cout<<"No"<<endl;
        }

    }
    return 0;
}

```



## [C: Nezzar and Symmetric Array](https://codeforces.com/contest/1478/problem/C)

### 解题方案：

&emsp;&emsp;当时做出这道题真是意料之外了，当时B卡了一会……C差点没看，最后5分钟A的。

&emsp;&emsp;数组A中的数互不相同，且每个数的相反数都在A中(即为n个不同的数及其相反数)。数组D由数组A构造而成。而构造的规则是验证是否可以构造数组A使得D成立。$di=∑^{2n}_{j=1}|a_i-a_j|$。我们发现，只考虑A中的n个正数(负数的di与相反数的整数相同)，$di=∑(a_i>a_j?2a_i:2a_j)$。这样，将n个整数从小到大排序，对应的D也是从小到大排序。

&emsp;&emsp;这样可以得到一个递推公式：`d[i]=d[i-1]-2*(i-1)*a[i-1]+2*(i-1)*a[i]`。根据递归公式，设a[1]为份，就可以得到所有的数的和sum份。对于最小的数d[1]等于数据总和的2倍(后续讨论只考虑了正数)。验证是否成立d[1]等等分成2*sum份即可。

&emsp;&emsp;有一些需要注意的细节。A中数据不能有0(因为0的相反数仍为0，但是A中数据不能相同)。同时每次地推a[i]增加的数必须是整数，否则将出现小数，不合理。

&emsp;&emsp;具体细节看代码注释。

### 代码

```c++
#include<bits/stdc++.h>

using namespace std;
using LL=long long;
using ULL=unsigned long long;
using LD=long double;
using P=pair<double,double>;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;

// |a[i]-a[j]|+|a[i]+a[j]|==a[i]>a[j]?2*a[i]:2*|a[j]|;
int main()
{
    int t;
    cin>>t;
    while(t--)
    {
        int n; cin>>n;
        vector<LL> d(2*n+1);
        bool flag=true;
        for(int i=1;i<=n*2;i++){
            cin>>d[i];
        }
        sort(d.begin()+1,d.end());
        for(int i=1;i<=2*n;i+=2)
            if(d[i]!=d[i+1]){//每个数必须出现偶数次(d[a]==d[-a])
                flag=false;
                break;
            }
        if(flag==false or d[1]<2*n or d[1]%2==1){//d[1]=2*sum,必须大于2*n且为偶数
            cout<<"No"<<endl;
            continue;
        }
        
        // d[i]==d[i-1]-2*(i-1)*a[i-1]+2*(i-1)*a[i]
        flag=true;;
        vector<LL> a(n+1);
        LL sum_dd=0;
        for(int i=2;i<=n;i++)
        {
            if( (d[2*i-1]-d[2*i-3])%(i*2-2)!=0 or (d[2*i-1]-d[2*i-3])/(i*2-2)==0){//必须正好分配，且不为0(每次必须增加)
                // 不是整数or相等
                flag=false;
                break;
            }
            a[i]=a[i-1]+(d[2*i-1]-d[2*i-3])/(i*2-2);
            sum_dd+=a[i];
        }
        if(flag==true)
        {
            LL sum=d[1]/2-sum_dd;
            if(sum>0 and sum%n==0) cout<<"Yes"<<endl;//a[1]!=0
            else cout<<"No"<<endl;
        }
        else cout<<"No"<<endl;
    }
    return 0;
}

```

