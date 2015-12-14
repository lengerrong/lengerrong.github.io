---
layout: post
title: Prince and Process 解题报告
tags: ACM DP LCS LIS
categories: 算法实例
---

![prince_and_process_1] [prince_and_process_1]

[prince_and_process_1]:  {{"/posts/prince_and_process_1.jpg" | prepend: site.imgrepo }}

![prince_and_process_2] [prince_and_process_2]

[prince_and_process_2]:  {{"/posts/prince_and_process_2.jpg" | prepend: site.imgrepo }}

由于数据量太大，DP LCS算法会超时。

记录princess的路径在prince的路径出现的index位置，问题就转化成一个LIS的问题。

这里如果使用数组遍历查询，因数据量大，依旧会超时，所以开了一个足够大的数组来模拟map<int, int>。

当然使用map是不会超时的，但系统规定不让用STL。

LIS问题有一个nlogn的算法。

还有一个问题，因为开的数组太大了，使用C编译出来的程序会挂掉，但使用了c++编译就OK了。

~~~C++
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#define MAX (250*250 + 1)
int pa[MAX];
int qa[MAX];
int f[MAX];
int binsearch(int x, int len)
{
	int left = 0, right = len, cnt = 0;
	while (left <= right)
	{
		int mid = (left + right) >> 1;
		if (x>f[mid])
		{
			cnt = mid;
			left = mid + 1;
		}
		else right = mid - 1;
	}
	return cnt;
}
int main()
{
	int T, n, p, q, rq;
	int i, j, id;
	int lcs, tl;
	scanf("%d", &T);
	for (id = 1; id <= T; id++) {
		lcs = 0;
		scanf("%d %d %d", &n, &p, &q);
		p++;
		q++;
		memset(pa, 0, sizeof(pa));
		for (i = 1; i <= p; i++) {
			int pv;
			scanf("%d", &pv);
			pa[pv] = i;
		}
		// LCS will exceed time limit
		// record the index of the value in qa that appears in pa
		rq = 0;
		for (j = 1; j <= q; j++) {
			int qv;
			scanf("%d", &qv);
			if (pa[qv] != 0)
				qa[rq++] = pa[qv];
		}
		// now become a LIS problem
		f[0] = -1;
		for (i = 0; i < rq; i++) {
			tl = binsearch(qa[i], lcs) + 1;
			if (tl > lcs) {
				f[tl] = qa[i];
				lcs = tl;
			}
			else if (f[tl] > qa[i])
				f[tl] = qa[i];
		}
		printf("Case %d: %d\n", id, lcs);
	}
	return 0;
}
~~~