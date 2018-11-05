作者：xiaoding133  
来源：CSDN  
原文：https://blog.csdn.net/xiaoding133/article/details/8037086  
版权声明：本文为博主原创文章，转载请附上博文链接！

---

问题：查找大量无序元素中最大的 K 个数。

### 方法一：常规解法,先排序(时间复杂度为 O(N\*logN))

该解法是大部分能想到的，也是第一想到的方法。假设数据量不大，可以先用快速排序或堆排序，他们的平均时间复杂度为 O(N*logN),然后取出前 K 个，时间复杂度为 O(K)，总的时间复杂度为 O(N*logN)+O(K).

当 K=1 时，上面的算法的时间复杂度也是 O(N*logN)，上面的算法是把整个数组都进行了排序，而原题目只要求最大的 K 个数，并不需要前 K 个数有限，也不需要后 N-K 个数有序。可以通过部分排序算法如选择排序和交换排序，把 N 个数中的前 K 个数排序出来，复杂度为 O(N*K),选择哪一个，取决于 K 的大小，在 K(K<logN)较小的情况下，选择部分排序。

### 方法二：利用快速排序原理(时间复杂度 O(N\*logK)(掌握)

(掌握)避免对前 K 个数进行排序来获取更好的性能(利用快速排序的原理)。假设 N 个数存储在数组 S 中，从数组中随机找一个元素 X，将数组分成两部分 Sa 和 Sb.Sa 中的元素大于等于 X,Sb 中的元素小于 X。

出现如下两种情况：

- (1)若 Sa 组的个数大于或等于 K，则继续在 sa 分组中找取最大的 K 个数字 。
- (2)若 Sa 组中的数字小于 K ，其个数为 T，则继续在 sb 中找取 K-T 个数字 。

一直这样递归下去，不断把问题分解成小问题，平均时间复杂度为 O(N\*logK)。

代码如下：

```
/*将数组a[s]...a[t]中的元素用一个元素划开，保存中a[k]中*/
void partition(int a[], int s, int t, int & k) {
	int i, j, x;
	x = a[s]; //取划分元素
	i = s; //扫描指针初值
	j = t;
	do {
		while ((a[j] < x) && i < j) j--; //从右向左扫描,如果是比划分元素小，则不动
		if (i < j) a[i++] = a[j]; //大元素向左边移
		while ((a[i] >= x) && i < j) i++; //从左向右扫描，如果是比划分元素大，则不动
		if (i < j) a[j--] = a[i]; //小元素向右边移

	} while (i < j); //直到指针i与j相等
	a[i] = x; //划分元素就位
	k = i;
}
/*查找数组前K个最大的元素，index:返回数组中最大元素中第K个元素的下标(从0开始编号),high为数组最大下标*/
int FindKMax(int a[], int low, int high, int k) {
	int q;
	int index = -1;
	if (low < high) {
		partition(a, low, high, q);
		int len = q - low + 1; //表示第几个位置
		if (len == k)
			index = q; //返回第k个位置
		else if (len < k)
			index = FindKMax(a, q + 1, high, k - len);
		else
			index = FindKMax(a, low, q - 1, k);
	}
	return index;

}
int main() {
	int a[] = {
		20,
		100,
		4,
		2,
		87,
		9,
		8,
		5,
		46,
		26
	};
	int Len = sizeof(a) / sizeof(int);
	int K = 4;
	FindKMax(a, 0, Len - 1, K);
	for (int i = 0; i < K; i++)
		cout << a[i] << " ";
	return 0;
}
```

### 方法三：利用最小堆的原理(时间复杂度为 O(N\*logK))(掌握)

(掌握)用容量为 K 的最小堆来存储最大的 K 个数。最小堆的堆顶元素就是最大 K 个数中的最小的一个。每次扫描一个数据 X，如果 X 比堆顶元素 Y 小，则不需要改变原来的堆。如果 X 比堆顶元素大，那么用 X 替换堆顶元素 Y，在替换之后，X 可能破坏了最小堆的结构，需要调整堆来维持堆的性质。调整过程时间复杂度为 O(logK)。 全部的时间复杂度为 O(N\*logK)。

这种方法当数据量比较大的时候，比较方便。因为对所有的数据只会遍历一次，第一种方法则会多次遍历数组。 如果所查找的 K 的数量比较大。可以考虑先求出 k`，然后再求出看k`+1 到 2 \* k`之间的数据，然后一次求取。

代码如下：

```
void heapifymin(int Array[], int i, int size) {
	if (i < size) {
		int left = 2 * i + 1;
		int right = 2 * i + 2;
		int smallest = i; //假设最小的节点为父结点
		//确定三个结点中的最大结点
		if (left < size) {
			if (Array[smallest] > Array[left])
				smallest = left;
		}
		if (right < size) {
			if (Array[smallest] > Array[right])
				smallest = right;
		}

		//开始交换父结点和最大的子结点
		if (smallest != i) {
			int temp = Array[smallest];
			Array[smallest] = Array[i];
			Array[i] = temp;
			heapifymin(Array, smallest, size); //对调整的结点做同样的交换
		}
	}
}

