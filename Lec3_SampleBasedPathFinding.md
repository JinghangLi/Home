# Lecture3 Sample-based Path Finding
----
## Content

1. Probabilistic Road Map
2. Rapidly-exploring Random Tree
3. Optimal sampling-based path planning methods
4. Advanced path planning methods
5. Implementation
## 1. Probabilistic Road Map 概率路图 (PRM)
- ### 简介
  一种 Sample-based 的寻找路径方法 (第一个介绍的,可能是代表算法), 将随机采样点连接为路图 (可以看做是对 Grid map 的简化),并在此基础上搜索路径,提高效率.
- ### 内容
  - 一种图结构（地图上生成的树状图结构，不是数据结构）
  - path finding过程分为两个阶段
    1. **Learning phase**
         1. 以一种采样方式在 c-space 中随机撒点
         2. 通过查询,删除落在障碍物上的采样点  
            ![LearningPhase-SampleNodes](https://gitee.com/jinghangli/imagebed/raw/master/20210223181702.png)
         3. 连接邻近的 sample nodes 构成 Road map
         4. 通过查询,删除越过障碍物的连线  
            ![LearningPhase-RoadMap](https://gitee.com/jinghangli/imagebed/raw/master/20210223182054.png)

    2. **Query phase**
         1. 在生成的 Road map (类似于简化版本的 Grid map) 上使用搜索算法找到最优路径  
            ![QueryPhase](https://gitee.com/jinghangli/imagebed/raw/master/20210223182649.png)

- ### 优点
  - 是概率完备的搜索算法,一定会有解(能够找到路径)
  - 不需要在整个地图环境中搜索Grid map, 而是搜索简化版本的 Road map, 效率高
- ### 缺点
  - 边界值问题: 没有考虑 Robot 的约束, 某些 Road map 的连线可能不适合 Robot 运动(eg 两个连线夹角太小)
  - 算法冗长,不够高效
- ### 优化
  - **Lazy collision-checking (针对解决算法冗长问题)** 算法流程

      1. **Learning phase**
          1. 以一种采样方式在 c-space 中随机撒点
          2. ~~通过查询,删除落在障碍物上的采样点~~ **(Lazy)**
          3. 连接邻近的 sample nodes 构成 Road map
          4. ~~通过查询,删除越过障碍物的连线~~ **(Lazy)**
            
            ![LazyRoadmap](https://gitee.com/jinghangli/imagebed/raw/master/20210223184755.png)

      2. **Query phase**
          1. 在生成的 Road map (类似于简化版本的 Grid map) 上使用搜索算法找到最优路径
          2. **Collision-checking if necessary**
            - If Collision
    
            ![IfCollision](https://gitee.com/jinghangli/imagebed/raw/master/20210223185844.png)

            ![](https://gitee.com/jinghangli/imagebed/raw/master/20210223185921.png)
            ![](https://gitee.com/jinghangli/imagebed/raw/master/20210223185939.png)

## 2. Rapidly-exploring Random Tree 快速搜索随机树 (RRT)
- ### 简介
  一种增量式的快速随机算法,在 Learning 的同时 Query, 当目标点或者其周围的点 (因为采样可能无法正好采集到目标点) 被采样到后, 则树结构的"生长"过程结束,由节点回溯父节点则可得到需要的路径. 该算法在复杂或高维环境下更高效,
- ### 内容
  1. 使用 **Sample 函数** (可选) 采样随机点 Xrand 
  2. 使用 **Near 函数** 找到树结构上在 Xrand 附近的点 Xnear
  3. 使用 Steer 函数 从点 Xnear 向 Xrand 移动一定的距离 (StepSize) 生成点 Xnew
  4. 连接 Xnew 和 Xnear 生成edge Ei
  5. 使用 **Collision-checking** 检测 Ei 是否 collision-free, 若是则 Xnew 和 Ei 加入树结构, 若不是则删除.
  6. 重复 1-5 直到采样到终点或终点附近点.
   
     ![RRT伪代码](https://gitee.com/jinghangli/imagebed/raw/master/20210224214755.png)

   ![RRTnear](https://gitee.com/jinghangli/imagebed/raw/master/20210224214838.png)
   ![RRTcollisioncheck](https://gitee.com/jinghangli/imagebed/raw/master/20210224215002.png)
   ![RRTresult](https://gitee.com/jinghangli/imagebed/raw/master/20210224215031.png)
   
   [RRT演示视频](https://www.youtube.com/watch?v=pOFtvZ_qVsA)
- ### 优点
  - 相比 PRM, 能够更针对的生成一条到终点的路线, 算法的速度提升.
- ### 缺点
  - 生成的路径可能不是最优的
  - 还是用直线连接生成的路径, 对 robot control 不是最优的
  - 在整个空间中采样
  - 效率不够高 (可以对 **Sample Near Collision-checking** 等函数改进, 提高效率)
- ### 改进
  #### 1. Kd-tree (改进 **Near()** 函数, 提高查找临近点的效率)   
  是一种数据的保存格式 (与 Road map 的树状结构不同)  
  ![Road map](https://gitee.com/jinghangli/imagebed/raw/master/20210227155539.png)
  ![Kd-tree划分](https://gitee.com/jinghangli/imagebed/raw/master/20210227155633.png)  
  ![data structure](https://gitee.com/jinghangli/imagebed/raw/master/20210227155724.png)  
  > **我觉得查找的过程有点像二分法查找**  
  > Other variants: Spatial grid, hill climbing,etc  
  > 参考: https://blog.csdn.net/junshen1314/article/details/51121582  

  #### 2. Bidirectional RRT / RRT Connect  
  从起点和终点同时生成树状 Road map, 当两个 "tree" 相连接的时候则找到路径. 可以提高查找路径的效率, 并且在存在 Narrow passage(狭窄区域)时效果较好[1], 但不是唯一解, 该问题更倾向于从改造 **Sample 函数**入手.   
   ![RRT Connect](https://gitee.com/jinghangli/imagebed/raw/master/20210227155936.png)
   ![1](https://gitee.com/jinghangli/imagebed/raw/master/20210227160557.png "1")

## 2. Optimal sampling-based path planning methods 基于RRT的优化方法

### 1. Rapidly-exploring Random Tree* (RRT star)   
- #### 简介  
  用于优化 RRT 生成路径
- #### 内容  
  ![RRT*伪代码](https://gitee.com/jinghangli/imagebed/raw/master/20210227162626.png)  
  **与 RRT 不同点 1:**  
  ![Near()](https://gitee.com/jinghangli/imagebed/raw/master/20210227163117.png)  
  以 Xnew 为中心, 设定距离为半径寻找所有的 Xnear, 而不是直接连接 Xnew 和 Xnear.  
  **与 RRT 不同点 2:**  
    ![ChooseParent](https://gitee.com/jinghangli/imagebed/raw/master/20210227163421.png)  
    ![AddNodeEdge](https://gitee.com/jinghangli/imagebed/raw/master/20210227163536.png)  
  根据 CostFunction (eg: Xnew 经过选中的 Xnear 到 Xinit 的距离) 选取合适的 Xnear 然后生成 Edge 加入到树结构.  
  **与 RRT 不同点 3:**  
  ![Rewire](https://gitee.com/jinghangli/imagebed/raw/master/20210227163608.png)  
  ![RewireEnd](https://gitee.com/jinghangli/imagebed/raw/master/20210227164953.png)  
  __RRT*优化重点:__ 在将 Xnew 和生成的 Edge 加入树结构后, 检查刚刚选中的 Xnear 点经过 Xnew 到达 Xinit 的 Cost, 然后对于 Cost 大的原路径剪枝, 重新连接 Cost 小的路径,进行优化 (该优化过程在找到路径后不会停止,而是继续剪枝, 优化当前找到的路径)  
  [RRT*演示视频](https://www.youtube.com/watch?v=YKiQTJpPFkA)

### 2. Kinodynamic-RRT* (RRT star 优化)   
- #### 简介  
  用于解决由于机器人运动约束导致的优化问题.  
- ### 内容  
  __与 RRT* 不同点:__  
  Steer() 连接 Xnew 与 Xnear 时用符合机器人约束的曲线, 而不是直线  
  ![Kinodynamic-RRT*](https://gitee.com/jinghangli/imagebed/raw/master/20210227170629.png)  
  - #### 可能的问题 (可以继续考虑的地方)  
    ![可能的问题](https://gitee.com/jinghangli/imagebed/raw/master/20210227170816.png)  
    [Kinodynamic-RRT*演示视频](https://www.youtube.com/watch?v=RB3g_GP0-dU)  

### 3. Anytime-RRT* (RRT star 优化)  
- #### 简介  
  在运动的过程中不断重新开始路径规划 (将机器人当前位置不断作为 Xinit)  
  [Anytime-RRT*论文](https://ieeexplore.ieee.org/document/5980479/;jsessionid=klziwfvJ6lsVx7VJRCzagoDgvQj2OaQFfd64VfLvXh9HKg99-rzO!-785819610)  

## 3. Advanced Sampling-based Methods 对采样生成路径的优化
### 1. Informed RRT*  
- #### 简介  
  RRT* 在找到路径后, 会在整个空间中继续 Sample 改进路径, 效率低.  
  Informed RRT* 将 Sample 过程限制在一个椭圆内, 针对性强, 效率高.  
  ![RRT* vs. Informed RRT*](https://gitee.com/jinghangli/imagebed/raw/master/20210227175143.png)  
- #### 内容  
  ![Informed RRT*](https://gitee.com/jinghangli/imagebed/raw/master/20210227175330.png)  
  **椭圆生成方法:** 椭圆上的动点到两焦点的距离和 = Path 的长度 (随时变化的)  
### 2. Cross-entropy motion planning
- #### 内容 
    1. 用某一方法生成一条路径
    2. 在每个节点的周围采样 (多个节点构成了一个多高斯模型)
    3. 由生成的最优路径计算得到一个均值 用于下一轮采样的高斯模型  
   ![Cross-entropy motion planning1](https://gitee.com/jinghangli/imagebed/raw/master/20210227175804.png)  
   ![Cross-entropy motion planning2](https://gitee.com/jinghangli/imagebed/raw/master/20210227175826.png)  
   ![Result](https://gitee.com/jinghangli/imagebed/raw/master/20210227175917.png)
   