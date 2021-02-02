# Lecture2 SEARCH-BASED PATH FINDING
----

## 1. 机器人的配置空间生成

将机器人转化为质点，将障碍物膨胀转化为C-space中的障碍物

## 2. 图和搜索算法

### **1. 图表示方法**

* 图由节点和边构成
* 图分为有向图和无向图
* 图的边分为有权重与无权重
* Grid map 可以天然表示为图结构，因为 grid 之间天然连接

### **2. Search based method**
* 根据 Grid map 生成搜索树
* General process：
  * Maintain a **container** to store all the nodes to be visited
  * The container is initialized with the start state Xs
  * **Loop**
    * **Remove** a node from the container according to some pre-defined score function
    * **Expansion** Obtain all neighbors of the node
    * **Push** them (neighbors) into the container
  * End Loop
* Methods

    Breadth First Search (BFS)|Depth First Search (DFS)
    :----:|:----:
    搜索树一层层推进|搜索树一条路走到头
    能找到全局最优路径，但是搜索时间长|搜索时间短，但可能不是最佳路径（因为可能在遍历所有节点之
    需要遍历所有节点|在遍历所有节点之前可能找到了最佳路径
    ![Breadth First Search (BFS) vs. Depth First Search (DFS)](https://github.com/JinghangLi/Homework_of_MotionPlanning/blob/main/picture/L2-BFSvsDFS.jpg)

### **3. Heuristic search**
为解决BFS均匀向外搜索，**遍历所有节点时间过多** 的问题，引入启发式搜索概念。但是算法过于 Greedy 可能会导致局部最优结果。

    Definition: A heuristic is a guess of how close you are to the target

 * 与BFS相比，启发式算法（Greedy Best-First Search）在遍历节点时更有目的性
 * 但是在遇到障碍物时 Greedy Best-First Search 会陷入局部最优，而BFS虽然耗时久但是能找到全局最优路径
  
  （插图）
    
### **4. Dijkstra and A***
考虑图中 cost of edge ，同时将启发式搜索与 BFS 结合，把BFS中的 **container** 变为 **priority queue**
    
    Strategy: expand/visit the node with cheapest accumulated cost g(n)

* g(n): The current best estimates of the accumulated cost from the start state to node “n”
* Update the accumulated costs g(m) for all unexpanded neighbors “m” of node “n”
* A node that has been expanded/visited is guaranteed to have the smallest cost from the start state

Dijkstra算法伪代码

![Dijkstra算法伪代码](https://github.com/JinghangLi/Homework_of_MotionPlanning/blob/main/picture/L2-Dijkstra%E4%BC%AA%E4%BB%A3%E7%A0%81.jpg)

![Dijkstra算法演示](https://github.com/JinghangLi/Homework_of_MotionPlanning/blob/main/picture/L2-Dijkstra%E6%BC%94%E7%A4%BA.jpg)

**Dijkstra算法优缺点**

优点：全局搜索 

缺点：
