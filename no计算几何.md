# 6-计算几何

## 扫描线

```cpp
const ll rmax=1e5;
const ll maxn=rmax+10;
const ll mod=998244353;
    ll x[maxn<<1];
struct segtree{
    #define ls (p<<1)
    #define rs (p<<1|1)
    //这里我们维护的是两个值而不是懒标记切记
    //所以你在代码里看不到pushdown
    //cnt维护该区间被题中的线段完全覆盖的次数
    //ans维护该区间内被覆盖的点数
    ll cnt[maxn<<3];
    ll ans[maxn<<3];//注意空间
    void pushup(ll p,ll l,ll r){
        if(cnt[p]) ans[p]=(x[r+1]-x[l]);
        else if (l==r) ans[p]=0;
        else ans[p]=ans[ls]+ans[rs];
    }
    void build(ll p,ll l,ll r){
        cnt[p]=0;
        ans[p]=0;
        if(l==r){
            return;
        }
        ll mid=(l+r)>>1;
        build(ls,l,mid);
        build(rs,mid+1,r);
        pushup(p,l,r);
    }
    void update(ll tl,ll tr,ll k,ll p,ll l,ll r){
        if(tl>r||l>tr) return;
        if(tl<=l&&r<=tr){
            cnt[p]+=k;
            pushup(p,l,r);
            return;
        }
        ll mid=(l+r)>>1;
        update(tl,tr,k,ls,l,mid);
        update(tl,tr,k,rs,mid+1,r);
        pushup(p,l,r);
    }
    #undef ls
    #undef rs
}seg;
struct line{
    ll x1,x2,y,op;
};
    line li[maxn<<1];
bool cmp(line a,line b){
    if(a.y==b.y) return a.op<b.op;
    return a.y<b.y;
}
void solve(){
    ll n;
    cin>>n;
    rep(i,1,n){
        ll x1,y1,x2,y2;
        cin>>x1>>y1>>x2>>y2;
        li[i*2-1]={x1,x2,y1,1};
        li[i*2]={x1,x2,y2,-1};
        x[i*2-1]=x1;
        x[i*2]=x2;
    }
    sort(li+1,li+2*n+1,cmp);
    sort(x+1,x+2*n+1);
    ll sz=unique(x+1,x+2*n+1)-x-1;
    //unique函数的特点：返回的是合法长度+1的指针
    //即vector.end()，所以多减1
    seg.build(1,1,sz-1);
    ll ans=0;
    rep(i,1,2*n-1){
        ll l,r;
        l=lower_bound(x+1,x+sz+1,li[i].x1)-x;
        r=lower_bound(x+1,x+sz+1,li[i].x2)-x;
        seg.update(l,r-1,li[i].op,1,1,sz-1);
        ans+=seg.ans[1]*(li[i+1].y-li[i].y);
    }
    cout<<ans<<endl;
}
```

## 二维坐标类（待补充）

```cpp
//点类
struct pt{
    ll x,y;//根据题目的需要将板子中的long long改为double
    bool operator<(pt t){
        if(x==t.x) return y<t.y;
        return x<t.x;
    }
    pt operator-(pt t){
        return pt{x-t.x,y-t.y};
    }
    pt operator+(pt t){
        return pt{x+t.x,y+t.y};
    }
    pt operator*(double a){
        return pt{a*x,a*y};
    }
    friend ll cross(pt a,pt b){
        return a.x*b.y-a.y*b.x;
    }
    friend ll cross(pt a,pt b,pt c){//这里是向量AB×向量BC//注意起始点
        return cross(b-a,c-b);
    }
    friend ll cross2(pt a,pt b,pt c){//这个是向量AB×向量AC//注意起始点
        return cross(b-a,c-a);
    }
    friend double dis(pt a,pt b){
        pt c=a-b;
        return sqrt(c.x*c.x+c.y*c.y);
    }
};
//线类
struct line{
    pt s,e;//两点式
    double angle(){
        pt k=e-s;
        return atan2(k.y,k.x);
    }
    bool operator<(line t){//按照方向向量进行极角排序
        double ag=angle(), tag=t.angle();
        if(fabs(ag-tag)<=eps) return cross2(s,e,t.e)<0;
        return ag<tag;
    }
    friend pt crosspt(line a,line b){//两直线求交点坐标
        //如果你要用到这个函数的话，那基本得开double了
        //即使一开始的坐标都是整点
        pt u=a.s-b.s, v=a.e-a.s, w=b.e-b.s;
        double t=cross(u,w)/(cross(w,v));
        return a.s+v*t;
    }
};
```

## 二维凸包

```cpp
    pt pa[maxn];
    ll st[maxn];
ll tubao(pt p[],ll n){
    sort(p+1,p+n+1);//先按横坐标排，再按纵坐标排
    ll tp=1;
    st[tp]=1;//维护一个单调栈
    //下凸包
    rep(i,2,n){
        while(tp>1&&cross(p[st[tp-1]],p[st[tp]],p[i])<=0){//叉积小于等于0则弹出
            tp--;
        }
        tp++;
        st[tp]=i;
    }
    ll tmp=tp;
    //上凸包
    per(i,n-1,1){
        while(tp>tmp&&cross(p[st[tp-1]],p[st[tp]],p[i])<=0){
            tp--;
        }
        tp++;
        st[tp]=i;
    }
    return tp;
    //最后栈内的(num+1)个元素按顺序就是构成凸包的点的下标(+1,因为id=1的点被记了两次)
    //首尾各一次
}
int main(){
    ll n;
    cin>>n;
    rep(i,1,n){
        cin>>pa[i].x>>pa[i].y;
    }
    ll tp=tubao(pa,n);
    double ans=0;
    rep(i,2,tp){
        ans+=dis(pa[st[i-1]],pa[st[i]]);
    }
    cout<<ans<<endl;
}
```

