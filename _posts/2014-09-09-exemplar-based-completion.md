---
layout: post
tags: image-completion inpainting
categories: vision
---

##Background
Old pictures rots, sometimes scratch appears. Inpainting means to fix the deteriorated areas. This was used to be done by artists, but later with computers, automatic methods was invented, to fill the missing parts of the image. 

Firstly, the PDE based algorithms. This class of methods based on pixels, not good at exploiting the higher level informations in an image. Since pixels contains extremely local features, it doesn't work for large holes.

Exemplar-based methods use patches to find similar contents in the same image, and fill the missing regions with these contents. A higher level features can be expressed by patches, so it works fine for larger holes.

<a name="bungee"/>
Following is a result from state of the art exemplar-based method:

|![Input](/images/completion/bungee.jpg)|![Mask](/images/completion/bungee-mask.png)|![Result](/images/completion/bungee-result.png)|![Shift-Map](/images/completion/bungee-label.png)|
| :---: | :--: | :----: | :-------: |
| Input | Mask | Result | Shift-Map |

Although patch is a higher level feature than pixel (almost the same level as SIFT if you count the bits), it's still not enough for complex contents because it doesn't contain any semantics. There are artefacts which are not artifacts until you put it in a view of semantics. In other words, the computer would think he worked perfectly if he knows nothing about semantics. Thus some problems are just impossible to solve in the patch level. That's why I'd say the further study of image completion must be on semantics. But that's the future story, while this post is about the patch based methods.

##The Straightforward method
CVPR'03 paper [Object Removal by Exemplar-Based Inpainting][^1] used a greedy approach. In each iteration, a patch was picked at the boundary of the missing regions, hence this patch contains some known pixels. These pixels can be used to find the most similar patch which contains all known pixels. Then copy this intact patch to the selected one, and replace the missing pixels. 

![Greedy](/images/completion/CVPR03-greedy.png)

In each iteration some of the missing pixels were filled. By defining priorities based on contour and gradient, this method fill holes from boundary to center. Like peeling a onion.

Back to 2003, this was the state of art algorithm. It was simple, straightforward and works much better than PDE methods. But a greedy approach usually provides a suboptimal result. Moreover, I think this paper does not have a big picture. It seems right in every step, but did not give a mathematical explanation on what 's going on behind. In Variational Method's view, what is the objective function to be optimized?

![Bungee](/images/completion/CVPR03-bungee.png)

##The Objective Function
[Shift-Map Image Editing][^2], an ICCV'09 paper provides a reasonable model. The image completion problem was defined as a shift optimization problem. Because exemplar-based method copies existed pixel to another place (in the hole), for each of the missing pixels, instead of asking what color it is, we can ask where should it come from. A Shift is a vector from the missing pixel to an existed one, the source pixel supposed to be copied to the missing place. Now a Shift-Map S can be defined on the missing area, where S(p) is the shift vector at position p. Once we know S, we know where to find the source pixel, copy S(p) to p then the work is done.

To solve S we must define the goodness of S, to distinct good Shift-Maps from bad ones. The objective function is a measurement of this goodness, only with that, we can say that we use the optimal S. 

Translate our requirements to mathematical language, the objective function was defined as:

$$
E(S) = \sum_{p \in H} E_d(p) + \lambda_1 \sum_{a,b \in N} E_s(a,b) + \lambda_2 \sum {p \in B} E_b(p)
$$

