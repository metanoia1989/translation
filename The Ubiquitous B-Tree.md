这篇论文篇幅太长，很多长难句，如果粗糙地忽略过就达不到训练自己的目的，精准地达成目的才是我要的。    
所以先粗略地翻译一遍，然后第二篇来审校，确定哪些句子有问题。

# The Ubiquitous B-Tree 无处不在的B树
Author: Douglas Comer       
Computer Sctence Department, Purdue Untverstty, West Lafayette, Indiana 47907

B-trees have become, de facto, a standard for file organization. File indexes of users,
dedicated database systems, and general-purpose access methods have all been proposed
and nnplemented using B-trees This paper reviews B-trees and shows why they have
been so successful It discusses the major variations of the B-tree, especially the B+-tree,
contrasting the relative merits and costs of each implementatmn. It illustrates a general
purpose access method which uses a B-tree.
Keywords and Phrases: B-tree, B*-tree, B+-tree, file organization, index
CR Categorws: 3.73 3.74 4.33 4 34

B树已经成为事实上的文件组织的标准。用户的文件索引、专用的数据库系统和所有建议及已补充的使用B树作为通用访问方法的。
这篇论文回顾B树为什么获得了如此的成功，它讨论B树的一些主要变体，特别是B+树，对照每一种实现相关的价值和代价。
它说明了使用B树的通用访问方法。

关键词和短语：B树、B*树、B+树、文件组织、索引   


INTRODUCTION    介绍        
Operations on a File    操作一个文件        
1 THE BASIC B-TREE  基础的B树       
    Balancing   平衡        
    Insertion   插入        
    Deletion    删除        
2 THE COST OF OPERATIONS    操作的花费      
    Retrieval Costs     检索的开销      
    Insertion and Deletion Costs    插入和删除的开销        
    Sequential  Processing 顺序处理     
3 B-TREE VARIANTS   B树变体         
    B*-Trees    B*树             
    B+-Trees    B+树    
    Prefix B+-Trees     前缀B+树    
    Virtual B-Trees     虚拟B树         
    Compression    压缩      
    Variable Length Entries    可变宽度入口      
    Binary B-Trees      二叉B树      
    2-3 Trees and Theoretical Results    2-3树和理论结果    
4 B-TREES IN A MULTIUSER ENVIRONMENT     B树在多用户环境       
    Security    安全            
5 A GENERAL PURPOSE ACCESS METHOD USING    通用访问方法的使用          
    B+-TREES     B+树    
    Performance Enhancements      优化性能        
    Tree-Structured File Directory      树结构文件目录             
    Other VSAM Facilities     
SUMMARY     摘要          
ACKNOWLEDGMENTS     致谢           
REFERENCES     参考引用      


# Introduction
The secondary storage facilities available on large computer systems allow users to store, 
update, and recall data from large collections of information called files. A computer must 
retrieve an item and place it in main memory before it can be processed. In order to make 
good use of the computer resources, one must organize files intelligently, making the retrieval 
process efficient.

在大型计算机系统上可用的辅助存储设备允许用户存储、更新、记忆来自被称作文件的大型集合信息的数据。
计算机必须检索一个项并且放置在主存中，然后它才能被处理。为了更好地使用计算机资源，必须聪明地组织文件，
让它能够被高效地检索。

The choice of a good file organization depends on the kinds of retrieval to be performed. 
There are two broad classes of retrieval commands which can be illustrated by the following examples:
Sequential: "From our employee file, prepare a list of all employees names and addresses," and      
Random: "From our employee file, extract the information about employee J. Smith".      

一个好的文件架构依赖着一种执行的检索。这里有两类广泛的检索命令，能够通过下面的示例进行说明。
按次序检索：从我们的职员文件，预处理一个所有职员名字和地址的列表。      
随机检索：从我们的职员文件，提取关于职员J. Smith的信息。        

We can imagine a filing cabinet with three drawers of folders, one folder for each employee. 
The drawers might be labeled "A-G," "H-R," and "S-Z," while the folders might be labeled with 
the employees' last names. A sequential request requires the searcher to examine the entire file, 
one folder at a time. On the other hand, a random request implies that the searcher, guided by 
the labels on the drawers and folders, need only extract one folder.

我们能想象成有三个文件抽屉的文件储藏柜，一个文件夹存储一个职员。这些戳提的标签可能是“A-G”、“H-R”和“S-Z”，
在此同时文件夹可能有着职员名字的标签。文件夹在一段时间内，一个顺序请求要求搜索者仔细检查全部的文件。另一种方式，
一个随机请求暗示搜索者，通过标签的指南在抽屉和文件夹中，仅仅提取所需要的一个文件夹。    

Associated with a large, randomly accessed file in a computer system is an index which, like the 
labels on the drawers and folders of the file cabinet, speeds retrieval by directing the searcher 
to the small part of the file containing the desired item. Figure 1 depicts a file and its index. 
An index may be physically integrated with the file, like the labels on employee folders, or phys-
ically separate, like the labels on the drawers. Usually the index itself is a file. If the index 
file is large, another index may be built on top of it to speed retrieval further, and so on. The 
resulting hierarchy is similar to the employee file, where the topmost index consists of labels on 
drawers, and the next level of index consists of labels on folders.

在一个计算机系统里使一个巨大的文件能够随机访问的是索引，像在文件存储柜的抽屉和文件夹上的标签一样，通过直接让搜索者
搜索包含所需部分的一小部分文件来加速检索。图1描绘了一个文件和它的索引。一个索引可能是物理集成在文件里的，像在职员
文件夹的标签一样，或者物理分离的，像在抽屉上的标签。通常索引它本身是一个文件。如果索引文件很大，另一个索引可能构建
在它上面用来加速未来的检索，等等。检索结果是有层次的类似职员文件，由在抽屉上的标签的顶部索引组成，和文件夹上的标签
组成的索引为下一个等级。

