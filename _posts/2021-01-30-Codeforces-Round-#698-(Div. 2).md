---
layout:     post
title:      Codeforces Round \#698 (Div. 2) ��� #����
subtitle:   �е���ս�ĸо���� #������
date:       2021-01-30 #����
author:     Lin_Xuan
header-img: img/post-Note-Head-�ӵ�-Lifeline.jpg #ͼƬ
catalog: true
tags:
   - ACM
   - ���
   - Codeforces
---

&emsp;&emsp;�����ţ�[Codeforces Round #698 (Div. 2)](https://codeforces.com/contest/1478) 

&emsp;&emsp;CF��ʷ��������óɼ�������ɭ

&emsp;&emsp;���������д��⡭��ɣ��

&emsp;&emsp;������D�⡭��������\~ 

## [A: Nezzar and Colorful Balls](https://codeforces.com/contest/1478/problem/A) 

&emsp;&emsp;ǩ��,��CF��ɫ�ˣ��������һ���飬�����˼�������ظ����ֵ�����������

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

### ���ⷽ����

&emsp;&emsp;��ʱ���˺þô����ϳ��ˣ������Ż���һ�´������˼·��ʵ�ܼ򵥡���

&emsp;&emsp;�涨ʮ����������d���ֵ�����Ϊ��������Ҫ����֤n�ܷ���������������ۼӵõ���

&emsp;&emsp;�����������һ����$n<10*d$�������Ļ����n��������ʮ���Ƶĵڶ�λ�Ͳ�������d��ֻ�ܿ��ǹ���10+d,20+d���������n%d��n/d���ص�����n%d�����������������(ʣ�µ��Ե�����d���ڼ���)�����ϳ���n%d+d,ֱ���������е�d���ж��Ƿ��ܵõ���λΪd���������ܵĻ��𰸼�Ϊ"No"��

&emsp;&emsp;��$n>=10*d$�ǣ�һ���������������ۼӵõ��������ǰ�n�ֳ�a1+a2������a1��ʮλ��d����λ��0����λ��nһ�¡�a2Ϊn-a1����ô��n������a1+a2%d��a2/d��d��ӵõ�����Щ��������������

### ���룺

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

### ���ⷽ����

&emsp;&emsp;��ʱ�����������������֮���ˣ���ʱB����һ�ᡭ��C���û�������5����A�ġ�

&emsp;&emsp;����A�е���������ͬ����ÿ�������෴������A��(��Ϊn����ͬ���������෴��)������D������A������ɡ�������Ĺ�������֤�Ƿ���Թ�������Aʹ��D������$di=��^{2n}_{j=1}|a_i-a_j|$�����Ƿ��֣�ֻ����A�е�n������(������di���෴����������ͬ)��$di=��(a_i>a_j?2a_i:2a_j)$����������n��������С�������򣬶�Ӧ��DҲ�Ǵ�С��������

&emsp;&emsp;�������Եõ�һ�����ƹ�ʽ��`d[i]=d[i-1]-2*(i-1)*a[i-1]+2*(i-1)*a[i]`�����ݵݹ鹫ʽ����a[1]Ϊ�ݣ��Ϳ��Եõ����е����ĺ�sum�ݡ�������С����d[1]���������ܺ͵�2��(��������ֻ����������)����֤�Ƿ����d[1]�ȵȷֳ�2*sum�ݼ��ɡ�

&emsp;&emsp;��һЩ��Ҫע���ϸ�ڡ�A�����ݲ�����0(��Ϊ0���෴����Ϊ0������A�����ݲ�����ͬ)��ͬʱÿ�ε���a[i]���ӵ������������������򽫳���С����������

&emsp;&emsp;����ϸ�ڿ�����ע�͡�

### ����

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
            if(d[i]!=d[i+1]){//ÿ�����������ż����(d[a]==d[-a])
                flag=false;
                break;
            }
        if(flag==false or d[1]<2*n or d[1]%2==1){//d[1]=2*sum,�������2*n��Ϊż��
            cout<<"No"<<endl;
            continue;
        }
        
        // d[i]==d[i-1]-2*(i-1)*a[i-1]+2*(i-1)*a[i]
        flag=true;;
        vector<LL> a(n+1);
        LL sum_dd=0;
        for(int i=2;i<=n;i++)
        {
            if( (d[2*i-1]-d[2*i-3])%(i*2-2)!=0 or (d[2*i-1]-d[2*i-3])/(i*2-2)==0){//�������÷��䣬�Ҳ�Ϊ0(ÿ�α�������)
                // ��������or���
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

