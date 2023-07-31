---
layout:     post
title:      Leetcode经典例题——单词接龙
subtitle:   
date:       2023-7-31
author:     xuoneyuan
header-img: 
catalog: false
tags:
    - 算法
---

### 题目
字典 wordList 中从单词 beginWord 和 endWord 的 转换序列 是一个按下述规格形成的序列 beginWord -> s1 -> s2 -> ... -> sk：
- 每一对相邻的单词只差一个字母
- 对于 1 <= i <= k 时，每个 si 都在 wordList 中。注意， beginWord 不需要在 wordList 中
- sk == endWord
给你两个单词 beginWord 和 endWord 和一个字典 wordList ，返回 从 beginWord 到 endWord 的 最短转换序列 中的 单词数目 。如果不存在这样的转换序列，返回 0

示例1 ：\
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]\
输出：5\
解释：一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog", 返回它的长度 5

示例2 ：\
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log"]\
输出：0\
解释：endWord "cog" 不在字典中，所以无法进行转换
### 分析
使用 BFS 算法在搜索空间中进行广度优先搜索，从而逐层寻找最短转换序列，在找到最短序列时，返回转换序列的长度
- 使用双向广度优先搜索：我们可以同时从 beginWord 和 endWord 开始搜索，然后在中间某个点相遇。这样可以减少搜索空间并加快搜索速度。
- 使用双向队列代替切片：在广度优先搜索中，双向队列比切片在插入和删除操作上更高效。
- 使用哈希集合代替哈希表：在这个问题中，我们只需要判断单词是否在 wordList 中，不需要存储其他额外信息。使用哈希集合比哈希表更简洁高效。
- 使用两个哈希集合：我们可以使用两个哈希集合分别存储从 beginWord 和 endWord 可达的单词，这样在搜索过程中可以及时交换两个集合，从而减少搜索的层数
### 题解
#### go
~~~go
func ladderLength(beginWord string, endWord string, wordList []string) int {
wordSet := make(map[string]bool)
	for _, word := range wordList {
		wordSet[word] = true
	}

	if !wordSet[endWord] {
		return 0
	}

	delete(wordSet, beginWord)

	beginSet := make(map[string]bool)
	beginSet[beginWord] = true

	endSet := make(map[string]bool)
	endSet[endWord] = true

	level := 1

	for len(beginSet) > 0 && len(endSet) > 0 {
		if len(beginSet) > len(endSet) {
			beginSet, endSet = endSet, beginSet
		}

		nextSet := make(map[string]bool)

		for word := range beginSet {
			for i := 0; i < len(word); i++ {
				// 将 word 中的第 i 个字符替换为 "*"
				// 形成通用状态
				genericWord := word[:i] + "*" + word[i+1:]

				for _, c := range "abcdefghijklmnopqrstuvwxyz" {
					newWord := word[:i] + string(c) + word[i+1:]

					if endSet[newWord] {
						return level + 1
					}

					if wordSet[newWord] {
						delete(wordSet, newWord)
						nextSet[newWord] = true
					}
				}

				delete(wordSet, genericWord)
			}
		}

		beginSet = nextSet
		level++
	}

	return 0
}
~~~

#### cpp
~~~cpp
class Solution {
public:
    unordered_map<string, int> wordId;
    vector<vector<int>> edge;
    int nodeNum = 0;

    void addWord(string& word) {
        if (!wordId.count(word)) {
            wordId[word] = nodeNum++;
            edge.emplace_back();
        }
    }

    void addEdge(string& word) {
        addWord(word);
        int id1 = wordId[word];
        for (char& it : word) {
            char tmp = it;
            it = '*';
            addWord(word);
            int id2 = wordId[word];
            edge[id1].push_back(id2);
            edge[id2].push_back(id1);
            it = tmp;
        }
    }

    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        for (string& word : wordList) {
            addEdge(word);
        }
        addEdge(beginWord);
        if (!wordId.count(endWord)) {
            return 0;
        }

        vector<int> disBegin(nodeNum, INT_MAX);
        int beginId = wordId[beginWord];
        disBegin[beginId] = 0;
        queue<int> queBegin;
        queBegin.push(beginId);

        vector<int> disEnd(nodeNum, INT_MAX);
        int endId = wordId[endWord];
        disEnd[endId] = 0;
        queue<int> queEnd;
        queEnd.push(endId);

        while (!queBegin.empty() && !queEnd.empty()) {
            int queBeginSize = queBegin.size();
            for (int i = 0; i < queBeginSize; ++i) {
                int nodeBegin = queBegin.front();
                queBegin.pop();
                if (disEnd[nodeBegin] != INT_MAX) {
                    return (disBegin[nodeBegin] + disEnd[nodeBegin]) / 2 + 1;
                }
                for (int& it : edge[nodeBegin]) {
                    if (disBegin[it] == INT_MAX) {
                        disBegin[it] = disBegin[nodeBegin] + 1;
                        queBegin.push(it);
                    }
                }
            }

            int queEndSize = queEnd.size();
            for (int i = 0; i < queEndSize; ++i) {
                int nodeEnd = queEnd.front();
                queEnd.pop();
                if (disBegin[nodeEnd] != INT_MAX) {
                    return (disBegin[nodeEnd] + disEnd[nodeEnd]) / 2 + 1;
                }
                for (int& it : edge[nodeEnd]) {
                    if (disEnd[it] == INT_MAX) {
                        disEnd[it] = disEnd[nodeEnd] + 1;
                        queEnd.push(it);
                    }
                }
            }
        }
        return 0;
    }
};
~~~
