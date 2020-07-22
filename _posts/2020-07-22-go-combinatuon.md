---
layout: post
title: golang Combination(조합) 구현
date: 2020-07-22
category: "Golang"
tags: [Golang]
author: Jaehyun Lee
comment: false
---
{% highlight go %}
import (
	"container/list"
	"fmt"
)

type Combination struct {
	n int
	c int
	list *list.List
}

func (comb *Combination) setParams(N, C int) *Combination {
	(*comb).n = N
	(*comb).c = C
	(*comb).list = list.New()
	return comb
}

func (comb *Combination) getLists() *list.List {
	return (*comb).list
}

func (comb *Combination) makeCombination() {
	selected := make([]bool, (*comb).n )
	comb.dfs(0,0, (*comb).c, selected)
}

func (comb *Combination) dfs(idx,cnt,target int, selected []bool) {
	if cnt == target {
		result := make([]bool, (*comb).n)
		for i := 0 ; i < len(result); i++ {
			result[i] = selected[i]
		}
		comb.list.PushBack(result)
		return
	}
	for i := idx ; i < (*comb).n ; i++ {
		if selected[i] {
			continue
		}
		selected[i] = true
		comb.dfs(i, cnt + 1, target, selected)
		selected[i] = false
	}
}

func New() Combination{
	return Combination{
		n : 0,
		c : 0,
		list : list.New(),
	}
}
{% endhighlight %}
