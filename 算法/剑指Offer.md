# 剑指Offer

## 链表

#### 1.输入一个链表，按链表从尾到头的顺序返回一个ArrayList。

思路：

1.递归

```java
ArrayList<Integer> list=new ArrayList();
public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
  if(listNode==null) return list;
  else{
    printListFromTailToHead(listNode.next);
    list.add(listNode.val);
  }
  return list;
}
```

2.栈

```java
	public static ArrayList reverse_(ListNode l){
        if(l==null) return null;
        Stack stack=new Stack();
        while (l.next!=null){
            stack.push(l.val);
        }
        while (!stack.isEmpty()){
            Object pop = stack.pop();
            list.add(pop);
        }
        return list;
    }
```

#### 2.用两个栈来实现一个队列

###### 思路：

​	**进队**直接进第一个栈S1

​	**出队**时首先判断S2有没有值，有的话将S2中的值直接出栈，S2无值则将S1的值放到S2中，S2出栈

```java
		private Stack<Integer> s1=new Stack();
    private Stack<Integer> s2=new Stack();

    //进队
    public  void push(int node){
        s1.push(node);
    }

    //出队
    public int pop(){

        if(s1.empty()&&s2.empty()){
            throw new RuntimeException("队列为空！");
        }

        if(!s2.isEmpty()){
            Integer pop = s2.pop();
            return pop;
        }else{
            while(!s1.isEmpty()){
                s2.push(s1.pop());
            }
            Integer pop2 = s2.pop();
            return pop2;
        }
    }
```

#### 3.输入一个链表，输出该链表中倒数第k个结点。

思路：

​	双指针，让第一个指针先走，每走一次k--，等到k=0第二个指针开始走，等到第一个指针到链表尾部，q指向的就是倒数第k个节点

```java
	public ListNode FindKthToTail(ListNode head,int k) {
        ListNode p=head;//前指针
        ListNode q=head;//后指针
        while(p!=null){
            p=p.next;
            if(k!=0){
                k--;
            }else{
                q=q.next;
            }
        }
        if(k>0) return null;
        return q;
    }
```

#### 4.输入一个链表，反转链表后，输出新链表的表头。

思路：

​	头插法

```java
public ListNode ReverseList(ListNode head){
  ListNode p=null;
  ListNode q=head;
  ListNode t;
  while(q!=null){
    t=q.next;
    q.next=p;
    p=q;
    q=t;

  }
  return p;  
}
```



#### 5.合并两个有序的链表（递增）

思路：

​	循环比值，直到任意一个单链表的最后一个值

​	将余下链表中的值直接加到合并的链表后

```java
	public ListNode Merge(ListNode list1, ListNode list2) {
        if (list1==null) return list2;
        if (list2==null) return list1;
        //创建一个头结点方便返回链表头的操作
    		ListNode head=new ListNode(-1);
        ListNode node=head;
				//进行比值
        while (list1!=null && list2!=null){
            if(list1.val < list2.val) {
                node.next=list1;
                list1=list1.next;
            }
            else{
                node.next=list2;
                list2=list2.next;
            }
          	//加入新的节点后指向新节点
            node=node.next;
        }
  			//将剩下的链表加入合并后的链表中
        if (list1!=null){
            node.next=list1;
        }
        if (list2!=null){
            node.next=list2;
        }
        return head.next;
    }
```



#### 6.栈的压入弹出序列

描述：

​	输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）



思路：

​	将压栈的序列进栈，每一个元素进栈时都判断一下和出栈的序列的元素是否相同，相同则弹出栈顶的值并循环判断栈中值，直到不相同为止，若判断完栈为空，则是，不空则不是

```java
		public boolean IsPopOrder(int [] pushA,int [] popA) {
        if(popA.length==0 || pushA.length==0) return false;
        Stack<Integer> stack=new Stack<>();
        int count=0;
        for (int i = 0; i < pushA.length; i++) {
            stack.push(pushA[i]);
           	//判断栈顶值与弹出序列值是否相同
          	while(!stack.empty()&&popA[count]==stack.peek()){
              	//相同就弹出
                stack.pop();
              	//判断弹出序列中下一个值
                count++;
            }
        }
        return stack.isEmpty();
    }
```



#### 7.链表中的第一个公共节点

描述：

​	输入两个链表，找出它们的第一个公共结点。（注意因为传入数据是链表，所以错误测试数据的提示是用其他方式显示的，保证传入数据是正确的）

思路：

​	循环遍历，p1到头就让p1执行第二个链表，p2到头就让p2遍历第一个链表，当p1==p2时，就是第一个公共节点	

```java
	public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
     if (pHead1==null || pHead2==null ) return null;
        ListNode p1=pHead1;
        ListNode p2=pHead2;
        while(p1!=p2){
            p1=p1==null?pHead2:p1.next;
            p2=p2==null?pHead1:p2.next;
        }
        return p1;
    }
```

#### 8.链表中环的入口节点

描述：

​	给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。

思路：

​	1.先判断是否有环

​	2.若有环，入口节点则是从头遍历的指针和以前相遇点遍历环的指针相遇的节点

```java
	public ListNode EntryNodeOfLoop(ListNode pHead) {
        if(pHead==null) return null;
        ListNode p=pHead;
        ListNode q=pHead;
        //先找出相遇点；如果是（ 1 null）的情况不加q.next!=null， q=q.next.next;会报空指针
        while(q!=null && q.next!=null){
            p=p.next;
            q=q.next.next;
            if(q==p) break;
        }
        //q.next==null 考虑的是可能未进入上面的循环
        if(q!=p || q.next==null) return null;
        //求入口节点
        p=pHead;
        while(q!=p){
            p=p.next;
            q=q.next;
        }
        return p;
    }
```

#### 9.删除链表中重复的节点

描述：

​	在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表1->2->3->3->4->4->5 处理后为 1->2->5

思路：

​	设置头尾两个节点 p指向当前确定的不重复节点, q判断重复节点，若重复，则一直找到不重复的值，找到后挪动指针删除；若不重复，继续遍历

```java
	public ListNode deleteDuplication(ListNode pHead) {
        if(pHead==null || pHead.next==null) return pHead;
        //添加一个头结点 防止第一个数就重复
        ListNode head=new ListNode(0);
        head.next=pHead;
        ListNode p=head;
        ListNode q=head.next;
        while(q!=null){
            //注意：要将q.next!=null放前，否则可能会报空指针异常
            if(q.next!=null&&q.val==q.next.val ){
                while(q.next!=null&&q.val==q.next.val){
                    q=q.next;
                }
                p.next=q.next;
                q=q.next;
            }else {
                p=p.next;
                q=q.next;
            }
        }
        return head.next;
    }
```



#### 10.包含min函数的栈

描述：

​	定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。

注意：保证测试中不会当栈为空的时候，对栈调用pop()或者min()或者top()方法。



思路：

​	stackTotal：存放所有元素； stackLittle：存放当前最小元素

​	**入栈**时stackTotal直接入栈，判断是不是最小值是则stackLittle入栈

