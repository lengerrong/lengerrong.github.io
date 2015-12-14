---
layout: post
title: Jetpack 解题报告
tags: ACM jetpack
categories: 算法实例
---

![jetpack1] [jetpack1]

[jetpack1]:  {{"/posts/jetpack_1.png" | prepend: site.imgrepo }}

We, the company, have made a new type of jetpack that can be used for transportation in the near future.
To travel using this jetpack, you need a certain time x (natural number ≥ 1 second/s). 
Though the transportation time differs for each person, it requires the same amount of time for a certain individual to use it for any section. 
For example, Mr. A may need 3 seconds to travel using the jetpack, while Mr. B needs 4 seconds. 
However, if Mr. A needs 3 seconds (x=3) to travel to a certain section using it, then he will need the same 3 seconds when using it for another section.

![jetpack2] [jetpack2]

[jetpack2]:  {{"/posts/jetpack_2.png" | prepend: site.imgrepo }}

On the graph above, each zenith indicates a point, and each line indicates a single route of direction. 
Also, the weighted value of each route indicates the amount of time needed to transport through the route. 
x, written on the weighted values, means you can transport through the route using the jetpack. 
For example, when transporting from point 2 to point 1 in the graph above, a person with x ≥ 5 can always travel in 5 seconds, 
while the other person with x < 5 can travel in x second(s).


Mr. Gang would like to know the fastest routes from point A to point B. How many routes are there?


Time limit: 1 second (Java: 2 seconds) (If your program exceeds this time limit, the answers that have been already printed are ignored and the score becomes 0. So, it may be better to print a wrong answer when a specific test case might cause your program to exceed the time limit. One guide for the time limit excess would be the size of the input.)


[Input]
Several test cases can be included for entry. 
T, The number of test cases, is given on the first line of entry, and after that, test cases of the number of T are given in order. (T ≤ 10)
N, the number of points, and M, the number of routes, are given orderly on each first line of each test case. (1 ≤ N ≤ 500, 0 ≤ M ≤ 10 000)
From the next line through to the lines of M, the route information, A, B, and L, are given in order. 
If L is a number, it means it needs L second(s) from A to B; if L is a character "x", it means the jetpack can be used for transportation from A to B. 
There can be several routes from A to B. (1 ≤ A, B ≤ N, A != B, 1 ≤ L ≤ 1 000 000) 
On the next line, Qs, the number of questions that Mr. Gang would like to know, are given. (1 ≤ Q ≤ 10)
Through to the next lines of the number of Qs, the points he would like to know, A and B, are given in order. 
This means he would like to know the fastest routes from A and B. (A != B)


[Output]

For each test case, you should print "Case #T" in the first line where T means the case number.

For each test case, you should output how many lengths of the separate minimum time routes are for each question, and the sum of those lengths of minimum time routes. 
Generate "inf" (except for quotation marks) if the answer is infinite. If there is no route from A to B, you may generate 0 for both. 
To increase your understanding of the output, see ‘[Note] Explanation on output for the input e.g. 1’ at the end of this question sheet.
Classify the test cases by generating an empty line of output at the end of each test case.


[I/O Example]

Input 
2
4 4 
1 2 x 
2 3 x 
3 4 x 
1 4 8 
3 
2 1 
1 3 
1 4 
3 5 
3 2 x 
2 1 x 
2 1 5 
1 3 10 
3 1 20 
6 
1 2 
2 3 
3 1 
2 1 
3 2 
1 3



Output 
Case #1

0 0 
inf 
3 17

Case #2
inf 
5 65 
15 185 
5 15 
inf 
1 10


[Note] Explanation on output for the input e.g. 1


- As there is no route from 2 to 1, it is 0 0. 
- The only route from 1 to 3 is by travelling from 1 to 2 using the jetpack and then travelling from 2 to 3 using the Jetpack. 
As x is a natural number ≥ 1, it can increase into 2, 4, 6, 8, ... infinitely so it is “inf”. 
- From 1 to 4, there is the first route from 1 to 2, 2 to 3 and 3 to 4 all using the jetpack; and the second route from 1 to 4 without the jetpack. 
It requires 3*x amount of time for using the jetpack, 8 without the jetpack; 
therefore, a person with x = 1, 2 can transport from point 1 to point 4 in 3, 6 seconds using the jetpack. 
The other cases are better without the jetpack, the number of lengths is 3: 3, 6, 8, and the sum of the lengths is 3+6+8=17.

