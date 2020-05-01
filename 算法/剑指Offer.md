### 剑指Offer

#### 链表

##### 1.输入一个链表，按链表从尾到头的顺序返回一个ArrayList。

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

##### 2.用两个栈来实现一个队列

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

##### 3.输入一个链表，输出该链表中倒数第k个结点。

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

##### 4.输入一个链表，反转链表后，输出新链表的表头。

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



##### 5.合并两个有序的链表（递增）

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



##### 6.栈的压入弹出序列

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



##### 7.链表中的第一个公共节点

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

##### 8.链表中环的入口节点

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

##### 9.删除链表中重复的节点

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



##### 10.包含min函数的栈

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



##### 11.复杂链表的复制

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





#### 二叉树

树的dfs用栈实现：（先/中/后）序遍历

树的bfs用队列实现：	层序遍历

##### 1.重建二叉树

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



##### 2.树的子结构

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



##### 3.二叉树镜像

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



##### 4.从上向下打印二叉树

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



##### 5.二叉搜索树的后序遍历序列

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



##### 6.二叉树中和为某一值的路径

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



##### 二叉搜索树和双向链表

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



##### 8.二叉树的深度

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



##### 9.判断平衡二叉树

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



##### 10.二叉树的下一个节点

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



##### 12.对称的二叉树

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



##### 13.之型打印二叉树

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



##### 14.序列化二叉树

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



##### 15.二叉搜索树的第k个节点

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

##### 附：树的遍历

###### 1.层序遍历（队列）

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

###### 2.先序遍历（栈）

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



###### 3.中序遍历

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



###### 4.后序遍历

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



#### 字符串



#### 数组