​	**出栈**则判断当前值是否为最小值，是则两栈同时出栈，不是则stackTotal出栈



```java
		Stack<Integer> stackTotal = new Stack<Integer>();
    Stack<Integer> stackLittle = new Stack<Integer>();
 
    public void push(int node) {
        stackTotal.push(node);
        if(stackLittle.empty()){
            stackLittle.push(node);
        }else{
            if(node <= stackLittle.peek()){
                stackLittle.push(node);
            }
        }
    }
 
    public void pop() {
        if(stackTotal.peek()==stackLittle.peek()){
            stackTotal.pop();
            stackLittle.pop();
        }else{
            stackTotal.pop();
        }
    }
 
    public int top() {
        return stackTotal.peek();
    }
 
    public int min() {
        return stackLittle.peek();
    }
```



#### 11.复杂链表的复制

描述：

​	输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

思路：

​	

```java
public class RandomListNode {
    int label;
    RandomListNode next = null;
    RandomListNode random = null;

    RandomListNode(int label) {
        this.label = label;
    }
}

public RandomListNode clone(RandomListNode pHead){
  if(pHead==null) return null;
  RandomListNode r1=pHead;
  //链表复制
  while (r1!=null){
    RandomListNode node=new RandomListNode(r1.label);
    node.next=r1.next;
    r1.next=node;
    r1=node.next;
  }
  //根据原链表的兄弟节点连接新链表的兄弟节点
  RandomListNode r2=pHead;
  while (r2!=null){
    //看原节点的节点是否为空，若不为空，指向原数组随机指针指向的节点的下个节点（新数组的节点）
    r2.next.random=r2.random==null?null:r2.random.next;
    r2=r2.next.next;
  }
  //拆分链表
  RandomListNode r3=pHead;
  RandomListNode clone=pHead.next;
  while(r3!=null){
    RandomListNode node=r3.next;
    r3.next=node.next;
    node.next=node.next==null?null:node.next.next;
    r3=r3.next;
  }
  return clone;
}
```

## 二叉树

树的dfs用栈实现：（先/中/后）序遍历

树的bfs用队列实现：	层序遍历

#### 1.重建二叉树

描述：

​		输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

思路：

```
 先根据树的两个遍历顺序找出树的根节点，然后拷贝数组进行左右子树的递归遍历,当数组长度为0
时开始从下往上返回树的左右节点（注意：copyOfRange()的取值为左开右闭）
```

```java
public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
        if(pre.length==0||in.length==0) return null;
        TreeNode tree=new TreeNode(pre[0]);//当前根节点肯定为前序遍历的第一个数
        //找出中序序列中root的位置
        int rootVal=0;
        for (int i = 0; i < in.length; i++) {
            if(pre[0]==in[i]){
                rootVal=i;//3 2 0
                break;
            }
        }
  			//根据中序序列的root位置判断左右子树
        tree.left=reConstructBinaryTree(Arrays.copyOfRange(pre,1,rootVal+1),
                              Arrays.copyOfRange(in,0,rootVal));
        tree.right=reConstructBinaryTree(Arrays.copyOfRange(pre,rootVal+1,pre.length),
                               Arrays.copyOfRange(in,rootVal+1,in.length));

        return tree;
    }
```



#### 2.树的子结构

描述：

​	输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

思路：

​	先判断A、B是不是空树，是则直接返回false

​	找到A中和B根节点相等的节点node，递归判断node的左右子树。

​	

```java
 		public boolean HasSubtree(TreeNode root1,TreeNode root2) {
        if(root2 ==null || root1==null) return false;
        boolean result=false;
        if(root2.val==root1.val){
            result=x(root1,root2);
        }
        if(!result){
            result=HasSubtree(root1.left,root2);
        }
        if(!result){
            result=HasSubtree(root1.right,root2);
        }
        return result;
    }

    public boolean x(TreeNode root1,TreeNode root2){
        if(root2==null) return true;
        if(root1==null) return false;
        if(root1.val != root2.val) return false;
        return (x(root1.left,root2.left) && x(root1.right,root2.right));
    }
```



#### 3.二叉树镜像

描述：

​	操作给定的二叉树，将其变换为源二叉树的镜像。

思路：

​	前序/中序/后序交换值，层序遍历用队列

```java
	public void Mirror(TreeNode root) {
        if(root==null) return;
        if(root.right==null && root.left==null) return;        
        TreeNode tem=root.left;
        root.left=root.right;
        root.right=tem;
        if(root.left!=null) Mirror(root.left);
        if(root.right!=null) Mirror(root.right);
    }
```



#### 4.从上向下打印二叉树

描述：

​	从上往下打印出二叉树的每个节点，同层节点从左至右打印。

思路：

​	层序遍历

```java
	public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        ArrayList<Integer> list=new ArrayList();
        if (root==null) return list;
        Queue<TreeNode> queue=new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()){
            TreeNode poll = queue.poll();
            if(poll.left!=null) queue.offer(poll.left);
            if(poll.right!=null) queue.offer(poll.right);
            list.add(poll.val);
        }
        return list;
    }
```



#### 5.二叉搜索树的后序遍历序列

描述：

​	输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。

思路：

​	根据有序和后序遍历的特点找到根节点和左右子树，判断值的大小

```java
		public boolean VerifySquenceOfBST(int [] sequence) {
        if(sequence==null || sequence.length==0) return false;
        return judge(sequence,0,sequence.length-1);
    }

    public boolean judge(int array[],int start,int end){
        if(start>=end) return true;
        int root=array[end];
      	//找到左子树
        int i=start;
        while(array[i]<root) i++;
      	//判断右子树是否比根节点大
        for(int k=i+1;k<end;k++){
            if(array[k]<root) return false;
        }
        boolean left=judge(array,start,i-1);
        boolean right=judge(array,i,end-1);
        return left&&right;
    }
```



#### 6.二叉树中和为某一值的路径

描述：

​	输入一颗二叉树的根节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。

思路：

​	走一个节点就用target减去这个节点的值，递归一直到叶子节点。递归返回时若有符合要求的路径则加入lists

注意：

​	从根到叶子才是一条路径。

```java
private ArrayList<ArrayList<Integer>> lists = new ArrayList<>();
private ArrayList<Integer> list = new ArrayList<>();

public  ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int target) {
  if (root == null) return lists;
  list.add(root.val);
  target -= root.val;
  if (target == 0 && root.left == null && root.right == null)
    lists.add(new ArrayList<Integer>(list));
  FindPath(root.left, target);
  FindPath(root.right, target);
  //用于回退遍历右子树
  list.remove(list.size() - 1);
  return lists;
}
```



#### 二叉搜索树和双向链表

描述：

​	输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。

思路：

​	      5					1 3 4 5 6 8 9

​    3           8

1	4     6      9

​	定义一个节点pre指向遍历时的上一个节点，BST的中序遍历就是有序的，以中序遍历为框架，中间做改变指针的操作，pre的右指针指向当前节点，当前节点的左指针指向pre，改变后pre后移。遍历完成转换。

