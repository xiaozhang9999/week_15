# week_15
## 字符串匹配
在源字符串s中寻找目标字符串t，t为s的子串，返回目标字符串首次出现在源字符串中的索引；找不到目标字符串时返回-1。
### 最朴素的算法—暴力搜索
i为索引遍历s;j为索引遍历t。依次比较s[i+j]与t[j]是否匹配，是则j++;否则j=0,i++。j==t.length，找到匹配，最后返回i；i>s.length-t.length，未找到匹配，返回-1。
```c++
//在s中寻找t，返回匹配的第一个索引，否则返回-1
int bruteforce(string s, string t) 
{
	if (s.length() < t.length()) return -1;
	for (int i = 0; i <= s.length() - t.length(); i++) {
		int j = 0;
		for (; j < t.length(); j++) {
			if (s[i + j] != t[j])
				break;
		}
		if (j == t.length())return i;//匹配
	}
	return -1;
}
```
最坏情况时算法执行次数为$(s.length-t.length)*t.length$，该算法的性能在一般情况下其实并不差，但是可以找到一种情况使其退化,时间复杂度为$O(n^2)$。
### 改进思路
比较两个字符串是否相等是O(n),比较两个整型是O(1)，可以将字符串转换成整型，使用哈希函数
hash(code)=((($c*B$+o)%$M * B$+d)%$M*B$+e)%M,//B为26，M需要取合适的值。
抛开匹配的问题，关于字符串转哈希的一个应用**leetcode 1147**段式回文，可以使用贪心法，动态规划。
每次添加新的字符都需要重新匹配，很费时，转换成哈希值每次更新哈希值进行比较就可以了。
未转换成哈希值的代码
```c++
class Solution {
public:
    int longestDecomposition(string text) {
        return solve(text,0,text.length()-1);
    }
    int solve(string s,int left,int right){
        if(left>right) return 0;
        for(int i=left,j=right;i<j;i++,j--){
            //判断i左边和j右边的两个子串是否匹配
            if(equal(s,left,i,j,right))
            return 2+solve(s,i+1,j-1);
        }
        return 1;
    }
    //s[l1,r1]?=s[l2,r2]
    bool equal(string s,int l1,int r1,int l2,int r2){
        for(int i=l1,j=l2;i<=r1&&j<=r2;i++,j++){
            if(s[i]!=s[j])return false;
        }
        return true;
    }
};
```
可以通过，但是性能并不好。
转换成哈希值的代码
```c++
class Solution {
public:
    long MOD=(long)(1e9+7);//十亿零七是一个很大的素数，作为取模运算的模，防止相加后的数据溢出
     int longestDecomposition(string text) {
         //计算26^i的值
         long pow26[text.length()];
         pow26[0]=1;
         for(int i=1;i<text.length();i++){
             pow26[i]=pow26[i-1]*26%MOD;
         }
        return solve(text,0,text.length()-1,pow26);
    }
    int solve(string s,int left,int right,long pow26[]){
        if(left>right) return 0;
        long prehash=0,posthash=0;
        for(int i=left,j=right;i<j;i++,j--){
            prehash=(prehash*26+(s[i]-'a'))%MOD;
            posthash=((s[j]-'a')*(pow26[right-j])+posthash)%MOD;
            //判断i左边和j右边的两个子串是否匹配
            if(prehash==posthash&&equal(s,left,i,j,right))//哈希值相等不一定绝对匹配，可能会有哈希冲突
            return 2+solve(s,i+1,j-1,pow26);
        }
        return 1;
    }
    //s[l1,r1]?=s[l2,r2]
    bool equal(string s,int l1,int r1,int l2,int r2){
        for(int i=l1,j=l2;i<=r1&&j<=r2;i++,j++){
            if(s[i]!=s[j])return false;
        }
        return true;
    }
};
```
通过提交结果后者比前者性能好很多。
