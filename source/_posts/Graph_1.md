---
title: Image processing using Graph_1
date: 2017-12-21 09:34:37
tags: ['image processing', '图像处理']
desc: 图像处理第一节笔记
---

### Lecture1

1. Graph-based methods in Image Processing for:

  * Segmentation 图像分割
  * Filtering 过滤
  * Classification and clustering 聚类/分类
  
<!-- more -->

2.  We will sometimes regard a **picture** as being a real-valued, non-negative
  function of two real variables; the value of this function at a point will be
  called the gray-level of the picture at the point.

  —— Rosenfeld

3. Storing the image in a computer requires digitization,

    图片存储：

    * Sampling(recoding image values at a finite set of samples points)
    * Quantization(discretizing the continuous functions values)

    > Typically, sampling points are located on a Cartesian grid.

4.  Basic model 

  * Generalized image modalities( multispectral images)
  * Generalized image domains(video, volume images MRI)
  * Generalized sampling point distributions( non-Cartesian girds)
  
  形态、样式、采用方法

5. Benefit for image processing

  * Discrete and mathematically simple representation that lends itself well to the development of efficient and provably correct methods.
  * A minimalistic image representation – flexibility in representing different types of images.
  * re-use existing algorithms and theorems for image analysis!

6.  Image as Graphs

  Graph based image processing methods typically **operate on pixel adjacency graphs**

  * graphs whose vertex set is the set of image elements,

  * whose edge set is given by an adjacency relation on the image elements
      ![TIM截图20171218155809.png](https://i.loli.net/2017/12/18/5a37753f92a08.png)
      ![TIM½ØÍ¼20171218155822.png](https://i.loli.net/2017/12/18/5a37753f93556.png)

7.  Graph segmentation

    * To segment an image represented as a graph, we want to partition the graph into a number of separate connected components.
    * The partitioning can be described either as a vertex labeling or as a
      graph cut.

8.  Graph partitioning

    * vertex labeling
	    Vertex labeling associates each node of the graph with an element in some set of labels. Each element in this set represents an object category.
    * graph cuts
	    A cut is a set of edges that, if they are removed from a graph, separates the graph into two or more connected components.  

  

### References

1. Space-Variant Machine Vision — A Graph Theoretic Approach.
2. A graph-based framework for sub-pixel image segmentation.

  

  