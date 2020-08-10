---
layout: post
title: System Parallelization
subtitle: Directed Acyclic Graph
#cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/dag.pngl
#share-img: /assets/img/path.jpg
tags: [dag, directed acyclic graph, c++]
---

### Summary
I've been working through ordering my ecs systems, such that they run in a valid order so it can be threaded safely.  Working through this issue i've found the needs of the structure to be the following:
1. Systems may only be run once in a given update
2. Dependencies must not result in circular dependencies
3. Systems can run in parallel (only if they do not have overlapping reading/writing of data in the ecs store)
4. Systems can have multiple tasks within them
5. Tasks within a system can specify if it can be run in parallel to other tasks within the system.

With those requirements it's begins to sounds like what we need is a Directed Acyclic Graph (DAG).

### Defining the basic structure

Let's start with setting up the node of the graph. 

```c++
struct Node {
	uint32_t id;
	std::vector<Node*> dependencies;
};
```

Let's walk through the node really quick. First we have have an identifier (`id`) that we will use when we are determining which nodes this task is dependent on. Next we need a list of nodes

```c++
struct Node {
	uint32_t id;
	std::vector<Node*> dependencies;
	Node *sibling;
	std::unordered_set<uint32_t> completed_dependencies;
};
```


```c++
struct graph {
	Node* root;
};
```