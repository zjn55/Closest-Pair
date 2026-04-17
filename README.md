# Closest-Pair
平面最近点对问题 —— 原创 —— 极值分段算法

## 算法简介
本算法基于**极点（山峰/山谷）分段**思想，是一种**非递归**的最近点对求解算法：
1. 对点按 x 坐标排序
2. 根据 y 的单调性将点集划分为多个单调段
3. 单调段内仅比较相邻点
4. 极点附近做局部跨段比较
5. 合并阶段保证全局正确性
6. 如果存在重复点可以在n次查找之内退出函数返回结果0

## 算法原理与正确性
1. 点按 x 坐标升序排列。 点按 x 坐标升序排列。
2. 根据 y 的上升/下降趋势，将序列划分为**单调段**，分段点为**极点（峰值/谷值）**。
3. **单调段内**：最小距离一定出现在相邻点之间，只需线性遍历。
4. **极点附近**：仅需局部检查，即可覆盖跨段最近点对。
5.如果你发现有不正确之处，希望你能告诉我

![算法分段示意图](image/单调分段证明.png)
![极点跨段比较](image/山峰山谷最小值证明.png)
![极点跨段比较](image/合并.png)
![极点跨段比较](image/复杂度证明.png)

## 时间复杂度
- 排序：O(n log n) 
- 分段 + 段内遍历 + 山峰山谷检查  
- **整体复杂度：O(n log n)**

## 核心函数说明
- `extremum()`：极点（峰值/谷值）附近跨段点对检查
- `findClosestPair()`：算法主入口

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
// 极值点跨段检查 
void extremum(const vector<Point>& points, int left, int mid, int right, double& minDist, bool isUpward) {
	int j = mid + 1;
	for (int i = mid - 1; i >= left; i--) {
		if (j > right || points[i].x - points[j].x > minDist) break;
		minDist = min(minDist, points[i].getDist(points[j]));
		if (isUpward) {
			if (points[i].y > points[j].y) continue;
		}
		else {
			if (points[i].y < points[j].y) continue;
		}
		j++; i++;
	}
}
// 极值分段算法主函数
double findClosestPair(vector<Point> points) {
	int n = points.size();
	if (n <= 1) return numeric_limits<double>::infinity();
	if (n == 2) return points[0].getDist(points[1]);
	sort(points.begin(), points.end(), cmpxx);
	// 极值点扫描，记录分段边界索引 segIndex,segX
	vector<int> segIndex(1, 0);
	vector<double> segX(n);
	vector<Point> strip(n);
	bool isRising = points[0].y < points[1].y, nextFalling;
	bool isPeak = !isRising;
	double minDist = 0x7FFF0000;

	for (int i = 1; i < n - 1; i++) {
		// 相邻点距离更新
		double d = points[i].getDist(points[i - 1]);
		if (d < minDist) minDist = d;
		if (minDist == 0) return 0;
		nextFalling = points[i].y > points[i + 1].y;
		if (nextFalling == isRising) {
			segIndex.push_back(i);
			segX[segIndex.size() - 1] = points[i].x;
			// 当有足够边界时，立即调用 extremum 检查以当前极值点为右边界的前一段
			if (segIndex.size() >= 3) {
				int k = segIndex.size() - 1;
				bool flag = (k % 2) ? isPeak : !isPeak;
				extremum(points, segIndex[k - 2], segIndex[k - 1], segIndex[k], minDist, flag);
			}
			isRising = !isRising;
		}
	}
	// 最后一段
	double dlast = points[n - 1].getDist(points[n - 2]);
	if (dlast < minDist) minDist = dlast;
	segIndex.push_back(n - 1);
	segX[segIndex.size() - 1] = points[n - 1].x;
	if (segIndex.size() >= 3) {
		int k = segIndex.size() - 1;
		bool flag = (k % 2) ? isPeak : !isPeak;
		extremum(points, segIndex[k - 2], segIndex[k - 1], segIndex[k], minDist, flag);
	}
	//带状区域合并检查
	int segCount = segIndex.size();
	for (int i = 2; i < segCount - 1; ) {
		double midX = points[segIndex[i]].x;
		int L = lower_bound(segX.begin(), segX.begin() + segCount, midX - minDist) - segX.begin();
		int R = upper_bound(segX.begin(), segX.begin() + segCount, midX + minDist) - segX.begin() - 1;
		L = max(0, min(L, segCount - 2));
		R = max(0, min(R, segCount - 2));

		if (L < i || R > i) {
			int start = segIndex[L];
			int end = segIndex[R + 1] - 1;
			strip.clear();
			for (int k = start; k <= end; k++)
				strip.push_back(points[k]);
			sort(strip.begin(), strip.end(), cmpy);
			int sz = strip.size();
			for (int a = 0; a < sz; a++) {
				for (int b = a + 1; b < sz && strip[b].y - strip[a].y < minDist; b++) {
					double d = strip[a].getDist(strip[b]);
					if (d < minDist) minDist = d;
				}
			}
		}
		// 跳步逻辑，减少无效比较
		if (R <= i + 2) i += 2;
		else i = (R % 2 == 0) ? R : R - 1;
		if (i >= segCount - 1) break;
	}
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
