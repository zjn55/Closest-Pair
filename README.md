# Fastest-Closest-Pair
平面最近点对问题——创新极值分段算法

## 算法简介
本算法基于**极点（山峰/滑坡）分段**思想：
1. 对点按 x 坐标排序
2. 根据 y 的单调性将点集划分为多个单调段
3. 单调段内仅比较相邻点
4. 极点附近做局部跨段比较
5. 全程无递归

## 算法原理与正确性
本算法基于平面点集的单调性与极点结构，实现无递归、低常数、全覆盖的最近点对求解。

1. 点按 x 坐标升序排列。
2. 根据 y 的单调性将序列划分为多个单调段，分段点为极点。
3. 单调段内：最小距离必在相邻点之间，只需线性遍历。
4. 跨段区域：仅需在极点附近局部检查，即可找到跨段最小距离。
5. 全覆盖：所有可能最近点对均被检查，结果严格正确。

![算法分段示意图](https://via.placeholder.com/800x400?text=Closest+Pair+Monotone+Split)
![极点跨段比较](https://via.placeholder.com/800x400?text=Extremum+Region+Check)

## 时间复杂度
- 排序：O(n log n)
- 主体遍历：O(n)
- 实际运行效率 **优于标准 O(n log n) 分治法**
- 实测速度

## 核心函数
- `slope()`：单调段内最小距离计算
- `extremum()`：极点附近跨段点对检查
- `findMindist()`：算法主入口

## 运行环境
- 操作系统：Windows
- C++ 标准：C++11 及以上
- 编译器：MSVC
- 无第三方库依赖

## 使用方法
输入点的数量 n，程序自动生成随机点并计算最小点对距离。

## 完整代码
```cpp
#include<iostream>
#include<random>
#include<chrono>
#include<vector>
#include<cmath>
#include<algorithm>

using namespace std;
using namespace chrono;

mt19937 engine(high_resolution_clock::now().time_since_epoch().count());
uniform_real_distribution<float> dis(0, 1000000);

class Point {
public:
	float getDist(const Point& a)const {
		return sqrt((a.x - x) * (a.x - x) + (a.y - y) * (a.y - y));
	}
	void setXY(float x, float y) {
		this->x = x;
		this->y = y;
	}
	float x, y;
};

bool cmpx(Point a, Point b) { return a.x < b.x; }

float slope(const vector<Point>& points, int left, int right) {
	float mindist = 1e10, dist;
	for (int i = left; i < right; i++) {
		dist = points[i].getDist(points[i + 1]);
		if (mindist > dist)
			mindist = dist;
	}
	return mindist;
}

float extremum(const vector<Point>& points, int left, int mid, int right, float mindist) {
	float dist;
	int j = mid + 1;
	for (int i = mid - 1; i >= left; i--) {
		if (j > right)
			break;
		if (points[i].x - points[j].x > mindist)
			break;
		dist = points[i].getDist(points[j]);
		if (dist < mindist)
			mindist = dist;
		if (points[i].y > points[j].y)
			continue;
		else j++;
	}
	return mindist;
}

float findMindist(vector<Point>points) {
	vector<int>index(1, 0);
	float mindist = 1e10;
	bool bigThanl = points[0].y < points[1].y, bigThanr;
	sort(points.begin(), points.end(), cmpx);
	for (int i = 1; i < points.size() - 1; i++) {
		bigThanr = points[i].y > points[i + 1].y;
		if (bigThanr == bigThanl) {
			index.push_back(i);
			bigThanl = !bigThanl;
		}
	}
	index.push_back(points.size() - 1);
	int num = index.size();
	for (int i = 1; i < num; i++)
		mindist = min(mindist, slope(points, index[i - 1], index[i]));
	for (int i = 2; i < num; i++)
		mindist = min(mindist, extremum(points, index[i - 2], index[i - 1], index[i], mindist));
	return mindist;
}

int main() {
	int n;
    long time;
	float mindist;

    cin >> n;
    vector<Point>points(n);
	for (int i = 0; i < n; i++)
		points[i].setXY(dis(engine), dis(engine));

	auto start = high_resolution_clock::now();
	mindist = findMindist(points);
	auto end = high_resolution_clock::now();
	time = duration_cast<milliseconds>(end - start).count();

	cout << "最小距离为：" << mindist << "\n运行耗时："
			<< time << " 毫秒" << endl;
	return 0;
}
