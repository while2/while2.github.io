---
layout: post
tags: c++ vision images
categories: code
title: Abstraction of Image Iterations
abstract: This post describes the idea of my image iteration methods. Which relief a lot of work when doing image processing. The code was hosted <a href="https://github.com/while2/his">here</a>.
---
While coding for image processing, I often need to write 2d iterations like this:

{% highlight cpp %}
for (int y = 0; y < image1.height(); ++y)
{
	for (int x = 0; x < image1.width(); ++x)
	{
		auto &element1 = image1(y, x);
		auto &element2 = image2(y, x);
		auto &element3 = image3(y, x);
		// ...
	}
}
{% endhighlight %}

This bunch of code is so ubiquitous and I have little tolerance to type such trival things again again and again.

Whatsmore, suppose you have already defined variables named `x` or `y` outside the iteration, then your must think of new names for iterative varibles, which makes the code confusing and easily leads to bugs. 

After all, the iteration is about images, why would I care about the iterative variables? What really matters is to process those pixels for each of the images. But look at the previous code, how many lines have I wrote, without even touch the real stuff?   

Iteration is an important concept that worth abstraction. STL provides iterators for their containers, they brought flexibility, but not convenient enough for me. In most of the cases, I want to iterate a whole container, while I must provide two variables(```begin()``` and ```end()```) 
to define the range. 

On the other hand, a 2D array iterator is not as intuitive, especially when you intend to support ROI(Region of Interest), or even more complicated interative ways.

After some mindstorm I have this series of iteration functions for my template class [__MatrixWrapper__](https://github.com/while2/his/blob/master/ImageProcessing/MatrixWrapper.hpp). 

`for_each` takes a few `MatrixWrapper`s (images with the same size) and a functor (takes one element for each image), then iterates all images, and feed the functor with elements at the same position in different images.

A rgb to gray example:

{% highlight cpp %}
his::MatrixWrapper<uchar[3]> image_wrapper;
his::MatrixWrapper<uchar> gray_wrapper;

his::for_each(image_wrapper, gray_wrapper, [](const char rgb[3], uchar &gray) {
	gray = rgb[0] * 0.299 
		 + rgb[1] * 0.587
		 + rgb[2] * 0.114;
}
{% endhighlight %}


With `for_each` defined as:

{% highlight cpp %}
template<class Mat1, class Mat2, class Func>
void for_each(Mat1 &mat1, Mat2 &mat2, Func func)
{
	CHECK_TYPE(Mat1);
	CHECK_TYPE(Mat2);

	assert(mat1.rows() == mat2.rows() && mat1.cols() == mat2.cols());
	
	for (int y = 0; y < mat1.rows(); ++y)
	{
		auto p1 = mat1[y];
		auto p2 = mat2[y];
		for (int x = 0; x < mat1.cols(); ++x)
		{
			func(*p1, *p2);
			p1 += 1, p2 += 1;
		}
	}
}
{% endhighlight %}

When binded to the template parameters, `Mat1` and `Mat2` are color/gray images with template variable `Pixel` and `uchar` respectively, and Func is the lambda expression which takes a `const Pixel &` and a `uchar &` to convert the RGB color to a gray scale intensity.

So far it's all straight forward. But there are some details need to explain:

###Type safety?
Some of you might prefer:

{% highlight cpp %}
template<typename T1, typename T2, class Func>
void for_each(his::MatrixWrapper<T1> &mat1, his::MatrixWrapper<T2> &mat2, Func func)
{
	// ...
}
{% endhighlight %}

Any type instead of `MatrixWrapper` should not compile! That's right, but in this way you need to provide two versions(const and non-const) for each image. In this two image iteration case, four functions must be defined. With the number of images increasing, this becomes infeasible. I known template is very powerful, so there must be some tricky way, but I don't want to make things too complicated.

With the privious method, `Mat1` can bind to `const his::MatrixWrapper<T1> &` or `his::MatrixWrapper<T1> &`, depending on the what's passed to this function. 

But how about he type safety? As designed, `for_each` supports only `MatrixWrapper`. When some other types was passed to it, there should be an explicit compile error. So I add a contract into `MatrixWrapper`:

{% highlight cpp %}
template<typename T>
class MatrixWrapper<T>
{
public:
	enum { FOR_EACH_ABLE, };
	// ...
};
{% endhighlight %}

And check this 'password' with a Macro:

{% highlight cpp %}
#define CHECK_TYPE(Mat) Mat::FOR_EACH_ABLE
{% endhighlight %}

If some other type rather than my ```MatrixWrapper``` was passed to the `for_each` methods, a compile error will be triggered, as 
`error C2039: 'FOR_EACH_ABLE' : is not a member of OtherType`

>Template specialization does not work for derived classes. So any subclasses of `Matrix` or `MatrixWrapper` can not benefit from `for_each` series. One alternative is to use [boost type trait](http://www.boost.org/doc/libs/1_55_0/libs/type_traits/doc/html/boost_typetraits/reference/is_base_of.html) to check the inheritance relationship. But that brought extra dependency and makes things too complicated. So I'll stick on my original idea.
 
###Pairwise iteration
In my exprience on image processing, it is commonly needed to iterate all neighboring pixels. Especially when use variational methods such as Poissin Image Editing, Graph Cuts, etc. And pairwise iterations are not that straight forward, since you need to check boundary conditions or, in a better solution, iterate them twice, vertically and horizontally. More trival code causes chaos and leads to potential bugs. While an abstraction of pairwise iteration saved my life.

With a similar interface, the only difference is that the functor takes two pixel(neighboring to each other) for each of the respective images.

Following is a laplacian example, where `b1` and `b2` are neighboring pixels from the src gray image, `f1` and `f2` are corresponding pixels from the dst float image. In the lambda expression, `f1` corresponds to `b1`, and a gray scale gradient towards neighbor `b2` was added to `f1`. In the whole process, this accumulates all neighbors for each of the dst pixels. With dst image initialized all zeros, this results a laplacian of the input.

{% highlight cpp %}
his::for_each_pair(gray_wrapper, laplace_wrapper,
	[](uchar b1, uchar b2, float &f1, float &f2) {
	f1 += (b1 - b2);
	f2 += (b2 - b1);
});
{% endhighlight %}

To build a Poisson eqaution, the lambda expression can easily capture your matrices or other equation builders.

###How about the iterative variables?
One advantage of these functions is that they hide the iterative variables and make the code more clear. But sometimes we really need iterative variables. The solution is to provide an extra ```IdxMap``` to the functions, which __*appears*__ like a 2-channels int image, with each element the x-y-position of the pixel:

{% highlight cpp %}
struct Idx { int x, y; };
typedef Matrix<Idx> IdxMap;
{% endhighlight %}

But with a tricky implementation, [IdxMap](https://github.com/while2/his/blob/master/ImageProcessing/IdxMap.hpp) does not need actually memories.

###How many images are supported?
Now I simply overload `for_each` functions to iterate one, two, three or four images correspondingly. However, with C++11 variadic template arguments, all of them can be unitied into a single defination. Unfortunately I work on Visual Studio, which is always slow to follow the standards. So far my compiler does not support variadic templates, I have to do with this.