```java
private TreeNode pre;
private TreeNode real;//存储链表的头结点用于返回

public TreeNode Convert(TreeNode pRootOfTree) {
  bs(pRootOfTree);
  return real;
}
public void bs(TreeNode pRootOfTree) {
  if(pRootOfTree==null) return;
  bs(pRootOfTree.left);

  if(pre==null){
    pre=pRootOfTree;
    real=pRootOfTree;
  }else{
    pre.right=pRootOfTree;
    pRootOfTree.left=pre;
    pre=pRootOfTree;
  }

  bs(pRootOfTree.right);
}
```



#### 8.二叉树的深度

描述：

​	输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。

```java
public int TreeDepth(TreeNode root){
  if(root==null) return 0;
	int left=TreeDepth(root.left);
	int right=TreeDepth(root.right);
  return (left>=right?left:right)+1;
}
```



#### 9.判断平衡二叉树

描述：输入一棵二叉树，判断该二叉树是否是平衡二叉树。

思路：平衡二叉树：左右子树的高度差值<=1

```java

public boolean IsBalanced_Solution(TreeNode root) {
  if(root==null) return true;
  int left=TreeDepth(root.left);
  int right=TreeDepth(root.right);
  if(Math.abs(left-right)<=1) return true;
  return false; 
}

public int TreeDepth(TreeNode root){
  if(root==null) return 0;
	int left=TreeDepth(root.left);
	int right=TreeDepth(root.right);
  return (left>=right?left:right)+1;
}
```



#### 10.二叉树的下一个节点

描述：

​	给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。

思路：

​	（给定的是特定节点，不给根节点）中序遍历的下一个节点分三种情况

​	1.有右孩子							  	 	下一节点是右孩子的最左孩子

​	2.无右孩子且是父节点的左孩子	 下一节点是父节点

​	4.无右孩子且是父节点的右孩子     下一节点是（属于右子树的）（最父节点的）父节点

```java
public class TreeLinkNode {
    int val;
    TreeLinkNode left = null;
    TreeLinkNode right = null;
    TreeLinkNode next = null;//指向父节点

    TreeLinkNode(int val) {
        this.val = val;
    }
}
public TreeLinkNode GetNext(TreeLinkNode pNode){
  	 //请况1
  	 if (pNode.right != null) {
         TreeLinkNode pRight = pNode.right;
         while (pRight.left != null) {
             pRight = pRight.left;
         }
         return pRight;
     }
     //情况2
     if (pNode.next != null && pNode.next.left == pNode) {
         return pNode.next;
     }
     //情况3
     if (pNode.next != null) {
         TreeLinkNode pNext = pNode.next;
         while (pNext.next != null && pNext.next.right == pNext) {
             pNext = pNext.next;
         }
         return pNext.next;
     }
     return null;
}
```



#### 12.对称的二叉树

描述：

​	请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

思路：

​	递归判断left.left, right.right和left.right, right.left

```java
public boolean isSymmetrical(TreeNode pRoot){
  if(pRoot == null){
    return true;
  }
  return check(pRoot.left,pRoot.right);
}
public boolean check(TreeNode left, TreeNode right){
  if(left== null && right == null){
    return true;
  }else if( left == null || right == null || (left.val != right.val)){
    return false;
  }
  return check(left.left, right.right)&&check(left.right, right.left);

}
```



#### 13.之型打印二叉树

描述：

​	请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。

思路：

​	层序遍历，定义一个flag，每一层根据flag交换同层的打印顺序

```java
public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
  ArrayList<ArrayList<Integer>> lists=new ArrayList<>();
  if(pRoot==null) return lists;
  Queue<TreeNode> queue=new LinkedList<>();
  queue.offer(pRoot);
  int count=1;
  while (!queue.isEmpty()){
    ArrayList<Integer> list=new ArrayList<>();
    int size=queue.size();
    for (int i = 0; i < size; i++) {
      TreeNode poll = queue.poll();
      list.add(poll.val);
      if (poll.left != null) queue.offer(poll.left);
      if (poll.right != null) queue.offer(poll.right);
    }
    if(count%2==1)
      lists.add(list);
    else{
      Collections.reverse(list);
      lists.add(list);
    }
    count++;
  }
  return lists;
}
```



#### 14.序列化二叉树

描述：

​	请实现两个函数，分别用来序列化和反序列化二叉树

​	二叉树的序列化是指：把一棵二叉树按照某种遍历方式的结果以某种格式保存为字符串，从而使得内存中建立起来的二叉树可以持久保存。序列化可以基于先序、中序、后序、层序的二叉树遍历方式来进行修改，序列化的结果是一个字符串，序列化时通过 某种符号表示空节点（#），以 ！ 表示一个结点值的结束（value!）。

​	二叉树的反序列化是指：根据某种遍历顺序得到的序列化字符串结果str，重构二叉树。

思路：

​	序列化：   先序遍历 节点之间用！分割，空节点以#表示

​	反序列化：先序遍历先分割字符串拿到节点值和#，得到的字符数组中一个#代表上一个数值节点的左/右子树结束，##代表上一个数值节点为叶子节点。

​	

```java
//先序遍历序列化
public String Serialize(TreeNode root) {
  if(root==null) return "#";
  return root.val+"!"+Serialize(root.left)+"!"+Serialize(root.right);
}

//反序列化
private int index=-1;//标记数组下标
public TreeNode Deserialize(String str){
  if(str.length()==0||str==null) return null;
  String s[]=str.spilt("!");
  TreeNode node = helpDeserialize(split);
  return node;
}
public TreeNode helpDeserialize(String[] strings){
  index++;//指向字符串数组的下一元素
  if(index>=strings.length) return null;
  if(strings[index].equals("#")) return null;
  TreeNode node=new TreeNode(Integer.parseInt(strings[index]));
  node.left=helpDeserialize(strings);
  node.right=helpDeserialize(strings);
  return node;
}
```



#### 15.二叉搜索树的第k个节点

描述：

​	给定一棵二叉搜索树，请找出其中的第k小的结点。例如， （5，3，7，2，4，6，8）    中，按结点数值大小顺序第三小结点的值为4。

思路：

​	就是求中序遍历第k个节点

```java
//dfs
TreeNode node=null;
int count=0;
TreeNode KthNode(TreeNode pRoot, int k){
  if(pRoot==null || k<=0) return null;
  help(pRoot,k);
  return node;
}

void help(TreeNode pnode,int k){
  if(pnode!=null){
    help(pnode.left,k);
    if(++count==k) node=pnode;
    help(pnode.right,k);
  }
}

//栈（中序遍历）
TreeNode KthNode_(TreeNode pRoot, int k) {
  if(pRoot==null) return null;
  if(k<=0) return null;
  Stack<TreeNode> stack=new Stack<>();
  TreeNode node=pRoot;
  while(!stack.isEmpty()|| node!=null){
    if(node!=null){
      //在此处进行操作为先序遍历
      stack.push(node);
      node=node.left;
    }else {
      node = stack.pop();
      if (--k==0) return node;
      node=node.right;
    }
  }
  return null;
}
```

#### 附：树的遍历

##### 1.层序遍历（队列）

思路：

