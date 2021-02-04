---
layout:     post
title:      "Codeforces Round #696 (Div. 2) 题解"
subtitle:   日常垫底掉分
date:       2021-01-20
author:     Lin_Xuan
header-img: "img/post-Note-Head-空への距離-banishment.jpg"
catalog: true
tags:
   - ACM
   - 题解
   - Codeforces
---

传送门：[Codeforces Round #696 (Div. 2)](https://codeforces.com/contest/1474) 

## [A：Puzzle From the Future](https://codeforces.com/contest/1474/problem/A)

签到题，控制着不要让c有相邻且相同的位数就可以了。

```c++
#include<bits/stdc++.h>

#define IOS ios::sync_with_stdio(false); cin.tie(0);cout.tie(0);

#define  m_p make_pair

#define x first

#define y second 

using namespace std;
using LL=long long;
using ULL=unsigned long long;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;

int main()
{
    IOS;
    int t;cin>>t;
    while(t--)
    {
        int n;
        cin>>n;
        string b;
        cin>>b;
        vector<int> c(n);
        string a;
        a.push_back('1');
        c[0]=b[0]=='0'?1:2;
        for(int i=1;i<n;i++)
        {
            if(c[i-1]==1){
                if(b[i]=='1') a.push_back('1');
                else a.push_back('0');
            }
            else if(c[i-1]==2){
                if(b[i]=='1') a.push_back('0');
                else a.push_back('1');
            }
            else if(c[i-1]==0){
                a.push_back('1');
            }
            c[i]=a[i]+b[i]-'0'-'0';
        }
        cout<<a<<endl;
    }
    return 0;
}

```

## [B：Different Divisors](https://codeforces.com/contest/1474/problem/B)

### 题目大意：

求一个最小的数x满足x的所有因子个数大于等于4，且因子差的最小值大于d。注意不是质因子而是所有因子。

### 解题方案：

x是一个除1和本身外还有两个因子的数。找大于d的质因子a，大于a+d的质因子b，a*b极为所求。$1\leqslant a,b\leqslant 1000 =>a\times b\leqslant 1E8$ ,使用欧拉筛筛出1E8内的质数，二分去求就可以了。

简单用反证法证明一下，x最小的质因子一定大于a，否则不满足因子最小差为d的条件。另一个质因子与a的差距也大于d。二者的积一定满足题意。我们证明一下不会有更小的答案。假设有更小的数字y满足题意，则将y不是质数且。分解质因数至少有两项a',b'且满足：$a'\times b'\leqslant y < x$。则由a为大于d的第一个数，有a'>a>d，b是大于b+a的第一个数，有b'>a'+d>a+d>b。$a'\times b'>a\times b$，与假设矛盾。

### 代码

```c++
#include<bits/stdc++.h>

#define IOS ios::sync_with_stdio(false); cin.tie(0);cout.tie(0);

#define  m_p make_pair

#define x first

#define y second 

using namespace std;
using LL=long long;
using ULL=unsigned long long;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;

void get_prime(vector<int> & prime,int upper_bound){ // 传引用
    if(upper_bound < 2)return;
    vector<bool> Is_prime(upper_bound+1,true);
    for(int i = 2; i <= upper_bound; i++){
        if(Is_prime[i])
            prime.push_back(i);
        for(int j = 0; j < prime.size() and i * prime[j] <= upper_bound; j++){
            Is_prime[ i*prime[j] ] = false;
            if(i % prime[j] == 0)break;// 保证了一个数只被筛一次。
        }
    }
}
int main()
{
    IOS;
    vector<int> prime;
    get_prime(prime, 10000001);
    int t;
    cin>>t;
    while(t--)
    {
        int d;cin>>d;
        LL a=lower_bound(prime.begin(),prime.end(),d+1)-prime.begin();
        a=prime[a];
        LL b=lower_bound(prime.begin(),prime.end(),d+a)-prime.begin();
        b=prime[b];
        cout<<a*b<<endl;

    }
    return 0;
}

```

**没用的碎碎念**：

*温馨提示，下面全是负能量* 

啊啊啊啊啊啊啊……好菜啊，选拔赛又垫底了！！

B开始读错题了啊，以为是质因子的最小差大于d，还打表找规律，上了ST表。差点用上线段树……太难了！

## [C：Array Destruction](https://codeforces.com/contest/1474/problem/C)

### 题目大意

有2*n个数字，开始选择一个数字m，接下来从2\*n个数字里选择两个数字x，y满足x+y=m的两个，将他们去除，同时更新m为x,y中的较大值。

如果重复上述过程可以移除所有的数字，则输出YES并输出每次一处的数字，否则输出NO

### 解题方案

注意一点，每次选择数字是都必须包含当前的最大数字，否则在下一次的匹配中最大数字同任意数字组合一定大于此次选择的数字中的较大值。

那，只要枚举第一次同最大值匹配的数字，就可以得到m的初始值，之后m每次的更新都是固定的，可以在$O(n)$的时间内检查是否可行，总共执行n次。

### 代码：

```c++
#include<bits/stdc++.h>

#define IOS ios::sync_with_stdio(false); cin.tie(0);cout.tie(0);

#define  m_p make_pair

#define x first

#define y second 

using namespace std;
using LL=long long;
using ULL=unsigned long long;
using P=pair<int,int>;
const int inf=0x3f3f3f3f;
const LL LLinf=0x3f3f3f3f3f3f3f3f;

// 补题了！最终要的一点，每次必须选择当前的最大值，否则最大值就不可能被消除了。
int main()
{
    int t;
    const int N=1e6;
    vector<int> num(N+5,0);
    cin>>t;
    while(t--)
    {
        int n; cin>>n;
        vector<int> a((n<<1)|1);
        for(int i=1;i<=n*2;i++){
            cin>>a[i];
            num[a[i]]++;
        }
        sort(a.begin(),a.end());
        vector<int> ans;//储存配对信息;
        for(int x=1;x<n*2;x++)
        {
            // 枚举与最大值结合的数字x;
            int target=a[2*n];
            ans.push_back(a[x]); num[a[x]]--;
            ans.push_back(a[2*n]); num[a[2*n]]--;
            for(int k=2*n-1;k>=1;k--)
            {
                if(num[a[k]]<=0) continue;
                ans.push_back(a[k]); num[a[k]]--;
                if(num[target-a[k]]<=0) break;
                ans.push_back(target-a[k]); num[target-a[k]]--;
                target=max(a[k],target-a[k]);
            }
            if(ans.size()==n*2){
                // 得到结果了
                break;
            }
            else{
                for(auto val:ans) num[val]++;
                ans.clear();
            }
        }
        if(ans.size()==2*n){
            cout<<"YES"<<endl;
            cout<<ans[0]+ans[1]<<endl;
            for(int i=0;i<ans.size();i++)
                cout<<ans[i]<<" \n"[i&1];
        }
        else{
            cout<<"NO"<<endl;
            for(auto val:ans) num[val]++;
            for(auto val:a) num[val]--;
        }

    }
    return 0;
}

```