![FIGURE 1. A file and its index on a secondary store.](./images/btree/figure01.png)

Natural hierarchies, like the one formed by considering last names as index entries, do not always 
produce the best performance when used in a computer system. Usually, a unique key is assigned to 
each item in the file, and all retrieval is requested by specifying the key. For example, each employee 
might be assigned a unique employee number which would identify that employee's record. Instead of 
labeling the drawers of the cabinet "A-G," etc., one would use ranges of employee numbers like "0001"-"3000".

自然的层次，像通过考虑把名字作为索引的整个，在计算机系统里这样使用不能总是获得最好的性能。通常，一个唯一的键分配给在
文件的一个项，而且所有的检索都要求指定这个键。例如，在标记职员记录的时候，每个职员可能被分配了唯一的职员号码。替代
在存储柜的抽屉上进行标签"A-G"等等，一个可能使用的职员号码范围是 "0001"-"3000"。

Many techniques for organizing a file and its index have been proposed; Knuth [KNuT73] provides a survey 
of the basics.  While no single scheme can be optimum for all applications, the technique of organizing 
a file and its index called the B-tree has become widely used. The B-tree is, de facto, the standard 
organization for indexes in a database system. This paper, intended for computer professionals who have 
heard of B-trees and want some explanation or direction for further reading, compares several variations 
of the B-tree, especially the B+-tree, showing why it has become popular. It surveys the literature on B-trees 
including recent papers not mentioned in textbooks. In addition, it discusses a general purpose file access 
method based on the Btree.

很多组织文件和它的索引的技术被提议，Kunth [KNUT73] 提供了一个基础的概述。虽然没有一个单一的规划能够适应所有的应用，
这个组织文件和索引的技术叫做B树，已经被广泛地使用了。B树是在数据库组织索引的事实标准。这个论文试图为有知道
B树和想更进一步地阐述或者直接为未来的阅读的计算机专业人士，比较各种B树的变体，尤其是B+树，表明它为什么会成为受欢迎的。
这篇论文罗列了B树相关的最近的论文没有提及的著作。

The starting point of our discussion is an internal storage structure called the binary search tree. 
In particular, we begin with balanced binary search trees because of their guaranteed low retrieval cost. 
For a survey of binary search trees and other internal storage mechanisms, the reader is referred to SEVE74 
and NIEV74. NIEV74 also explains the graph theoretic terms "tree," "node," "edge," "root," "path," and "leaf," 
which will be used throughout the discussion.  The remainder of this Introduction presents a model of the 
retrieval process and outlines the file operations to be considered.  Section 1 presents the basic B-tree as 
proposed by Bayer and McCreight, giving the methods for inserting, deleting, and locating items. Then for 
each type of operation, Section 2 examines the cost and concludes that sequential processing can be expensive. 
In many cases, changes in implementation can lower the costs; Section 3 shows variations of the B-tree 
which have been developed to do so. Extending the variations of B-trees, Section 4 reviews the problems 
of maintaining a B-tree in a multiple user environment and outlines solutions for concurrency and security problems. Finally, Section 5 presents IBM's general purpose file access method which is based on the B-tree.

我们讨论的一开始的点是一个叫做二叉搜素树的内存结构。特别地，我们从平衡二叉搜索树开始，疑问它能保证检索的低开销。
为了概述二叉搜索树和其他的内存机制，读者需要查阅 SEVE74和NIEV74 论文。NIEV74 论文详细解释了图论的术语 树、节点、边、
根、路径和叶子，这些术语将会被整个讨论中使用。这个介绍的余下部分将描述检索处理和文件操作大纲的模型。第一部分将描述Bayer
和McCreight提议的基础B树，给了插入、删除和定位项的方法。然后为每个类型的操作；第二部分详细描述这些操作的开销和顺序
处理之所昂贵的推断。在很多情况下，改变实现能够降低开销；第三部分展示了已经开发使用的B树变体。延伸描述B树的变体，第四部分
审核在多用户环境下维护B树的问题和解决安全、并发问题的大纲解决方案。最后，第五部分描述了IBM基于B树结构基础的通用文件访问方法。

# Operations on a File 文件的操作
For purposes of this paper, we think of a file as a set of n records, each of the form ri = (ki, ai), 
in which k*i* is called the key for the *i*th record, and a*i* the associated information. For example, the key 
for a record in an employee file might be a five-digit employee number, while the associated information 
might consist of the employee's name, address, salary, and number of dependents.

由于这篇论文的目的，我们把一个文件看做n条记录的集合，每一个记录用 `ri = (ki, ai)` 表示，其中 ki 是第i条记录的键，ai表示关联的信息。
例如，一条记录的键可能是职员文件中的五位数职员号码，而关联的信息可能是由职员姓名、地址、薪水和职员编号组成的。

We assume that key k*i* uniquely identifies record r*i*. Furthermore, we assume that although the key is much 
shorter than the associated information, the set of all keys is too large to fit into main memory. These
assumptions imply that if records are to be retrieved randomly using the keys, it would be advantageous to 
construct an index to speed retrieval. Since the set of all keys does not fit in main memory, the index
itself must be external. Finally, we assume that the keys have a natural order, say alphabetical, so we can 
refer to the keysequence order of a file.

我们假设键 ki 是记录 ri 的唯一标识符。此外，我们假设尽管这些键比关联的信息更短，但是所有键的集合是非常巨大的，会填满整个内存。
我们假定这些记录使用键来进行随机访问，使用索引的优点是提高检索速度。一开始所有键的集合并不会填满内存，但是这些索引一定会扩展。
最终，我们假设这些键有一个自然的顺序，叫做字母顺序，所以我们能引用关键序列的文件。           

