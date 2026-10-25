# 560. Subarray Sum Equals K

https://leetcode.com/problems/subarray-sum-equals-k/

---

## Step 1

和が k になる連続部分配列を，開始位置と終了位置の二重ループで全部試した．

```python
class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        ans = 0
        n = len(nums)
        for i in range(n):
            for j in range(i, n):
                total = sum(nums[i:j + 1])
                if total == k:
                    ans += 1
        return ans
```

結果は TLE

外側 n 回 × 内側 n 回 × sum が n 回  →  O(n³)

さらに `nums[i:j + 1]` はスライスなので，毎回新しいリストを作っている
時間だけでなくメモリも O(n) 使う

---

## Step 2

### 考え方

`i` から `j` までの和は，2つの「先頭からの和」の差で書ける

`total[j] - total[i] == k`

→`total[i] == total[j] - k`

となるような`j`未満の`i`を探す

### O(n) の形

```python
class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        ans = 0
        sum_counts = {0: 1}
        total = 0
        for num in nums:
            total += num
            if total - k in sum_counts:
                ans += sum_counts[total - k]
            sum_counts[total] = sum_counts.get(total, 0) + 1
        return ans
```

`O(n)` 時間 / `O(n)` 空間。各要素を1回ずつ見て，辞書の参照と登録が定数時間

### 計測：`in` + `[ ]` と `.get` の比較

数える側の書き方として、次の2つがある。

```python
if total - k in sum_counts:              # ハッシュ探索 2回（外れたら1回）
    ans += sum_counts[total - k]

ans += sum_counts.get(total - k, 0)      # ハッシュ探索 1回 + メソッド呼び出し 1回
```

claudeに `timeit` で比較するよう要請したところ，`if[]`の方が速いという結果が出たのでこちらを採用した

```
辞書の探索 2回  <  メソッド呼び出し 1回 + 探索 1回
```

---

## Step 3

ノーエラー・3回連続：6分 / 2分 / 1分20秒

```python
class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        ans = 0
        sum_counts = {0: 1}
        total = 0
        for num in nums:
            total += num
            if total - k in sum_counts:
                ans += sum_counts[total - k]
            sum_counts[total] = sum_counts.get(total, 0) + 1
        return ans
```

---