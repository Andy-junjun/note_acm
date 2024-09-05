# trie树、字典树

## 板子

 [P8306 【模板】字典树](https://www.luogu.com.cn/problem/P8306)
[P2580 于是他错误的点名开始了](https://www.luogu.com.cn/problem/P2580)

**字典树**，一种**前缀数据结构**，根节点是空串，每个边是一个字符，从根节点到某个节点按照依次经过边会形成字符串。
每次往树上插入一个字符串就是可以按照字符顺序走边(如果没有边就建边，开点)，走完这个字符串，并在最后一个节点打标记。
查询就是按照字符顺序走边，如果没有边走就一定没有，如果走完，要看一下该节点有没有标记。

```cpp
#include<bits/stdc++.h>
using namespace std;
int get_num(char a){
	if (a>='0'&&a<='9')return a-'0';
	if (a>='a'&&a<='z')return a-'a'+10;
	if (a>='A'&&a<='Z')return a-'A'+36;
	return 0;
}
string s;
int n,q,T;
int vis[3000010][77];
int cnt,to[3000010][77],jsq,ans[3000010];
void insert(){//建树插点
	int m=s.size(),now=0;
	for (int i=0;i<m;i++){
		if (vis[now][get_num(s[i])]!=T+1){
			vis[now][get_num(s[i])]=T+1;
			to[now][get_num(s[i])]=++cnt;
			ans[cnt]=0;
		}
		now=to[now][get_num(s[i])];
		ans[now]++;
	}
	return ;
}
int query(){//查询字符串
	int m=s.size(),now=0;
	for (int i=0;i<m;i++){
		if (vis[now][get_num(s[i])]!=T+1)return 0;
		now=to[now][get_num(s[i])];
	}
	return ans[now];
}
int main(){
	scanf("%d",&T);
	while (T--){
		scanf("%d%d",&n,&q);
		cnt=0;
		for (int i=1;i<=n;i++){
			cin>>s;
			insert();
		}
		for (int i=1;i<=q;i++){
			cin>>s;
			printf("%d\n",query());
		}
	}
	return 0;
} 
```



## [P3065 USACO12DEC First! G ](https://www.luogu.com.cn/problem/P3065)

题目大意：有多少个字符串**不能通过字典序重排**变成字典序最小的字符串（其实直接求补集就可以了求多少个字母可以通过字典序重排变成最小的）。

思路：
可以根据这多个字符串建立一个字典树，找字典序最小的字符串的步骤是：从根往下走，如果当前节点有多个去路，优先走字典序小的边，所以得知其他边的字母的字典序都是大于这个边上的字母（即可得知一系列的字母的字典序大小关系）。**注意**，如果一个字母的前缀是另一个字母，那没该字母不可能是字典序最小的字母（因为**空字符**字典序肯定是字典序最小）举个栗子：aabc 和  a 中aabc的字典序一定比a大！

那我们怎么维护那一系列的字典序的大小关系是否成立？其实这就是维护优先级的问题，如果**存在环就证明一定无解**。
于是，我们可以利用**拓扑排序**从入度为0的点跑有向图，如果**全部的点都进队了一次就证明无环，有解**。

**tag：trie树+拓扑排序**

## [P2536 AHOI2005 病毒检测](https://www.luogu.com.cn/problem/P2536)

题目大意：字符串匹配，RNA病毒基因中包含五种字符：A、C、G、T、？、* （？为可以匹配任何种类碱基，*可以匹配若干个碱基（**包含空字符**）），给出m个基因序列让你来判断这这些基因序列是否携带病毒？

它打破了我们“传统”的字符串的一一匹配的刻板印象，一个字符位置可以匹配4种边，甚至有0个或者若干个边。
这变得很符合我们对**搜索**的印象，所以我们可以变成**trie树上记忆化dfs**，前面四种碱基就是正常的一一配对转移，我们讨论$?和*$该如何转移？

**？**：直接枚举四种边能否走，可以走就往下走。
*****：
1、空字符：当前匹配下一个目标串的下一个字符
2、如果若干个字符，其实可以变成$？$，先往下走一条边，匹配位置后移一位或者 $? *$，往下走一条边，匹配位置不变还是$*$。

要**记忆化**，设当前节点位置x和匹配位置level，$vis[x][y]$访问过，那么就返回，因为这个状态之前被搜索过。

**tag：trie树+dfs记忆化搜索**

[P4592 TJOI2018 异或 ](https://www.luogu.com.cn/problem/P4592)

# 01trie

01trie在异或运算中有重要地位，异或运算大概率是线性基或者01tri

## 板子

## [P4551 最长异或路径](https://www.luogu.com.cn/problem/P4551)

题目大意：有一颗树，两点之间的距离是经过的边权的异或和，求两点间最大的距离。
思路：
异或其实有一种**”撤销“**的性质，设一个数为x，$x \oplus k \oplus k=x$，就是再异或一次相同的数相当于撤销上一次的异或。
我们把这种性质用到树上，设树的根节点为$root$，两个节点x，y，他们的$LCA$为k：
$$
dis(x,y) = dis(x,k) \oplus dis(y,k)\\
dis(x,y) = dis(x,k) \oplus dis(k,root) \oplus dis(k,root) \oplus dis(y,k)\\
dis(x,y) = dis(x,root) \oplus dis(y,root) 
$$
可以理解为$k$到$root$的路径上边被异或了两次，相当于没有。
我们可以维护一个$dis(x,root)$的$01trie$树,然后每个点$dis(x,root)$在$01trie$树找与它异或最大的数(贪心，高位选异或为1的边走)。

### 01trie树建树

可以看作字符只有01两种，但是为了统一位权，我们在前面用0占位，所有的数字长度一样

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=200000+10;
const int M=5000010;
int hd[N],cnt_edge,cnt,val[N],ans,x,y,z,n;
bool vis1[N];
int vis2[M][2];
struct st{
	int to,next,val;
}edge[N];
void add(int u,int v,int k){
	edge[++cnt_edge].to=v;
	edge[cnt_edge].next=hd[u];
	edge[cnt_edge].val=k;
	hd[u]=cnt_edge;
	return ;
}
void dfs(int now,int nowval){
	vis1[now]=true;
	val[now]=nowval;
	for (int i=hd[now];i;i=edge[i].next){
		if (vis1[edge[i].to])continue;
		dfs(edge[i].to,nowval^edge[i].val);
	}
	return ;
}
int now,to,ansnow;
void insert(int s){
	now=0;
	for (int i=30;i>=0;i--){
		to=((s>>i)&1);
		if (!vis2[now][to])vis2[now][to]=++cnt;
		now=vis2[now][to];
	}
	return ;
}
void query(int s){
	now=0;
	ansnow=0;
	for (int i=30;i>=0;i--){
		to=((s>>i)&1);
		if (vis2[now][to^1]){
			now=vis2[now][to^1];
			ansnow+=(1<<i);
		}else now=vis2[now][to];
	}
	ans=max(ans,ansnow);
	return ;
}
int main(){
	scanf("%d",&n);
	for (int i=1;i<n;i++){
		scanf("%d%d%d",&x,&y,&z);
		add(x,y,z);
		add(y,x,z);
	}
	dfs(1,0);
	for (int i=1;i<=n;i++){
		insert(val[i]);
		ans=max(ans,val[i]);
	}
	for (int i=1;i<=n;i++){
		query(val[i]);
	}
	printf("%d",ans);
	return 0;
}
```

参考文献[P4551 最长异或路径 题解 - 洛谷专栏 (luogu.com.cn)](https://www.luogu.com.cn/article/0x1smhhc)

# trie树合并（待）

## 前置芝士

- **线段树合并**
- **trie树以及01trie树**

[P6623 省选联考 2020 A 卷\] 树 ](https://www.luogu.com.cn/problem/P6623)
[P6018 Ynoi2010 Fusion tree](https://www.luogu.com.cn/problem/P6018)

# 可持久化trie树

## 前置芝士

- **可持久化（建议学了可持久化线段树之后再来）---关键动态开点、版本维护**
- **trie树以及01trie树**

## 板子

[P4735 最大异或和](https://www.luogu.com.cn/problem/P4735)

给定一个非负整数序列 {𝑎}{*a*}，初始长度为 𝑁*N*。

有 𝑀*M* 个操作，有以下两种操作类型：

1. `A x`：添加操作，表示在序列末尾添加一个数 𝑥*x*，序列的长度 𝑁*N* 加 11。
2. `Q l r x`：询问操作，你需要找到一个位置 𝑝*p*，满足 𝑙≤𝑝≤𝑟*l*≤*p*≤*r*，使得：𝑎[𝑝]⊕𝑎[𝑝+1]⊕...⊕𝑎[𝑁]⊕𝑥*a*[*p*]⊕*a*[*p*+1]⊕...⊕*a*[*N*]⊕*x* 最大，输出最大值。

思路：
利用上面的异或**撤销**性质，我们可以把后缀异或和转化为两个前缀异或和异或(有点像前缀和)
$$
s[i]=a[1] \oplus a[2] \oplus… \oplus a[i]\\
a[p] \oplus a[p+1] \oplus… \oplus a[n] = s[n] \oplus s[p-1]\\
所以原式子化为：s[i]\oplus s[n] \oplus x(l-1 \leq i\leq r-1)
$$
现在问题转化成维护$[l-1,r-1]$之间的$01trie树$，我们不可能直接开n棵$01trie树$，我们就会想到**可持久化数据结构的动态开点**
设的某一个节点x的版本$l-2$编号为$nowx$，版本$r-1$编号为$nowy$，当前节点x是否走0/1边，看$jsq[nowy][0/1]-jsq[nowx][0/1]$是否大于0，大于0则这边下面有数字可以走！

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=6e5+10;
int a[N],root[N],cnt;
int n,m,jsq,l,r,x,k;
char opt;
int id[20000100][2],ans[20000100],to,ansh;
void insert(int nowx,int nowy,int x){
	for (int i=25;i>=0;i--){
		to=(x>>i)&1;
		id[nowx][to^1]=id[nowy][to^1];
		id[nowx][to]=++cnt;
		nowx=id[nowx][to];
		nowy=id[nowy][to];
		ans[nowx]=ans[nowy]+1;
	}
	return ;
}
void query(int nowx,int nowy,int x){
	ansh=0;
	for (int i=25;i>=0;i--){
		to=(x>>i)&1;
		if (ans[id[nowy][to^1]]>ans[id[nowx][to^1]]){
			nowy=id[nowy][to^1];
			nowx=id[nowx][to^1];
			ansh+=(1<<i);
		}else{
			nowy=id[nowy][to];
			nowx=id[nowx][to];
		}
	}
	return ;
}
int main(){
	scanf("%d%d",&n,&m);
	a[0]=0;
	root[0]=++cnt;
	insert(root[0],0,a[0]);
	for (int i=1;i<=n;i++){
		scanf("%d",&a[i]);
		a[i]^=a[i-1];
		root[i]=++cnt;
		insert(root[i],root[i-1],a[i]);
	}
	jsq=n;
	for (int i=1;i<=m;i++){
		cin>>opt;
		if (opt=='A'){
			jsq++;
			scanf("%d",&a[jsq]);
			a[jsq]^=a[jsq-1];
			root[jsq]=++cnt;
			insert(root[jsq],root[jsq-1],a[jsq]);
		}else{
			scanf("%d%d%d",&l,&r,&x);
			k=a[jsq]^x;
			if (l==1){
				query(0,root[r-1],k);
				ansh=max(ansh,k);
			}
			else query(root[l-2],root[r-1],k);
			printf("%d\n",ansh);
		}
	}
}
```

