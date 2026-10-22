# 78. Subsets

https://leetcode.com/problems/subsets/

---

## Step 1

各要素について「入れる / 入れない」を決めていく二分木を、再帰でたどる形で書いた。
葉（全要素を決め終わったところ）で、そのときの選択を答えに積む。

```python
class Solution(object):
    def subsets(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        self.nums = nums
        self.length = len(nums)
        self.answers = []
        list1 = []
        self._decide_put(0, list1)
        return self.answers

    def _decide_put(self, depth, include_list):
        if depth != self.length:
            self._decide_put(depth + 1, include_list[:])
            include_list.append(self.nums[depth])
            self._decide_put(depth + 1, include_list[:])
        else:
            self.answers.append(include_list)
```

### 詰まったところ

再帰は書けたが出力が全部同じになった。

```
入力 [1,2,3]
出力 [[3,2,3,1,3,2,3]] が 8 個
```

原因は2つあった。

- `self.answers.append(include_list)` が参照を積んでいた → 8個が同じオブジェクトを指す
- `append` した要素を戻していなかった → 中身が7要素まで伸び続ける

同じ症状に見えるが別の原因で、片方だけ直しても通らない。
参照だけ直すと `[], [3], [3,2], [3,2,3], [3,2,3,1], ...` となり、存在しない部分集合が出る。
Step 1 では、渡すときにコピーする（`include_list[:]`）ことで両方を回避した。

---

## Step 2

Step 1 は再帰呼び出しのたびにコピーを作っていた。
コピーを渡す代わりに、同じリストを使い回して、`append` したものを `pop` で戻す形にした。

```python
self._decide_put(depth + 1, chosen)     # 入れない側
chosen.append(self.nums[depth])
self._decide_put(depth + 1, chosen)     # 入れる側
chosen.pop()                            # 戻す
```

`_decide_put` に入ったときと出たときで `chosen` が同じ状態に戻る、という形になる。
代わりに `chosen` は最後まで同じオブジェクトなので、答えに積む瞬間にコピーが必要になる。

あわせて名前と構造も直した。

- `include_list` → `chosen`（何のリストかが名前に出ていなかった。「根から今のノードまでで、入れると決めた要素の並び」）
- `self.length` を削除（`len(self.nums)` と同じものを二重に持っていた）
- 終了条件を先に書く（再帰は「終了条件 → 再帰」の順のほうが読む順と合う）
- `list1` を削除（一度しか使わないのでそのまま `[]` を渡す）

```python
class Solution(object):
    def subsets(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        self.nums = nums
        self.answers = []
        self._decide_put(0, [])
        return self.answers

    def _decide_put(self, depth, chosen):
        if depth == len(self.nums):
            self.answers.append(chosen[:])
        else:
            self._decide_put(depth + 1, chosen)
            chosen.append(self.nums[depth])
            self._decide_put(depth + 1, chosen)
            chosen.pop()
```

### 計測

コピーが減るぶんメモリも減ると思ったが、測るとピークは変わらなかった。
途中のコピーは再帰から戻った時点で解放されるので、答えの `2^n` 個のリストが支配的になる。
差が出たのは時間のほうだった。

`n = 18`（部分集合 262,144 個）、`timeit` 5回平均 / `tracemalloc` のピーク：

| | 時間 | ピークメモリ |
|---|---|---|
| Step 1（渡すときにコピー） | 32.93 ms | 34.21 MB |
| Step 2（pop で戻す） | 27.87 ms | 34.20 MB |

前回 #208 で「LeetCode の Runtime は実行ごとのブレが大きい」とご指摘をいただいたので、
今回は LeetCode の表示は使わず、手元で `timeit` と `tracemalloc` で測った。
LeetCode の Memory 表示もプロセス全体の値なので、上のピークメモリとは別物として扱っている。

---

## Step 3

ノーエラー・3回連続：6分 / 2分 / 2分

```python
class Solution(object):
    def subsets(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        self.nums = nums
        self.answers = []
        self._decide_put(0, [])
        return self.answers

    def _decide_put(self, depth, chosen):
        if depth == len(self.nums):
            self.answers.append(chosen[:])
        else:
            self._decide_put(depth + 1, chosen)
            chosen.append(self.nums[depth])
            self._decide_put(depth + 1, chosen)
            chosen.pop()
```

---