Users conduct transactions against a file, inserting, deleting, retrieving, and updating records. In 
additions, users frequently process the file sequentially, in key-sequence order, starting at a given point. 
Most often, that starting point is the beginning of the file. A set of basic operations which support
such transactions are:
insert: add a new record, (ki, ai), checking that ki, is unique,      
delete: remove record (ki, ai) given ki,     
find: retrieve ai given ki,     
next: retrieve a~i+1 given that ai, was just retrieved (i.e., process the file sequentially).

用户对文件进行组织事务，插入、删除、检索和更新记录。更多的，用户频繁处理文件排序，通过键排序，给一个开始的指针。
常常地，这个起点是文件的开始。一个基础操作的集合支持如下的事务：
插入：添加一个新的记录，(ki, ai)，检测这个k是唯一的。
删除：通过给的键ki移除记录(ki, ai)
查找：通过给的键ki检索值ai
下一个：根据给的文件ai检索下一个a~i+1（顺序地处理文件序列）

For a given file organization, there are costs associated with maintaining the index and with performing 
each of these operations. Since the index is intended to speed retrieval, processing time is usually taken
as the primary cost measure. With current hardware technology, the time required to access secondary storage 
is the main component of the total time required to process the data. Furthermore, most random access
devices transfer a fixed amount of data per read operation, so that the total time required is linearly 
related to the number of reads. Therefore, the number of secondary storage accesses serves as a reasonable 
cost measure for evaluating index methods.  Other less important costs include the time to process data 
once it has been placed in main memory, the secondary storage space utilization, and the ratio of the space 
required by the index to the space required by the associated information.

为了给出文件架构，这些维持索引的关联和执行每一种操作是有开销的。一开始所以是为了提高检索速度的，处理时间是主要的开销衡量标注。
关于当前的硬件技术，这个时间取决于访问辅助存储设备，其请求处理数据的时间是主要的组成部分。而且，很多随机访问设备传输数据是
一个固定数量的读取操作数据量，所以总共的时间取决于读取数据的数量成线性比。因此，二级存储访问的次数可以作为评价指标方法的一个合理的成本度量。其他少数有重要开销的包括处理数据一次性放入主存的操作时间，辅助存储空间的利用，通过分配信息索引到空间来伸缩空间。

# 1. THE BASIC B-TREE 基础的B树
The B-tree has a short but important history. In the late 1960s computer manufacturers and independent 
research groups competitively developed general purpose file systems and so-called "access methods"
for their machines. At Sperry Univac Corporation (in conjunction with Case Western Reserve University) 
H. Chiat, M. Schwartz, and others developed and implemented a system which carried out insert and find
operations in a manner related to the B- tree method which we will describe shortly. Independently, 
B. Cole, S. Radcliffe, M.  Kaufman, and others developed a similar system at Control Data Corporation (in
conjunction with Stanford University). R.  Bayer and E. McCreight, then at Boeing Scientific Research Labs, 
proposed an external index mechanism with relatively low cost for most of the operations defined in
the previous section; they called it a B-tree ~ [BAYE72].

B树有一个简短还是很重要的历史。在1960年代之前计算机制造商和相关的研究组织为了他们的机器竞相开发出叫做“access methods”
的通用文件系统。在Sperry Univac公司（与西储大学一起）的H. Chiat、M. Schwartz 和其他人开发并实现了一个系统，完成了
与B树相关的插入和查找操作，我们将简短地描述它。另外的，B. Cole、S.Radcliffe、M. kaufman 和其他人在Control Data公司
（与斯坦福大学一起）开发了一个类似的系统。那个时候，R. Bayer和E. McCreight在Boeing科学研究实验室，提议一个关于上一部分
定义的常用操作的低开销的扩展索引机制。他们都叫做B树。

PS: 欧美那边真是人才济济啊，各个大学和大公司，都有一流的人才。

This section presents the basic B-tree data structure and maintenance algorithms as a generalization of 
the binary search tree in which more than two paths leave a given node; the next section discusses costs 
for each operation. Other general introductions may be found in HORO76, KNuT73, and WIRT76.
    
这个部分将描述基础的B树数据结构和维持算法，维持一般化的二叉搜索树，其一个节点有比两个更多的路径叶子。下一部分讨论
每个操作的开销。其他关于B树的一般介绍可以在 HORO76、KNuT73 和 WIRT76 中找到。

Recall that in a binary search tree the branch taken at a node depends on the outcome of a comparison of 
the query key and the key stored at the node. If the query is less than the stored key, the left branch
is taken; if it is greater, the right branch is followed. Figure 2 shows part of such a tree used to store 
employee numbers, and the path taken for the query "15."

回想一下，在一个二叉搜索树种，获取分支的一个节点，取决于查询的键与存储的键的比较结果。如果查询的比存储的键更小，
做分支将会获取；如果查询的比存储的键更大，右分支将被选择。图二展示了一个部分，这个树使用职员号码作为存储的值，
而且查询的值是“15”。

![Figure02 Part of a binary search tree for employee numbers The path taken for query "15" is darkened.](./images/btree/figure02.png)

Now consider Figure 3 which shows a modified search tree with two keys stored in each node. Searching 
proceeds by choosing one of three paths at each node. In the figure, the query, 15, is less than 42 
so the leftmost would be taken at the root. For those queries between 42 and 81 the center
path would be selected, while the rightmost path would be followed for queries greater than 81. The 
decision procedure is repeated at each node until an exact match occurs (success) or a leaf is encountered 
(failure).

现在考虑一下展示图3的情况，修改一个每个节点存储两个键的二叉搜索树。搜索处理将在每个节点的三个路径中选择一个。在示图中，
查询的是15，比42更小，所以将会从根节点的左侧子树获取。为了那些在42和81之间的查询，将会选中中间的子树。当查询的值大于81时，
右侧路径将会被选择。这个选择程序将会在每个节点上重复进行，直到提取匹配发生（成功）或者到了叶子节点（失败）。

