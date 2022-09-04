# 不相交集ADT（并查集）
## 动态等价问题

 - 输入数据最初是N个集合的类，每个集合含有一个元素。初始的描述是所有的关系均为false（自反的关系除外）。每个集合都有一个不同的元素，从而Si∩Sj = ∅。
 - 此时有两种运算允许进行。第一种是**Find**，**它返回包含给定元素的集合（即等价类）的名字**。第二种运算是**添加关系**。如果想要添加关系a~b，那么**首先**要看是否a和b已经有关系，**这**可以通过对a和b执行Find并检验它们是否在同一个等价类中来完成。如果它们不在同一类中，那么我们使用求并运算Union，这种运算把含有a和b的两个等价类合并成一个新的等价类。**∪的结果是建立一个新集合Sk = Si∪Sj，去掉原来两个集合而保持所有的集合的不相交性**。
 - 该算法是**动态的**，因为在算法执行过程中，集合可以通过Union运算发生改变。这个算法还必然是**联机操作**：当Find执行时，它必须给出答案算法才能继续进行。另一种可能是**脱机算法**：它需要观察全部的Union和Find序列。它对每个Find给出的答案必须和所有执行到该Find的Union一致，而该算法在看到所有的问题以后再给出它的所有的答案。
 - **解决动态等价问题**的方案有两种。一种方案是保证指令Find能够以常数最坏情形运行时间执行，而另一种方案则保证指令Union能够以常数最坏情形运行时间执行。
## 基本数据结构
 - 我们的问题不要求Find操作返回任何特定的名字，而只是要求当且仅当两个元素属于相同的集合时，作用在这两个元素上的Find返回相同的名字。一种想法是可以用树来表示每一个集合，因为书上的每一个元素都有相同的根，**该根可以用来命名所在的集合**。
 - **为了执行两个集合的Union运算，我们使一个节点的根指针指向另一棵树的根节点，这个操作花费常数时间。其中，Union（X,Y）后新的根是X的约定。**
 - 对元素X的一次Find(X)操作通过返回包含X的树的根而完成。执行这次操作花费的时间与表示X的节点的深度成正比，前提是假设以常数时间找到表示X的节点。**如此能建立一棵深度为N-1的树，使得一次Find的最坏情形运行时间是O(N)。**一般情况下，运行时间是对连续混合使用M个指令来计算的。在这种情况下，M次连续操作在最坏情形下可能花费O(M*N)时间。
 - 并查集基本实现：
 

	    #pragma once
		#ifndef _DisjSet_H
		constexpr auto NumSets = 10;
		typedef int DisjSet[NumSets + 1];
		typedef int SetType;
		typedef int ElementType;
		void Initialize(DisjSet S);
		void SetUnion(DisjSet S, SetType Root1, SetType Root2);
		SetType Find(ElementType X, DisjSet S);
		#endif
		#include"DisjSet.h"
		void Initialize(DisjSet S) {
			int i;
			for (i = NumSets;i > 0;i--) {
				S[i] = 0;
			}
		}

		void SetUnion(DisjSet S, SetType Root1, SetType Root2) {
			S[Root2] = Root1;
		}
		//Find操作返回的是包含X的树的根
		SetType Find(ElementType X, DisjSet S) {
			if (S[X] <= 0)
				return X;
			else
				return Find(S[X], S);
		}

 - 上面的Union的执行是相当任意的，它通过使第二棵树成为第一棵的子树而完成合并。**对其进行简单改进是借助任意的方法打破现有关系，使得总让较小的树成为较大的树的子树，该方法叫作按大小求并**。**如果没有对大小进行探测而直接执行Union，那么结果将会形成更深的树**。如果这些Union都是按照大小进行的，那么任何节点的深度均不会超过log N。Find操作的运行时间是O(log N)，而连续M次操作则花费O(M*log N)。
 - **实现按大小求并的需要**记住每一棵树的大小。由于实际上只使用了一个数组，因此可以让每个根的数组元素包含它的树的大小的负值，这样初始时树的数组表示就都是-1了。当执行一次Union时要检查树的大小，新的大小是老的大小的和。，**已证明若使用按大小求并则连续M次运行需要O(M)平均时间**，是因为当随机的诸Union执行时整个算法一般只有一些很小的集合（通常含一个元素）与大集合合并。
 - 另一种方法：按高度求并，它同样保证所有树的深度最多是O(log N)，我们**跟踪每棵树的高度而不是大小**并执行那些Union使得浅得树成为深的树的子树。

 ## 路径压缩
 

 - 迄今所描述的Union/Find算法对于大多数情形都是完全可接受的（对于连续M个指令（在所有模型下）平均是线性的）。不过，O(M*log N)的最坏情形还是可能相当容易并发生的。如果运算Find比Union多很多，那么其运行时间就比快速查找算法的运行时间要糟，而且对于Union算法没有更多改进的可能。因此，对Find操作会更可靠。
 - **改进Find之路径压缩**。路径压缩在一次Find操作期间执行，而与用来执行Union的方法无关。（Find返回的是包含X的树的根）。

 - **方法**：

		SetType Find(ElementType X, DisjSet S) {
			if (S[X] <= 0)
				return X;
			else
				return S[X] = Find(S[X], S);
		}
	 S[X]等于由Find返回的值；这样在集合的根被递归地找到以后，X就直接指向它。对通向根的路径上的每一个节点这将递归地出现，从而实现路径压缩。

