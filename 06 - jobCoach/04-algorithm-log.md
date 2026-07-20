### 执行节奏：
```
周一到周五：
每天 2 题，1 个主题，90 分钟以内。

周六：
补 3-4 题，集中解决本周薄弱题型。

周日：
只复盘错题，不开新题型。
```

### 算法记录模式
```
## 日期：2026-06-03
主题：Redis 面试日 + 算法启动 / 哈希表

### 题目 1：
题型：
是否独立完成：
耗时：
卡点：
错因分类：
正确思路：
模板/套路：
下次遇到类似题的识别信号：
复习日期：

### 题目 2：
题型：
是否独立完成：
耗时：
卡点：
错因分类：
正确思路：
模板/套路：
下次遇到类似题的识别信号：
复习日期：
```


### day03

```
## 日期：2026-06-03

### 题目 1：【两数之和】
题型：【哈希表应用】
是否独立完成：【否】
耗时：【20分钟】
卡点：【哈希表之于数组实现的高效点】
错因分类：题型没识别出来、提示使用哈希表后重复获取哈希表元素
正确思路：重复使用O(n)的时间复杂度，可以考虑用hashmap降为O(1)
模板/套路：无
下次遇到类似题的识别信号：重复循环遍历，可用hashmap
复习日期：

### 题目 2：【最长无重复子串】
题型：【滑动窗口】
是否独立完成：【否】
耗时：半小时
卡点：没有滑动窗口思路认识
错因分类：题型没识别出来
正确思路：
模板/套路：

下次遇到类似题的识别信号：符合某条件的子数组/字符串
复习日期：
```

```java
回答三个问题：
1.何时扩大窗口
2.何时缩小窗口
3.什么时候应该更新答案

public void solution(String s){
	//当前窗口的结果（各字符个数、取值结果）
	Object window=...
	
	//窗口滑动逻辑
	//1.默认左开右闭
	int left=0;int right=0;
	//扩大窗口
	while(right<s.length()){
		//往窗口塞元素
		char c= s.charAt(right);
		right++;
		
		window.add(c);
		//窗口内元素更新...
		
	
		//缩小窗口
		while(left<right&&window need shrink){
			char c=s.charAt(left);
			window.remove(c);
			left++;
			
			//窗口内元素更新...
		}
	}
}
```


### day04

```

### 题目 1：合并2个有序链表
题型：双指针
是否独立完成：是
耗时：10min
卡点：思维重点放在“如何将一个链表的元素放到另一个链表，且维护前驱后继关系”
错因分类：“链表操作要原地修改”的定式思维
正确思路：新建链表，双指针移动比较。
模板/套路：
下次遇到类似题的识别信号： 合并有序
复习日期：

```


```java
public void template(ListNode l1,ListNode l2){
	//新建dummyNode
	ListNode dummy=new ListNode();
	while(l1!=null&&l2!=null){
	//每次取链表的最小值
		if(l1.val<l2.val){
			p.next=l1;
			l1=l1.next;
		}else{
			p.next=l2;
			l2=l2.next;
		}
		p=p.next;
	}
	
	//拼接剩下的链表
	if(l1==null){
		p.next=l2;
	}
	if(l2==null){
		p.next=l1;
	}
}
```



```

### 题目 2：反转链表
题型：双指针
是否独立完成：是
耗时：10min
卡点：nxt提早初始化；循环以nxt作为条件
错因分类：
正确思路：cur充当反转后的头节点，pre充当前反转链表头节点，nxt充当未反转链表头节点；cur作为循环条件，nxt在循环内初始化
模板/套路：
下次遇到类似题的识别信号：
复习日期：

```

```java
public ListNode reverseList(ListNode head) {
    ListNode pre = null;
    ListNode cur = head;

    while (cur != null) {
        ListNode nxt = cur.next; // 🔥 每一步都保存 next
        cur.next = pre;          // 反转
        pre = cur;               // 前驱前移
        cur = nxt;               // 当前节点前移
    }
    return pre; // 返回新头
}
```

### day07

```
### 题目 1： 前缀和
题型：数组不可变
是否独立完成：否
耗时：
卡点：1.res[a..b]的累加和≠ 累加和数组sumArr[b]-sumArr[a];2.无法处理[0,0]边界问题
错因分类：没有考虑边界问题、没有画图验证
正确思路：累加数组预留sum[0]=0作为边界处理；做图验证后 res[a..b]=sumArr[b+1]-sumArr[a]
模板/套路：
下次遇到类似题的识别信号：多次重复遍历+计算累计和/差
复习日期：次日
```

```java
前缀和模板

public void solution(int[] nums){
	//sums[0]=0;方便处理边界条件
	int[] sums=new int[nums.length+1];
	//前缀和=上个前缀和+原数组当前元素
	for(int i=1;i<sums.length;i++){
		sums[i]=sums[i-1]+nums[i-1];
	}
	
}
//[a,b]区间的前缀和=sum[a+1]-sums[b]差值
public int sumRange(int start,int end,int[] sums){
	return sums[end+1]-sums[start];
}
```


```
### 题目 2：二维前缀和
题型：矩阵不可变
是否独立完成：否
耗时：
卡点：二维数组的矩阵和如何表示？目标矩阵和又如何表示？
错因分类：识别不出题型
正确思路：
1.左侧各加一行一列处理矩阵和边界问题；
2.矩阵和求解规律
3.认识目标矩阵和求解规律
模板/套路：
下次遇到类似题的识别信号：二维数组求和
复习日期：次日
```

```java
// 构建二维前缀和
public int[][] build2DPrefix(int[][] matrix) {
    int m = matrix.length;
    int n = matrix[0].length;
    int[][] pre = new int[m + 1][n + 1]; // 行列都+1
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            pre[i][j] = pre[i - 1][j] + pre[i][j - 1] 
                      - pre[i - 1][j - 1] + matrix[i - 1][j - 1];
        }
    }
    return pre;
}

/**
 * 查询二维矩阵 (i1,j1) 左上 ~ (i2,j2) 右下 闭区间和
 * @param i1 左上角行
 * @param j1 左上角列
 * @param i2 右下角行
 * @param j2 右下角列
 * @param pre 二维前缀和数组
 */
public int query2D(int i1, int j1, int i2, int j2, int[][] pre) {
    return pre[i2+1][j2+1] - pre[i1][j2+1] - pre[i2+1][j1] + pre[i1][j1];
}
```