![FIGURE 3. A search tree with 2 keys and 3 branches per node. The path taken for query "15" is darkened.](./images/btree/figure03.png)

In general, each node in a B-tree of order d contains at most 2d keys and 2d + 1 pointers, as shown in 
Figure 4. Actually, the number of keys may vary from node to node, but each must have at least d keys
and d + 1 pointers. As a result, each node is at least 1/2 full. In the usual implementation a node 
forms one record of the index file, has a fixed length capable of accommodating 2d keys and 2d pointers, 
and contains additional information telling how many keys correctly reside in the node.

一般情况下，在度为d的B树中，每个节点最有2d个键和2d+1个指针，展示在示图4中。事实上，键的数量可能会受到节点到节点的影响，
但是每个节点必须有至少d个键和d+1个指针。所以，每个节点至少是1/2满的。一个通常的实现中，一个节点由文件索引的记录组成，
有假设为2d个键和2d个指针的固定长度的容量，而且包含了有多少个键正确放置在节点中的扩展信息。

![FIGURE 4. A node in a B-tree of order d with 2d keys and 2d + 1 pointers.](./images/btree/figure04.png)

Usually, large, multikey nodes cannot be kept in main memory and require an access to secondary storage 
each time they are to be inspected. Later, we will see how, under our cost criterion, maintaining more than
one key per node lowers the cost of find, insert, and delete operations.

通常，巨大的、很多件的节点无法保存在主存中，需要一个能够访问的辅助存储设备，随时能够进行检查。之后，我们将能看到如何
在我们的开销标准下，维持不止一个键的节点进行查找、插入、删除操作保持很小的开销。

## Balancing 平衡
The beauty of B-trees lies in the methods for inserting and deleting records that always leave the tree 
balanced. As in the case of binary search trees, random insertions of records into a file can leave a tree 
unbalanced. While an unbalanced tree, like the one shown in Figure 5a has some long paths and some short 
ones, a balanced tree, like the one shown in Figure 5b, has all leaves at the same depth. Intuitively, 
B-trees have a shape as shown in Figure 6. The longest path in a B-tree of n keys contains at most
about logdn nodes, d being the order of the B-tree. A find operation may visit n nodes
in an unbalanced tree indexing a file of n records, but it never visits more than 1 + logdn nodes in a 
B-tree of order d for such a file. Because each visit requires a secondary storage access, balancing 
the tree has large potential savings. Many schemes to balance trees have been proposed (see NIEV74, 
FOST65, KARL76 for examples).  Each scheme requires some computation time to perform the balancing, 
so the savings during retrieval operations must be greater than the cost of balancing itself.  The B-tree 
balancing scheme restricts changes in the tree to a single path from a leaf to the root, so it cannot 
introduce "runaway" overhead. Furthermore, the balancing mechanism uses extra storage to lower the 
balancing costs (presumably, secondary storage is inexpensive compared to retrieval time). Hence, B-trees 
gain the advantages of balanced tree schemes while avoiding some of the time-consuming maintenance.

完美的B树存在插入和删除记录的方式时失去平衡。作为二叉搜索树的一种，随机插入记录到一个文件中能使树变得不平衡。
当一个不平衡的树，像图5b展示的那样，所有的叶都有同样的深度。很自然的，B树有一个形状像示图6展示的那样。在B树中
最长的路径有着n个键，包含最多logdn个节点，b表示B树的度。在有n条记录的文件中，索引是一个不平衡的树，一个查找操作
可能经过n个节点。在度为d的B树结构的索引文件中，其搜索经过的次数不会比 logdn+1 多。很多平衡树的方案被提出了，像
NIEV74、FOST65、KARL76等。 每个方案都被要求一些计算时间来执行平衡，例如保存在检索操作必须大于平衡操作的开销。
B树平衡方案限制改变B树成为一条从叶子到根的单一路径，所以它不能失控地直接到顶。此外，这个平衡机制使用扩展的存储来
降低平衡的开销（大概，比起检索时间来说辅助存储设备是廉价的）。所以，B树获得平衡树方案的优势，通过避免一些耗费时间的机制。   

![FIGURE 5. (a) An unbalanced tree with many long paths, and (b) a balanced tree with all paths to leaves exactly the same length.](./images/btree/figure05.png)

![FIGURE 6. The shape of a B-tree of order d indexing a file of n records.](./images/btree/figure06.png)



## Insertion
To see how balance is maintained during insertion, consider Figure 7a which shows a B-tree of order 2. 
Since each node in a B-tree of order d contains between d and 2d keys, each node in the example has 
between 2 and 4 keys. Some indicator which is not depicted must be present in each node to mark the 
current number of keys. Insertion of a new key requires a two-step process.  First, a find proceeds 
from the root to locate the proper leaf for insertion. Then the insertion is performed, and balance 
is restored by a procedure which moves from the leaf back toward the root. Referring to Figure 7a, 
one can see that when inserting the key "57" the find terminates unsuccessfully at the fourth leaf. 
Since the leaf can accommodate another key, the new key is simply inserted, yielding the tree shown in
Figure 7b. If the key "72" were inserted, however, complications would arise because the appropriate 
leaf is already full.  Whenever a key needs to be inserted in a node that is already full, a split occurs: 
the node is divided as shown in Figure 8. Of the 2d + 1 keys, the smallest d are placed in one node, 
the largest d are placed in another node, and the remaining value is promoted to the parent node where 
it serves as a separator. Usually the parent node will accommodate an additional key and the insertion 
process terminates. If the parent node happens to be full too, then the same splitting process is applied 
again. In the worst case, splitting propagates all the way to the root and the tree increases in height
by one level. In fact, a B-tree only increases in height because of a split at the root.