# 练习

 - 回顾复习散列表

		#include <malloc.h>
		#define HASHSIZE 12
		#define NULLKEY -32768

		typedef struct {
			int* elem;
			int count;
		}HashTable;

		int Init(HashTable* H) {//用的是指针
			H->count = HASHSIZE;
			H->elem = (int*)malloc(HASHSIZE * sizeof(int));
			if (!H->elem)
				return -1;
			for (int i = 0;i < HASHSIZE;i++) {
				H->elem[i] = NULLKEY;//让表中先存入不可能出现的数据
			}
			return 0;
		}
		int Hash(int key) {
			return key % HASHSIZE;
		}
		//插入关键字到散列表
		void InsertHash(HashTable* H, int key) {
			int addr;
			addr = Hash(key);
			while (H->elem[addr] != NULLKEY) {//如果不为空，则冲突出现
				addr = (addr + 1) % HASHSIZE;//每次加1取余//开放定址法的线性探测
			}
			H->elem[addr] = key;
		}
		//散列表查找关键字
		int SearchHash(HashTable H, int key, int* addr) {
			*addr = Hash(key);
			while (H.elem[*addr] != key) {
				*addr = (*addr + 1) % HASHSIZE;
				if (H.elem[*addr] == NULLKEY || *addr == Hash(key))//查到最后一个元素和循环回到原点，说明不存在这个元素
					return -1;
			}
			return 0;
		}

		 //使用哈希表，可以将寻找 target - x 的时间复杂度降低到从 O(N)O(N) 降低到 O(1)O(1)。
		//
		//这样我们创建一个哈希表，对于每一个 x，我们首先查询哈希表中是否存在 target - x，
		//然后将 x 插入到哈希表中，即可保证不会让 x 和自己匹配。

		struct hashTable {
		    int key;//键
		    int val;//值
		    UT_hash_handle hh;//使定义的结构具有 “哈希性” ，它可以命名为任何名称。（这一步必须有！！！）
		};//自定义数据结构

		struct hashTable* hashtable;////定义哈希指针，指向前面的 hashTable 数据结构

		struct hashTable* find(int ikey) {//通过key值来查找
		    struct hashTable* tmp;////定义指向 hashTable 的指针变量 tmp
		    HASH_FIND_INT(hashtable, &ikey, tmp);//第一个参数 hashtable 是哈希表，第二个参数是 ikey 的地址（一定要传递地址）。
		    //最后 tmp 是输出变量。当可以在哈希表中找到相应键值时，tmp 返回给定键的结构，当找不到时 tmp 返回NULL。
		    return tmp;
		}

		void insert(int ikey, int ival) {
		    struct hashTable* it = find(ikey);//返回ikey值对应的节点it
		    if (it == NULL) {//如果没有找到，则创建项目添加
		        struct hashTable* tmp = malloc(sizeof(struct hashTable));//为 tmp 申请一个空间，
		                                                                  //进行动态分配
		        tmp->key = ikey, tmp->val = ival;//将结构体变量的键指向 ikey，将值指向 ival
		        HASH_ADD_INT(hashtable, key, tmp);//向 hashtable 添加元素
		        //第一个参数 hashtable 是哈希表，第二个参数 key 是键字段的名称。最后一个参数 tmp 是指向要添加的结构的指针。
		    }
		    else {//如果能找到，则替换即可
		        it->val = ival; //将值val 替换成 ival
		    }
		}

		int* twoSum(int* nums, int numsSize, int target, int* returnSize) {
		    hashtable = NULL;//初始化
		    for (int i = 0; i < numsSize; i++) {
		        struct hashTable* it = find(target - nums[i]);//寻找满足条件的数据
		        if (it != NULL) {//如果找到
		            int* ret = malloc(sizeof(int) * 2);//为 ret 申请一个空间，进行动态分配
		            ret[0] = it->val, ret[1] = i;//ret[0]为target-nums[i]对应的元素下标，ret[1]即为 
		                                          //nums[i]对应元素下标

                *returnSize = 2;//设置返回数组的长度为 2
                return ret;
            }
            insert(nums[i], i);//如果没有找到，就将其插入进哈希表
        }
        *returnSize = 0;//遍历所有数据后仍未找到匹配元素，则设置返回数组的长度为 0
        return NULL;
    }

 - 省份数量（并查集）

		void Init(int* parent, int Size) {
			for (int i = 0;i < Size;i++) {
				parent[i] = i;
			}
		}//初始化
		//因为这里已经初始化过了，如果if语句中的内容成立，说明该节点不是根节点，则继续向上遍历，直到找到根节点
		int Find(int* parent, int index) {
			if (parent[index] != index) {
				parent[index] = Find(parent, parent[index]);
			}
			return parent[index];
		}
		//合并是两根节点合并
		void Union(int* parent, int Element1, int Element2) {
			//parent[Element1] = Element2; //此写法也对
			parent[Find(parent, Element1)] = Find(parent, Element2);
		}
		int findCircleNum(int** isConnected, int isConnectedSize, int* isConnectedColSize) {
			int cities = isConnectedSize;//相当于代表数组大小
			int parent[cities];
			Init(parent, cities);//初始化
			for (int i = 0;i < cities;i++) {
				for (int j = 0;j < cities;j++) {
					if (isConnected[i][j] == 1)//说明两城市相连
						Union(parent, i, j);//则合并
				}
			}
			int provinces = 0;
			for (int i = 0;i < cities;i++) {
				if (parent[i] == i) {//利用初始化的条件。
					provinces++;
				}
			}
			return provinces;
		}

 - 搜索插入位置

	     //二分查找又称折半查找，适合对已经排好序的数据集合进行查找，时间复杂度为O(log2n),效率高。
	    int searchInsert(int* nums, int numsSize, int target) {
	    	int begin = 0, end = numsSize - 1;
	    	int ans = numsSize;
    
    	   while (begin <= end) {
	    		int mid = ((end - begin) >> 1) + begin;//这样写或(end-begin)/2+begin都是为了防止数组越界
	    		if (nums[mid] >= target) {/即便nums[mid]等于了target仍要继续进行，因为nums[mid] == target不是判断程序结束的标志
	    			ans = mid;//ans实际上没什么用
	    			end = mid - 1;
	    	}
	    		if (nums[mid] < target) {
	    			begin = mid + 1;
    		}
 
    	}
    	return ans;
	    }

 - 合并两个排序的链表
 

	    struct ListNode {
	        int val;
	        struct ListNode *next;
	    };
	    
	    
	    
	    struct ListNode* mergeTwoLists(struct ListNode* l1, struct ListNode* l2) {
	        struct ListNode* head = malloc(sizeof(struct ListNode));//对于新创建的链表要申请空间
	        struct ListNode* l3 = head;//这里不用dummyHead，本解法不用头节点存数据。
	        while (l1 != NULL && l2 != NULL) {
	            if (l1->val > l2->val) {
	                l3->next = l2;
	                l2 = l2->next;
	            }
	            else {
	                l3->next = l1;
	                l1 = l1->next;
	            }
	            l3 = l3->next;
	        }
	        l3->next = l1 == NULL ? l2 : l1;
	        return head->next;
	    }

 - 多项式ADT的数组实现

	     #include<iostream>
	    using namespace std;
	    constexpr auto MaxDegree = 10;;
	    typedef struct
	    {
	        int CoeffArray[MaxDegree + 1];//各个多项式的系数A，对应位置存对应幂。
	        int HighPower;  //最高的幂ni
	    } *Polynomial;
	    //将多项式初始化为0的过程
	    void ZeroPolynomial(Polynomial Poly) {//需要一个多项式，才能初始化
	        int i;
	        for (i = 0;i < MaxDegree + 1;i++) {
	            Poly->CoeffArray[i] = 0;
	        }
	        Poly->HighPower = 0;
	    }
	    //多项式相加
	    void AddPolynomial(const Polynomial Poly1, const Polynomial Poly2, Polynomial PolySum) {
	        int i;
	        ZeroPolynomial(PolySum);//Poly1, Poly2不需要初始化，因为这是人为输入的。
	        PolySum->HighPower = max(Poly1->HighPower, Poly2->HighPower);
	        for (i = PolySum->HighPower;i >= 0;i--) {
	            PolySum->CoeffArray[i] = Poly1->CoeffArray[i] + Poly2->CoeffArray[i];
	        }
	    }
	    //多项式相乘
	    void AddPolynomial(const Polynomial Poly1, const Polynomial Poly2, Polynomial PolyProd) {
	        int i, j;
	        ZeroPolynomial(PolyProd);
	        PolyProd->HighPower = Poly1->HighPower + Poly2->HighPower;//乘法
	        if (PolyProd->HighPower > MaxDegree) {
	            cout << "Exceeded array size" << endl;
	        }
	        else {
	            for (i = 0;i < Poly1->HighPower;i++) {
	                for (j = 0;j < Poly2->HighPower;j++) {
	                    PolyProd->CoeffArray[i + j] += Poly1->CoeffArray[i] * Poly2->CoeffArray[j];
	                }
	            }
	        }
	    }

 - x的平方根

	    int mySqrt(int x) {
	    	int left, right;
	    	left = 0;right = x;
	    	int ans = -1;
	    	while (left <= right) {
	    		int mid = (right - left) / 2 + left;
	    		if ((long long)mid * mid <= x) {
	    			ans = mid;//由于<的情况也有可能找到ans，所以就保存该middle值
	    			//当k*k<x，那么说明k可能是x的算术平方根（题目的要求小数点的数会被舍掉，所以说，k * k< x是有可能成为x的算术平方根的值）；
	    			left = mid + 1;
	    		}
	    		else
	    			right = mid - 1;
	    	}
	    	return ans;
	    }

 - 有效的括号（栈）

	    #include<string.h>
	    char pairs(char s) {
	    	if (s == ']')return '[';
	    	if (s == '}')return '{';
	    	if (s == ')')return '(';
	    	return 0;
	    }
	    
	    bool isValid(char* s) {
	    	int size = strlen(s);
	    	//strlen 是一个函数，它用来计算指定字符串 str 的长度，但不包括结束字符（即 null 字符）。
	    	int stk[size + 1], top = 0;
	    	if (size % 2 == 1)
	    		return false;
	    	for (int i = 0;i < size;i++) {
	    		int ch = pairs(s[i]);
	    		if (ch) {
	    			if (top == 0 || stk[top - 1] != ch)//非空栈中的栈顶指针始终在栈顶元素的下一个位置。
	    				//条件是栈中没有存入元素，或者是栈中的元素不等于要查的元素。
	    				return false;
	    			top--;
	    		}
	    		else {
	    			stk[top++] = s[i];//当出现的是左半部分时，将它存入栈中。
	    		}
	    	}
	    	return top == 0;//程序执行完成后，如果栈为空
	    }
	    //栈：空栈中的栈顶指针的指向和栈底指针一致，非空栈中的栈顶指针始终在栈顶元素的下一个位置。

 - 二叉树的中序遍历
 

	    void inorderTraversal(struct TreeNode* root, int* res, int* resSize) {
		 if (!root)
			 return;
		 inorderTraversal(root->left, res, resSize);
		 res[(*resSize)++] = root->val;//更新数组。
		 inorderTraversal(root->right, res, resSize);
		}
		int* inorderTraversal(struct TreeNode* root, int* returnSize) {
			int* res = malloc(sizeof(int) * 100);
			*returnSize = 0;
			inorderTraversal(root, res, returnSize);
			return res;//返回一个指针
		}

 - 二叉树的前序遍历

	    void preorder(struct TreeNode* root, int*res,int* resSize) {
	    	 if (!root)
	    		 return;
	    	 res[(*resSize)++] = root->val;
	    	 preorder(root->left, res, resSize);
	    	 preorder(root->right, res, resSize);
	    }
	    
	    int* preorderTraversal(struct TreeNode* root, int* returnSize) {
	    	int* res = malloc(sizeof(int) * 100);
	    	*returnSize = 0;
	    	preorder(root, res, returnSize);
	    	return res;
	    }

 - 二叉树的后序遍历

	    void postorder(struct TreeNode* root, int*res,int* resSize) {
	    	 if (!root)
	    		 return;
	    	 postorder(root->left, res, resSize);
	    	 postorder(root->right, res, resSize);
	    	 res[(*resSize)++] = root->val;
	    }
	    
	    int* postorderTraversal(struct TreeNode* root, int* returnSize) {
	    	int* res = malloc(sizeof(int) * 100);
	    	*returnSize = 0;
	    	postorder(root, res, returnSize);
	    	return res;
	    }
	    //前序遍历：根->左->右
	    //中序遍历：左->根->右
	    //后序遍历：左->右->根

 - 回文链表

	     bool isPalindrome(struct ListNode* head) {
	    	if (head == NULL)
	    		return NULL;
	    	int arr[100000], i = 0;
	    	while (head != NULL) {
	    		arr[i++] = head->val;
	    		head = head->next;
	    	}//先把链表中的元素转移到数组中，这里不用指针存储是因为指针出了该循环之后会被释放掉。
	    	for (int k = 0, l = i - 1;k < l;k++, l--) {
	    		if (arr[k] != arr[l])
	    			return false;
	    	}
	    	return true;
	    }

 - 存在重复元素

	    #include <cstddef>
	    #include <malloc.h>
	    struct hashTable {
	        int key;
	        UT_hash_handle hh;
	    };
	    
	    bool containsDuplicate(int* nums, int numsSize) {
	        struct hashTable* set = NULL;
	        for (int i = 0; i < numsSize; i++) {
	            struct hashTable* tmp;
	            HASH_FIND_INT(set, nums + i, tmp);//第一个参数 hashtable 是哈希表，第二个参数是 ikey 的地址（一定要传递地址）。
	         //最后 tmp 是输出变量。当可以在哈希表中找到相应键值时，tmp 返回给定键的结构，当找不到时 tmp 返回NULL。
	            if (tmp == NULL) {
	                tmp = malloc(sizeof(struct hashTable));
	                tmp->key = nums[i];
	                HASH_ADD_INT(set, key, tmp);//向 hashtable 添加元素
	             //第一个参数 hashtable 是哈希表，第二个参数 key 是键字段的名称。最后一个参数 tmp 是指向要添加的结构的指针。
	            }
	            //如果不为空，则说明了该元素重复了。
	            else {
	                return true;
	            }
	        }
	        return false;
	    }