​	加入根节点

​	循环 弹出队列中节点，向队列中加入此节点的左右孩子，直到队列为空

```java
public ArrayList<Integer> level(TreeNode root) {
  ArrayList<Integer> list=new ArrayList();
  if (root==null) return list;
  Queue<TreeNode> queue=new LinkedList<>();
  queue.offer(root);
  while(!queue.isEmpty()){
    TreeNode poll = queue.poll();
    if(poll.left!=null) queue.offer(poll.left);
    if(poll.right!=null) queue.offer(poll.right);
    list.add(poll.val);
  }
  return list;
}
```

##### 2.先序遍历（栈）

思路：

​	打印当前节点，将当前节点左孩子压栈直至无左孩子

​	回退（出栈），将右孩子压栈。

```java
public void pre(TreeNode root){
  Stack stack=new Stack();
  while(!stack.isEmpty() || root!=null){
    if(root!=null){
      print(root.val);
    	stack.push(root.left);
    	root=root.left;
    }else{
    	root= stack.pop();
      root=root.right;
    }  
  }
}
```

##### 3.中序遍历

思路：

​	将当前节点左孩子压栈直至无左孩子

​	回退（出栈），打印节点信息，将右孩子压栈。

```java
public void in(TreeNode root){
  Stack stack=new Stack();
  while(!stack.isEmpty() || root!=null){
    if(root!=null){
      stack.push(root.left);
      root=root.left;
    }else{
      root=stack.pop();
      print(root.val);
      root=root.right;
    }
  }
}
```

##### 4.后序遍历

思路：

​	使用辅助栈s2（因为无法判断回退时是左孩子回退还是右孩子回退）

​	s1先右孩子压栈直至无右孩子，回退，左孩子压栈。s2保存压栈顺序。

​	s2出栈顺序则为后序遍历

```java
public void post(TreeNode root){
  Stack s1=new Stack();
  Stack s2=new Stack();//辅助栈，存储后序遍历的逆序
  while(!stack.isEmpty() || root!=null){
    if(root!=null){
 	  	s1.push(root);
  	  s2.push(root);
      root=root.right;//先右孩子压栈
    }else{
      root=s1.pop();
      root=root.left;//左孩子压栈
    }
  }
  while(!s2.isEmpty()){
		print(s2.pop());//s2出栈顺序则为后序遍历
  }
}
```

## 数组

#### 1.二维数组的查找

描述：在一个二维数组中，（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下的递增顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组是否含有该整数

思路： 利用数组特点，先定位右上角的元素，然后用target与数组元素进行比较，如果target小则 列++，如果target大则 行--；

```java
//初始化二维数组
public static int[][] init(){
  int array[][]=new int[5][5];
  int x=0;
  //        Scanner scanner=new Scanner(System.in);
  for(int m=0;m<5;m++){
    for(int n=0;n<5;n++){
      //                array[m][n]=scanner.nextInt();
      array[m][n]=++x;
    }
  }

  for (int i=0;i<array.length;i++){
    for (int j=0;j<array[i].length;j++){
      if (j<4)
        System.out.print(array[i][j]);
      else {
        System.out.println(array[i][j]);
      }
    }
  }
  return array;
}
//完成的功能函数
public static boolean find(int array[][],int target){
  int row=0;
  int col=array[0].length-1;
  while (row>array.length-1&&col<0) {
    if (target == array[row][col]) {
      return true;
    } else if (target > array[row][col]) {
      row++;

    } else if (target < array[row][col]) {
      col--;
    }
  }
  return false;

}


public static void main(String[] args) {
  int[][] arrays = init();
  boolean b = find(arrays, 88);
  System.out.println(b);

}
```

#### 2.调整数组顺序使奇数位于偶数前面

描述：输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

思路：

```
插入排序（Insertion sort）是一种简单直观且稳定的排序算法。如果有一个已经有序的数据序列，
要求在这个已经排好的数据序列中插入一个数，但要求插入后此数据序列仍然有序，这个时候就要用到一种新的排序方法——插入排序法

让t指定一个数
若满足条件 前一个数后移
直到不满足这个条件 把t放到空位置
```

```java
//int array[]={1,2,3,4,5,6};
public static int[] reOrderArray(int array[]){
  if(array.length==0 || array.length==1) return array;
  for (int i = 0; i < array.length; i++) {
    int temp=array[i];// i=4 t=5
    if(array[i]%2==1){
      int j=i;//j=4        [1,3,2,4,5,6]
      while(j>=1 && array[j-1]%2==0){
        array[j]=array[j-1];//[13 246 ]
        j--;//j=3
      }
      array[j]=temp;
    }
  }

  return array;
}
```

#### 3.顺时针打印矩阵

描述：输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵：1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.

思路：一个循环。先向右，再向下，在想左，再向上（到这一圈的第二行），到最后判断将上下左右的数值进行更改

```java
public static ArrayList<Integer> printMatrix(int [][] matrix) {
  int row=matrix.length,col=matrix[0].length;
  if(row==0 || col==0) return null;
  int num=row*col;
  int x=0,y=0;
  int up=0,tail=row-1,left=0,right=col-1;
  ArrayList<Integer> list=new ArrayList<>(num);
  while(list.size()<num){
    list.add(matrix[x][y]);
    if(x==up){
      if(y<right) y++;
      else if(y==right) x++;
    } else if(y==right){
      if(x<tail) x++;
      else if(x==tail) y--;
    } else if(x==tail){
      if(y>left) y--;
      else if(y==left) x--;
    } else if(y==left){
      if(x>up+1) x--;
      else if(x==up+1){y++;left++;up++;right--;tail--;}
    }
  }
  return list;
}
```

#### 4.数组中出现次数超过一半的数字

描述：数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0

思路：

```
思路2：
    准备两个数 x y
    x为当前的数
    y为x在数组中存在的数量
    若遍历中数与x相等 y++ 若不同 y-- 等y为0 换成数组中下一个数并将此数的y置为1
    循环完 x就是数组中数量最多的数
    在循环计算x的数量
```

```java
//hashMap
public  int MoreThanHalfNum_Solution(int [] array) {
  int len=array.length;
  if(len==0||array==null) return 0;
  if(len==1) return array[0];
  Map<Integer,Integer> map=new HashMap<>();
  for (int i = 0; i < array.length; i++) {

    if (!map.containsKey(array[i])) {
      map.put(array[i], 1);
    } else {
      int count = map.get(array[i]);
      map.put(array[i], ++count);
      if (count > len / 2) return array[i];
    }
  }
  //        Set<Map.Entry<Integer, Integer>> entrySet = map.entrySet();
  //        Iterator<Map.Entry<Integer, Integer>> it=entrySet.iterator();
  //        while(it.hasNext()){
  //            Map.Entry<Integer, Integer> num=it.next();
  //            Integer value = num.getValue();
  //            if(value>len/2) return num.getKey();
  //        }
  return 0;
}


//思路2
public  int MoreThanHalfNum_Solution_2(int [] array){
  if(array.length==0||array==null) return 0;
  if(array.length==1) return array[0];
  int result=1,x=array[0];
  for (int i = 1; i < array.length; i++) {
    if(array[i]==x) result++;
    else {
      result--;
      if(result==0) {
        x=array[i];
        result=1;
      }
    }
  }
  result=0;
  for (int i=0;i<array.length;i++){
    if(array[i]==x) result++;
  }
  if(result>array.length/2) return x;
  return 0;
}

```