来看看怎么在插入的时候维护平衡，参见示图7a中度为2的B树。在度为d的B树中，每个节点包含了d到2d之间的键，每个节点在示例中
有2到4个键。一些指示器没有描述但是必须表示每个节点标记的当前键的数量。插入一个新的键，要求两个处理步骤。首先，找到一个
从根节点前进定位合适的叶子节点来插入。然后执行插入，通过移动叶子向根节点的步骤来恢复平衡。引用示图7a，一个能看到的示例，
插入键57时，在第4叶子上找到不成功的结束。一开始叶子节点能够容纳另一个键，新的键是可以简单地插入，很符合的树展示在示图7b。
如果键72插入了，然而，产生了一个复杂的问题，因为这个合适的叶子已经满了。无论如何，当一个键需要插入一个节点，而它已经满
的时候，一个分割发生了；这个节点被分割，如示图8所示。2d+1个键，小于d的放在一个节点中，大于d的放在另一个节点中，作为分割符
的剩余的值提升到父节点中。通常这个父节点将提供扩展的键容纳位置，然后插入过程结束。如果父节点也发生满了的情况，那时再进行一次
一样的分割处理操作。在最糟糕的情况中，分割操作蔓延所有的路径直到根节点，并且树的高度会递增一层。实际上，因为分割B树的
根节点，所以会增加高度。


![FIGURE 7. (a) A B-tree of order 2, and (b) the same tree after insertion of key "57". Note that the number of keys m the root node may be less than d, the order of the B-tree All other nodes have at least d keys in them.](./images/btree/figure07.png)

![FIGURE 8. (a) a leaf and its ancestor in a B-tree, and (b) the same subtree after insertion of key "72".  Each node retains between 2 and 4 keys (d and 2d ).](./images/btree/figure08.png)

## Deletion
Deletion in a B-tree also requires a find operation to locate the proper node. There are then two 
possibilities: the key to be deleted resides in a leaf, or the key resides in a nonleaf node. A nonleaf 
deletion requires that an adjacent key be found and swapped into the vacated position so that it finds 
work correctly. To locate an adjacent key in key-sequence order, one merely searches for the leftmost 
leaf in the right subtree of the now empty slot. As in a binary search tree, the needed value always
resides in a leaf. Figure 9 demonstrates these relationships.  

在B树中，删除要求先进行查找操作定位到合适的节点。这里有两种可能性：删除的键位于叶子节点上，或者键位于非叶子节点上。
非叶子节点的删除要求一个邻接的键找到并且进入空出的位置，以便达到正确地工作。定位有序的键序列上的邻接键，在不是空槽的右侧
子树上仅搜索一次最左侧的叶子节点。作为一个二叉搜索树，这个需要的值总是在叶子节点上。

![FIGUR~ 9. Deletion of key "17" requires that the next sequential key, "21" be found and swapped into the vacant position. The next sequential key always resides in the leftmost leaf of the subtree given by the right pointer of the empty position.](./images/btree/figure09.png)

Once the empty slot has been "moved" to a leaf, we must check to see that at least d keys remain. If 
less than d keys occupy the leaf, then an underflow is said to occur, and redistribution of the keys 
becomes necessary. To restore balance (and the B-tree property that each node has at least d keys) only 
one key is needed--it could be obtained by borrowing from a neighboring leaf. But since the operation 
requires at least two accesses to secondary storage, a better redistribution would evenly divide the 
remaining keys between the two neighboring nodes, lowering the cost of successive deletions from the 
same node. Redistribution is illustrated by Figure 10.  

空槽一次性地移动到叶子节点上，我们必须检查看看最少的d个键是否能够保持。如果这个叶子发生了少于d个键的情况，这是
下溢的情况发生，并且重新分配键成为了必须要做的。要达到恢复平衡（B树的属性：每个节点至少有d个键）仅仅需要一个键时，
它能获得从相邻的叶子借来的键。但是开始这个操作要求至少两次访问辅助存储设备，一个更好的重新分配方法是在两个相邻的节点
均匀地分割剩余的键，从同一个节点上进行连续删除操作的开销是很低的。重新分配的插图参见示图10。

![FIGURE 10. (a) Part of a B-tree before, and (b) after redistribution of keys among two neighbors. Note the final position of the separator key, "50". Redistribution into equal size nodes helps avoid underflow on successive deletions.](./images/btree/figure10.png)

Of course, the distribution of keys among two neighbors will suffice only if there are at 
least 2d keys to distribute. When less than 2d values remain, a concatenation must occur. During a concatenation, 
the keys are simply combined into one of the nodes, and the other is discarded (note that concatenation is 
the inverse of splitting).  Since only one node remains, the key separating the two nodes in the ancestor 
is no longer necessary; it too is added to the single remaining leaf. Figure 11 shows an example of 
concatenation and the final location of the separator key. 

当然，只有在至少有2d键可供分配的情况下，在两个邻居之间分配键才足够。 当剩余的少于2d个值时，会发生串联。在这串联的情况下，
这些键简单地混合在一个节点中，并且其他的将会被丢弃（注意那个关联的将会被分割逆转）。因为只剩下一个节点，所以不再需要祖先
节点中分割为两个节点的键；它会被添加到剩余的单个叶子节点中。示图11展示了这个例子，串联和最终定位到的作为分割的键。

![FIGURE 11. (a) A deletion causing concatenation, and (b) the rebalanced tree.](./images/btree/figure11.png)

When some node loses a separator key due to concatenation of two of its children, it too may underflow 
and require redistribution from one of its neighbors.  The process of concatenating may force concatenating 
at the next higher level, and so on, to the root level.  Finally, if the descendants of the root are 
concatenated, they form a new root, decreasing the height of the B-tree by 1.

当一些节点串联它的两个子节点时会丢失分割的键，它也可能下溢并且要求再分配到它的邻居节点。处理串联可能会在更高的层级强制串联，
并且直到根节点。最终，如果根节点的后代也串联了，他们会成为新的根节点，B数的高度减去1。

Algorithms for insertion and deletion may be found in BAYE72. Simple examples programmed in PASCAL are provided by
Wirth [WIRT76].

