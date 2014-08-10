---
layout: post
tags: code
title: Abstraction of 2d Filter
abstract: This post explained the details of function filter in <a href='https://github.com/while2/his/blob/master/ImageProcessing/Filter.hpp'>his lib</a>, which provides a convenient interface to define linear, non-linear, and even non-numerical filters.
---
With the convenience of [abstraction of iterations]({% post_url 2014-07-14-abstraction-of-image-iterations %}), I felt necessary abstract filter with similar technics.

Recently in my program I need to apply different non-linear filters to my images, which leads to terrible iterations like:

~~~ cpp
for (int y = 0; y < image.rows(); ++y)
{
	for (int x = 0; x < image.cols(); ++x)
	{
		int top = MAX(0, y - size);
		int bottom = MIN(y + size + 1, image.rows());
		int left = MAX(0, x - size);
		int right = MIN(x + size + 1, image.cols());

		for (int iy = top; iy < bottom; ++iy)
		{
			for (int ix = left; ix < right; ++ix)
			{
				// ...
			}
		}
	}
}
~~~

I'd like to put them in a function so I can focus on the filter process without typing these regular code again and again.

Filter is a kind of image transformation where each output pixel depends on a local region of pixels(typically a weighted sum) from the input image. So usually we need two steps, __accumulation__ and __evaluation__.

__Accmulation__ computes over all the neighboring pixels and generates some intermediate results. __Evaluation__ transorm these intermediate results to the output pixels.

So a straightforward Filter function comes like this:

~~~ cpp
template<class Mat1, class Mat2, class Mat3, class AccmFunc, class EvalFunc>
void filter(Mat1 &src, Mat2 &dst, Mat3 &kernel, AccmFunc accm, EvalFunc eval)
{
	CHECK_TYPE(Mat1);
	CHECK_TYPE(Mat2);
	CHECK_TYPE(Mat3);

	assert(src.rows() == dst.rows() && src.cols() == dst.cols());
	for (int y = 0; y < src.rows(); ++y)
	{
		for (int x = 0; x < src.cols(); ++x)
		{
			int x0 = max(x - kernel.cols() / 2, 0);
			int x1 = min(x + kernel.cols() / 2 + 1, src.cols());
			int y0 = max(y - kernel.rows() / 2, 0);
			int y1 = min(y + kernel.rows() / 2 + 1, src.rows());
			for (int dy = y0; dy < y1; ++dy)
				for (int dx = x0; dx < x1; ++dx)
					accm(src(dy, dx), kernel(dy - y + kernel.rows()/2, dx - x + kernel.cols()/2));
			
			eval(dst(y, x));
		}
	}
}
~~~

With which a gaussian blur appears like:

~~~ c++
float sum_rgb[3] = {0, 0, 0}, sum_w = 0;
his::filter(input_image, output_image, kernel,
	[&](const Pixel &pixel, float w)
{
	for (int c = 0; c < 3; ++c)
		sum_rgb[c] += pixel.rgb[c] * w;
	sum_w += w;
},
	[&](Pixel &dst)
{
	for (int c = 0; c < 3; ++c)
	{
		dst.rgb[c] = sum_rgb[c] / sum_w;
		sum_rgb[c] = 0;
	}
	sum_w = 0;
});
~~~

where `input_image` and `output_image` are two images, `kernel` is a gaussian kernel of the type `his::Matrix<float>`. Following are two lambda expressions define the process of __Accumulation__ and __Evaluation__ respectively. __Accumulation__ sums the color values and the weights, while __Evaluation__ computes the output pixel color based on the weighted sum. A full sample can be viewed at [Github](https://github.com/while2/his/blob/master/sample.cpp).
 
The trick is to use lambda expression to capture intermediate variables(the sum of rgb color and the sum of weights in above example). In such a way __Accumulation__ and __Evaluation__ can share informations to finish the work together.

A [more sophisticated implementation](https://github.com/while2/his/blob/master/ImageProcessing/Filter.hpp) removes unnecessary boundary checks at the central part of the image.

With two lambda expressions, a filter, linear or non-linear, or even non-numerical ones, can be esaily defined. TPAMI 2014 paper 'Stereo Matching Using Tree Filtering' introduced a complicated 'Tree Filter' which builds a minimum spanning tree with local input pixels and use the distances as weights. This can be implemented by `filter` function too. 