## 旋转卡壳（求直径）

```cpp
//一款基于凸包的求凸多边形直径的算法
    ll st[maxn];
    pt pb[maxn];
double getdia(pt p[],ll n){
    ll tp=tubao(pb,n);//凸包见上
    double ans=0;
    if(tp<4){
        ans=dis(p[1],p[2]);
        return ans;
    }
    ll j=3;
    tp--;
    rep(i,1,tp){
        while(llabs(cross(p[st[i]],p[st[i+1]],p[st[j]]))<=
              llabs(cross(p[st[i]],p[st[i+1]],p[st[j%tp+1]]))){
                    j=j%tp+1;
        }//llabs根据题目要求可能改成fabs
        ans=max(ans,max(dis(p[st[i]],p[st[j]]),dis(p[st[i+1]],p[st[j]])));
    }
    return ans;
}
int main(){
    ll m;
    cin>>m;
    rep(i,1,m){
        cin>>pb[i].x>>pb[i].y;
    }
    double ans=getdia(pb,m);
    cout<<ans<<endl;
}
```

## 半平面交
```cpp
bool right(line a,line b,line c){//b与c的交点是否在a的右侧？
    pt p=crosspt(b,c);
    return cross2(a.s,a.e,p)<0;
}
    line a[maxn];//直线们
    line q[maxn];//队列
pll halfplane(line li[],ll n){
    //给出一堆(有方向的)直线
    //求解这些直线**左侧**所包围的的半平面交
    sort(li+1,li+n+1);
    ll l=1,r=1;
    q[1]=li[1];
    rep(i,2,n){
        if(a[i].angle()-a[i-1].angle() <= eps) continue;
        while(l<r && right(a[i],q[r],q[r-1]) ){
            r--;
        }
        while(l<r && right(a[i],q[l],q[l+1]) ){
            l++;
        }
        q[++r] = a[i];
    }
    while(l<r-1 && right(q[l],q[r],q[r-1])){
        r--;
    }
    q[++r]=q[l];
    return {l,r};
}
    pt p[maxn];
double area(pt p[],ll n){//求解n个点构成的凸多边形(保证)的面积
    double ans=0;
    rep(i,2,n-1){
        ans+=cross2(p[1],p[i],p[i+1]);
    }
    return ans/2;
}
void solve(){
    ll n;
    cin>>n;
    ll sz=0;
    rep(_,1,n){
        ll m;
        cin>>m;
        vector<pt> tmp(m);
        rep(i,0,m-1){
            cin>>tmp[i].x>>tmp[i].y;
        }
        rep(i,1,m){
            a[sz+i].s=tmp[i-1];
            a[sz+i].e=tmp[i%m];
        }
        sz+=m;
    }
    auto [l,r]=halfplane(a,sz);
    ll sz2=0;
    rep(i,l,r-1){
        sz2++;
        p[sz2]=crosspt(q[i],q[i+1]);
    }
    double ans=area(p,sz2);
    cout<<fixed<<setprecision(3)<<ans<<endl;
}
```

## 极角排序

### 使用atan2函数

```cpp
bool cmp1(pt a,pt b){
    if(atan2(a.y,a.x)==atan2(b.y,b.x)) return a.x<b.x;
    return atan2(a.y,a.x)<atan2(b.y,b.x);
}
```

排完序后的效果：从$\theta = -\pi$的地方逆时针旋转，依次经过第三、四、一，二象限，最尽头到达$\theta = \pi$

特点：快、但可能有精度问题

### 使用叉积

```cpp
double cross(pt a,pt b){
    return a.x*b.y-a.y*b.x;
}
bool cmp2(pt a,pt b){
    if(cross(a,b)==0) return a.x<b.x;
    return cross(a,b)>0;
}
```

效果：逆时针排序

特点：无精度损失，较慢，且只能在总角度范围在180°之内使用。

### 想在360°的情况下使用叉积进行排序

方法：先对点的象限做判断，再做叉积

```cpp
int Quadrant(pt a){
    if(a.x>0&&a.y>=0)  return 1;
    if(a.x<=0&&a.y>0)  return 2;
    if(a.x<0&&a.y<=0)  return 3;
    if(a.x>=0&&a.y<0)  return 4;
}
bool cmp3(pt a,pt b){
    if(Quadrant(a)==Quadrant(b)) return cmp2(a,b);
    return Quadrant(a)<Quadrant(b);
}
```

效果：从$\theta = 0$的地方逆时针旋转，依次经过第一，二，三，四象限，最尽头到达$\theta = 2\pi$

（不过要是真要360°的话，不如直接atan2）

### 给定某坐标轴，以该坐标轴为基准逆时针排序

> 下面以o为原点，o oo向量为基准向量进行逆时针排序

```cpp
pt o,oo;
o={0,0},oo={-1,0};
sort(p+1,p+n+1,[&](pt a,pt b) -> bool {
    if(cross2(o,oo,a)==0 && (oo.x-o.x)*(a.x-o.x)>0) return true;
    if(cross2(o,oo,b)==0 && (oo.x-o.x)*(b.x-o.x)>0) return false;
    if((cross2(o,oo,a)>0) != (cross2(o,oo,b)>0)) return cross2(o,oo,a)>cross2(o,oo,b);
    return cross2(o,a,b) > 0 ;
});
```

效果：这个例子中，和你用atan2进行排序的效果类似（由于这里的基准坐标轴比较特殊），但小区别在于位于该坐标轴上的点会被认为是最前面的点而非最后面的点

注意：极角排序的时候不要把原点放到你想要排序的点里面，因为原点的极角是没有办法良定义的，不要这么做。