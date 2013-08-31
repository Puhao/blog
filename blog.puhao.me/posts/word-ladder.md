---
title: Word Ladder
date: '2013-08-28'
description:
categories:
- 算法
tags:
- LeetCode
- BFS
- DFS
---
LeetCode上面有两道关于单词变换的题目。

Word Ladder
---
>Given two words (start and end), and a dictionary, find the length of shortest transformation sequence from start to end, such that:

>Only one letter can be changed at a time
>Each intermediate word must exist in the dictionary
>For example,

>Given:
>start = "hit"  
>end = "cog"  
>dict = ["hot","dot","dog","lot","log"]

>As one shortest transformation is "hit" -> "hot" -> "dot" -> "dog" -> "cog",
return its length 5.

>Note:

>Return 0 if there is no such transformation sequence.  
>All words have the same length.  
>All words contain only lowercase alphabetic characters.

由于需要找到最短的变换路径，因此使用BFS查找，相同深度的在同一轮遍历每个单词进行检查是否有下一个跳转的路径，因此从单词首位开始到末位，每次只能变换一个位置，如果变换的单词在dict中，则加入队列，供下一轮遍历查找。当找到最终跳转单词的时候，返回当前遍历的深度即可。

```
int ladder(string start, string end, set<string> &dict)
{
	if (start == end)
		return 1;
	int step = 1;
	queue<string> q;
	q.push(start);
	while(q.size())	//BFS队列有元素就继续查找
	{
		step++;
		int len = q.size();
		while(len--)	//相同深度的节点数目
		{
			string check = q.front();
			for(int i = 0; i<check.size(); i++)
			{
				char tmp = check[i];
				for(char j='a'; j<='z'; j++)
				{
					check[i] = j;
					if (check == end)
						return step;
					if (dict.count(check))
					{
						q.push(check);	
						dict.erase(check);
					}
		
				}
				check[i] = tmp;
			}
			q.pop();
		}
	}
	return 0;
}
```

Word Ladder 2
---
>Given two words (start and end), and a dictionary, find all shortest transformation sequence(s) from start to end, such that:  
  
>Only one letter can be changed at a time
>Each intermediate word must exist in the dictionary
>For example,

>Given:  
>start = "hit"  
>end = "cog"  
>dict = ["hot","dot","dog","lot","log"]

>Return

>>  [  
>>    ["hit","hot","dot","dog","cog"],  
>>    ["hit","hot","lot","log","cog"]  
>>  ]
  
>Note:

>All words have the same length.  
>All words contain only lowercase alphabetic characters.

这个时候，我们需要多维护一个目录，就是每个单词的前序列表，这个单词可以由哪几个单词到达，每个单词的前序列表其实是一个set（没有重复元素），这里有一个小陷阱，那就是每一轮遍历的时候，找到dict中有的元素，不能马上删除dict中的元素，要等到这轮遍历完毕，因为有可能还有其他单词能够到达这个单词，同时这个单词也不能马上放入队列供下一轮查找，首先要检查这一轮的时候是否已经放置过这个单词，防止下一轮遍历检查的单词有重复。当我们通过BFS查找到变化序列的时候，根据每个单词的前序单词，使用DFS来输出所有可到达路径。

```
//查看每个单词的前序序列
void PrintPreList(map<string,vector<string> > &LadPre)
{
	map<string,vector<string> >::const_iterator  it;
	vector<string>::const_iterator vt_it;
	for (it = LadPre.begin();it != LadPre.end();it++)
	{
		cout << it->first << ":";
		for(vt_it=(it->second).begin();vt_it < (it->second).end(); vt_it++)
		{
			cout << *vt_it << " ";
		}
		cout << endl;
	}
	return;
}

//根据前序序列，通过DFS找到所有路径
void FindPath(map<string,vector<string> > &LadPre, vector<vector<string> > &LadRoad,string start,string end,vector<string> &OneRoad)
{
	if (start == end)
	{
		OneRoad.push_back(start);
		reverse(OneRoad.begin(), OneRoad.end());
		LadRoad.push_back(OneRoad);
		reverse(OneRoad.begin(), OneRoad.end());
		OneRoad.pop_back();
		return;
	}
	OneRoad.push_back(end);
	vector<string>::const_iterator it;
	vector<string> list = LadPre[end];
	for (it=list.begin(); it < list.end(); it++)
	{
		FindPath(LadPre,LadRoad,start,*it,OneRoad);
	}
	OneRoad.pop_back();
	return;
}

//BFS查找最少变化路径
vector<vector<string> > findLadders(string start, string end, set<string> &dict) 
{
        // Start typing your C/C++ solution below
        // DO NOT write int main() function
	vector<vector<string> > LadRoad;
	vector<string> OneRoad;
	map<string,vector<string> >LadPre;	//记录每个单词，在相同最少跳跃次数下可以由哪几个单词跳转来到
	if (start == end)
		return LadRoad;

	queue<string>	LadQue;
	queue<string>	DelQue;
	set<string> RoundSet;	//确保每轮queue入队的元素没有重复的

	LadQue.push(start);
	//有一个小陷阱，如果开始的元素已经在dict里面，在变化检索的时候会产生重复元素
	//if(dict.count(start))
	//	dict.erase(start);
	bool FindEnd = false;
	while(!LadQue.empty())
	{
		int length = LadQue.size();
		while(length--)
		{
			string check = LadQue.front();
			string PreWord = check;
			for(int i=0; i<check.size(); i++)
			{
				char tmp = check[i];	//每次变化一位
				for(char j='a'; j<='z'; j++)
				{
					check[i] = j;
					if (check == end)
					{
						LadPre[check].push_back(PreWord);
						FindEnd = true;						
					}
					else if (dict.count(check))	//if the string is in dict
					{
						if (!RoundSet.count(check))
						{
							RoundSet.insert(check);
							DelQue.push(check);		//这个深度的全部检查完之后，才能从dict中删除
							LadQue.push(check);
						}
						LadPre[check].push_back(PreWord);
					}
				}
				check[i] = tmp;
			}
			LadQue.pop();	//单词检查完毕,出队
		}
		RoundSet.clear();

		if (FindEnd)	//在这个深度的时候找到了最终跳转结果
		{
			//PrintPreList(LadPre);
			FindPath(LadPre,LadRoad,start,end,OneRoad);
			return LadRoad;
		}

		while(!DelQue.empty())
		{
			string tmp = DelQue.front();
			dict.erase(tmp);
			DelQue.pop();
		}
	}
    return LadRoad;    
}
```

[点击我](https://github.com/Puhao/leetcode)查看更多LeetCode题目解答。