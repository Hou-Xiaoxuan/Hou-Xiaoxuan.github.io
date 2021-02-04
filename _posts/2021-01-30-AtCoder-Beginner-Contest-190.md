---
layout:     post
title:      AtCoder Beginner Contest 190 题解 #标题
subtitle:   很舒服 #副标题
date:       2021-01-30 #日期
author:     Lin_Xuan
header-img: img/post-Note-Head-002.jpg #图片
catalog: true
tags:
   - ACM
   - AtCoder
   - 题解
---

传送门: [AtCoder Beginner Contest 190](https://atcoder.jp/contests/abc190) 

5道题……没有AK过的菜鸡想AK啊！！



## [A:Very Very Primitive Game](https://atcoder.jp/contests/abc190/tasks/abc190_a) 

很简单的逻辑判断

```c++
    int a,b,c;
    cin>>a>>b>>c;
    if(a>b) cout<<"Takahashi"<<endl;
    else if(a<b) cout<<"Aoki"<<endl;
    else cout<<(c==0?"Aoki":"Takahashi")<<endl;
```

## [B:Magic 3](https://atcoder.jp/contests/abc190/tasks/abc190_b) 

签到\~

```c++
    int n,s,d;
    cin>>n>>s>>d;
    bool flag=false;
    for(int i=1;i<=n;i++)
    {
        int a,b;
        cin>>a>>b;
        if(a<s and b>d) flag=true;
    }
    if(flag) cout<<"Yes"<<endl;
    else cout<<"No"<<endl;
```

## [C:Bowls and Dishes](https://atcoder.jp/contests/abc190/tasks/abc190_c) 

二选一，数据范围16，直接暴力，dfs枚举$2^{16}$中情况即可。

```c++
#include<bits/stdc++.h>

#define IOS ios::sync_with_stdio(false); cin.tie(0);cout.tie(0);

#define x first

#define y second 

using namespace std;
using LL=long long;
using ULL=unsigned long long;
using LD=long double;
using P=pair<int,int>;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;

int n,m,k;
vector<P> condition,opration;
vector<int> vis;
int dfs(int i)
{
    if(i>k)
    {
        int cnt=0;
        for(int i=1;i<=m;i++)
            if(vis[condition[i].x]>0 and vis[condition[i].y]>0) cnt++;
        return cnt;
    }
    vis[opration[i].x]++;
    int ans1=dfs(i+1);
    vis[opration[i].x]--;
    vis[opration[i].y]++;
    int ans2=dfs(i+1);
    vis[opration[i].y]--;
    return max(ans1,ans2);
}
int main()
{
    cin>>n>>m;
    condition.assign(m+1,P(0,0));
    vis.assign(n+1,0);
    for(int i=1;i<=m;i++)
        cin>>condition[i].x>>condition[i].y;
    cin>>k;
    opration.assign(k+1,P(0,0));
    for(int i=1;i<=k;i++)
        cin>>opration[i].x>>opration[i].y;
    int ans=dfs(1);
    cout<<ans<<endl;
    return 0;
}

```



## [D:Staircase Sequences](https://atcoder.jp/contests/abc190/tasks/abc190_d) 

### 解题方法：

差为1的等差数列的和为n，则设等差数列的项数为k项，中间值为m。(*中间值为奇数项的等差中项，偶数项两个等差中项的平均值*)，则$k\times m=n$ 。如果m为分数，则只能为$\frac{1}{2}$的倍数(*差为1的两个整数的平均数*)。令M=2m必为整数。$k\times M=2n$，$k=\frac{2n}{M}$必须为整数，M，k是2n的因数。同时注意，M为偶数的时候说明m是整数，则k必须为奇数(奇数项m为整数)。如果M为奇数，则说明m为分数，一定是偶数项。

枚举2n的因数，判断即可。

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
// AC
int main()
{
    LL n;
    cin>>n;
    LL m=n<<1;
    LL cnt=0;
    for(int i=1;i<=sqrt(m);i++)
    {
        if(m%i==0)
        {
            LL tmp=i;
            if((tmp&1)==1 and ((m/tmp)&1)==0) cnt++;
            else if((tmp&1)==0 and ((m/tmp)&1)==1) cnt++;
            tmp=m/i;
            if(tmp!=i)
                if((tmp&1)==1 and ((m/tmp)&1)==0) cnt++;
                else if((tmp&1)==0 and ((m/tmp)&1)==1) cnt++;
        }
    }
    cout<<cnt<<endl;
    return 0;
}

```



## [E:Magical Ornament](https://atcoder.jp/contests/abc190/tasks/abc190_e) 

**待补题** 

## [F:Shift and Inversions](https://atcoder.jp/contests/abc190/tasks/abc190_f) 

### 解题方案：

简单理解一下，题目为每次将数列的第一个数放到队尾，分别求逆序对的个数。采用树状数组求逆序对的方法即可。

树状数组可以快速求得比队首的数小的数的数量，极为拿掉队首逆序对将减少的数量。同时将这个数放到队尾，使用树状数组求该数前面大于它的数的数量，极为逆序对增加的数量。重复k-2次该过程即可。

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
//  AC

const int N=5e5+5;
//区间查询，单点修改
int a[N]={0};
LL c[N]={0};
int lowbit(int x){
    return x&(-x);
}
void update(int i,int n,int v)
{
    while(i<=N)
    {
        c[i]+=v;
        i+=lowbit(i);
    }
}

LL get_sum(int i)
{
    LL x=0;
    while(i>0)
    {
        x+=c[i];
        i-=lowbit(i);
    }
    return x;
}

int main()
{
    int n;
    cin>>n;
    queue<int> que;
    LL ans=0;
    for(int i=1;i<=n;i++){
        int x; cin>>x;
        x++;//要从1计
        ans+=get_sum(n)-get_sum(x);//比x大的数量
        update(x,n,1);
        que.push(x);
    }
    cout<<ans<<endl;
    for(int k=1;k<=n-1;k++)
    {
        int val=que.front();
        que.pop();
        ans-=get_sum(val-1);//有多少比val小
        ans+=get_sum(n)-get_sum(val);//有多少比val大
        cout<<ans<<endl;
    }
    return 0;
}

```

