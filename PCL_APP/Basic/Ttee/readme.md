# 存储结构 点搜索
## kdTree 二叉树
## Octree 八叉树
    ##################################################
# L 点云数据管理 
    ###################################################

    点云压缩，点云索引（KDtree、Octree），点云LOD（金字塔），海量点云的渲染

    通过雷达，激光扫描，立体摄像机等三维测量设备获取的点云数据，具有数据量大，
    分布不均匀等特点，作为三维领域中一个重要的数据来源，点云主要是表征目标表面的海量点的集合，
    并不具备传统网格数据的几何拓扑信息，所以点云数据处理中最为核心的问题就是建立离散点间的拓扑关系，
    实现基于邻域关系的快速查找。

    建立空间索引在点云数据处理中有着广泛的应用，
    常见的空间索引一般 是自顶而下逐级划分空间的各种空间索引结构，
    比较有代表性的包括BSP树，KD树，KDB树，R树，四叉树，八叉树等索引结构，
    而这些结构中，KD树和八叉树使用比较广泛 


## KDTree　　一种递归的邻近搜索策略
     kd树（k-dimensional树的简称），是一种分割k维数据空间的数据结构。
    主要应用于多维空间关键数据的搜索（如：范围搜索和最近邻搜索）。
    其实KDTree就是二叉搜索树的变种。这里的K = 3.

    K-D树是二进制空间分割树的特殊的情况。用来组织表示K维空间中点的几何，
    是一种带有其他约束的二分查找树，为了达到目的，通常只在三个维度中进行处理因此所有的kd_tree
    都将是三维的kd_tree,kd_tree的每一维在指定维度上分开所有的字节点，
    在树 的根部所有子节点是以第一个指定的维度上被分开。

    k-d树算法可以分为两大部分，
    一部分是有关k-d树本身这种数据结构建立的算法，       构建算法
    另一部分是在建立的k-d树上如何进行最邻近查找的算法。  

    =====================
### 1 构建算法
    k-d树是一个二叉树，每个节点表示一个空间范围。
    表1给出的是k-d树每个节点中主要包含的数据结构。

    父节点
    k-d树是一个二叉树，每个节点表示一个空间范围。表1给出的是k-d树每个节点中主要包含的数据结构。
    域名         数据类型       描述
    Node-data    数据矢量      数据集中某个数据点，是n维矢量（这里也就是k维）
    Range	     空间矢量      该节点所代表的空间范围
    split        整数          垂直于分割超平面的方向轴序号
    Left         k-d树         由位于该节点分割超平面左子空间内所有数据点所构成的k-d树
    Right        k-d树         由位于该节点分割超平面右子空间内所有数据点所构成的k-d树
    parent       k-d树         父节点


    先以一个简单直观的实例来介绍k-d树算法。假设有6个二维数据点
    {
    （2,3），
    （5,4），
    （9,6），
    （4,7），
    （8,1），
    （7,2）}，
    数据点 位于二维空间内（如图1中黑点所示）。
    k-d树算法就是要确定图1中这些分割空间的分割线（多维空间即为分割平面，一般为超平面）。
    下面就要通过一步步展 示k-d树是如何确定这些分割线的。

    由于此例简单，数据维度只有2维，所以可以简单地给x，y两个方向轴编号为0,1，也即split={0,1}。
    （1）确定split域,首先该取的值。分别计算x，y方向上数据的方差, x方向上的方差最大，所以split域值首先取0，也就是x轴方向；
    （2）确定Node-data的域值。根据x轴方向的值2,5,9,4,8,7排序选出中值为7，所以Node-data = （7,2）。
         这样，该节点的分割超平面就是通过（7,2）并垂直于split = 0（x轴）的直线x = 7；
    （3）确定左子空间和右子空间。分割超平面x = 7将整个空间分为两部分，如图2所示。
         x < = 7的部分为左子空间，包含3个节点{（2,3），（5,4），（4,7）}；
         另一部分为右子空间，包含2个节点{（9,6），（8,1）}。
    （4）k-d树的构建是一个递归的过程。然后对左子空间和右子空间内的数据重复根节点的过程就可以得到下一级子节点（5,4）和（9,6）
    （也就是 左右子空间的'根'节点），同时将空间和数据集进一步细分。
    如此反复直到空间中只包含一个数据点，如图1所示。最后生成的k-d树如图3所示。