插入和删除的算法能够在论文 BAYE72 中找到。简单的PASCAL编程示例在论文 WIRT76中提供了。   

# 2. THE COST OF OPERATIONS
Since visiting a node in a B-tree requires an access to secondary storage, the number of nodes visited 
during an operation provides a measure of its cost. Bayer and McCreight [BAYE72] give a precise analysis 
of the costs of insertion, deletion, and retrieval. They also provide comprehensive experimental results 
which relate the theoretical bounds to actual devices. Knuth [KNUT73] also derives bounds for the cost 
of operations in a B-tree using a slightly different definition. The next section gives a simple explanation 
of the asymptotic bound on costs.

因为查看B树中的一个节点要求访问辅助存储设备，访问一定数量节点的操作提供了开销的衡量标准。Bayer 和 McCreight 给了一个
关于插入、删除和检索的开销的清晰分析。他们也提供了在于理论相关的真实设备上的广泛的实验结果。Knuth 也得到了关于定义
稍微不同的B树的操作开销。下一个章节将提供渐进关联开销的简单描述。   

## Retrieval Costs

First, consider the cost of a find operation.  Except for the root, each node in the B-tree has at least 
d direct descendants since there are between d and 2d keys per node; the root has at least 2 descendants. 
So the number of nodes at depths^2 0, 1, 2, ..., must be at least 2, 2d, 2d^2, 2d^3 .... All leaves lie at 
the same depth h so there are [formula01] nodes with at least d keys each. The height of a tree with n total 
keys is therefore constrained so that `2d(d^h - 1)/(d - 1) <= n` with a little work one can show that
`2d^h <= n + 1`, or h <= logd((n+1)/2). Thus, the cost of processing a find operation grows as the logarithm 
of the file size.

首先，考虑一下查找操作的开销。除了根节点，每个节点在B树中至少有d个直接后代，因为每个节点都有d到2d个键；根节点至少有
两个直接后代。所以节点的数目是深度的2次方，0、1、2等，必须有至少 2、2d、2d^2、2d^3。所有的叶子节点位于同样的高度h，
这个高度的树总共有n个键，因此限制。。。。。太难了额。。。。  因此，处理查找操作的开销是随着文件大小对数增长的。

![formula01](./images/btree/formula01.png)

Table I shows how reasonable logarithmic cost can be, even for large fries. A B-tree of order 50 which 
indexes a file of one million records can be searched with only 4 disk accesses in the worst case. Later we
will see that this estimate is too high; simple implementation techniques lower the worst case cost to 3, 
and the average cost to less.

表I展示了合理的对数开销，甚至更大。度为50的B树，索引文件将有一百万条记录，在最糟糕的情况下只需要4次访问。随后我们来
看看这个树估计有多高；简单地实现技术在最糟的情况下是3，而平均开销会更小。

[AHO74] provide another perspective on the cost of finds in a B-tree. They show that for the decision-tree 
model of computation, one where searching is based on comparison at each node, no asymptotically faster 
retrieval algorithm can be devised. Of course, this model does rule out some methods, such as hashing
[MAUE75]. Nevertheless, B-trees exhibit low retrieval costs in both a practical and theoretical sense.

AH074 提供了又一个关于B树查找开销的观点。他们展示了决定树计算模型，一次基于比较每个节点的搜索，没有接近更快的检索
算法被发明。当然，这个模型用一些方法定义了规则，例如散列法。尽管如此，在实践和理论中，B树都表现出很低的检索开销。

## Insertion and Deletion Costs 插入和删除的开销

An insert or delete operation may require additional secondary storage accesses beyond the cost of a 
find operation as it progresses back up the tree. Overall, the costs are at most doubled, so the height 
of the tree still dominates the expressions for these costs. Therefore, in a B-tree of order d for a 
file of n records, insertion and deletion take time proportional to logdn in the worst case.

当返回树时，插入或删除操作可能要求额外的辅助存储设备访问，会超过查找操作的开销，总体的，开销最多是双倍的，
所以树的高度仍然制约着开销的表达式。因此，在度为d的B树中一个文件有n条记录，在最糟糕的情况下插入和删除消耗时间比例
达到logdn。

The advantage of nodes containing a large number of keys should now be clear.  As the branch factor, d, 
increases, the logarithmic base increases, and the costs of find, insert, and delete operations decrease.
There are, however, practical limits on the size of a node: most hardware systems bound the amount of 
data that can be transferred with one access to secondary storage.  Besides, our cost hides the constant 
factor which grows as the size of data transferred increases. Finally, each device has some fixed track 
size which must be accommodated to avoid wasting large amounts of space. So, in practice, optimum 
node size depends critically on the characteristics of the system and the devices on which the
file is allocated.

节点包含大量键的优势是变得很简洁。作为分叉的因子，d增加的时候，对数增加，并且查找、插入和删除操作的开销减少。
然而，实际上节点的大小是有限制的：很多硬件系统限制了数量的数量，能够转换到辅助存储设备上访问。与之相比，当数据传输大小
增长时，我们的开销隐藏了常量因子。 最终，每个设备有一些固定轨迹大小，必须容纳避免浪费大量的空间。所以，在实际中，
最适合的节点大小取决于系统和设备中文件严苟分配的大小。

Bayer and McCreight [BAYE72] give some loose guidelines for choosing node sizes based on rotational delay 
time, transfer rate, and key size. Their experiments verify that the model's optimal values perform well 
in practice.

Bayer 和 McCreight 给了一些零散的指南，基于转动延迟时间、传输速率和键的大小来选择节点大小。他们的实验验证了这个模型的最佳值，
在实际中是起作用的。

## Sequential Processing 序列处理

So far we have considered random transactions conducted by specifying a key. Often, users wish to view the 
file as a sequential one, using the next operation to process all records in key-sequence order. In fact,
one alternative to B-trees, the so called Indexed Sequential Access Method (ISAM) [GHOS69], assumes that 
sequential accesses occur very frequently.