#### 5.最小的K个数

描述:输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

```java
public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
  ArrayList<Integer> list=new ArrayList<>(k);
  if(k==0||input.length==0||input==null) return list;
  if(k>input.length) return list;
  for (int i = 0; i < input.length; i++) {
    if(list.size()<k) list.add(input[i]);
    else{
      int max = 0;
      for(int j=0;j<k-1;j++){
        if(list.get(j)>list.get(j+1)){
          max=j;
        }
      }
      if(list.get(max)>input[i]){
        list.remove(max);
        list.add(input[i]);
      }
    }
  }
  return list;
}
```

#### 6.连续子数组的最大和

描述：

```
HZ偶尔会拿些专业问题来忽悠那些非计算机专业的同学。今天测试组开完会后,他又发话了:
在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。
但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？

例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。
给一个数组，返回它的最大连续子序列的和，你会不会被他忽悠住？(子向量的长度至少是1)
```

思路：

```
使用动态规划
  max: 与前一个数和当前数相加后 再与当前数比较的 最大值 （ max相当于累计和 若累积的和大于0 则继续加下个数，若小于0 则放弃前面的数从当前数算起 ）
  res: 存放当前最大子序列和
  如6,-3,-2,7,-15,1,2,2
      初始 max=res=array[0]=6
( max=max(array[i]+max,array[i])  res=max(max,res) )
      i=1 max=max(6-3,-3)=3 res=6
      i=2 max=(3-2,-2)=1    res=6
      i=3 max=(1+7,7)=8     res=8
      i=4 max=(8-15,-15)=-7 res=8
      i=5 max=(-7+1,1)=1    res=8
      i=6 max=(1+2,2)=3     res=8
      i=7 max=(3+2,2)=5     res=8
```

```java
public int FindGreatestSumOfSubArray(int[] array) {
  if(array.length<1 || array==null) return 0;
  int sum=array[0];
  int max=array[0];
  for (int i = 1; i < array.length; i++) {
    max = Math.max(max + array[i], array[i]);
    sum=Math.max(max,sum);
  }
  return sum;
}
```

#### 7.把数组排成最小的数

描述：

```
输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。
例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。
```

思路：将 332（3，32） 和323（32，3）比较，若大则交换3 和 32

```java
public String PrintMinNumber(int [] numbers) {
  if(numbers.length==0||numbers==null) return "";
  if(numbers.length==1) return String.valueOf(numbers[0]);
  StringBuilder sb=new StringBuilder();
  String str[]=new String[numbers.length];
  for (int i = 0; i < numbers.length; i++) {
    str[i]=String.valueOf(numbers[i]);
  }
  String tem;
  for (int i = 0; i < str.length; i++) {
    for(int j=0;j<str.length-1-i;j++){
      if((str[j]+str[j+1]).compareTo((str[j+1]+str[j]))>0){
        tem=str[j];
        str[j]=str[j+1];
        str[j+1]=tem;
      }
    }
  }
  for (int i = 0; i < str.length; i++) {
    sb.append(str[i]);
  }
  return sb.toString();
}
```

#### 8.数组中的逆序对

描述:

```
在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。
输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007
```

```java
public int InversePairs(int [] array) {
  if(array.length<=1 || array==null) return 0;
  int result=0;
  int tem;
  for (int i = 0; i < array.length; i++) {
    tem=array[i];
    for (int j = i+1; j < array.length; j++) {
      if(tem>array[j]) result++;
    }
  }
  return result%1000000007;
}
```

#### 9.数字在排序数组中出现的次数

描述：统计一个数字在排序数组中出现的次数。

思路：有序则二分查找

```java
//利用排序数组，二分查找 找k+0.5 和 k-0.5
public int GetNumberOfK_(int [] array , int k) {
  int result=0;
  if(array.length==0||array==null) return result;
  int left=seek(array,k-0.5);
  int right=seek(array,k+0.5);
  return right-left;
}
public int seek(int array[],double k){
  int low=0;
  int high=array.length-1;
  while(low<high){
    int mid=low+(high-low)/2;
    if(k<array[mid])
      high=mid;
    else
      low=mid;
  }
  return low;
}
```

#### 10.数组中只出现一次的数字

描述：一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

思路：

```
1.数组内异或   异或完的结果为两个只出现一次的数字的异或结果
2.找异或结果最低位为1的位置x
3.将两个数组分为两个子数组   第一个数组x位全为1 第二个数组x位全为0
4.两个数组分别异或 异或完的结果则为所求
```

```java
//num1,num2分别为长度为1的数组。传出参数
//将num1[0],num2[0]设置为返回结果
public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
    int result=0;
    for (int i = 0; i < array.length; i++) {
        result^=array[i];
    }
    int index=1;
    while((index&result)==0)
        index=index<<1;//index标志着异或结果中的最低位为1的位置    10110 index=2
    int result1=0;
    int result2=0;
    for(int i=0;i<array.length;i++){
        if((index&array[i])==0){//分拨 数组中第index位的为0的是一组
            result1^=array[i];
        }else {                 //    数组中第index位的为1的是一组
            result2^=array[i];// ^完就剩下这子数组中只出现一个的数字
        }
    }
    num1[0]=result1;
    num2[0]=result2;
    
}
```

#### 11.数组中重复的数字

描述：

```
在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。
请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。
```

思路：因为所有数字的值都是1-n 所以让数组下标的位置放对应的值 //空间复杂度O(1) 时间复杂度O(n)

```java
public boolean duplicate_(int numbers[],int length,int [] duplication) {
    if(numbers==null||length==0) return false;
    for (int i = 0; i < length; i++) {
        while(numbers[i]!=i){//如果换不到0 则把所有数都放到对应的位置上
            if(numbers[i]==numbers[numbers[i]]){
                duplication[0]=numbers[i];
                return true;
            }else {
                int temp=numbers[i];
                numbers[i]=numbers[temp];
                numbers[temp]=temp;
            }
        }
    }
    return false;
}
```

#### 12.构建乘积数组

描述：

```
给定一个数组A[0,1,...,n-1],请构建一个数组B[0,1,...,n-1],其中B中的元素B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]。
不能使用除法。（注意：规定B[0] = A[1] * A[2] * ... * A[n-1]，B[n-1] = A[0] * A[1] * ... * A[n-2];）
```

思路：

```
时间复杂度O(2n) 将B[i]的值以对角线分为两部分

   A 0 1 2 3 4 5
   0 1
   1   1
   2     1
   3       1
   4         1
   5           1

  下三角(从上往下）      上三角(从下往上)
  B[0]=1          *    A[1]*t[1]=t[0]
  B[1]=A[0]*B[0]  *    A[2]*t[2]=t[1]
  B[2]=A[1]*B[1]  *    A[3]*t[3]=t[2]
  B[3]=A[2]*B[2]  *    A[4]*t[4]=t[3]
  B[4]=A[3]*B[3]  *    A[5]*t[5]=t[4]
  B[5]=A[4]*B[4]  *            1=t[5]

  乘积为B[i]*t[i]
```