~~~C++
#include<math.h>
#include<stdio.h>
#include<string.h>
const int NMAX = 503;
const int MMAX = 10003;
const int INT_MAX = 2021161080;
int TC; // test cases
int N, M; // N points, M edges
int Q; // questions
int S, T; // start point, terminal point in questions
typedef long long LL;
// each path from S to T, the sumary of the path is (kx+b), 0 <= k <= n-1, 0 <= b <= oo
// for the paths with same k, we just care about the min b
int f[NMAX][NMAX]; // f[i][j] stores the min value of b that from start point to point j via i number edges with x value 
// handle with n numbers line y = kx + b
// if b in each path is oo, then it means there is no route from S to T
// if b in the path with k = 0 is oo, the answer is inf
// the rest has the limit answsers
// line y=kx+b
struct TLine {
	int k;
	int x;
	inline TLine(){}
	inline TLine(int _k, int _x) { k = _k; x = _x; }
};
TLine stLines[NMAX];
struct Edge{
	int to;
	int weight;
};
Edge adjmap[NMAX][MMAX];
bool vis[NMAX][NMAX]; // vis[i][j] means the flag that whether we already access to j point with i number edges with x value or not, ture means accessed, false means not
int questack[NMAX*NMAX][2]; //questack[i][0] means k(0<= k <= n-1) value in the i index in the path, questack[i][1] means to the i index point value frome 1 to n
inline void spfa()
{
	int front = 0, rear = 1; // que head and tail
	memset(vis, 0, sizeof(vis));
	memset(f, 120, sizeof(f));
	f[0][S] = 0; // from S to S, the min path is 0
	questack[front][0] = 0; questack[front][1] = S; // start from the S point and with k = 0
	vis[0][S] = 1;
	while (front != rear) {
		int k = questack[front][0], t = questack[front][1], v = f[k][t];
		for (int i = 0; i < adjmap[t][MMAX-1].weight; i++) {
			int to = adjmap[t][i].to;
			int tv = adjmap[t][i].weight;
			if (!tv) { // edge with x value 
				if ((k + 1) < N && f[k + 1][to] > v) {
					f[k + 1][to] = v;
					if (!vis[k + 1][to]) {
						vis[k + 1][to] = 1;
						questack[rear][0] = k + 1;
						questack[rear++][1] = to;
					}
				}
			} else {
				if (f[k][to] > (v + tv)) {
					f[k][to] = v + tv;
					if (!vis[k][to]) {
						vis[k][to] = 1;
						questack[rear][0] = k;
						questack[rear++][1] = to;
					}
				}
			}
		}
		front++;
		vis[k][T] = 0;
	}
}
inline bool spj()
{
	int x = INT_MAX;	
	for (int i = 0; i < N; i++)
		x = x < f[i][T] ? x : f[i][T];
	if (x >= INT_MAX) {
		printf("0 0\n");
		return true;
	} else if (f[0][T] >= INT_MAX) {
		printf("inf\n");
		return true;
	} else
		return false;
}
inline int kcalc(int k, int x)
{
	return k*x + f[k][T];
}
int main()
{
	scanf("%d", &TC);
	for (int tc = 1; tc <= TC; tc++) {
		scanf("%d %d", &N, &M);
		memset(adjmap, 0x00, sizeof(adjmap));
		for (int i = 1; i <= M; i++) {
			int A, B, L = 0;
			char lch[20] = { 0, };			
			scanf("%d %d %s", &A, &B, lch);
			if (lch[0] != 'x') {
				L = lch[0] - '0';
				int j = 0;
				while (lch[++j] != '\0' && j < 7)
					L = L * 10 + lch[j] - '0';
			}			
			adjmap[A][adjmap[A][MMAX - 1].weight].to = B;
			adjmap[A][adjmap[A][MMAX - 1].weight].weight = L;
			adjmap[A][MMAX - 1].weight += 1;
		}		
		scanf("%d", &Q);
        printf("Case #%d\n", tc);
		while (Q--) {
			scanf("%d %d", &S, &T);			
			spfa();
			if (spj())
				continue;
			int top = 0;
			for (int i = N - 1; i >= 0; i--) {
				if (f[i][T] >= INT_MAX)
					continue;
				while (top) {
					if (kcalc(i, stLines[top].x) <= kcalc(stLines[top].k, stLines[top].x))
						top--;
					else
						break;
				}
				if (top) {
					double k1 = stLines[top].k, k2 = i, b1 = f[stLines[top].k][T], b2 = f[i][T];
					int x = (int)ceil((b2 - b1) / (k1 - k2));
					stLines[++top] = TLine(i, x);
				} else
					stLines[++top] = TLine(i, 0);
			}
			int tot = 1; 
			LL sum = f[0][T];
			for (int i = 1; i < top; i++) {
				int st = stLines[i].x, ed = stLines[i + 1].x - 1;
				if (st == 0)
					st++;
				tot += ed - st + 1;
				sum += LL(ed - st + 1)*(kcalc(stLines[i].k, st) + kcalc(stLines[i].k, ed)) / 2;
			}
			printf("%d %lld\n", tot, sum);
		}
	}
	return 0;
}
~~~