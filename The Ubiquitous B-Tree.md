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
    B*-Trees        
    B*-Trees        
    Prefix B*-Trees     
    Virtual B-Trees     
    Compression     
    Variable Length Entries     
    Binary B-Trees      
    2-3 Trees and Theoreucal Results        
4 B-TREES 1N A MULTIUSER ENVIRONMENT        
    Security        
5 A GENERAL PURPOSE ACCESS METHOD USING     
    B*-TREES        
    Performance Enhancements        
    Tree-Structured Fde Directory       
    Other VSAM Facdltms     
SUMMARY     
ACKNOWLEDGMENTS     
REFERENCES      