```java
public int[] multiply_(int[] A){
    if(A==null) return null;
    if(A.length==0) return new int[0];
    int B[]=new int[A.length];
    B[0]=1;
    for(int i=1;i<A.length;i++){
        B[i]=B[i-1]*A[i-1];
    }
    int tem=1;//从下往上记录A[n-1]*A[n-2]*...*A[0]的值
    for(int i=A.length-2;i>=0;i--){
        tem=tem*A[i+1];
        B[i]*=tem;
    }

    return B;
}
```

## 算法

#### 1.斐波那契数列

描述：大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。

```java
//斐波那契数列（从1开始）
public static int feiBo(int n){
  if(n==1) return 1;
  else if (n==2) return 1;
  else return (feiBo(n-1)+feiBo(n-2));
}

//动态规划的斐波那契数列（从0开始）
private static int feiBo_dong(int n) {
  int i=0,j=1;
  while(n-- > 0){
    j=i+j;
    i=j-i;
  }
  return i;
}

//迭代法
public static int feiBo_die(int n){
  int a=1,b=1,c;
  int i;
  for (i=3;i<=39;i++){
    c=a+b;
    a=b;
    b=c;
    if (n==c) break;
  }
  return i;
}

```

#### 2.跳台阶

描述：一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）

思路：n个台阶 如果跳一个为f(n-1) 如果跳两个为f(n-2) 所以f(n)=f(n-1)+f(n-2)

```java
public static int val(int n){
  if (n==1) return 1;
  if (n==2) return 2;
  else return (val(n-1)+val(n-2));
}
```

#### 3.变态跳台阶

描述：一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

思路：

```
						f(n)   = f(n-1)+f(n-2)+f(n-3)+....+f(n-n)
            f(n-1) = f(n-2)+f(n-3)+....+f(n-n)
做差：  f(n)-f(n-1) =  f(n-1)
            f(n)   =   2*f(n-1)
```

```java
public int sum(int n){
        if(n == 1) return 1;
        else return 2*sum(n-1);
    }
```

#### 4.矩形覆盖

描述：我们可以用2\*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2\*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？

```java
 public static int val(int n){
        if (n==0) return 0;
        if (n==1) return 1;
        if (n==2) return 2;
        else return (val(n-1)+val(n-2));

    }
```

#### 5.二进制中1的个数

描述：输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

思路：

```
*   一个整数如1100 减去1 为 1011
*   按位与运算： 为1000
*   1000 0111 按位与 0000
*   得到的次数为2
*   一个数的二进制有多少个1，就能进行几次这样的运算
*
*   负数：补码就是负数在计算机中的二进制表示方法。
*   如 -5 二进制为 11111011 和上面步骤一样
```

```java
public static int val(int x){
  int count = 0;
  while(x!=0){
    x=x&(x-1);
    count++;
  }
  return count;
}
```

#### 6.数值的整数次方

描述：给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。

思路：

```
 给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。
保证base和exponent不同时为0
    所有可能
        base = 0 ： exponent = 0    不合法
                    exponent < 0    0
                    exponent > 0    0
        base > 0 ： exponent = 0    1
                    exponent < 0    1/base^exponent
                    exponent > 0    base^exponent
        base < 0 ： exponent = 0    1
                    exponent < 0    1/base^exponent (exponent为偶数)
                                    -1/base^exponent(exponent奇数)
                    exponent > 0    base^exponent (exponent为偶数)
                                    -base^exponent(exponent奇数)
    简单快速幂
        传统n*n*n 时间复杂度为O(n)，简单快速幂为O(log n)
        如：a^10，10的二进制为1010
            a^10=a^8 * a^0 * a^2 * a^0
            而 （1010 & 1）计算的结果是二进制最右侧的位是1还是0 是1则乘a*a 是0则a*1
            a*a是保证下一个是正确的基数
            运算后1011右移一位  直到所有位全为0
```

```java
public double pow(double base,int exponent){
  double result=1;
  boolean flag=false;
  int exponent_abs=Math.abs(exponent);
  if(base==0 && exponent== 0) throw new RuntimeException("数据不合法");
  if(base==0) return 0;
  if(exponent==0) return 1;
  if(exponent<0) flag=true;
  while(exponent_abs>0){
    if((exponent_abs & 1) != 0){
      result *= base;
    }
    base*=base;
    exponent_abs>>=1;
  }
  if(base<0 && (exponent_abs%2!=0)) result=-result;
  return flag?1/result:result;
}
```

#### 7.整数中1出现的次数

描述：求出1\~13的整数中1出现的次数,并算出100\~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从1 到 n 中1出现的次数）

思路：

```
一位数  r=1  1
二位数  r=11+8=19
三位数  r=100*1+9*19
四位数  r=1000*1+9*(100*1+9*19)
n/m=4
```

```java
//14ms 9408k
//n=2123
public int NumberOf1Between1AndN_Solution_1(int n) {
  int result=0;
  for (int i = 1; i <= n; i*=10) {
    //	n/(i*10)*i 算的是整数中(2000)的1； Math.min(Math.max(n%(i*10)-i+1,0),i)算的是余数(123)中的1

    //	n/(i*10)：代表整数部分中多少个 位
    //	*i:代表每个 位 中有多少个1
    //	Math.max(n%(i*10)-i+1,0):如果余数中当前位为0 取0  如果大于0，取 余数+1（如 100-123）
    //	Math.min(Math.max(n%(i*10)-i+1,0),i):代表当前位的余数有多少个1(比当前位小，取余数，比当前位大，取整数)

    //	个位的1 i=1     n/10=200  200*1=200     （3-1+1）， 1   =201   (1   是 1-3     中的1）
    //	十位的1 i=10    n/100=20  20*10=200      (21-10+1),10  =210   (10  是 10-20   中的1)
    //	百位的1 i=100   n/1000=2  2*100=200      (123-100+1),100 =224 (24  是 100-123 中的1)
    //	千位的1 i=1000  n/10000=0 0*1000=0  +    (2123-1000+1),1000=1000(1000是1000-2121中的1)
    result+=n/(i*10)*i+Math.min(Math.max(n%(i*10)-i+1,0),i);
  }
  return result;
}


//  暴力破解  39ms 12120k
public int NumberOf1Between1AndN_Solution(int n) {
  int result=0;
  for (int i = 1; i <= n; i++) {
    String s=String.valueOf(i);
    for(int j=0;j<s.length();j++){
      if(s.charAt(j)=='1') result++;
    }
  }
  return result;
}
```

#### 8.丑数

描述：

```
把只包含质因子2、3和5的数称作丑数（Ugly Number）。
例如6、8都是丑数，但14不是，因为它包含质因子7。
习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。
```

思路：

