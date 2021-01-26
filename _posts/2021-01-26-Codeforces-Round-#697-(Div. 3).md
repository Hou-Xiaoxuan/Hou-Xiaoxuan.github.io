---
layout:     post
title:      "Codeforces Round #697 (Div. 3) 题解" #标题
subtitle:   "艰难的比赛" #副标题
date:       2021-01-26 #日期
author:     Lin_Xuan
header-img: img/post-Note-Head-河灯-Lifeline.jpg #图片
catalog: true
tags:
   - ACM
   - 题解
   - Codeforces
---



&emsp;&emsp;传送门：[Codeforces Round #697 (Div. 3)](https://codeforces.com/blog/entry/87161) 

&emsp;&emsp;经历了两次延迟，一次评测机炸掉，最终un-rating，这场比赛太难了……虽然发挥的不错，但是不排名没办法了……

## [A: Odd Divisor](https://codeforces.com/contest/1475/problem/A)

&emsp;&emsp;题目要求判断一个数x有没有奇数因子。如果数字是奇数，那么自己本身就是奇数因子。如果偶数，那就不断除2，直到变成奇数。最后剩余的数字大于1的话，这个数就可以当作原数x的奇数因子。否则就没有奇数因子。

### 核心代码：

```c++
    int t;  cin>>t;
    while(t--)
    {
        LL x;
        cin>>x;
        bool flag=false;
        while(x>1 and x%2==0)
            x>>=1;

        if(x>1) flag=true;
        if(flag==true) cout<<"Yes"<<endl;
        else cout<<"No"<<endl;
    }
```

## [B: New Year's Number](https://codeforces.com/contest/1475/problem/B)

&emsp;&emsp;判断一个数x能否由若干个2020和2021组成，

&emsp;&emsp;x应该介于k\*2020\~k\*2021之间。直接判断x/2020和x%2020的大小即可

### 核心代码：

```c++
    int t;  cin>>t;
    while(t--)
    {
        LL x;
        cin>>x;
        if(x/2020>=x%2020) cout<<"Yes"<<endl;
        else cout<<"No"<<endl;
    }
```

## [C: Ball in Berland](https://codeforces.com/contest/1475/problem/C)

### 题目大意：

&emsp;&emsp;有a个男生、b个女生。现在又k种配对情况，从中选择两对(不能有重复的人)，问有几种选择情况。

### 解题方法：

&emsp;&emsp;使用容斥原理。先认为两对的选择是有序的(后续答案除以2即可)。枚举第一队选择，容斥地计算出冲突的情况，让总情况数目减去冲突的情况数目即可。

&emsp;&emsp;开始时记录每个男生配对的女生的数目`num_a[i]`和每个女生配对的男生的数目`num_b[i]`。当第一队选择2-4时，冲突的数目为男生2配对的女生数目+女生4配对的男生数目，2和4的配对个数计算了两次，但是自己和本身的冲入时不用计算的，因此减去2。$num\_a[2]+num\_b[4]-2$。配对总数是$k\times{(k-1)}$。

&emsp;&emsp;直接累加计算即可，最后记得除以2。

### 代码：

```c++
#include<bits/stdc++.h>

using namespace std;
using LL=long long;
using ULL=unsigned long long;
using LD=long double;
using P=pair<int,int>;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;

int main()
{
    int t;
    cin>>t;
    while(t--)
    {
        int a,b,k;
        cin>>a>>b>>k;
        vector<P> pairs(k+1);
        for(int i=1;i<=k;i++)	cin>>pairs[i].x;
        for(int i=1;i<=k;i++)	cin>>pairs[i].y;
        vector<int> num_a(a+1),num_b(b+1);
        for(int i=1;i<=k;i++){
            num_a[pairs[i].x]++;
            num_b[pairs[i].y]++;
        }
        LL ans=LL(k)*(k-1);
        for(int i=1;i<=k;i++){
            LL tmp=num_a[pairs[i].x]+num_b[pairs[i].y]-2;
            ans-=tmp;;
        }
        ans/=2;
        cout<<ans<<endl;
    }
    return 0;
}

```



## [D: Cleaning the Phone](https://codeforces.com/contest/1475/problem/D)

~~已补题~~ 

### 题目大意：

&emsp;&emsp;现在又n个手机程序，每个程序对应有内存大小a[i]和对应的舒适度b[i]\(2或1)。现在选择若干个程序删除，使得释放的内存大小大于m，输出最小的舒适度减少量。

### 接替方案：

&emsp;&emsp;用两个数组分别储存舒适度为1和舒适度为2的程序的内存大小`cnt[1]和cnt[2]`，从大到小排序，并求前缀和。现在只要确定了cnt[1]中选择程序的个数，就能二分地求出cnt[2]中使用的程序的个数。枚举求最小值即可。

&emsp;&emsp;注意，前缀和大小会宝int，要使用long long。

### 代码：

```c++
#include<bits/stdc++.h>

using namespace std;
using LL=long long;
using ULL=unsigned long long;
using LD=long double;
using P=pair<int,int>;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;
// 补题
int main()
{
    int t;cin>>t;
    while(t--)
    {
        int n,m;
        cin>>n>>m;
        vector<LL> a(n+1);
        for(int i=1;i<=n;i++)   cin>>a[i];
        vector<LL> cnt[3];
        cnt[2].push_back(0);
        cnt[1].push_back(0);
        for(int i=1;i<=n;i++){
            int b;  cin>>b;
            cnt[b].push_back(a[i]);
        }

        sort(cnt[1].begin()+1,cnt[1].end(),greater<int>());
        sort(cnt[2].begin()+1,cnt[2].end(),greater<int>());
        for(size_t i=1;i<cnt[1].size();i++) cnt[1][i]+=cnt[1][i-1];
        for(size_t i=1;i<cnt[2].size();i++) cnt[2][i]+=cnt[2][i-1];

        LL ans=LLinf;
        for(size_t i=0;i<cnt[1].size();i++)//枚举1中取的个数
        {
            size_t k=lower_bound(cnt[2].begin(),cnt[2].end(),m-cnt[1][i])-cnt[2].begin();
            if(k==cnt[2].size()) continue;//没找到
            ans=min(ans,LL(i+k*2));
        }
        // 特殊考虑1中一个都不取
        cout<<(ans==LLinf?-1:ans)<<endl;
    }
    return 0;
}

```



## [E: Advertising Agency](https://codeforces.com/contest/1475/problem/E)

### 题目大意：

&emsp;&emsp;在n个数里面选择k个数，使得总和最大。问有几种选择方式(数值一样的数当作一样的)

### 解题方案：

&emsp;&emsp;综合最大，则选择的数值的大小是已经确定的(从大到小前k个)。但是第k个数数值如果有多个的话，就有组合数中选择情况。因此计算除前k个数中a[k]的个数m和n个数总a[k]的个数m，求组合数C(n,m)即可。组合数的计算方法多样。这里选择求逆元的方法。

### 代码：



```c++
#include<bits/stdc++.h>

using namespace std;
using LL=long long;
using ULL=unsigned long long;
using LD=long double;
using P=pair<int,int>;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;
LL qpow(LL a,LL b,LL m){
   // 快速幂板子
    LL ans=1;
    LL k=a;
    while(b){
        if(b&1)ans=ans*k%m;
        k=k*k%m;
        b>>=1;
    }
    return ans;
}
int main()
{
    // 求阶乘及其逆元
    const int M=1005;
    const int mod=1e9+7;
    vector<LL> fac(M+1);
    vector<LL> inv(M+1);
    fac[0]=inv[0]=1;
    for(int i=1;i<=M;i++)
        fac[i]=fac[i-1]*i%mod;
    inv[1]=1;
    for(int i=1;i<=M;i++)
        inv[i]=qpow(fac[i],mod-2,mod);
    function<LL(LL,LL)> C=[&](LL n,LL m)->LL{
        return fac[n]*inv[m]%mod*inv[n-m]%mod;
    };
    
    int t;
    cin>>t;
    while(t--)
    {
        int n,k;
        cin>>n>>k;
        vector<int> a(n+1);
        for(int i=1;i<=n;i++) cin>>a[i];
        sort(a.begin()+1,a.end(),greater<int>());

        int val=a[k];
        int cnt[2]={0};
        for(int i=1;i<=n;i++){
            if(a[i]==val) cnt[1]++;
            if(a[i]==val and i<=k) cnt[0]++;
        }
        cout<<C(cnt[1],cnt[0])%mod<<endl;
    }
    return 0;
}

```

