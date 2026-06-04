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
卡点：没有
错因分类：没有
正确思路：cur充当反转后的头节点，pre充当前反转链表头节点，next充当未反转链表头节点
模板/套路：
下次遇到类似题的识别信号：
复习日期：

```

```java
public void reverseList(ListNode l){
	ListNode pre=null;
	ListNode cur=l;
	ListNode nxt=cur.next;
	
	while(nxt!=null){
		cur.next=pre;
		pre=cur;
		cur=next;
		if(nxt.next!=null)
		nxt=nxt.next;
	}

}
```