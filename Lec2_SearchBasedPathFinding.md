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
    
    ![Breadth First Search (BFS) vs. Depth First Search (DFS)](https://gitee.com/jinghangli/imagebed/raw/master/L2-BFSvsDFS.jpg)

### **3. Heuristic search**
- ### 优点与缺点
  可以解决BFS均匀向外搜索，**遍历所有节点时间过多** 的问题，引入启发式搜索概念。但是算法过于 Greedy 可能会导致局部最优结果。

      Definition: A heuristic is a guess of how close you are to the target

  * 与BFS相比，启发式算法（Greedy Best-First Search）在遍历节点时更有目的性
  * 但是在遇到障碍物时 Greedy Best-First Search 会陷入局部最优，而BFS虽然耗时久但是能找到全局最优路径
    
    红色路径是贪心搜索算法生成的,蓝色路径是BFS生成的
    
    ![Greedy Best-First Search VS BFS](http://theory.stanford.edu/~amitp/GameProgramming/concave1.png?2017-11-25-21-35-04)
    
***
### **4. Dijkstra**
- ### 优点
  比BFS需要拓展的节点少, 考虑图中 cost of edge 即 g(n), 同时将启发式搜索与 BFS 结合，把BFS中的 **container** 变为 **priority queue** (比BFS多了一个优先序列,根据g(n)值为节点n排序,g(n) 值最小的被优先扩展 )
      
      Strategy: expand/visit the node with cheapest accumulated cost g(n)

  * g(n): The current best estimates of the accumulated cost from the start state to node “n”
  * Update the accumulated costs g(m) for all unexpanded neighbors “m” of node “n”
  * A node that has been expanded/visited is guaranteed to have the smallest cost from the start state

  Dijkstra算法伪代码
  ```
  • Maintain a priority queue to store all the nodes to be expanded
  • The priority queue is initialized with the start state Xs
  • Assign g(Xs)=0, and g(n)=infinite for all other nodes in the graph

  • Loop
    • If the queue is empty, return FALSE; break;
    • Remove the node “n” with the lowest g(n) from the priority queue (openlist)
    • Mark node “n” as expanded
    • If the node “n” is the goal state, return TRUE; break;
    • For all unexpanded neighbors “m” of node “n”
      • If g(m) = infinite (说明m是未发现的新节点)
        • g(m)= g(n) + Cnm 
        • Push node “m” into the queue
      • If g(m) > g(n) + Cnm
        • g(m)= g(n) + Cnm
    • end
  • End Loop
  ```
  Dijkstra算法过程
  ![Dijkstra算法演示](https://gitee.com/jinghangli/imagebed/raw/master/20210209170027.png)

  **Dijkstra算法优缺点**

  优点：全局搜索得到最优结果

  缺点：
  1. 对于n节点的 neighbors 的搜索是均匀扩散的（n的所有扩展节点都被压进了openlist，并计算g(n)值） 
  2. 如果边的数值是 equal 的（Cnm），则 Dijkstra 就是 BFS 算法
  3. 没有 goal location 的信息 (没有方向性？)

  **为了解决 uniform cost search 的问题，引入 启发式（Heuristic） 函数 即关于终点的估计值从而提出A***

***
### 5. A*算法 (Dijkstra with a Heuristic h(n)) 
- ### 优点
  比 Dijkstra 对目标更有指向性，采取的方法是： openlist 的排序由 f(n)=g(n)+h(n) 决定，最小 f(n) 值最先弹出并拓展 **（与Dijkstra算法流程唯一不同的地方）**
  ```
  h(n): The estimated least cost from node n to goal state (i.e. goal cost)
  ```


- ### 算法内容

  ![A* Example](https://gitee.com/jinghangli/imagebed/raw/master/20210209165107.png)

- ### 缺点及原因
  当对节点的估计值过大时（即 h(n)值）openlist的节点排序会出错

  错误实例

  ![错误实例](https://gitee.com/jinghangli/imagebed/raw/master/20210209200126.png)

  对节点 A 的估计h(n) > 实际h(n), 导致在openlist中最先被压出的是点 G
- ### 可能的解决思路
  使得每个节点的 估计h(n) < 实际h(n)

  **一个可接受的启发方法的条件 A Heuristic h is admissible (optimistic) if:**

    __h(n) <= h*(n) for all node “n”, where h*(n) is the true least cost to goal from node “n”__
    - If the heuristic is admissible, the A* search is optimal
    - Coming up with admissible heuristics is most of what’s involved in using A* in practice.
    - 启发式方法的设计对每个实例都不同 **h(n)的选择**
      - Euclidean distance (L2 norm) 一定是小于实际值
      - Manhattan distance (L1 norm) 在机器人不能对角线移动时是最优的
      - L∞ norm distance 总是最优的
      - 0 distance 时A*退化为 Dijkstra 算法
  
  黄色:Euclidean Distance  绿色: Manhattan Distance

  ![](https://gitee.com/jinghangli/imagebed/raw/master/20210209211719.png)
  

- ### 次优解 (Sub-optimal Solution)
  __令 h(n) >= h*(n) [估计值大于等于实际值] 来提升算法的计算速度,得到次优解,即 Weighted A*__
  
  __Weighted A*: Expands states based on f = g + εh, ε > 1 =bias towards states that are closer to goal.__

  ε 取值; f=g+εh|算法
  :----:|:----:
    ε = 0; f=g | Dijkstra
  ε = 1; f=g+h | A* 算法
  ε = ∞; f= εh | 贪心算法

  Weighted A*:  ε 取值是算法速度和算法最优性之间的取舍, ε 增大时则用**算法的最优性来换取计算速度** 

***
### 6. Greedy Best First Search vs. Weighted A* vs. A*
  Dijkstra : a = 1, b = 0

  ![Greedy Best First Search vs. Weighted A* vs. A*](https://gitee.com/jinghangli/imagebed/raw/master/20210209215431.png)

***
### 7. A* 算法的工程实现
1. Heuristic 的选择
  - Diagonal Heuristic 的效果比 Euclidean Heuristic 效果更好,因为 Diagonal Heuristic 有固定解,其结果就是真实的距离值
  ![示例](https://gitee.com/jinghangli/imagebed/raw/master/20210209225545.png)
    ```
    dx=abs(node.x −goal.x); 
    dy=abs(node.y −goal.y); 
    h=(dx+dy)+(√2−2)∗min(dx,dy)
    ```
  - Diagonal Heuristic VS. Euclidean Heuristic
  ![](https://gitee.com/jinghangli/imagebed/raw/master/20210209225719.png)
  Diagonal Heuristic 比 Euclidean Heuristic 扩展的节点数要少.
2. Tie Breaker
  - 需要 Tie Breaker 的原因
  
    许多路径中的节点有相同的 f(n) 值, 这是由于路径具有对称性 (起点到终点的路径长度相同,但是路径形状不同),导致算法在两条相似的路径中来回跳转,拓展了很多不必要的节点.

  - 多种解决办法
    
    **Core idea of tie breaker:** Find a preference among same cost paths
  
    - 当两个节点的 f(n) 相同时,可以轻微的放大其中一个节点的 h(n) ,使算法选择其中一条路走下去.
    $$ h=h×(1.0+p)\text {，}\quad p<\frac{minimum cost of one step}{expected maximum path cost}\text {，例如 } \frac{一步的cost}{地图最大路径的cost}$$ 
    - When nodes having same f, compare their h.
    - Add deterministic random numbers to the heuristic or edge costs (A hash of the coordinates).
    - Prefer paths that are along the straight line from the starting point to the goal.
    - ... Many customized ways
  
   - Tie breaker 的可能缺点
      
     在遇到障碍物时, 可能会生成虽然轨迹最优, 但是对轨迹优化带来困难的路径, 如下图所示
    ![Tie breaker 遇到障碍物时](https://gitee.com/jinghangli/imagebed/raw/master/20210210162653.png)

### 8. Jump Point Search (JPS)

- ### 优点
  在保留 A* 算法的框架的同时，进一步优化了 A*算法寻找后继节点的操作, 减少了需要探索和拓展的节点.
- ### 算法内容
  - **Core Idea of JPS:** Find symmetry and break them.

  - **Look Ahead Rule:** JPS探索节点的规则
    - Neighbor Pruning (邻居节点修剪)

      判断节点是否需要被探索: 如果从 x 到被探索节点的总 cost, 小于等于 x 的父节点从其他支路(不经过x)的总 cost, 则该节点可以被探索.

      **直线运动的修剪规则是小于等于; 对角运动的修剪规则是小于** (因此对角运动中节点 2,5 被保留)

      ![Neighbor Pruning 规则](https://gitee.com/jinghangli/imagebed/raw/master/20210210165800.png)

      白色节点为被探索的节点, 灰色为被修剪的节点
    - Forced Neighbors (在有障碍物的情况下)
      
      ![](https://gitee.com/jinghangli/imagebed/raw/master/20210210171746.png)

      红色节点为 forced neighbors, 可以被探索, 因为从 x 的父节点到红色节点的最短路径被障碍物阻断了.

  - **Jumping Rule:** JPS 跳跃规则
    - 对所有节点优先考虑直线跳跃
    - 发现特殊节点时 (由 Look Ahead Rule 决定) 则回溯该方向跳跃的父节点或者中继节点 (因为跳跃是直线的,不是折线的,所以中继节点也要加入 openlist),并将他们加入 openlist (eg: y 节点)
      
      ![直线跳跃](https://gitee.com/jinghangli/imagebed/raw/master/20210210173432.png)
        
        上图中, 在节点 y 跳跃时发现了Forced neighbor节点 z, 因此回溯该次跳跃, 将特殊节点 y 加入openlist.

      ![对角跳跃](https://gitee.com/jinghangli/imagebed/raw/master/20210210174756.png)
    
        上图中, 在节点 z 跳跃时发现了Forced neighbor节点 (红色区域), 因此回溯该次跳跃, 将特殊节点 z 加入openlist, 同时节点 y 为本次跳跃的中继节点, 因此把节点 y 也加入 openlist.
  - **JPS伪代码** 
      
    流程与A* 算法相同, 不同的是A* 算法拓展的是所有几何连接的邻居节点, JPS拓展的是跳跃节点.

    ![JPS伪代码](https://gitee.com/jinghangli/imagebed/raw/master/20210210175540.png)
    
- ### 缺点及原因

  JPS 算法在复杂环境下表现优于A* 算法 (尤其在解决迷宫问题的时候), 但是当地图很大且在视角中没有检测到障碍物时, JPS会花费大量的时间遍历空旷地区的节点. 这在机器人导航中很常见 (有限机器人FOV, 大的全局/局部地图). 
- ### 结论
  - JPS 算法在复杂环境下表现优于A* 算法 (尤其在解决迷宫问题的时候)
  - JPS 虽然减少了 openlist 中的节点数量,但是增加了 status query 的数量(对是否有障碍物的碰撞检查 )
  - **JPS的局限性:** 只适用于 uniform grid map (边有权值时JPS无法使用)