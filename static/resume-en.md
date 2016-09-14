---
layout: page
---

# He Yi

1987-03-09 |  (+86)18868105691 | [heyi.pub@gmail.com](mailto:heyi.pub@gmail.com) | [while2.github.io](http://while2.github.io)

## Experience:

- 2015.5 – **now**        Sofeware Engineer at Bigdata Innovation Center of Creditease
- 2012.4 – 2015.3    M.S. in State Key Lab of CAD&CG, Zhejiang University, China
- 2009.6 – 2010.1    Sofeware Engineer in Virtuos Games, China
- 2005.9 – 2009.6    B.S. in Software Engineering, Tongji University, China

## Skills:

- C++, Java, Hadoop, Spark, Python, Javascript.
- Computer Vision and Image Processing

## Projects:

- Bigdata anti-fraud system

  We integrate data from creditease product lines and crawlers, provide search, visualisation, auto-decision services. My contributions:

  1. Integrate data from different systems. I designed [json-cook](https://github.com/while2/json-cook), a JSON transformation tool, so we can drive Spark programs with JSON configurations. With the plugin mechanism, native Java code can be "embeded" into JSON files to implement complicated cleaning logic.
  2. Java Data service. Developed a MySQL-based blacklist API, maintains our Knowledge-Graph based search API.
  3. Based on Data Services, I developed the graph search web page, to visualise the user relationship networks. The graph-related logic was abstracted as [d3graph](https://github.com/while2/d3graph).

- Multi-viewpoint Panorama

  Research for a surveillance system. Our method relaxed the limit for traditional panoramas, achieved state of the art performance on wide-baseline data sets. Accepted by [IEEE Transactions on Image Processing](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=7420659). I designed, implemented the algorithm, developed related tools(feature editing and seamless composition, etc.), drafted the paper and helped my partner on existed methods for comparison.

- Image/Video Completion

  A video completion algorithm based on my improvement for [image completion](http://while2.github.io/exemplar-based-completion), can fill large missing areas in a video clip. I designed a hierarchical approach to significantly improve the performance of Space-Time-Fusion. Implemented in C++, used in a stereo conversion system and a 3d photograph system for object removal.