```
动态规划
    包含质因子2、3和5的数肯定为丑数*2 或*3 或*5得到
    1*因子 = 2 3 5

    选最小的2加入数组 t2++
    2*因子=4 6 10
    此时t2=1 指向 4 (array[t2]*2=4)
       t3=0 指向 3
       t5=0 指向 5

    选最小的3加入数组 t3++
    3*因子=6 9 15
    此时t2=1 指向 4
       t3=1 指向 6 (array[t3]*3=6)
       t5=0 指向 5

    选最小的4加入数组 t2++
    4*因子=8 12 20
    此时t2=2 指向 6 (array[t2]*2=6)
       t3=1 指向 6
       t5=0 指向 5


    选最小的5加入数组 t5++
    5*因子=10 12 20
    此时t2=2 指向 6
       t3=1 指向 6
       t5=1 指向 10 (array[t5]*5=10)

 丑数： 2 3 5 4 6 10 6 9 15 8 12 20 10 12 20

 数组： 1 2 3 4 5 6 8 9 10 12
```

```java
public int GetUglyNumber_Solution(int index) {
  if(index<7) return index;
  int array[]=new int[index];
  array[0]=1;
  int t2=0,t3=0,t5=0;//记录三个队列的位置
  for(int i=1;i<index;i++){
    //array[t2]*2 代表的是丑数
    //Math.min(array[t2]*2,Math.min(array[t3]*3,array[t5]*5)); 求出下一个最小的丑数
    array[i]=Math.min(array[t2]*2,Math.min(array[t3]*3,array[t5]*5));
    if(array[i]==array[t2]*2) t2++;
    if(array[i]==array[t3]*3) t3++;
    if(array[i]==array[t5]*5) t5++;
  }
  return array[index-1];
}
```

#### 9.和为S的连续正数序列

描述：

```
小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。但是他并不满足于此,
他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。
没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。
现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列? Good Luck!

输出：
    输出所有和为S的连续正数序列。序列内按照从小至大的顺序，序列间按照开始数字从小到大的顺序
```

思路：

```
思路：
    设置两个指针 左指针先指向第一个数 右指针指向下一个数
    如果 左+右<sum 右++ sum+右指针指向的数
        左+右>sum 左++ sum-左指针指向的数
        左+右=sum 添加从左到右所有的数到list   加完后right++ sum加上right 进行下一次循环
```

```java
public ArrayList<ArrayList<Integer>> FindContinuousSequence(int sum) {
    ArrayList<ArrayList<Integer>> lists=new ArrayList<>();
    if(sum<3) return lists;
    int left=1;
    int right=2;
    int s=left+right;
    while(sum>right){
        if(s<sum){
            right++;
            s+=right;
        }else if(s>sum){
            s-=left;
            left++;
        }else {
            ArrayList<Integer> list=new ArrayList<>();
            for(int i=left;i<=right;i++){
                list.add(i);
            }
            lists.add(list);
            right++;
            s+=right;
        }
    }


    return lists;
}
```

#### 10.和为S的两个数字

描述：

```
//输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。
//乘积最小和相同 第一个数肯定为最小的那个数    2 3 4 5 6 7 8 10 和为10 那乘积最小的为3*7
//对应每个测试案例，输出两个数，小的先输出。
```

```java
public ArrayList<Integer> FindNumbersWithSum(int [] array, int sum) {
    ArrayList<Integer> list=new ArrayList<>();
    if(array.length==0||array==null) return list;
    for (int i = 0; i < array.length; i++) {
        if(array[i]<sum) list.add(array[i]);
    }
    int min,max;
    for (int i = 0; i < list.size(); i++) {
        min=list.get(i);
        for(int j=i+1;j<list.size();j++){
            if(list.get(j)+min==sum) {
                max=list.get(j);
                list.clear();
                list.add(min);
                list.add(max);
                return list;
            }
        }
    }
    list.clear();
    return list;
}

//双指针 O(n)
public ArrayList<Integer> FindNumbersWithSum_(int [] array, int sum) {
    ArrayList<Integer> list=new ArrayList<>();
    if(array.length==0||array==null) return list;
    int low=0;
    int high=array.length-1;
    while(low<high){
        if(array[low]+array[high]==sum){
            list.add(array[low]);
            list.add(array[high]);
            return list;
        }else if(array[low]+array[high]>sum){
            high--;
        }else if(array[low]+array[high]<sum){
            low++;
        }
    }
    return list;
}
```

#### 11.求1+2+3+...+n

描述：求1+2+3+...+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）

```java
public static int Sum_Solution(int n) {
    if(n==1) return 1;
    return Sum_Solution(n-1)+n;
}
```

#### 12.不用加减乘除做加法

描述：写一个函数，求两个整数之和，要求在函数体内不得使用+、-、*、/四则运算符号。

思路：

```
正确的加法计算：11+01 = 100
使用位运算实现二位加法：
    按位加法： res1 = 11 ^ 01 = 10
    与运算进位： res2 = (11 & 01) << 1 = ( 01 ) << 1 = 010
    res1 ^ res2 = 10 ^ 010 = 00
    (10 & 10) << 1 = 100
```

```java
public static int Add(int num1,int num2) {
    int result=0;
    int carry=0;
    do{
        result=num1^num2;//不算进位相加  5+7=2
        carry=(num1&num2)<<1;//进位
        num1=result;
        num2=carry;
    }while ((carry!=0));
    return result;
}
```

## 字符串

#### 1.字符串的排列

描述：

```
输入一个字符串,按字典序打印出该字符串中字符的所有排列。
例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

数量：设总数为n 相同字符数为x
     所排列出的总数：Ann/Axx
```

思路：回溯法

```java
public ArrayList<String> Permutation(String str) {
    ArrayList<String> list=new ArrayList<>();
    if(str.length()!=0 && str!=null){
        x(str.toCharArray(),0,list);
    }
    Collections.sort(list);//保证字典序打印
    return list;
}
public void x(char chars[],int i,ArrayList<String> list){
    if(i==chars.length-1){
        String str=String.valueOf(chars);
        if(!list.contains(str)) list.add(str);
    }else {
        for(int j=i;j<chars.length;j++){
            swap(chars,i,j);// 1 2  a c b     0 1 b a c  1 2 b c a
            x(chars,i+1,list);//            1 1
            swap(chars,i,j);// 1 2  a b c
        }
    }

}
public void swap(char chars[],int i,int j){
    if(i!=j){
        char c=chars[i];
        chars[i]=chars[j];
        chars[j]=c;
    }
}
```

#### 2.第一个只出现一次的字符位置

描述：在一个字符串(0<=字符串长度<=10000，全部由字母组成)中找到第一个只出现一次的字符,并返回它的位置, 如果没有则返回 -1（需要区分大小写）

思路：

```
    hash思想
       str全部由字符组成，a-z A-Z
       用int数组存放每一个字符的数量
       循环一次 遇到一个字符 字符数量+1
       然后循环判断 数组中的元素个数为1时，输出此时的i

       数组长度为58而不是52的原因 a-z是97-122 A-Z是65-90   其中91-96是其他字符 为了不另判断是大写还是小写，统一减去65
 
```