[参考](https://blog.csdn.net/silangquan/article/details/41483689)
    ==============================================================

### 查找算法
    在k-d树中进行数据的查找也是特征匹配的重要环节，其目的是检索在k-d树中与查询点距离最近的数据点。
    这里先以一个简单的实例来描述最邻近查找的基本思路。
    星号表示要查询的点（2.1,3.1）。通过二叉搜索，顺着搜索路径很快 就能找到最邻近的近似点，
    也就是叶子节点（2,3）。而找到的叶子节点并不一定就是最邻近的，最邻近肯定距离查询点更近，
    应该位于以查询点为圆心且通过叶 子节点的圆域内。为了找到真正的最近邻，还需要进行'回溯'操作：
    算法沿搜索路径反向查找是否有距离查询点更近的数据点。此例中先从（7,2）点开始进行 二叉查找，然后到达（5,4），最后到达（2,3），
    此时搜索路径中的节点为<（7,2），（5,4），（2,3）>，首先以（2,3）作为 当前最近邻点，计算其到查询点（2.1,3.1）的距离为0.1414，
    然后回溯到其父节点（5,4），并判断在该父节点的其他子节点空间中是否有距离查 询点更近的数据点。
    以（2.1,3.1）为圆心，以0.1414为半径画圆，如图4所示。发现该圆并不和超平面y = 4交割，因此不用进入（5,4）节点右子空间中去搜索。



    =================================================
    ====================================================

## 八叉树（Octree）是一种用于描述三维空间的树状数据结构。
    八叉树的每个节点表示一个正方体的体积元素，每个节点有八个子节点，
    这八个子节点所表示的体积元素加在一起就等于父节点的体积。
    一般中心点作为节点的分叉中心。

[参考](https://blog.csdn.net/u013019296/article/details/70052307)

    OcTree是一种更容易理解也更自然的思想。
    对于一个空间，如果某个角落里有个盒子我们却不知道在哪儿。
    但是"神"可以告诉我们这个盒子在或者不在某范围内，
    显而易见的方法就是把空间化成8个卦限，然后询问在哪个卦限内。
    再将存在的卦限继续化成8个。
    意思大概就是太极生两仪，两仪生四象，四象生八卦，
    就这么一直划分下去，最后一定会确定一个非常小的空间。
    对于点云而言，只要将点云的立方体凸包用octree生成很多很多小的卦限，
    那么在相邻卦限里的点则为相邻点。

    显然，对于不同点云应该采取不同的搜索策略，如果点云是疏散的，
    分布很广泛，且每什么规律（如lidar　雷达　测得的点云或双目视觉捕捉的点云）kdTree能更好的划分，
    而octree则很难决定最小立方体应该是多少。太大则一个立方体里可能有很多点云，太小则可能立方体之间连不起来。
    如果点云分布非常规整，是某个特定物体的点云模型，则应该使用ocTree，
    因为很容易求解凸包并且点与点之间相对距离无需再次比对父节点和子节点，更加明晰。
    典型的例子是斯坦福的兔子。



    1点云获取　(智能扫描　点云配准)
    2点云处理　( 点云去噪　特征增强　法向估计)
    3点云表示　( 点云渲染　骨架提取)
    4点云重构　(静态建模　动态建模)

### 实现八叉树的原理 
      (1). 设定最大递归深度。
      (2). 找出场景的最大尺寸，并以此尺寸建立第一个立方体。
      (3). 依序将单位元元素丢入能被包含且没有子节点的立方体。
      (4). 若没达到最大递归深度，就进行细分八等份，再将该立方体所装的单位元元素全部分担给八个子立方体。
      (5). 若发现子立方体所分配到的单位元元素数量不为零且跟父立方体是一样的，
           则该子立方体停止细分，因为跟据空间分割理论，细分的空间所得到的分配必定较少，
           若是一样数目，则再怎么切数目还是一样，会造成无穷切割的情形。
      (6). 重复3，直到达到最大递归深度。

### 八叉树的逻辑结构如下：

       假设要表示的形体V可以放在一个充分大的正方体C内，C的边长为2n，形体V=C，
       它的八叉树可以用以下的递归方法来定义：八 叉树的每个节点与C的一个子立方体对应，

       树根与C本身相对应，如果V＝C，那么V的八叉树仅有树根，如果V≠C，则将C等分为八个子立方体，
       每个子立方体 与树根的一个子节点相对应。只要某个子立方体不是完

       全空白或完全为V所占据，就要被八等分，从而对应的节点也就有了八个子节点。
       这样的递 归判断、分割一直要进行到节点所对应的立方体或是完全空白，或是完全为V占

       据，或是其大小已是预先定义的体素大小，并且对它与V之交作一定的“舍入”，
       使 体素或认为是空白的，或认为是V占据的。


    可以十分高效的实现八叉树的建立管理等操作，并且节点实现对临近树节点的结构的探测，对应到空间点云，
    其就可以对空间曲面的动态变化进行探测，在进行空间动态变化探测中非常有用
    
    
    ############################################
### 变化检测 Octree 八叉树算法
    ########################################################

[参考](https://blog.csdn.net/u013019296/article/details/70052307)

    当无序点云在连续变化中，八叉树算法常常被用于检测变化，
    这种算法需要和关键点提取技术结合起来，八叉树算法也算是经典中的经典了。
     octree是一种用于管理稀疏3D数据的树状数据结构，
    我们学习如何利用octree实现用于多个无序点云之间的空间变化检测，
    这些点云可能在尺寸、分辨率、密度和点顺序等方面有所差异。
    通过递归地比较octree的树结构，
    可以鉴定出由octree产生的体素组成之间的区别所代表的空间变化，
    此外，我们解释了如何使用PCL的octree“双缓冲”技术，
    以便能实时地探测多个点云之间的空间组成差异。


#### 八叉树空间分割进行点分布区域的压缩
    通过对连续帧之间的数据相关分析，检测出重复的点云，并将其去除掉再进行传输.

    点云由庞大的数据集组成，这些数据集通过距离、颜色、法线等附加信息来描述空间三维点。
    此外，点云能以非常高的速率被创建出来，因此需要占用相当大的存储资源，
    一旦点云需要存储或者通过速率受限制的通信信道进行传输，提供针对这种数据的压缩方法就变得十分有用。
    PCL库提供了点云压缩功能，它允许编码压缩所有类型的点云，包括“无序”点云，
    它具有无参考点和变化的点尺寸、分辨率、分布密度和点顺序等结构特征。
    而且，底层的octree数据结构允许从几个输入源高效地合并点云数据
    .

    Octree八插树是一种用于描述三维空间的树状数据结构。
    八叉树的每个节点表示一个正方体的体积元素，每个节点有八个子节点，
    将八个子节点所表示的体积元素加在一起就等于父节点的体积。
    Octree模型：又称为八叉树模型，若不为空树的话，
    树中任一节点的子节点恰好只会有八个，或零个，也就是子节点不会有0与8以外的数目。

    Log8(房间内的所有物品数)的时间内就可找到金币。
    因此，八叉树就是用在3D空间中的场景管理，可以很快地知道物体在3D场景中的位置，
    或侦测与其它物体是否有碰撞以及是否在可视范围内。

    基于Octree的空间划分及搜索操作

    octree是一种用于管理稀疏3D数据的树状数据结构，每个内部节点都正好有八个子节点，
    本小节中我们学习如何用octree在点云数据中进行空间划分及近邻搜索，特别地，
    解释了如何完成
    “体素内近邻搜索(Neighbors within Voxel Search)”、
    “K近邻搜索(K Nearest Neighbor Search)”和
    “半径内近邻搜索(Neighbors within Radius Search)”。

    