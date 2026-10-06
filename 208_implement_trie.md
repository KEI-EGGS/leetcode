# 208. Implement Trie (Prefix Tree)

https://leetcode.com/problems/implement-trie-prefix-tree/

---

## Step 1

Trie を扱うのが初めてだったので、ノードをクラスにして「子ノードの辞書」と「ここで終わる単語があるか」
の2つを持たせる形から書いた。

Runtime: 99 ms

```python
class Node(object):
    def __init__(self):
        self.children = {}
        self.isEnd = False


class Trie(object):
    def __init__(self):
        self.root = Node()

    def insert(self, word):
        cur = self.root
        for c in word:
            if c not in cur.children:
                cur.children[c] = Node()
            cur = cur.children[c]
        cur.isEnd = True

    def search(self, word):
        cur = self.root
        for c in word:
            if c in cur.children:
                cur = cur.children[c]
            else:
                return False
        return cur.isEnd

    def startsWith(self, prefix):
        cur = self.root
        for c in prefix:
            if c in cur.children:
                cur = cur.children[c]
            else:
                return False
        return True
```

---

## Step 2

`search` と `startsWith` が、同じキーを2回引いているのが気になった。

```python
if c in cur.children:        # 1回目
    cur = cur.children[c]    # 2回目
```

存在確認と取り出しを分ける必要はないので、`.get()` で1回にまとめた。
見つからなければ `None` が返るので、それをそのまま終了条件にできる。

Runtime: 99 ms → 68 ms（31 ms 減）

```python
    def search(self, word):
        cur = self.root
        for c in word:
            cur = cur.children.get(c)
            if cur is None:
                return False
        return cur.isEnd

    def startsWith(self, prefix):
        cur = self.root
        for c in prefix:
            cur = cur.children.get(c)
            if cur is None:
                return False
        return True
```

### 検討して見送ったこと

`search` と `startsWith` は走査部分が完全に同じなので、`_walk` のような共通メソッドに
切り出すことを考えた。ただし呼び出し元が2つだけで、切り出しても各メソッドは
「`_walk` を呼んで戻り値を見る」になるだけだったので、読みやすさが変わらないと判断して見送った。

---

## Step 3

ノーエラー・3回連続：9分 / 5分30秒 / 3分

1回目は Step 1 の形（2回引き）で書いてしまったので、2回目以降は Step 2 の形で書き直した。
最終的に落ち着いたのは Step 2 と同じコード。

```python
class Node:
    def __init__(self):
        self.children = {}
        self.is_end = False


class Trie(object):
    def __init__(self):
        self.root = Node()

    def insert(self, word):
        node = self.root
        for c in word:
            if c not in node.children:
                node.children[c] = Node()
            node = node.children[c]
        node.is_end = True

    def search(self, word):
        node = self.root
        for c in word:
            node = node.children.get(c)
            if node is None:
                return False
        return node.is_end

    def startsWith(self, prefix):
        node = self.root
        for c in prefix:
            node = node.children.get(c)
            if node is None:
                return False
        return True
```

---