//建堆过程，建立最小堆,从最后一个结点开始调整为最小堆
void min_heapify(int Array[], int size) {
	int i;
	for (i = size - 1; i >= 0; i--)
		heapifymin(Array, i, size);

}
//k为需要查找的最大元素个数，size为数组大小，kMax存储k个元素的最小堆
void FindMax(int Array[], int k, int size, int kMax[]) {

	for (int i = 0; i < k; i++)
		kMax[i] = Array[i];
	//对kMax中的元素建立最小堆
	min_heapify(kMax, k);
	printf("最小堆如下所示 : \n");
	for (i = 0; i < k; i++)
		printf("%4d", kMax[i]);
	printf("\n");

	for (int j = k; j < size; j++) {
		if (Array[j] > kMax[0]) //如果最小堆的堆顶元素，替换
		{
			int temp = kMax[0];
			kMax[0] = Array[j];
			Array[j] = temp;
			//可能破坏堆结构，调整kMax堆
			min_heapify(kMax, k);
		}
	}
}

int main() {

	int a[] = {
		10,
		23,
		8,
		2,
		52,
		35,
		7,
		1,
		12
	};
	int length = sizeof(a) / sizeof(int);

	//最大四个元素为23,52,35,12
	/***************查找数组中前K个最大的元素****************/
	int k = 4;
	int * kMax = (int * ) malloc(k * sizeof(int));
	FindMax(a, k, length, kMax);

	printf("最大的%d个元素如下所示 : \n", k);
	for (int i = 0; i < k; i++)
		printf("%4d", kMax[i]);
	printf("\n");
	return 0;
}
```

### 方法四：二分法(在实际应用中效果不佳)

解法四：这也是寻找Ｎ个数中的第Ｋ大的数算法。利用二分的方法求取 TOP k 问题。 首先查找 max 和 min，然后计算出 mid = (max + min) / 2 该算法的实质是寻找最大的 K 个数中最小的一个。

```
const int N = 8;
const int K = 4;

/*
利用二分的方法求取TOP k问题。
首先查找 max 和 min，然后计算出 mid = (max + min) / 2
该算法的实质是寻找最大的K个数中最小的一个。


*/

int find(int * a, int x) //查询出大于或者等于x的元素个数
{
	int sum = 0;

	for (int i = 0; i < N; i++) {
		if (a[i] >= x)
			sum++;
	}
	return sum;
}


int getK(int * a, int max, int min) //最终max min之间只会存在一个或者多个相同的数字
{
	while (max - min > 1) //max - min的值应该保证比两个最小的元素之差要小
	{
		int mid = (max + min) / 2;
		int num = find(a, mid); //返回比mid大的数字个数
		if (num >= K) //最大的k个数目都要比min值大
			min = mid;
		else
			max = mid;
	}
	cout << "end" << endl;
	return min;
}

int main() {
	int a[N] = {
		54,
		2,
		5,
		11,
		554,
		65,
		33,
		2
	};
	int x = getK(a, 554, 2);
	cout << x << endl;
	getchar();
	return 0;
}
```

该算法在实际应用中效果不佳。

### 方法五：用空间换取时间的方法

解法五：如果 N 个数都是正数，取值范围不太大，可以考虑用空间换时间。申请一个包括 N 中最大值的 MAXN 大小的数组 count[MAXN]，count[i]表示整数 i 在所有整数中的个数。这样只要扫描一遍数组，就可以得到第 K 大的元素。

代码如下：

```
for(sumCount = 0, v = MAXN -1; v >=0; v--)
{
    cumCount += count[v];
       if(sumCount >= k)
            break;
}
return v;
```

扩展问题：

1. 如果需要找出 N 个数中最大的 K 个不同的浮点数呢？比如，含有 10 个浮点数的数组（1.5，1.5，2.5，3.5，3.5，5，0，- 1.5，3.5）中最大的 3 个不同的浮点数是（5，3.5，2.5）。
   解答：除了解法五不行，其他的都可以。因为最后一种需要是正数。

2. 如果是找第 k 到第 m（0<k<=m<=n)大的数呢？  
   解答：可以用小根堆来先求出 m 个最大的，然后从中输出 k 到 m 个。

3. 在搜索引擎中，网络上的每个网页都有“权威性”权重，如 page rank。如果我们需要寻找权重最大的 K 个网页，而网页的权重会不断地更新，那么算法要如何变动以达到快速更新（incremental update）并及时返回权重最大的 K 个网页？  
   解答：(解法三)用堆排序当每一个网页权重更新的时候，更新堆。

举一反三：查找最小的 K 个元素

解答：最直观的方法是用快速排序或堆排序先排好，在取前 K 小的数据。最好的办法是利用解法二和解法三的原理进行查找。