```java
public int FirstNotRepeatingChar(String str) {
    int words[]=new int[58];
    for (int i = 0; i < str.length(); i++) {
        words[str.charAt(i)-65]++;
    }
    for (int i = 0; i < str.length(); i++) {
        if(words[str.charAt(i)-65]==1) return i;
    }

    return -1;
}
```

#### 3.左旋转字符串

描述：

```
汇编语言中有一种移位指令叫做循环左移（ROL），现在有个简单的任务，就是用字符串模拟这个指令的运算结果。
对于一个给定的字符序列S，请你把其循环左移K位后的序列输出。
例如，字符序列S=”abcXYZdef”,要求输出循环左移3位后的结果，即“XYZdefabc”。是不是很简单？OK，搞定它！
```

```java
public String LeftRotateString(String str,int n) {
    if(str.length()==0||str==null) return str;
    String substring = str.substring(0, n);
    String substring2 = str.substring(n, str.length());
    str=substring2+substring;
    return str;
}

public String LeftRotateString_(String str,int n) {
    if(str.length()==0||str==null) return str;
    char[] chars = str.toCharArray();
    char[] temp=new char[n-1];
    for (int i = 0; i < temp.length; i++) {
        temp[i]=chars[i];
    }
    for (int i = n; i < chars.length; i++) {
        chars[i-n]=chars[i];
    }
    for (int i = chars.length-n; i < chars.length; i++) {
        chars[i]=temp[i-(chars.length-temp.length)];
    }
    str=new String(chars);
    return str;
}
```

#### 4.翻转单词顺序列

描述：

```
牛客最近来了一个新员工Fish，每天早晨总是会拿着一本英文杂志，写些句子在本子上。同事Cat对Fish写的内容颇感兴趣，
有一天他向Fish借来翻看，但却读不懂它的意思。例如，“student. a am I”。后来才意识到，这家伙原来把句子单词的顺序翻转了，
正确的句子应该是“I am a student.”。Cat对一一的翻转这些单词顺序可不在行，你能帮助他么？
```

```java
public static String ReverseSentence(String str) {
    if(str.trim().length()==0||str==" "||str==null) return str;

    String[] s1 = str.split(" ");
    String s="";
    for (int i = s1.length-1; i >=0;i--) {
        s+=s1[i]+" ";
    }
    return s;
}
```

#### 5.扑克牌顺子

描述：

```
LL今天心情特别好,因为他去买了一副扑克牌,发现里面居然有2个大王,2个小王(一副牌原本是54张^_^)...
他随机从中抽出了5张牌,想测测自己的手气,看看能不能抽到顺子,如果抽到的话,他决定去买体育彩票,嘿嘿！！“
红心A,黑桃3,小王,大王,方片5”,“Oh My God!”不是顺子.....LL不高兴了,他想了想,决定大\小 王可以看成任何数字,
并且A看作1,J为11,Q为12,K为13。上面的5张牌就可以变成“1,2,3,4,5”(大小王分别看作2和4),“So Lucky!”。LL决定去买体育彩票啦。
 现在,要求你使用这幅牌模拟上面的过程,然后告诉我们LL的运气如何，
如果牌能组成顺子就输出true，否则就输出false。为了方便起见,你可以认为大小王是0。
```

```java
//思路：看有几张大王 再看数能不能连在一起
public static boolean isContinuous(int [] numbers) {
    if(numbers==null||numbers.length!=5) return false;
    int max=-1;
    int min=14;
    //数组下标表示牌 数组的元素值表示数量
    int d[]=new int[14];
    //把大王小王的值设置为5
    d[0]=-5;
    for (int i = 0; i < numbers.length-1; i++) {
        d[numbers[i]]++;
        //如果有两张相同的牌，返回false
        if(d[numbers[i]]>1) return false;
        if(numbers[i]==0){
            continue;
        }
        if(numbers[i]>max){
            max=numbers[i];
        }
        if(numbers[i]<min){
            min=numbers[i];
        }
    }

    int shun=max-min;
    if(shun<5) return true;
    return false;
}
```

#### 6.把字符串转换为整数

描述：

```
将一个字符串转换成一个整数，要求不能使用字符串转换整数的库函数。 数值为0或者字符串不是一个合法的数值则返回0

输入描述：输入一个字符串,包括数字字母符号,可以为空
输出描述：如果是合法的数值表达则返回该数字，否则返回0
```

```java
public static int StrToInt(String str) {
    if(str==null||str.length()==0) return 0;
    int target=0;
    boolean flag=true;
    int i=0;
    //如果第一位是符号，记录下符号并判断下一个字符
    if(str.charAt(0)=='+'||str.charAt(0)=='-'){
        if(str.charAt(0)=='-') flag=false;
        i++;
    }
    for (; i < str.length(); i++) {
        //如果字符串部分为0-9 进行累加运算
        if(str.charAt(i)>='0'&&str.charAt(i)<='9'){
            int x= (int)str.charAt(i)-48;
            target=target*10+x;//target=(target<<1)+(target<<3)+x
        }else {//若出现异常字符 返回0
            return 0;
        }
    }
    if(flag){
        //当值为正数时 算出的结果却小于0 说明已经超出整型最大值（2147483647+1=-2147483648）
        if(target<0) return 0;
        return target;
    }else {
        //当值为负数时 算出的结果却大于0 说明已经超出整型最小值（-2147483648-1=2147483647）
        if(-target>0) return 0;
        return -target;
    }
}
```

#### 7.字符流中第一个不重复的字符

描述：

```
请实现一个函数用来找出字符流中第一个只出现一次的字符。例如，当从字符流中只读出前两个字符"go"时，第一个只出现一次的字符是"g"。
当从该字符流中读出前六个字符“google"时，第一个只出现一次的字符是"l"。

输出描述：如果当前字符流没有存在出现一次的字符，返回#字符。
```

思路：字符与数量对应用hash()

```java
//空间为n 时间为1 因为不会循环超过128 所以时间复杂度为常数级
int chars[]=new int[128];//一共128个字符
Queue<Character> queue=new LinkedList<>();
public void Insert_2(char ch) {
    if(chars[ch-' ']==0) queue.offer(ch);
    chars[ch-' ']++;
}

public char FirstAppearingOnce_2() {
    while(!queue.isEmpty()){
        if(chars[queue.element()-' ']>1){
            queue.poll();
        }else if(chars[queue.element()-' ']==1){
            return queue.peek();
        }
    }
    return '#';
}
```

#### 8.替换空格

描述：请实现一个函数，将一个字符串中的每个空格替换成"%20"。例如，当字符串为We are happy 则经过替换之后的字符串为We%20are%20happy

思路：用springBuilder的append和string的charAt

```java
public static String exchange(String str){
    if (str == null){
        return null;
    }
    StringBuilder sb=new StringBuilder();
    for(int i=0;i<str.length();i++){
        if(str.charAt(i) == ' '){
            sb.append("%20");
        }else{
            sb.append(str.charAt(i));
        }
    }
    return sb.toString();
}
```