到目前为止，我们考虑通过指定一个键来进行随机事务的引导。用户希望把文件看做一个序列，使用下一个操作，在有序的键序列中处理
所有的记录。实际上，一个替代B树的，叫做索引序列访问方法（ISAM），假设序列访问是非常频繁的。

Unfortunately, a B-tree may not do well in a sequential processing environment.  While a simple preorder 
tree walk [KNUT68] extracts all the keys in order, it requires space for at least h = logd(n + 1)
nodes in main memory since it stacks the nodes along a path from the root to avoid reading them twice. 
Additionally, processing a next operation may require tracing a path through several nodes before reaching
the desired key. For example, the smallest key is located in the leftmost leaf; finding it requires 
accessing all nodes along a path from the root to that leaf as shown in Figure 12.

不幸地，在序列处理环境中B可能做的并不好。在一个简单的前序树迭代提取所有有序的键期间，要求的最少 h = logd(n + 1)
在主存中的节点空间，因为它们堆积的节点沿着从根节点开始的路径，避免被读两次。此外，处理下一个操作可能要求回溯一个路径，
在抵达需要的键之前通过各个节点。例如，最小的键是定位在最左侧的叶子上；查找它要求访问所有的节点，沿着一条路径从根节点
到示图12展示的叶子节点。

![FIGURE 12. The locahon of the smallest key m the leftmost leaf of a B-tree. Reaching it requires logdn accesses.](./images/btree/figure12.png)

What can be done to improve the cost of the next operation? This question and others will be answered in 
the next section, under the topic "B+-trees."

我们改进下一个操作的开销吗？这个问题和其他的都将会在下一个章节中被回答，在B+树的主题下。

# 3. B-TREES VARIANTS B树变体
As with most file organizations, variations of B-trees abound. Bayer and McCreight [BAYE72] suggest several 
implementation alternatives in their original paper. For example, the underflow condition, resulting from 
a deletion, is handled without concatenation by redistributing keys from neighboring nodes (unless the 
requisite number of keys cannot be obtained). Applying the same strategy to the overflow condition can 
delay splitting and eliminate the associated overhead. Thus, instead of splitting a node as soon as it fills 
up, keys could merely be distributed into a neighboring node, splitting only when two neighbors fill.

作为最常见的文件组织形式，产生了各种B树变种。Bayer 和 McCreight 建议了几个替代源论文的是心啊。例如，下溢条件，
由删除而产生的，是操作除了通过重新分配来自邻居节点的键的串（除非无法获取键的数目了）。应用一些策略到下溢条件上能够延迟
分割和清除联系的顶部。因此，替代分割的节点很快就填满它，键仅仅分配给邻居节点，分割时仅仅当个邻居被填充。
=_= 这一部分狗屁不通了，晕倒啊简直了          

Other variations of B-trees have concentrated on improvements in the secondary costs. Clampet [CLAM64] 
considers the cost of processing a node once it has been retrieved from secondary storage. He suggests 
using a binary search instead of a linear lookup to locate the proper descendent pointer. Knuth [KNUT73] 
points out that a binary search might be useful if the node is large, while a sequential search
might be best for small nodes. There is no reason to limit internal searching to sequential or binary search; 
any number of techniques from KNUT73 might be used.  In particular, Maruyama and Smith [MARc77] mention an
extrapolation technique they call the square root search.

其他的B树变种专注提高次要的开销。Clampet 考虑处理一次节点的开销，从辅助存储设备检索。它建议使用二分搜索来替换
线性查找定位合适的后裔指针。Knuth 指出如果节点非常大，二分搜索是有用的，而一些顺序搜索可能更适合小少的节点。
那不是限制内部搜索使用顺序或者二分搜索的理由；来自KNUT73中的任意数量的技术都可能被使用。特别的，Maruyama 和 Smith
提及了一个推理技术，他们叫做平方根搜索。

In their general treatment of index creation for a file, Ghosh and Senko [GHos69] consider the use of an 
interpolation search to eliminate a secondary storage access.  The analysis presented generalizes to B-
trees and indicates that it might be cost effective to eliminate some of the index levels just above the 
leaves. Since a search would terminate with several possible candidate leaves, the correct one would be
found by an "estimate" based on the key value and the key distribution within the file. When the estimate 
produced the wrong leaf, a sequential search could be carried out. Although some estimates might miss,
the method would pay off on the average.

通常他们处理由文件创造的索引，Ghosh 和 Senko 考虑使用一个插补搜索来排除辅助存储设备的访问。这个分析表示泛化B树
和标示，肯能会影响开销，清除一些仅仅在叶子等级上的索引。开始一个搜索可能在各种可能下合适离开结束，正确的发现可能是
基于键值和在文件内键分配的预测。当预测生成错误的叶子时，一个顺序搜索可能完成了。尽管一些一些预测会错过，但这个方法
在平均过程中能取得成功。

Knuth [KNUT73] suggests a B-tree variation which has varying "order" at each depth. Part of the motivation 
comes from his observation that pointers in leaf nodes waste space and should be eliminated. It also makes 
sense to have a different shape for the root (which is seldom very full compared to the other nodes). 
Maintenance costs for this implementation seem rather high compared to the benefits, especially since secondary 
storage is both inexpensive and well suited to fixed length nodes.

Knuth 建议的一个B树变体，在每个深度都有不同的度。部分意图来自他的观察，在叶子节点中的指针浪费空间并且应该被清除。
它也生成了一个不同形状的根节点（很少但是与其他节点比起来完全不同）。与优势相比，这个实现的维护开销似乎很高，
特别当辅助存储设备不昂贵和很好地固定节点的宽度。

