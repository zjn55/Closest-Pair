# Closest-Pair
平面最近点对问题 —— 原创 —— 极值分段算法

## 算法简介
本算法基于**极点（山峰/山谷）分段**思想，是一种**非递归、高速、常数小**的最近点对求解算法：
1. 对点按 x 坐标排序
2. 根据 y 的单调性将点集划分为多个单调段
3. 单调段内仅比较相邻点
4. 极点附近做局部跨段比较
5. 滑动窗口保证全局正确性
6. 全程无递归，速度超过传统分治法
7. 如果存在重复点可以在n次查找之内退出函数返回结果0

## 算法原理与正确性
1. 点按 x 坐标升序排列。 点按 x 坐标升序排列。
2. 根据 y 的上升/下降趋势，将序列划分为**单调段**，分段点为**极点（峰值/谷值）**。
3. **单调段内**：最小距离一定出现在相邻点之间，只需线性遍历。
4. **极点附近**：仅需局部检查，即可覆盖跨段最近点对。
5. **滑动窗口**：对远距离分段进行剪枝检查，确保 100% 正确且无冗余计算。

![算法分段示意图](image/单调分段证明.png)
![极点跨段比较](image/山峰山谷最小值证明.png)

## 时间复杂度
- 排序：O(n log n) 
- 分段 + 段内遍历 + 山峰山谷检查 + 寻找陡坡：O(n) 
- 滑动窗口：O(n)~O(n^2) 
- **整体复杂度：O(n log n)**
- **实际运行效率：远优于标准分治法**

## 核心函数说明
- `extremum()`：极点（峰值/谷值）附近跨段点对检查
- `checkSegments()`：滑动窗口检查所有陡坡，保证正确性
- `findClosestPair()`：算法主入口
- 
#### 运行环境 
- 操作系统：Windows 
- C++ 标准：C++11 及以上 
- 编译器：MSVC 
- **无任何第三方库依赖**

## 使用方法
输入点的数量 n，程序自动生成随机点并计算最小点对距离。

## 完整代码
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>
#include <chrono>
#include <random>

using namespace std;
using namespace chrono;

struct Point {
    float x, y;

    void setXY(float x_, float y_) {
        x = x_;
        y = y_;
    }

    float getDist(const Point& other) const {
        float dx = x - other.x;
        float dy = y - other.y;
        return sqrt(dx * dx + dy * dy);
    }
};

bool cmpx(const Point& a, const Point& b) {
    return a.x < b.x;
}
void extremum(const vector<Point>& points, int left, int mid, int right, float& minDist, bool isUpward) {
	int j = mid + 1;
	for (int i = mid - 1; i >= left; i--) {
		if (j > right || points[i].x - points[j].x > minDist)
			break;
		minDist = min(minDist, points[i].getDist(points[j]));
		if (isUpward) {
			if (points[i].y > points[j].y) continue;
		}
		else if (points[i].y < points[j].y) continue;
		j++; i++;
	}
}
// 滑动窗口全局检查（远距离分段剪枝）
void checkSegments(const vector<Point>& points, const vector<int>& hill, float& minDist) {
	int n = hill.size(), left, right, end;
	for (int i = 0; i < n; i += 2) {
		left = hill[i], end = hill[i + 1];
		for (right = hill[i] + 1; right <= end; right++) {
			while (points[right].x - points[left].x > minDist)left++;
			for (int j = left; j < right; j++)
				if (abs(points[j].y - points[right].y) < minDist)
					minDist = min(minDist, points[right].getDist(points[j]));
		}
	}
}
// 主函数：极值分段法求最近点对
float findClosestPair(vector<Point> points) {
	vector<int> segIndex(1, 0);
	float minDist = 1e10;
	int n = points.size();
	if (n <= 1) return 0x7F800000;
	if (n == 2) return points[0].getDist(points[1]);

	sort(points.begin(), points.end(), cmpx);

	bool isRising = points[0].y < points[1].y, isPeak = !isRising, nextFalling;
	for (int i = 1; i < n - 1; i++) {
		if (minDist > points[i].x - points[i - 1].x)
			minDist = min(minDist, points[i].getDist(points[i - 1]));
		if (minDist == 0)return 0;
		nextFalling = points[i].y > points[i + 1].y;
		if (nextFalling == isRising) {
			segIndex.push_back(i);
			isRising = !isRising;
		}
	}
	minDist = min(minDist, points[n - 1].getDist(points[n - 2]));
	segIndex.push_back(n - 1);

	int segCount = segIndex.size();
	for (int i = 2; i < segCount; i++) {
		bool flag = (i % 2) ? isPeak : !isPeak;
		extremum(points, segIndex[i - 2], segIndex[i - 1], segIndex[i], minDist, flag);
	}

	vector<int>hill;
	int num = 0, segi;
	for (int i = 1; i < segCount - 1; i++) {
		segi = segIndex[i];
		if (points[segi + 1].x - points[segi].x < minDist)
			num++;
		else {
			if (num >= 2) {
				hill.push_back(segIndex[i - num - 1]);
				hill.push_back(segi);
			}
			num = 0;
		}
	}
	if (num >= 2) {
		hill.push_back(segIndex[segCount - num - 2]);
		hill.push_back(segIndex[segCount - 1]);
	}
	checkSegments(points, hill, minDist);

	return minDist;
}

mt19937 engine(high_resolution_clock::now().time_since_epoch().count());
uniform_real_distribution<float> dis(0, 1000000);

int main() {
    int n;
    float minDist;

    cout << "请输入点的数量：";
    cin >> n;

    vector<Point> points(n);
    for (int i = 0; i < n; i++)
        points[i].setXY(dis(engine), dis(engine));

    auto start = high_resolution_clock::now();
    minDist = findClosestPair(points);
    auto end = high_resolution_clock::now();

    cout << "最小距离为：" << minDist << "\n运行耗时："
        << duration_cast<milliseconds>(end - start).count() << " ms" << endl;

    return 0;
}