where the data term $$E_d$$ measure the goodness of a single pixel, the smoothness term $$E_s$$ measure the goodness of the neighborhood relationship of two pixels, while boundary condition $$E_b$$ penalize the deviation of the boundary pixels. $$\lambda$$s are weights to balance their contributions. $$\lambda_2$$ was set large to force the constraint. As for $$\lambda_1$$, we will later put it into [$$E_s$$](#Es) and use different weights for different pixels for smoothness term. See [Smoothness term weights](#smoothness-term-weights) for details.

###Data term
If we want to use S(p) to replace p, S(p) would better be a known pixel. If the shift at p shifts to another missing pixel, it does not help. So we'd like those shifts that can lead us out of the hole.

Hence the data term:

$$
E_d(p)=
\begin{cases}
0, S(p) \notin H \\
+\infty, S(p) \in H
\end{cases}
$$

where $$H$$ is the hole, a set of all missing pixels. A big value was used instead of $$+\infty$$ in real life.

###Smoothness term
The preferred Shift-Map is the one resulting to an image without seams, which means it's hard to tell the completed part from the original part. The seam can also be explained as the inconsistency of colors, so the objective function (the goodness measurement of S) is about the consistency of colors. 

As optimal Shift-Map is shown in the [result](#bungee). 
Each color indicates a unique shift used to fill the missing pixel. It is perfectly seamless inside each of the colored regions, while the seams between them are unnoticeable because of the consistency of image colors.

Typically consistency is about two adjacent pixels. Imagine that two adjacent pixels has the same shift, that will lead them to a pair of adjacent source pixels. 

<pre>
    +---+---+
    | a'| b'|
    +---+---+
    |   |   |
+---+---+---+
| a | b |
+---+---+
</pre>
As the figure shows, adjacent pixels __a__ and __b__, has the same shift (1, -2). Our image coordinates define top left as the origin.

In this condition, source pixels __a'__ and __b'__ will be used to replace __a__ and __b__. There will be no seam between __a__ and __b__, since __a'__ and __b'__ are neighbors by ground truth. The intact content must be seamless.

Generally things are not so perfect, adjacent pixels may take different shifts. However, even if they do, it does not always introduce a seam. The different shifts can be applied to two adjacent pixels, resulting to 2 pairs of adjacent pixels, as long as these 2 pairs appears similar, the seam can be very weak.
<pre>
            +---+---+
            | a1| b1|
+---+---+---+---+---+
| a2| b2|   |   |
+---+---+---+---+
        | a | b |
        +---+---+
</pre>
adjacent missing pixels __a__ and __b__, has different shifts. Say __a__ want to shift to __a1__ with shift (1, -2), while b want to shift to __b2__ with shift (-2, -1). The result puts __a1__ beside __b2__, which are not neighbors originally. A seam was introduced between. But what if __b2__ has the same color with __b1__? Then it is equivalent to replace __a__ __b__ by __a1__ __b1__! So it's still perfect. As long as __a1__ __b1__ are close to __a2__ __b2__, the seam appears weak.

> Although it seams that either a __a1__ closed to __a2__ or a __b1__ close to __b2__ is enough, in fact pixels are still too local to define consistency. With enough candidates, you can always find such rare pixels to fill in seam and reduce the objective function. But that just ends up to a seam with 1 pixel wider. The paper used both pixels to measure the consistency, while I recommend to use even more.

<a name="Es"/>
So here comes the smoothness term:

$$
E_s(a,b)=\sum_{x \in N(a,b) } \lambda_{(a,b)} Dist(I(x+S(a)) - I(x+S(b)))
$$  

where Dist is a similarity measurement for colors, usually $$L_1$$ or $$L_2$$ norm on RGB space, sometimes gradients are also involved to produce better performance. $$I$$ and $$S$$ are the image and Shift-Map, $$N(a,b)$$ a adjacent area around a,b. $$\lambda_{(a,b)}$$ is a weight depend on the position of pixel a and b, to fix the [position bias](#smoothness-term-weights) of the original function.

Seams, or inconsistency, was introduced by adjacent pixels with different shifts. So take some local pixels, and apply the different shifts to them and see how different they become. The total difference can measure the inconsistency.

This definition differs from the original paper. The original smoothness term comes from SIGGRAPH'03 paper [Graphcut Textures: Image and Video Synthesis Using Graph Cuts][^4] and is a special case of the above one where $$N(a,b)$$ takes no more pixels other than $$a$$ and $$b$$. In my implementation I sum the difference of pixels within radius 1 or 2.

###Boundary condition
The consistency is defined on two adjacent pixels, you can always copy an entire intact part to the missing region and preserve the consistency. Then the central contents are all good except that they don't fit in the environment, because of the seam between filled pixels and the intact ones. Inconsistency must be penalised not only on missing areas, but also on boundaries. The same function can be applied on boundaries only that one of the two adjacent pixels is known now. To simplify the problem we can expand our unknown regions by 1 pixel, and constraint the boundary pixels to be close to their original value.

$$
E_b(p) = Dist(I(p),I^*(p))
$$

###Smoothness term weights
There are still one problem. As the hole expand, boundary pixels grow linearly, while inner pixels grow quadratically. Thus to reduce the objective function, it is much easier to work on the inner pixels. The optimization will take that advantage, and overlook the importance of boundaries. An extreme result is to copy an entire region to the hole, put the all seams on the boundary. Since boundary pixels are minority, this may lead to a low function value, but the result is not good at all. The fact is that this function does not consider the importance and the weights of pixels. 

In TPAMI'07 paper [Space-Time Video Completion][^3], the same problem was addressed with a position based weight map. (Although they use a totally different objective function). The idea is to compensate the minority with larger weights. A distance transformation was a applied on the mask to figure out how far a pixel lies from the boundary. Closer pixels have larger weight for their smoothness terms. 

The author use an exponential function as: $$\lambda = \gamma ^d$$. The reason is that exponential function has a recursive property to maintain the priority on each layer of the hole.

<a name="lambda"/>
Similarly our 
$$
\lambda_{(a,b)}=\gamma ^{ {d_a+d_b} \over 2}
$$
where $$d_a$$/$$d_b$$ are the distances from pixel a/b to the nearest boundary and $$\gamma$$ is experientially set to 1.3.

##Optimization
Discretizing the unknown space, above function can be transformed to a discrete labeling problem, where each label represents a possible shift, and the solution is to assign labels to pixels. With the above objective function, the optimization belong to a well studied type: [Markov Random Field][^5] (MRF) problems, where the objective function comprises data terms(depending on a single node/pixel) and smoothness terms(depending on two adjacent nodes/pixels). More complicated patterns are not considered. Our boundary condition can be combined to the data terms.

Optimization of MRF is typically NP-hard, but it can be efficiently approximated by [Belief Propagation][^6] or Graphcuts. [Boykov-Kolmogorov algorithm][^7] use an alpha-beta-swap approach to solve multi-labeled MRF problem by Graphcuts. They provide Matlab and C++ code.

##Reduce unknown space
There are too many possible shifts, not mention their combinations. [Shift-Map Image Editing][^2] adopted a hierarchical approach but it's still slow. ECCV'12 paper [Statistics of Patch Offsets for Image Completion][^8] use the prior to eliminate those improbable shifts and significantly improved the performance. As the author pointed out, unrelated shifts not only slow down the optimization, but also causes "overfitting".

They use [Patch Match][^9] to compute a NNF(Nearest Neighbor Field), where the Nearest Neighbor of a pixel was defined as the most similar patch to the patch centered at this pixel. Then the offset from each pixel to its nearest neighbor can be counted, and they found that the distribution of a natural image's offsets are very sparse. 80% of the offsets gathered in 7% of all the possibilities. So they use the top 60 offsets as the candidate shifts for the previous Objective Function.

As shown in the optimal [Shift-Map](#bungee), there are only tens of shifts were used to reconstruct the missing contents.

[^1]: [Object Removal by Exemplar-Based Inpainting](http://research.microsoft.com/pubs/67273/criminisi_cvpr2003.pdf)
[^2]: [Shift-Map Image Editing](http://www.vision.huji.ac.il/shiftmap/)
[^3]: [Space-Time Video Completion](http://www.wisdom.weizmann.ac.il/~vision/VideoCompletion.html)
[^4]: [Graphcut Textures: Image and Video Synthesis Using Graph Cuts](http://www.cc.gatech.edu/cpl/projects/graphcuttextures/)
[^5]: [Markov Random Field](http://en.wikipedia.org/wiki/Markov_random_field)
[^6]: [Belief Propagation](http://en.wikipedia.org/wiki/Belief_propagation)
[^7]: [Boykov-Kolmogorov algorithm](http://www.csd.uwo.ca/faculty/yuri/Abstracts/pami04-abs.shtml)
[^8]: [Statistics of Patch Offsets for Image Completion](http://research.microsoft.com/en-us/um/people/kahe/eccv12/index.html)
[^9]: [PatchMatch: A Randomized Correspondence Algorithm for Structural Image Editing](http://gfx.cs.princeton.edu/pubs/Barnes_2009_PAR/index.php)