## B*-Trees
Perhaps the most misused term in B-tree literature is B*-tree. Actually, Knuth [KNuT73] defines a B*-tree 
to be a B-tree in which each node is at least 2/3 full (instead of just 1/2 full). B*-tree insertion 
employs a local redistribution scheme to delay splitting until 2 sibling nodes are full.  Then the 2 nodes 
are divided into 3, each 2/3 full. This scheme guarantees that storage utilization is at least 66%, while 
requiring only moderate adjustment of the maintenance algorithms. It should be pointed out that increasing 
storage utilization has the side effect of speeding up the search since the height of the resulting tree is
smaller.  

也许B*-tree是B树这一术语的最大滥用。事实上，Knuth 定义一个B*-tree作为B树，每个姐弟啊至少有2/3满（而不是仅仅1/2满），
B*-tree插入利用了一个本地再分配方案，延迟分割直到有两个兄弟节点是满的。那时两个节点分割为3，每个都是2/3满的。
这个方案保证存储利用率达到至少66%，并且要求适度调整维护算法。需要指出的是提高存储利用率是有副作用的，加速了搜索，
因为生成树的高度更小了。


The term B*-tree has frequently been applied to another, very popular variation of B-trees also suggested 
by Knuth (cf.  [KNuT73, WEDE74, BAYE77]). To avoid confusion, we will use the term B+-tree for Knuth's 
unnamed implementation.

B*-tree术语也被频繁地应用到其他方面，非常受欢迎的B树变体也被Knuth建议。避免混淆，我们将使用术语B+树作为Knuth的
未命名实现。

## B+-Trees
In a B+-tree, all keys reside in the leaves.  The upper levels, which are organized as a B-tree, consist 
only of an index, a roadmap to enable rapid location of the index and key parts. Figure 13 shows the logical 
separation of the index and key parts. Naturally, index nodes and leaf nodes may have different formats or 
even different sizes. In particular, leaf nodes are usually linked together left-to-right, as shown. The 
linked list of leaves is referred to as the sequence set. Sequence set links allow easy sequential processing.

在一个B+数中，所有的键都放置在叶子中。在较高的层级中，是B树的组织形式，仅仅由索引组成的一个路标，开启快速定位的索引和键部分。
图12展示了分隔索引和键部分的逻辑。 自然地，索引节点和叶子节点可能有不同的格式甚至不同的大小。特别的，叶子节点通常是从左到右的，
如图所示。叶子的链表是顺序集合的引用。顺序集合链接允许很容易地进行顺序处理。

![FIGURE 13. A B+-tree with separate index and key parts. Operations "by key" begin at the root as in a B-tree, sequential processing begins at the leftmost leaf.](./images/btree/figure13.png)

To fully appreciate a B+-tree, one must understand the implications of having an independent index and sequence 
set. Consider for a moment the find operation.

要完整地领悟B+树，必须明白索引和序列集有依赖的隐含事实。进行查找操作时考虑一会儿。

Searching proceeds from the root of a B+-tree through the index to a leaf. Since all keys reside in the 
leaves, it does not matter what values are encountered as the search progresses as long as the path leads 
to the correct leaf.

搜索处理从B+树的根节点通过索引到达叶子节点。因为所有的键都放在叶子节点上，搜索过程中在很长的路径引导到合适的叶子，
是没有关系的。

During deletion in a B+-tree, the ability to leave non-key values in the index part as separators simplifies
processing. The key to be deleted must always reside in a leaf so its removal is simple. As long as the leaf
remains at least half full, the index need not be changed, even if a copy of the key had been propagated up 
into it. Figure 14 shows how the copy of a deleted key can still direct searches to the correct leaf. Of
course, if an underflow condition arises, the redistribution or concatenation procedures may require adjusting 
values in the index as well as in the leaves.

在B+树删除操作的期间，在索引部分离开非叶子节点值的能力作为分割简单的处理。键的删除必须总是在叶子节点上进行，这样移除
是比较简单的。只要叶子节点剩余的能满足一般满，索引不需要改变，甚至如果复制键也扩散到节点里面。示图14展示了怎么拷贝一个
被删除的键，仍然能正确地搜索到叶子节点。当然，如果下溢的情况出现了，重新分配或串联操作要求调整除叶子节点外的索引的值。

Insertion and find operations in a B+-tree are processed almost identically to insertion and find operations 
in a B-tree. When a leaf splits in two, instead of promoting the middle key, the algorithm promotes a copy of
the key, retaining the actual key in the right leaf. Find operations differ from those in a B-tree in that 
searching does not stop if a key in the index equals the query value. Instead, the nearest right pointer is 
followed, and the search proceeds all the way to a leaf.

在B+树中，插入和查找操作跟处理B树的插入和查找操作是一样的。当叶子节点分割为两个时，替代提升中间的键，这个算法提升复制的键，
保留真实的键在右叶子善。查找操作跟B树有所不同，如果一个键的索引等于查找的值，搜索不会停止。反而，最近的右指针将被继承，
而且所有处理所有的路径到叶子节点。  

We have seen that B-trees, which support low-cost find, insert, and delete operations, may require logdn 
accesses to secondary storage to process a next operation. The B+-tree implementation retains the loga-
rithmic cost properties for operations by key, but gains the advantage of requiring at most 1 access to 
satisfy a next operation.  Moreover, during the sequential processing of a file, no node will be accessed 
more than once, so space for only 1 node need be available in main memory. Thus, B+-trees are well suited 
to applications which entail both random and sequential processing.

我们有这样的B树，支持低开销的查找、插入和删除操作，可能要求 logdn 的访问辅助存储设备来处理下一个操作。B+树实现了
保留通过键操作的对数开销属性，但是添加的好处是要求每一次访问满足下一次操作。更多地，在顺序处理一个文件的期间，没有
节点会被访问一次以上，所以在主存中只需要满足一个节点的空间。因此，B+树是很好的集合，应用随机访问和顺序处理两种操作的。      