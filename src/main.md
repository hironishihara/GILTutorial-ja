---
layout: default
---

<!-- Copyright 2014 Hiroaki Nishihara

     Distributed under the Boost Software License, Version 1.0.
     (See accompanying file LICENSE_1_0.txt or copy at
     http://www.boost.org/LICENSE_1_0.txt)
-->

<!-- Copyright 2008 Lubomir Bourdev and Hailin Jin

     Distributed under the Boost Software License, Version 1.0.
     (See accompanying file LICENSE_1_0.txt or copy at
     http://www.boost.org/LICENSE_1_0.txt)
-->

<!--
    Copyright 2005-2007 Adobe Systems Incorporated
    Distributed under the MIT License (see accompanying file LICENSE_1_0_0.txt
    or a copy at http://stlab.adobe.com/licenses.html)

    Some files are held under additional license.
    Please see "http://stlab.adobe.com/licenses.html" for more information.
-->


# Generic Image Library チュートリアル

#### 著者:
Lubomir Bourdev (<bourdev@adobe.com>)  
Hailin Jin (<hljin@adobe.com>)  
Adobe Systems Incorporated

#### Version:
2.1

#### 作成日時:
2007年9月15日  

<!--
The Generic Image Library (GIL) is a C++ library that abstracts image representations from algorithms and allows writing code that can work on a variety of images with performance similar to hand-writing for a specific image type.
This document will give you a jump-start in using GIL.
It does not discuss the underlying design of the library and does not cover all aspects of it.
You can find a detailed library design document on the main GIL web page at http://opensource.adobe.com/gil
-->

Generic Image Library (GIL)は、画像に適用されるアルゴリズムから画像の形式を抽象化することで、書き上げたアルゴリズムが様々な形式の画像でも動作することを可能にします。
また同時に、特定の形式の画像に特化したアルゴリズムに匹敵する速度で動作するコードの生成も実現します。
本文章は、GILの使用に関するジャンプスタートを提供します。
ただし、ライブラリの内部設計については説明しませんし、ライブラリの全体像については取り扱いません。
詳細なライブラリ設計についての文章は、GILのWebサイト<http://opensource.adobe.com/gil>を参照ください。

<!--
* Installation
* Example - Computing the Image Gradient
    * Interface and Glue Code
    * First Implementation
    * Using Locators
    * Creating a Generic Version of GIL Algorithms
    * Image View Transformations
    * 1D pixel iterators
    * STL Equivalent Algorithms
    * Color Conversion
    * Image
    * Virtual Image Views
    * Run-Time Specified Images and Image Views
    * Conclusion
* Appendix
    * Naming convention for GIL concrete types
-->

* [インストール](#section_01)
* [例題 - Gradient画像の算出](#section_02)
    * [インタフェースとグルーコード](#section_02_01)
    * [はじめてのインプリメンテーション](#section_02_02)
    * [Locatorの使い方](#section_02_03)
    * [ジェネリックなGILアルゴリズムの作成](#section_02_04)
    * [Image Viewの変換](#section_02_05)
    * [1次元Pixel Iterator](#section_02_06)
    * [STL-Styleアルゴリズム](#section_02_07)
    * [色変換](#section_02_08)
    * [Image](#section_02_09)
    * [Virtual Image View](#section_02_10)
    * [実行時に型を指定するImageとImage View](#section_02_11)
    * [まとめ](#section_02_12)
* [付録](#section_03)
    * [GILが定める型の命名規則](#section_03_01)

[section_01]: index.md#section_01 "section_01"
[section_02]: index.md#section_02 "section_02"
[section_02_01]: index.md#section_02_01 "section_02_01"
[section_02_02]: index.md#section_02_02 "section_02_02"
[section_02_03]: index.md#section_02_03 "section_02_03"
[section_02_04]: index.md#section_02_04 "section_02_04"
[section_02_05]: index.md#section_02_05 "section_02_05"
[section_02_06]: index.md#section_02_06 "section_02_06"
[section_02_07]: index.md#section_02_07 "section_02_07"
[section_02_08]: index.md#section_02_08 "section_02_08"
[section_02_09]: index.md#section_02_09 "section_02_09"
[section_02_10]: index.md#section_02_10 "section_02_10"
[section_02_11]: index.md#section_02_11 "section_02_11"
[section_02_12]: index.md#section_02_12 "section_02_12"
[section_03]: index.md#section_03 "section_03"
[section_03_01]: index.md#section_03_01 "section_03_01"

<!--
Installation
-->

## <a name="section_01"> インストール

<!--
The latest version of GIL can be downloaded from GIL's web page, at http://opensource.adobe.com/gil.
GIL is approved for integration into Boost and in the future will be installed simply by installing Boost from http://www.boost.org.
GIL consists of header files only and does not require any libraries to link against.
It does not require Boost to be built.
Including boost/gil/gil_all.hpp will be sufficient for most projects.
-->

最新バージョンのGILは、GILのWebサイト <http://opensource.adobe.com/gil> からダウンロードすることができます。
GILはBoostへの統合が承認されており、近いうちに <http://www.boost.org> からBoostをインストールする際にGILも同時にインストールされるようになるでしょう。
GILはヘッダーファイルだけで構成されており、他のライブラリのリンクは不要です。
また、Boostのビルドも必要ありません。
ほとんどのプロジェクトでは、`boost/gil/gil_all.hpp`をインクルードするだけで十分です。

<!--
Example - Computing the Image Gradient
-->

## <a name="section_02"> 例題 - Gradient画像の算出

<!--
This tutorial will walk through an example of using GIL to compute the image gradients.
We will start with some very simple and non-generic code and make it more generic as we go along.
Let us start with a horizontal gradient and use the simplest possible approximation to a gradient - central difference.
The gradient at pixel x can be approximated with the half-difference of its two neighboring pixels: D[x] = (I[x-1] - I[x+1]) / 2
For simplicity, we will also ignore the boundary cases - the pixels along the edges of the image for which one of the neighbors is not defined.
The focus of this document is how to use GIL, not how to create a good gradient generation algorithm.
-->

このチュートリアルは、Gradient画像を算出するというGILの使用例を通して進めていくことにしましょう。
始めは極めてシンプルでジェネリックでないコードからスタートし、それを少しずつジェネリックなコードにしていきます。
まずは水平方向のGradientから始めることとし、Gradientの最もシンプルな近似と思われる中心差分を使います。
位置xにあるPixelのGradientは、その両隣のPixelの差分の1/2で近似されます。

D[x] = (I[x-1] - I[x+1]) / 2

簡単のために、境界ケース(すなわち、対象のPixelが画像の端にあって隣接Pixelの片方が定義されていないケース)は無視することにしましょう。
この文章のフォーカスは、GILの使い方であり、上質なGradient画像生成アルゴリズムの作り方ではないのです。

<!--
Interface and Glue Code
-->

### <a name="section_02_01"> インタフェースとグルーコード

<!--
Let us first start with 8-bit unsigned grayscale image as the input and 8-bit signed grayscale image as the output.
Here is how the interface to our algorithm looks like:
-->

入力は8ビット符号なしグレイスケール画像、出力は8ビット符号付きグレイスケール画像として始めましょう。
まずは、GILを使って作られたアルゴリズムのインタフェースがどのような感じなのか示します。

{% highlight C++ %}

#include <boost/gil/gil_all.hpp>
using namespace boost::gil;

void x_gradient(const gray8c_view_t& src, const gray8s_view_t& dst) {
    assert(src.dimensions() == dst.dimensions());
    ...    // compute the gradient
}

{% endhighlight %}

<!--
gray8c_view_t is the type of the source image view - an 8-bit grayscale view, whose pixels are read-only (denoted by the "c").
The output is a grayscale view with a 8-bit signed (denoted by the "s") integer channel type.
See Appendix 1 for the complete convension GIL uses to name concrete types.
-->

`gray8c_view_t`は入力画像の型です。Pixelがread-only ("c"で表されています)の8ビットグレイスケールViewです。
出力は8ビット符号付き("s"で表されています)整数型のグレイスケール画像です。
GILが定める型の命名規則については、付録を参照ください。

<!--
GIL makes a distinction between an image and an image view.
A GIL image view, is a shallow, lightweight view of a rectangular grid of pixels.
It provides access to the pixels but does not own the pixels.
Copy-constructing a view does not deep-copy the pixels.
Image views do not propagate their constness to the pixels and should always be taken by a const reference.
Whether a view is mutable or read-only (immutable) is a property of the view type.
-->

GILはImageとImage Viewを区別します。
GILのImage Viewは、長方形格子状のPixelの範囲を指し示す、浅く軽いViewです。
ViewはPixelへのアクセスを提供しますが、Pixelそのものではありません。
ViewのコピーコンストラクションはPixelのディープコピーではありません。
Image Viewに付加されたconst性はPixelまで伝播しないので、Image Viewは常にconst参照で使用すべきです。
Viewがmutableであるかread-only (immutable)であるかは、Viewの型のプロパティです。

<!--
A GIL image, on the other hand, is a view with associated ownership.
It is a container of pixels; its constructor/destructor allocates/deallocates the pixels, its copy-constructor performs deep-copy of the pixels and its operator== performs deep-compare of the pixels.
Images also propagate their constness to their pixels - a constant reference to an image will not allow for modifying its pixels.
-->

一方、GILのImageは、所有権と関連づけられたViewの一種であり、Pixelのコンテナです。
すなわち、Imageのコンストラクタ/デストラクタはPixelのメモリの確保/解放を行い、コピーコンストラクタはPixelのディープコピーを行い、`operator==`はPixelのディープな比較を行います。
Imageに付加されたconst性はPixelまで伝播するので、Imageのconst参照はPixelの編集を許しません。

<!--
Most GIL algorithms operate on image views; images are rarely needed.
GIL's design is very similar to that of the STL.
The STL equivalent of GIL's image is a container, like std::vector, whereas GIL's image view corresponds to STL's range, which is often represented with a pair of iterators.
STL algorithms operate on ranges, just like GIL algorithms operate on image views.
-->

ほとんどのGILアルゴリズムはImage Viewの上で動作します。Imageが必要になることはめったにありません。
GILの設計は、STLの設計とよく似ています。
GILのImageはSTLにおける`std::vector`などといったコンテナに相当し、GILのImage ViewはSTLにおけるRange(しばしば、`begin()`と`end()`のようなIteratorの組で表現されています)に相当します。
STLアルゴリズムがRangeの上で動作するのと同じ様に、GILアルゴリズムはImage Viewの上で動作するのです。

<!--
GIL's image views can be constructed from raw data - the dimensions, the number of bytes per row and the pixels, which for chunky views are represented with one pointer.
Here is how to provide the glue between your code and GIL:
-->

GILのImage Viewは生データ(widthとheight、1行あたりのバイト数、かたまりごとにポインタで表現されたPixelデータ)から構成することができます。
ここで、自身のコードとGILのグルーをどのように提供するかを示します。

{% highlight C++ %}

void ComputeXGradientGray8(const unsigned char* src_pixels, ptrdiff_t src_row_bytes, int w, int h,
                                   signed char* dst_pixels, ptrdiff_t dst_row_bytes) {
    gray8c_view_t src = interleaved_view(w, h, (const gray8_pixel_t*)src_pixels,src_row_bytes);
    gray8s_view_t dst = interleaved_view(w, h, (     gray8s_pixel_t*)dst_pixels,dst_row_bytes);
    x_gradient(src,dst);
}

{% endhighlight %}

<!--
This glue code is very fast and views are lightweight - in the above example the views have a size of 16 bytes.
They consist of a pointer to the top left pixel and three integers - the width, height, and number of bytes per row.
-->

このグルーコードはとても高速であり、2つのViewはとても軽量(上記の例では16バイト)です。
それぞれのViewは、左上隅のPixelを示すポインタと3個の整数(width、height、1行あたりのバイト数)から構成されています。


<!--
First Implementation
-->

### <a name="section_02_02"> はじめてのインプリメンテーション

<!--
Focusing on simplicity at the expense of speed, we can compute the horizontal gradient like this:
-->

処理速度への影響がわかりやすくなるように、水平方向のgradientをまずは次のように計算します。

{% highlight C++ %}

void x_gradient(const gray8c_view_t& src, const gray8s_view_t& dst) {
    for (int y=0; y<src.height(); ++y)
        for (int x=1; x<src.width()-1; ++x)
            dst(x,y) = (src(x-1,y) - src(x+1,y)) / 2;
}

{% endhighlight %}

<!--
We use image view's operator(x,y) to get a reference to the pixel at a given location and we set it to the half-difference of its left and right neighbors.
operator() returns a reference to a grayscale pixel.
A grayscale pixel is convertible to its channel type (unsigned char for src) and it can be copy-constructed from a channel.
(This is only true for grayscale pixels).
While the above code is easy to read, it is not very fast, because the binary operator() computes the location of the pixel in a 2D grid, which involves addition and multiplication.
Here is a faster version of the above:
-->

Image Viewの`operator(x,y)`を使用して与えられた座標のPixel参照を取得し、そこに両隣のPixelの差分の1/2を代入します。
`operator()`はグレイスケールPixelの参照を返します。
グレイスケールPixelはそのChannel (`src`における`unsigned char`)と変換可能であり、Channelからのコピーコンストラクションが可能です。(これが可能なのはグレイスケールPixelだけです。)
上記のコードは読みやすいのですが、それほど高速ではありません。というのも、ふたつの引数を取る`operator()`が2次元格子上の座標を算出する際に和と積を用いているからです。
上記のコードをより高速にしたバージョンは次の様になります。

{% highlight C++ %}

void x_gradient(const gray8c_view_t& src, const gray8s_view_t& dst) {
    for (int y=0; y<src.height(); ++y) {
        gray8c_view_t::x_iterator src_it = src.row_begin(y);
        gray8s_view_t::x_iterator dst_it = dst.row_begin(y);

        for (int x=1; x<src.width()-1; ++x)
            dst_it[x] = (src_it[x-1] - src_it[x+1]) / 2;
    }
}

{% endhighlight %}

<!--
We use pixel iterators initialized at the beginning of each row.
GIL's iterators are Random Access Traversal iterators.
If you are not familiar with random access iterators, think of them as if they were pointers.
In fact, in the above example the two iterator types are raw C pointers and their operator[] is a fast pointer indexing operator.
-->

このコードでは、各行の先頭を指すように初期化されたPixel Iteratorを使用します。
GILのIteratorはランダムアクセス走査Iteratorです。
もしランダムアクセスIteratorに詳しくないのであれば、ひとまずポインタだと考えておきましょう。
事実、上記の例における2個のIteratorはCポインタであり、`operator[]`はポインタの高速なインデクシングを行う演算子です。

<!--
The code to compute gradient in the vertical direction is very similar:
-->

垂直方向のgradientを計算するコードも、上記のコードにとてもよく似ています。

{% highlight C++ %}

void y_gradient(const gray8c_view_t& src, const gray8s_view_t& dst) {
    for (int x=0; x<src.width(); ++x) {
        gray8c_view_t::y_iterator src_it = src.col_begin(x);
        gray8s_view_t::y_iterator dst_it = dst.col_begin(x);

        for (int y=1; y<src.height()-1; ++y)
            dst_it[y] = (src_it[y-1] - src_it[y+1])/2;
    }
}

{% endhighlight %}

<!--
Instead of looping over the rows, we loop over each column and create a y_iterator, an iterator moving vertically.
In this case a simple pointer cannot be used because the distance between two adjacent pixels equals the number of bytes in each row of the image.
GIL uses here a special step iterator class whose size is 8 bytes - it contains a raw C pointer and a step.
Its operator[] multiplies the index by its step.
-->

各行のループを回すかわりに各列のループを回し、その中で垂直方向に移動するIteratorである`y_iterator`を作成します。
ただし、垂直方向に隣接するPixel間のメモリ上での距離はその画像の1行分のバイト数に等しいので、垂直方向Iteratorにシンプルなポインタを使用することはできません。
ここでGILはサイズが8バイトの特別なステップIterator (Cポインタとステップ幅の値を含みます)を使用します。
`operator[]`はインデクス値とステップ幅の値との積を求めます。

<!--
The above version of y_gradient, however, is much slower (easily an order of magnitude slower) than x_gradient because of the memory access pattern;
traversing an image vertically results in lots of cache misses.
A much more efficient and cache-friendly version will iterate over the columns in the inner loop:
-->

しかし、上記のバージョンにおける`y_gradient`は、そのメモリアクセスパターンが原因で、`x_gradient`と比べて非常に低速です。
垂直方向の画像の走査が、多くのキャッシュを無駄にするからです。
より効率的でキャッシュフレンドリなバージョンでは、垂直方向のループの内部で各行の処理を反復します。

{% highlight C++ %}

void y_gradient(const gray8c_view_t& src, const gray8s_view_t& dst) {
    for (int y=1; y<src.height()-1; ++y) {
        gray8c_view_t::x_iterator src1_it = src.row_begin(y-1);
        gray8c_view_t::x_iterator src2_it = src.row_begin(y+1);
        gray8s_view_t::x_iterator dst_it = dst.row_begin(y);

        for (int x=0; x<src.width(); ++x) {
            *dst_it = ((*src1_it) - (*src2_it))/2;
            ++dst_it;
            ++src1_it;
            ++src2_it;
        }
    }
}

{% endhighlight %}

<!--
This sample code also shows an alternative way of using pixel iterators - instead of operator[] one could use increments and dereferences.
-->

このサンプルコードでは、`operator[]`を通してインクリメントと間接参照を行う方法のかわりに、Pixel Iteratorを用いる代替手段を示しています。

<!--
Using Locators
-->

### <a name="section_02_03"> Locatorの使い方

<!--
Unfortunately this cache-friendly version requires the extra hassle of maintaining two separate iterators in the source view.
For every pixel, we want to access its neighbors above and below it.
Such relative access can be done with GIL locators:
-->

残念なことに、このキャッシュフレンドリなバージョンでは、入力Viewの中で2個のIteratorを扱うという余計な手間が掛かっています。
ここで私たちが行いたいのは、各Pixelの上下に隣接するPixelへのアクセスです。
そういった相対アクセスは、GILのLocatorによって行うことができます。

{% highlight C++ %}

void y_gradient(const gray8c_view_t& src, const gray8s_view_t& dst) {
    gray8c_view_t::xy_locator src_loc = src.xy_at(0,1);
    for (int y=1; y<src.height()-1; ++y) {
        gray8s_view_t::x_iterator dst_it  = dst.row_begin(y);

        for (int x=0; x<src.width(); ++x) {
            (*dst_it) = (src_loc(0,-1) - src_loc(0,1)) / 2;
            ++dst_it;
            ++src_loc.x();                  // each dimension can be advanced separately
        }
        src_loc+=point2<std::ptrdiff_t>(-src.width(),1);    // carriage return
    }
}

{% endhighlight %}

<!--
The first line creates a locator pointing to the first pixel of the second row of the source view.
A GIL pixel locator is very similar to an iterator, except that it can move both horizontally and vertically.
src_loc.x() and src_loc.y() return references to a horizontal and a vertical iterator respectively, which can be used to move the locator along the desired dimension, as shown above.
Additionally, the locator can be advanced in both dimensions simultaneously using its operator+= and operator-=.
Similar to image views, locators provide binary operator() which returns a reference to a pixel with a relative offset to the current locator position.
For example, src_loc(0,1) returns a reference to the neighbor below the current pixel.
Locators are very lightweight objects - in the above example the locator has a size of 8 bytes -
it consists of a raw pointer to the current pixel and an int indicating the number of bytes from one row to the next (which is the step when moving vertically).
The call to ++src_loc.x() corresponds to a single C pointer increment.
However, the example above performs more computations than necessary.
The code src_loc(0,1) has to compute the offset of the pixel in two dimensions, which is slow.
Notice though that the offset of the two neighbors is the same, regardless of the pixel location.
To improve the performance, GIL can cache and reuse this offset:
-->

最初の行では、入力Viewの2行目の先頭を指し示すLocatorを作成しています。
GILのPixel Locatorは、水平方向と垂直方向のどちらにも移動できること以外は、Iteratorとよく似ています。
上記のコードにある通り、`src_loc.x()`と`src_loc.y()`はそれぞれ水平方向Iteratorの参照と垂直方向Iteratorの参照をそれぞれ返すので、それらを使ってLocatorを好きな方向に動かすことができます。
加えて、Locatorは`operator+=`と`operator-=`を用いることで、両方向へ同時に移動することが出来ます。
Image Viewと同じように、Locatorは現在位置を基準にした相対的オフセットで指定されるPixelへの参照を返す`operator()`(この演算子は2個の引数を取ります)を提供します。
例えば、`src_loc(0,1)`は現在指し示しているPixelの下側に隣接するPixelの参照を返します。
Locatorはとても軽量なオブジェクトであり、上記の例ではわずか8バイトです。
現在のPixelを指す生ポインタと、ある行から次の行までのバイト数を表す整数型(垂直移動の際のステップ数として使用します)で構成されています。
`++src_loc.x()`の呼び出しは、Cポインタのインクリメント1回に相当します。
ところが、上記のコードの内部では必要以上の計算が行われてしまっています。
`src_loc(0,1)`のコードは2方向のPixelオフセットを計算をしなければならず、低速なのです。
と言いつつ、両隣のPixelとのオフセットはPixelの座標にかかわらず一定であることに着目しましょう。
パフォーマンス向上のために、GILはこのオフセットをキャッシュして再利用することができるのです。

{% highlight C++ %}

void y_gradient(const gray8c_view_t& src, const gray8s_view_t& dst) {
    gray8c_view_t::xy_locator src_loc = src.xy_at(0,1);
    gray8c_view_t::xy_locator::cached_location_t above = src_loc.cache_location(0,-1);
    gray8c_view_t::xy_locator::cached_location_t below = src_loc.cache_location(0, 1);

    for (int y=1; y<src.height()-1; ++y) {
        gray8s_view_t::x_iterator dst_it = dst.row_begin(y);

        for (int x=0; x<src.width(); ++x) {
            (*dst_it) = (src_loc[above] - src_loc[below])/2;
            ++dst_it;
            ++src_loc.x();
        }
        src_loc+=point2<std::ptrdiff_t>(-src.width(),1);
    }
}

{% endhighlight %}

<!--
In this example "src_loc[above]" corresponds to a fast pointer indexing operation and the code is efficient.
-->

この例における`src_loc[above]`はポインタの高速なインデクシングに相当しており、このコードは効率的です。

<!--
Creating a Generic Version of GIL Algorithms
-->

### <a name="section_02_04"> ジェネリックなGILアルゴリズムの作成

<!--
Let us make our x_gradient more generic.
It should work with any image views, as long as they have the same number of channels.
The gradient operation is to be computed for each channel independently.
Here is how the new interface looks like:
-->

`x_gradient`のコードをよりジェネリックにしていきましょう。
まず、同じChannel数をもつどのようなImage Viewでも動作すべきです。
gradientの計算は各Channelが独立に計算されます。
新しいインタフェースがどのようになるのか示します。

{% highlight C++ %}

template <typename SrcView, typename DstView>
void x_gradient(const SrcView& src, const DstView& dst) {
    gil_function_requires<ImageViewConcept<SrcView> >();
    gil_function_requires<MutableImageViewConcept<DstView> >();
    gil_function_requires<ColorSpacesCompatibleConcept<
                                typename color_space_type<SrcView>::type,
                                typename color_space_type<DstView>::type> >();

    ... // compute the gradient
}

{% endhighlight %}

<!--
The new algorithm now takes the types of the input and output image views as template parameters.
That allows using both built-in GIL image views, as well as any user-defined image view classes.
The first three lines are optional;
they use boost::concept_check to ensure that the two arguments are valid GIL image views, that the second one is mutable and that their color spaces are compatible (i.e. have the same set of channels).
-->

この新しいアルゴリズムでは、テンプレートのパラメータとして入力のImage Viewと出力のImage Viewを取ります。
このアルゴリズムは、GILのビルトインImage Viewとユーザ定義のImage Viewのどちらでも使うことができます。
そして、上記の関数内の最初の3行は任意です。
この3行では、2個の仮引数がそれぞれ有効なGIL Image Viewであること、2個目の仮引数のImage Viewがmutableであること、2個のImage ViewのColor Spaceの間に互換性があることを、`boost::concept_check`を用いて保証しています。

<!--
GIL does not require using its own built-in constructs.
You are free to use your own channels, color spaces, iterators, locators, views and images.
However, to work with the rest of GIL they have to satisfy a set of requirements; in other words, they have to model the corresponding GIL concept.
GIL's concepts are defined in the user guide.
-->

GILは、ビルトインコンストラクトの中でも、`boost::concept_check`の使用を要求していません。
ユーザが、独自のChannel、Color Space、Iterator、Locator、View、Imageを使うことは自由です。
しかし、その他の部分を担うGILと一緒に動作させるためには、独自のコンストラクトは`boost::concept_check`による要求のセットを満たさなければなりません。
言い換えると、独自のコンストラクトは、関連するGIL Conceptに基づいたModelでなければなりません。
GIL Conceptはユーザガイドの中で定義されています。

<!--
One of the biggest drawbacks of using templates and generic programming in C++ is that compile errors can be very difficult to comprehend.
This is a side-effect of the lack of early type checking - a generic argument may not satisfy the requirements of a function, but the incompatibility may be triggered deep into a nested call, in code unfamiliar and hardly related to the problem.
GIL uses boost::concept_check to mitigate this problem.
The above three lines of code check whether the template parameters are valid models of their corresponding concepts.
If a model is incorrect, the compile error will be inside gil_function_requires, which is much closer to the problem and easier to track.
Furthermore, such checks get compiled out and have zero performance overhead.
The disadvantage of using concept checks is the sometimes severe impact they have on compile time.
This is why GIL performs concept checks only in debug mode, and only if BOOST_GIL_USE_CONCEPT_CHECK is defined (off by default).
-->

C++のテンプレートとジェネリックプログラミングの最大の欠点のひとつは、コンパイルエラーの意味を読み取ることが非常に困難だということです。
これは、型の判定が先延ばしされていることの副作用です。
ジェネリックな引数が関数からの要求を満たしていない可能性がありますが、その不一致は、コードの中の見慣れない、問題とほとんど関係のない、幾重にもネストされた関数コールがきっかけとなって発生するのです。
GILでは、この問題を軽減するために`boost::concept_check`を用います。
先の3行は、テンプレートのパラメータが関連するConceptを満たした有効なModelであるかどうかを調べています。
Modelが正しくない場合、問題に近く追跡が簡単な`gil_function_requires`のなかでコンパイルエラーが発生します。
加えて、これらのチェックはコンパイル時に行われ、パフォーマンスへの影響はありません。
コンセプトチェックを用いることの欠点は、コンパイル時間に深刻なインパクトを与える場合があるということです。
このことから、GILはデバッグモードで且つ`BOOST_GIL_USE_CONCEPT_CHECK`が定義されている場合にだけコンセプトチェックを行います。
(ちなみに、`BOOST_GIL_USE_CONCEPT_CHECK`はデフォルトでOFFです。)

<!--
The body of the generic function is very similar to that of the concrete one.
The biggest difference is that we need to loop over the channels of the pixel and compute the gradient for each channel:
-->

ジェネリックな関数の中身は、型が決まっている一般の関数の中身とよく似ています。
もっとも大きな違いは、Pixelの各Channelをループして各Channel毎にgradientの計算が必要だということです。

{% highlight C++ %}

template <typename SrcView, typename DstView>
void x_gradient(const SrcView& src, const DstView& dst) {
    for (int y=0; y<src.height(); ++y) {
        typename SrcView::x_iterator src_it = src.row_begin(y);
        typename DstView::x_iterator dst_it = dst.row_begin(y);

        for (int x=1; x<src.width()-1; ++x)
            for (int c=0; c<num_channels<SrcView>::value; ++c)
                dst_it[x][c] = (src_it[x-1][c]- src_it[x+1][c])/2;
    }
}

{% endhighlight %}

<!--
Having an explicit loop for each channel could be a performance problem.
GIL allows us to abstract out such per-channel operations:
-->

単純な各Channelのループは、パフォーマンスの問題になる可能性があります。
GILは、各Channelの操作を次のように抽象化することができます。

{% highlight C++ %}

template <typename Out>
struct halfdiff_cast_channels {
    template <typename T> Out operator()(const T& in1, const T& in2) const {
        return Out((in1-in2)/2);
    }
};

template <typename SrcView, typename DstView>
void x_gradient(const SrcView& src, const DstView& dst) {
    typedef typename channel_type<DstView>::type dst_channel_t;

    for (int y=0; y<src.height(); ++y) {
        typename SrcView::x_iterator src_it = src.row_begin(y);
        typename DstView::x_iterator dst_it = dst.row_begin(y);

        for (int x=1; x<src.width()-1; ++x)
            static_transform(src_it[x-1], src_it[x+1], dst_it[x],
                               halfdiff_cast_channels<dst_channel_t>());
    }
}

{% endhighlight %}

<!--
static_transform is an example of a channel-level GIL algorithm.
Other such algorithms are static_generate, static_fill and static_for_each.
They are the channel-level equivalents of STL's generate, transform, fill and for_each respectively.
GIL channel algorithms use static recursion to unroll the loops; they never loop over the channels explicitly.
Note that sometimes modern compilers (at least Visual Studio 8) already unroll channel-level loops, such as the one above.
However, another advantage of using GIL's channel-level algorithms is that they pair the channels semantically, not based on their order in memory.
For example, the above example will properly match an RGB source with a BGR destination.
-->

`static_transform`はChannelレベルのGILアルゴリズムのひとつです。
この他には、`static_genrerate`、`static_fill`、`static_for_each`といったアルゴリズムがあります。
これらのアルゴリズムは、`generate`、`transform`、`fill`、`for_each`といったSTLアルゴリズムと同等なChannelレベルのアルゴリズムです。
GILのChannelアルゴリズムは、ループを回すことがないように、静的な再帰を用います。
これらのアルゴリズムは、各Channelの単純なループを決して行いません。
上記の例などであれば、モダンなコンパイラ(たとえVisual Studio 8であっても)はChannelレベルのループを行うことはないでしょう。
しかし、GILのChannelレベルのアルゴリズムを用いるもうひとつの利点は、メモリ上での順序ではなくセマンティックなChannel順序を用いるということです。
例を挙げると、上記のコードは入力ViewがRGBで出力ViewがBGRであっても問題なく適合します。

<!--
Here is how we can use our generic version with images of different types:
-->

型が異なるImageに対して、ジェネリックなコードをどのように用いるのか示します。

{% highlight C++ %}

// Calling with 16-bit grayscale data
void XGradientGray16_Gray32(const unsigned short* src_pixels, ptrdiff_t src_row_bytes, int w, int h,
                                  signed int* dst_pixels, ptrdiff_t dst_row_bytes) {
    gray16c_view_t src=interleaved_view(w,h,(const gray16_pixel_t*)src_pixels,src_row_bytes);
    gray32s_view_t dst=interleaved_view(w,h,(     gray32s_pixel_t*)dst_pixels,dst_row_bytes);
    x_gradient(src,dst);
}

// Calling with 8-bit RGB data into 16-bit BGR
void XGradientRGB8_BGR16(const unsigned char* src_pixels, ptrdiff_t src_row_bytes, int w, int h,
                                 signed short* dst_pixels, ptrdiff_t dst_row_bytes) {
    rgb8c_view_t  src = interleaved_view(w,h,(const rgb8_pixel_t*)src_pixels,src_row_bytes);
    rgb16s_view_t dst = interleaved_view(w,h,(    rgb16s_pixel_t*)dst_pixels,dst_row_bytes);
    x_gradient(src,dst);
}

// Either or both the source and the destination could be planar - the gradient code does not change
void XGradientPlanarRGB8_RGB32(
           const unsigned short* src_r, const unsigned short* src_g, const unsigned short* src_b,
           ptrdiff_t src_row_bytes, int w, int h,
           signed int* dst_pixels, ptrdiff_t dst_row_bytes) {
    rgb16c_planar_view_t src=planar_rgb_view (w,h, src_r,src_g,src_b,         src_row_bytes);
    rgb32s_view_t        dst=interleaved_view(w,h,(rgb32s_pixel_t*)dst_pixels,dst_row_bytes);
    x_gradient(src,dst);
}

{% endhighlight %}

<!--
As these examples illustrate, both the source and the destination can be interleaved or planar, of any channel depth (assuming the destination channel is assignable to the source), and of any compatible color spaces.
-->

これらの例が示す通り、入力と出力はともにインタリーブ形式であってもプラナー形式であっても構いませんし、それらのChannle深度は(入力から出力に代入可能であれば)どのようであれ構いませんし，それらのColor Spaceは(互換性さえあれば)どのようであっても構いません。

<!--
GIL 2.1 can also natively represent images whose channels are not byte-aligned, such as 6-bit RGB222 image or a 1-bit Gray1 image.
GIL algorithms apply to these images natively.
See the design guide or sample files for more on using such images.
-->

GIL 2.1は、6ビットRGB222 Imageや1ビットGray1 Imageといったバイト単位ではないChannelをもつImageについてもネイティブに扱うことができます。
GILのアルゴリズムは、ネイティブにこれらのImageへ適用することができます。
このようなImageの用例ついては、デザインガイドやサンプルファイルを参照ください。

<!--
Image View Transformations
-->

### <a name="section_02_05"> Image Viewの変換

<!--
One way to compute the y-gradient is to rotate the image by 90 degrees, compute the x-gradient and rotate the result back.
Here is how to do this in GIL:
-->

`y_gradient`を計算する方法のひとつに、画像を90度回転させて`x_gradient`を計算し、その結果を逆方向に90度回転させて元の向きに戻すというものがあります。
それをGILでどのように行うかを次に示します。

{% highlight C++ %}

template <typename SrcView, typename DstView>
void y_gradient(const SrcView& src, const DstView& dst) {
    x_gradient(rotated90ccw_view(src), rotated90ccw_view(dst));
}

{% endhighlight %}

<!--
rotated90ccw_view takes an image view and returns an image view representing 90-degrees counter-clockwise rotation of its input.
It is an example of a GIL view transformation function.
GIL provides a variety of transformation functions that can perform any axis-aligned rotation, transpose the view, flip it vertically or horizontally, extract a rectangular subimage, perform color conversion, subsample view, etc.
The view transformation functions are fast and shallow - they don't copy the pixels, they just change the "coordinate system" of accessing the pixels.
rotated90cw_view, for example, returns a view whose horizontal iterators are the vertical iterators of the original view.
The above code to compute y_gradient is slow because of the memory access pattern; using rotated90cw_view does not make it any slower.
-->

`rotated90ccw_view`は、あるImage Viewを引数に取り、それを90度反時計回りに回転させたImage Viewを返します。
これは、GILのView変換関数の一例です。
GILは、軸に沿った回転、転置、垂直方向または水平方向の反転、矩形領域の切り出し、色空間の変換、サブサンプリングなど、様々なView変換関数を提供します。
View変換関数は浅く、高速です。
View変換関数では、Pixelのコピーは行わず、Pixelにアクセスする際の"座標系"を変更するだけです。
例を挙げると、`rotated90cw_view`はオリジナルのViewの垂直方向Iteratorを水平方向IteratorとしてもつViewを返します。
先に挙げた`y_gradient`を計算するコードはメモリのアクセスパターンが原因で低速でしたが、`rotated90cw_view`を用いたコードは全く遅くなりません。

<!--
Another example: suppose we want to compute the gradient of the N-th channel of a color image.
Here is how to do that:
-->

もうひとつの例として、カラー画像のN番目Channelのgradientを計算してみましょう。

{% highlight C++ %}

template <typename SrcView, typename DstView>
void nth_channel_x_gradient(const SrcView& src, int n, const DstView& dst) {
    x_gradient(nth_channel_view(src, n), dst);
}

{% endhighlight %}

<!--
nth_channel_view is a view transformation function that takes any view and returns a single-channel (grayscale) view of its N-th channel.
For interleaved RGB view, for example, the returned view is a step view - a view whose horizontal iterator skips over two channels when incremented.
If applied on a planar RGB view, the returned type is a simple grayscale view whose horizontal iterator is a C pointer.
Image view transformation functions can be piped together. For example, to compute the y gradient of the second channel of the even pixels in the view, use:
-->

`nth_channel_view`は、あらゆるViewを引数に取り、そのN番目Channelだけをもつ単Channel (グレイスケール) Viewとして返すView変換関数です。
インタリーブRGB Viewの場合、この関数から返されるViewはインクリメントされたときに2つのChannelを飛び越す水平方向IteratorをもつスキップViewです。
プラナーRGB Viewに用いられた場合には、Cポインタが水平方向Iteratorになっている、シンプルなグレイスケールViewが返されます。
View変換関数は、互いを連結させることが可能です。
例えば、Viewの2番目ChannelのY方向gradientを計算する場合、次のようにします。

{% highlight C++ %}

y_gradient(subsampled_view(nth_channel_view(src, 1), 2,2), dst);

{% endhighlight %}

<!--
GIL can sometimes simplify piped views.
For example, two nested subsampled views (views that skip over pixels in X and in Y) can be represented as a single subsampled view whose step is the product of the steps of the two views.
-->

GILでは、連結された複数のViewを簡略化できる場合があります。
例を挙げると、入れ子になった2つのサブサンプルView (X軸方向でスキップするViewとY軸方向でスキップするView)は、その2つのViewのステップの積をステップとする1つのサブサンプルViewに置き換えることが可能です。

<!--
1D pixel iterators
-->

### <a name="section_02_06"> 1次元Pixel Iterator

<!--
Let's go back to x_gradient one more time.
Many image view algorithms apply the same operation for each pixel and GIL provides an abstraction to handle them.
However, our algorithm has an unusual access pattern, as it skips the first and the last column.
It would be nice and instructional to see how we can rewrite it in canonical form.
The way to do that in GIL is to write a version that works for every pixel, but apply it only on the subimage that excludes the first and last column:
-->

ここでもう一度`x_gradient`の話に戻りましょう。
多くのImage Viewアルゴリズムでは各Pixelに同じ処理を行います。GILは、そのような一連の手順を抽象化します。
振り返ってみると、先ほどの`x_gradient`アルゴリズムでは、画像の最初の列にあるPixelと最後の列にあるPixelについてはスキップするという変則的なアクセスパターンを用いていました。
これをどのようにして規則的なアクセスパターンに書き直すことができるのかを知っておくのも有意義でしょう。
GILでこれを実現するには、View内の全てのPixelに対して処理を行うバージョンを作成し、それを最初の列と最後の列を除いたサブイメージに対して適用するという方法をとります。

{% highlight C++ %}

void x_gradient_unguarded(const gray8c_view_t& src, const gray8s_view_t& dst) {
    for (int y=0; y<src.height(); ++y) {
        gray8c_view_t::x_iterator src_it = src.row_begin(y);
        gray8s_view_t::x_iterator dst_it = dst.row_begin(y);

        for (int x=0; x<src.width(); ++x)
            dst_it[x] = (src_it[x-1] - src_it[x+1]) / 2;
    }
}

void x_gradient(const gray8c_view_t& src, const gray8s_view_t& dst) {
    assert(src.width()>=2);
    x_gradient_unguarded(subimage_view(src, 1, 0, src.width()-2, src.height()),
                         subimage_view(dst, 1, 0, src.width()-2, src.height()));
}

{% endhighlight %}

<!--
subimage_view is another example of a GIL view transformation function.
It takes a source view and a rectangular region (in this case, defined as x_min,y_min,width,height) and returns a view operating on that region of the source view.
The above implementation has no measurable performance degradation from the version that operates on the original views.
-->

`subimage_view`はGIL View変換関数のもうひとつの例です。
`subimage_view`は、入力Viewと矩形領域(ここでは、`x_min`、`y_min`、`width`、`height`)を引数にとり、入力Viewの指定した矩形領域を指し示すViewを返します。
上記の実装であれば、元のViewに対して処理を行っていたバージョンと比較しても、測定できるようなパフォーマンスの低下は生じません。

<!--
Now that x_gradient_unguarded operates on every pixel, we can rewrite it more compactly:
-->

また、全Pixelに対して処理を行う`x_gradient_unguarded`は、よりコンパクトに書き換えることができます。

{% highlight C++ %}

void x_gradient_unguarded(const gray8c_view_t& src, const gray8s_view_t& dst) {
    gray8c_view_t::iterator src_it = src.begin();
    for (gray8s_view_t::iterator dst_it = dst.begin(); dst_it!=dst.end(); ++dst_it, ++src_it)
      * dst_it = (src_it.x()[-1] - src_it.x()[1]) / 2;
}

{% endhighlight %}

<!--
GIL image views provide begin() and end() methods that return one dimensional pixel iterators which iterate over each pixel in the view, left to right and top to bottom.
They do a proper "carriage return" - they skip any unused bytes at the end of a row.
As such, they are slightly suboptimal, because they need to keep track of their current position with respect to the end of the row.
Their increment operator performs one extra check (are we at the end of the row?), a check that is avoided if two nested loops are used instead.
These iterators have a method x() which returns the more lightweight horizontal iterator that we used previously.
Horizontal iterators have no notion of the end of rows.
In this case, the horizontal iterators are raw C pointers.
In our example, we must use the horizontal iterators to access the two neighbors properly, since they could reside outside the image view.
-->

GILのImage Viewは、View内の全てのPixelを左から右かつ上から下の順序で走査する1次元Pixel Iteratorを返す`begin()`と`end()`という関数を提供します。
これらの1次元Pixel Iteratorは完璧な"キャリッジリターン"を行います。
すなわち、各行の末尾に未使用のバイトがあればスキップします。
そしてこのことから、1次元Pixel Iteratorはわずかに最適化の余地を残しています。というのは、これらの1次元Pixel Iteratorは行の末尾を考慮するために自身の現在位置を記録しておく必要があるのです。
1次元Pixel Iteratorのインクリメントを行う演算子は追加のチェック(現在位置は行の末尾か?)を行っており、代わりとして2段にネストされたループを用いる場合には、このチェックをさけることが出来ます。
これらの1次元Pixel Iteratorは、さらに軽量な水平方向Iteratorを返す`x()`という関数(前のコードで使用)をもっています。
水平方向Iteratorは行の末尾についての情報をもっていません。
今回のケースでは、水平方向Iteratorは生のCポインタです。
この例では、ふたつの隣接PixelにはImage Viewが指し示す範囲外に配置されている可能性があることから、それらに適切にアクセスするために水平方向Iteratorを用いなければなりません。

<!--
STL Equivalent Algorithms
-->

### <a name="section_02_07"> STL-Styleアルゴリズム

<!--
GIL provides STL equivalents of many algorithms.
For example, std::transform is an STL algorithm that sets each element in a destination range the result of a generic function taking the corresponding element of the source range.
In our example, we want to assign to each destination pixel the value of the half-difference of the horizontal neighbors of the corresponding source pixel.
If we abstract that operation in a function object, we can use GIL's transform_pixel_positions to do that:
-->

GILは、STL-Styleの多くのアルゴリズムを提供しています。
例えば、`std::transform`は、出力コンテナの指定範囲にある各要素に対して、対応する入力要素にジェネリック関数を適用した結果をセットするSTLアルゴリズムです。
これまで挙げてきた例では、出力Image Viewの各Pixelに対して、対応する入力Pixelの両隣のPixelの差分の1/2を割り当てる処理を行ってきました。
計算部分を関数オブジェクトとして抽象化すると、この処理はGILの`transform_pixel_position`を用いて次のように書くことができます。

{% highlight C++ %}

struct half_x_difference {
    int operator()(const gray8c_loc_t& src_loc) const {
        return (src_loc.x()[-1] - src_loc.x()[1]) / 2;
    }
};

void x_gradient_unguarded(const gray8c_view_t& src, const gray8s_view_t& dst) {
    transform_pixel_positions(src, dst, half_x_difference());
}

{% endhighlight %}

<!--
GIL provides the algorithms for_each_pixel and transform_pixels which are image view equivalents of STL's std::for_each and std::transform.
It also provides for_each_pixel_position and transform_pixel_positions, which instead of references to pixels, pass to the generic function pixel locators.
This allows for more powerful functions that can use the pixel neighbors through the passed locators.
GIL algorithms iterate through the pixels using the more efficient two nested loops (as opposed to the single loop using 1-D iterators)
-->

GILは、Image ViewにおけるSTLの`std::for_each`や`std::transform`相当のものとして、`for_each_pixel`や`transform_pixel`を提供します。
また、ジェネリック関数に対してPixel参照の代わりにPixel Locatorを渡す、`for_each_pixel_positions`や`transform_pixel_positions`も提供します。
このことは、渡されたPixel Locatorを通じて隣接Pixelを使用する、さらに強力な関数を可能にします。
GILアルゴリズムは、(1次元Pixel Iteratorを用いた1段のループではなく)より効率的な2段にネストされたループを用いて反復を行います。

<!--
Color Conversion
-->

### <a name="section_02_08"> 色変換

<!--
Instead of computing the gradient of each color plane of an image, we often want to compute the gradient of the luminosity.
In other words, we want to convert the color image to grayscale and compute the gradient of the result.
Here how to compute the luminosity gradient of a 32-bit float RGB image:
-->

画像の各色平面のgradientを計算するのではなく、明度のgradientを計算したい場合も多々あります。
言い換えると、カラー画像からグレイスケール画像に変換して、その結果のgradientを計算したい場合です。
32ビット浮動小数点型RGB画像の明度のgradientをいかに算出するか、次に示します。

{% highlight C++ %}

void x_gradient_rgb_luminosity(const rgb32fc_view_t& src, const gray8s_view_t& dst) {
    x_gradient(color_converted_view<gray8_pixel_t>(src), dst);
}

{% endhighlight %}

<!--
color_converted_view is a GIL view transformation function that takes any image view and returns a view in a target color space and channel depth (specified as template parameters).
In our example, it constructs an 8-bit integer grayscale view over 32-bit float RGB pixels.
Like all other view transformation functions, color_converted_view is very fast and shallow.
It doesn't copy the data or perform any color conversion.
Instead it returns a view that performs color conversion every time its pixels are accessed.
-->

`color_converted_view`は、あらゆるImage Viewを引数に取り、(テンプレートのパラメータで指定した)目標のColor SpaceとChannel深度をもったViewを返す、GILのView変換関数です。
ここで示した例では、`color_converted_view`は32ビット浮動小数点型RGB Pixelを指すViewから8ビット整数型グレイスケールViewを構築します。
他のView変換関数と同様に、`color_converted_view`は浅く、非常に高速です。
この関数はデータのコピーも色変換も行いません。
そのかわりに、この関数はPixelへのアクセス毎にそのPixelの色変換を実行するViewを返します。

<!--
In the generic version of this algorithm we might like to convert the color space to grayscale, but keep the channel depth the same.
We do that by constructing the type of a GIL grayscale pixel with the same channel as the source, and color convert to that pixel type:
-->

このアルゴリズムのジェネリックなバージョンで、Color Spaceをグレイスケールに変換してもそのChannel深度は変更しないほうがよいでしょう。
入力画像と同じChannel型をもつグレイスケールPixelを構築してそのPixelへ色変換することで、これを実現します。

{% highlight C++ %}

template <typename SrcView, typename DstView>
void x_luminosity_gradient(const SrcView& src, const DstView& dst) {
    typedef pixel<typename channel_type<SrcView>::type, gray_layout_t> gray_pixel_t;
    x_gradient(color_converted_view<gray_pixel_t>(src), dst);
}

{% endhighlight %}

<!--
When the destination color space and channel type happens to be the same as the source one, color conversion is unnecessary.
GIL detects this case and avoids calling the color conversion code at all - i.e. color_converted_view returns back the source view unchanged.
-->

目標のColor SpaceとChannel型が偶然にも入力のそれらと同じだった場合、色変換は必要ありません。
GILはこのようなケースを検出し、色変換のコードが呼び出されるのを完璧に回避します。
つまり、このような場合の`color_converted_view`は、入力Viewそのものを返すのです。

<!--
Image
-->

### <a name="section_02_09"> Image

<!--
The above example has a performance problem - x_gradient dereferences most source pixels twice, which will cause the above code to perform color conversion twice.
Sometimes it may be more efficient to copy the color converted image into a temporary buffer and use it to compute the gradient - that way color conversion is invoked once per pixel.
Using our non-generic version we can do it like this:
-->

上記の例はパフォーマンス上の問題を抱えています。
`x_gradient`は、ほぼ全ての入力Pixelを2回ずつ間接参照しており、それが原因で上記のコードは色変換を各Pixelで2回実行しています。
色変換を行った画像を一時的なバッファにコピーしてそのgradientを計算する(この方法であれば、色変換は各Pixelで1回で済みます)ほうが効率的な場合もあるかもしれません。
ジェネリックでないバージョンは、次のようになります。

{% highlight C++ %}

void x_luminosity_gradient(const rgb32fc_view_t& src, const gray8s_view_t& dst) {
    gray8_image_t ccv_image(src.dimensions());
    copy_pixels(color_converted_view<gray8_pixel_t>(src), view(ccv_image));

    x_gradient(const_view(ccv_image), dst);
}

{% endhighlight %}

<!--
First we construct an 8-bit grayscale image with the same dimensions as our source.
Then we copy a color-converted view of the source into the temporary image.
Finally we use a read-only view of the temporary image in our x_gradient algorithm.
As the example shows, GIL provides global functions view and const_view that take an image and return a mutable or an immutable view of its pixels.
-->

まず、入力画像と同じサイズの8ビットグレイスケールImageを構築します。
そして、この一時的なImageに色変換Viewから取得した各Pixelの値をコピーします。
最後に、一時的なImageのread-onlyなViewに、私たちが作成した`x_gradient`を適用します。
上記の例で示されているように、GILは、あるImageを引数に取り、そのPixelを指し示すmutableなViewとimmutableなViewを返す、グローバル関数`view`と`const_view`を提供します。

<!--
Creating a generic version of the above is a bit trickier:
-->

上記のコードのジェネリックバージョンは、少々トリッキーです。

{% highlight C++ %}

template <typename SrcView, typename DstView>
void x_luminosity_gradient(const SrcView& src, const DstView& dst) {
    typedef typename channel_type<DstView>::type d_channel_t;
    typedef typename channel_convert_to_unsigned<d_channel_t>::type channel_t;
    typedef pixel<channel_t, gray_layout_t>  gray_pixel_t;
    typedef image<gray_pixel_t, false>       gray_image_t;

    gray_image_t ccv_image(src.dimensions());
    copy_pixels(color_converted_view<gray_pixel_t>(src), view(ccv_image));
    x_gradient(const_view(ccv_image), dst);
}

{% endhighlight %}

<!--
First we use the channel_type metafunction to get the channel type of the destination view.
A metafunction is a function operating on types.
In GIL metafunctions are structs which take their parameters as template parameters and return their result in a nested typedef called type.
In this case, channel_type is a unary metafunction which in this example is called with the type of an image view and returns the type of the channel associated with that image view.
-->

まず、出力ViewのChannel型を取得するために`channel_type`というメタ関数を使用します。
メタ関数とは、型を扱う関数です。
GILのメタ関数は、テンプレートのパラメータを自身のパラメータとして取り、ネストされた`typedef`で定義された型を返します。
今回のケースで言えば、上記の例の中の`channel_type`は、Image Viewの型を引数に取り、そのImage Viewに関連づけられているChannel型を返す、ひとつの引数を取るメタ関数です。

<!--
GIL constructs that have an associated pixel type, such as pixels, pixel iterators, locators, views and images, all model PixelBasedConcept, which means that they provide a set of metafunctions to query the pixel properties, such as channel_type, color_space_type, channel_mapping_type, and num_channels.
-->

Pixel、Pixel Iterator、Locator、View、Imageといった関連づけられたPixel型をもつGILコンストラクトは全て`PixelBasedConcept`に基づいたModelであり、このことは、`channel_type`、`color_space_type`、`channel_mapping_type`、`num_channels`といったPixelのプロパティの問い合わせを行うメタ関数一式をGILが提供するということを意味します。

<!--
After we get the channel type of the destination view, we use another metafunction to remove its sign (if it is a signed integral type) and then use it to generate the type of a grayscale pixel.
From the pixel type we create the image type.
GIL's image class is templated over the pixel type and a boolean indicating whether the image should be planar or interleaved.
Single-channel (grayscale) images in GIL must always be interleaved.
There are multiple ways of constructing types in GIL.
Instead of instantiating the classes directly we could have used type factory metafunctions.
The following code is equivalent:
-->

出力ViewのChannel型を取得した後、(それが符号付き整数型だった場合を考慮して)符号を取り除くメタ関数を用い、それから、グレイスケールPixelの型をつくるためにそのChannel型を使用します。
出来上がったPixelの型からImageの型を作成します。
GILのImageクラスは、Pixelの型と画像がプラナー形式なのかを示すboolean(インタリーブ形式の場合、`false`を指定)でテンプレート化されています。
GILの単Channel (グレイスケール) Imageは、常にインタリーブ形式でなければなりません。
GILで型を構築する方法はいくつかあります。
直接的にクラスを具体化する代わりに、型生成メタ関数を使用することもできます。
上記のコードと次に示すコードは等価です。

{% highlight C++ %}

template <typename SrcView, typename DstView>
void x_luminosity_gradient(const SrcView& src, const DstView& dst) {
    typedef typename channel_type<DstView>::type d_channel_t;
    typedef typename channel_convert_to_unsigned<d_channel_t>::type channel_t;
    typedef typename image_type<channel_t, gray_layout_t>::type gray_image_t;
    typedef typename gray_image_t::value_type gray_pixel_t;

    gray_image_t ccv_image(src.dimensions());
    copy_and_convert_pixels(src, view(ccv_image));
    x_gradient(const_view(ccv_image), dst);
}

{% endhighlight %}

<!--
GIL provides a set of metafunctions that generate GIL types - image_type is one such meta-function that constructs the type of an image from a given channel type, color layout, and planar/interleaved option (the default is interleaved).
There are also similar meta-functions to construct the types of pixel references, iterators, locators and image views.
GIL also has metafunctions derived_pixel_reference_type, derived_iterator_type, derived_view_type and derived_image_type that construct the type of a GIL construct from a given source one by changing one or more properties of the type and keeping the rest.
-->

GILは、GILの型を生成するメタ関数一式を提供します。
`image_type`は、与えられたChannel型、Color Layout、プラナー形式/インタリーブ形式を決めるオプション(インタリーブ形式がデフォルトに指定されています)からImageの型を構築するメタ関数です。
GILは、ひとつ以上のプロパティを変更した(それ以外は元のままの)与えられた型からGILコンストラクトの型を構築する`derived_pixel_reference_type`、`derived_iterator_type`、`derived_view_type`、`derived_image_type`といったメタ関数ももっています。

<!--
From the image type we can use the nested typedef value_type to obtain the type of a pixel.
GIL images, image views and locators have nested typedefs value_type and reference to obtain the type of the pixel and a reference to the pixel.
If you have a pixel iterator, you can get these types from its iterator_traits.
Note also the algorithm copy_and_convert_pixels, which is an abbreviated version of copy_pixels with a color converted source view.
-->

Imageの型からは、Pixelの型を取得するために、ネストされた`typedef`である`value_type`を使うことが出来ます。
GILのImage、Image View、Locatorは、ネストされた`typedef`である`value_type`をもっており、Pixelの型とそのPixelへの参照を取得するための参照をもっています。
あるPixel Iteratorをもっている場合には、その`iterator_traits`からImageの型やImage Viewの型やLocatorの型を得ることができます。
色変換を伴った`copy_pixels`の短縮表記バージョンである、`copy_and_converted_pixels`アルゴリズムにも注目しておいてください。

<!--
Virtual Image Views
-->

### <a name="section_02_10"> Virtual Image View

<!--
So far we have been dealing with images that have pixels stored in memory.
GIL allows you to create an image view of an arbitrary image, including a synthetic function.
To demonstrate this, let us create a view of the Mandelbrot set.
First, we need to create a function object that computes the value of the Mandelbrot set at a given location (x,y) in the image:
-->

ここまでは、メモリ上に保存されたPixelをもつ画像を扱ってきました。
GILは、合成関数を含む任意の画像についてのImage Viewを作成することが出来ます。
これを実演するために、マンデルブロ集合の画像を作ってみましょう。
最初に、与えられた画像中の座標(x,y)におけるマンデルブロ集合の値を計算する関数オブジェクトを作成する必要があります。

{% highlight C++ %}

// models PixelDereferenceAdaptorConcept
struct mandelbrot_fn {
    typedef point2<ptrdiff_t>   point_t;

    typedef mandelbrot_fn       const_t;
    typedef gray8_pixel_t       value_type;
    typedef value_type          reference;
    typedef value_type          const_reference;
    typedef point_t             argument_type;
    typedef reference           result_type;
    BOOST_STATIC_CONSTANT(bool, is_mutable=false);

    mandelbrot_fn() {}
    mandelbrot_fn(const point_t& sz) : _img_size(sz) {}

    result_type operator()(const point_t& p) const {
        // normalize the coords to (-2..1, -1.5..1.5)
        double t=get_num_iter(point2<double>(p.x/(double)_img_size.x*3-2, p.y/(double)_img_size.y*3-1.5f));
        return value_type((bits8)(pow(t,0.2)*255));   // raise to power suitable for viewing
    }
private:
    point_t _img_size;

    double get_num_iter(const point2<double>& p) const {
        point2<double> Z(0,0);
        for (int i=0; i<100; ++i) {     // 100 iterations
            Z = point2<double>(Z.x*Z.x - Z.y*Z.y + p.x, 2*Z.x*Z.y + p.y);
            if (Z.x*Z.x + Z.y*Z.y > 4)
                return i/(double)100;
        }
        return 0;
    }
};

{% endhighlight %}

<!--
We can now use GIL's virtual_2d_locator with this function object to construct a Mandelbrot view of size 200x200 pixels:
-->

ここで、200x200 PixelのマンデルブロViewを構築するために、GILの`virtual_2d_locator`と共にこの関数オブジェクトを用います。

{% highlight C++ %}

typedef mandelbrot_fn::point_t point_t;
typedef virtual_2d_locator<mandelbrot_fn,false> locator_t;
typedef image_view<locator_t> my_virt_view_t;

point_t dims(200,200);

// Construct a Mandelbrot view with a locator, taking top-left corner (0,0) and step (1,1)
my_virt_view_t mandel(dims, locator_t(point_t(0,0), point_t(1,1), mandelbrot_fn(dims)));

{% endhighlight %}

<!--
We can treat the synthetic view just like a real one.
For example, let's invoke our x_gradient algorithm to compute the gradient of the 90-degree rotated view of the Mandelbrot set and save the original and the result:
-->

合成関数によるViewは実態をもつViewと同じように扱うことが出来ます。
例として、マンデルブロ集合を90度回転させたViewのgradientを計算するために私たちの`x_gradient`アルゴリズムを実行してみましょう。

{% highlight C++ %}

gray8s_image_t img(dims);
x_gradient(rotated90cw_view(mandel), view(img));

// Save the Mandelbrot set and its 90-degree rotated gradient (jpeg cannot save signed char; must convert to unsigned char)
jpeg_write_view("mandel.jpg",mandel);
jpeg_write_view("mandel_grad.jpg",color_converted_view<gray8_pixel_t>(const_view(img)));

{% endhighlight %}

<!--
Here is what the two files look like:
-->

ふたつのファイルがどのようになったかを示します。

![マンデルブロ集合](http://hironishihara.github.com/GILTutorial-ja/src/img/mandel.jpg "マンデルブロ集合")


<!--
Run-Time Specified Images and Image Views
-->

### <a name="section_02_11"> 実行時に型を指定するImageとImage View

<!--
So far we have created a generic function that computes the image gradient of a templated image view.
Sometimes, however, the properties of an image view, such as its color space and channel depth, may not be available at compile time.
GIL's dynamic_image extension allows for working with GIL constructs that are specified at run time, also called variants.
GIL provides models of a run-time instantiated image, any_image, and a run-time instantiated image view, any_image_view.
The mechanisms are in place to create other variants, such as any_pixel, any_pixel_iterator, etc.
Most of GIL's algorithms and all of the view transformation functions also work with run-time instantiated image views and binary algorithms, such as copy_pixels can have either or both arguments be variants.
-->

ここまで、テンプレート化されたImage Viewのgradient画像を算出するジェネリック関数を作成してきました。
しかし、Color SpaceやChannel深度などといったImage Viewのプロパティをコンパイル時に利用できない場合もあるかもしれません。
GILの`dynamic_image` extensionは、variantとも呼ばれる実行時に型が決まるGILコンストラクトと共に動作することを可能にします。
GILは、実行時に型が決まる即席のImageのModelである`any_image`と、実行時に型が決まる即席のImage ViewのModelである`any_image_view`を提供します。
このようなメカニズムは、`any_pixel`や`any_pixel_iterator`など他のvariantを作成するためのものです。
`copy_pixels`が2個の引数の片方もしくは両方にvariantを取ることが可能であるように、ほとんどのGILアルゴリズムと全てのView変換関数も、実行時に型が決まる即席のImage Viewや実行時に型が決まる即席のアルゴリズムと共に動作することが可能です。

<!--
Lets make our x_luminosity_gradient algorithm take a variant image view.
For simplicity, let's assume that only the source view can be a variant.
(As an example of using multiple variants, see GIL's image view algorithm overloads taking multiple variants.)
-->

variantなImage Viewをとる`x_luminosity_gradient`アルゴリズムを作ってみましょう。
簡単のために、入力Viewだけvariantにすることができるようにしてみましょう。
(複数のvariantを使った例は、複数のvariantを引数に取るようにオーバーロードしたGILのImage Viewアルゴリズムを参照ください。)

<!--
First, we need to make a function object that contains the templated destination view and has an application operator taking a templated source view:
-->

まず始めに、テンプレート化された出力Viewをもち、テンプレート化された入力Viewを引数に取るアプリケーションオペレータをもつ、関数オブジェクトを作成する必要があります。

{% highlight C++ %}

#include <boost/gil/extension/dynamic_image/dynamic_image_all.hpp>

template <typename DstView>
struct x_gradient_obj {
    typedef void result_type;        // required typedef

    const DstView& \_dst;
    x_gradient_obj(const DstView& dst) : \_dst(dst) {}

    template <typename SrcView>
    void operator()(const SrcView& src) const { x_luminosity_gradient(src, \_dst); }
};

{% endhighlight %}

<!--
The second step is to provide an overload of x_luminosity_gradient that takes image view variant and calls GIL's apply_operation passing it the function object:
-->

つづいての手順は、Image View variantを引数に取り、それを関数オブジェクトに渡すGILの`apply_operation`を呼び出す、`x_luminosity_gradient`のオーバーロードを提供することです。

{% highlight C++ %}

template <typename SrcViews, typename DstView>
void x_luminosity_gradient(const any_image_view<SrcViews>& src, const DstView& dst) {
    apply_operation(src, x_gradient_obj<DstView>(dst));
}

{% endhighlight %}

<!--
any_image_view<SrcViews> is the image view variant.
It is templated over SrcViews, an enumeration of all possible view types the variant can take.
src contains inside an index of the currently instantiated type, as well as a block of memory containing the instance.
apply_operation goes through a switch statement over the index, each case of which casts the memory to the correct view type and invokes the function object with it.
Invoking an algorithm on a variant has the overhead of one switch statement.
Algorithms that perform an operation for each pixel in an image view have practically no performance degradation when used with a variant.
-->

`any_image_view<SrcViews>`は、Image View variantです。
これは、variantが取りうるViewの型を列挙したリストである`SrcViews`でテンプレート化されています。
メモリのブロック内にインスタンスが含まれるのと同じように、`src`には現在インスタンス化されている型のインデクスが含まれます。
`apply_operation`はインデクスによるswitch文判定を実施し、各ケースごとに正しいViewの型へメモリをキャストし、そのViewと共に関数オブジェクトを実行します。
variant上で呼び出されたアルゴリズムは、switch文1個分のオーバーヘッドをもちます。
Image Viewの各Pixelに処理を実行するアルゴリズムは、variantと共に使用した場合にもパフォーマンスの低下はありません。

<!--
Here is how we can construct a variant and invoke the algorithm:
-->

ここで、いかにvariantを構築し、いかにアルゴリズムを呼び出すのかを示します。

{% highlight C++ %}

#include <boost/mpl/vector.hpp>
#include <boost/gil/extension/io/jpeg_dynamic_io.hpp>

typedef mpl::vector<gray8_image_t, gray16_image_t, rgb8_image_t, rgb16_image_t> my_img_types;
any_image<my_img_types> runtime_image;
jpeg_read_image("input.jpg", runtime_image);

gray8s_image_t gradient(runtime_image.dimensions());
x_luminosity_gradient(const_view(runtime_image), view(gradient));
jpeg_write_view("x_gradient.jpg", color_converted_view<gray8_pixel_t>(const_view(gradient)));

{% endhighlight %}

<!--
In this example, we create an image variant that could be 8-bit or 16-bit RGB or grayscale image.
We then use GIL's I/O extension to load the image from file in its native color space and channel depth.
If none of the allowed image types matches the image on disk, an exception will be thrown.
We then construct a 8 bit signed (i.e. char) image to store the gradient and invoke x_gradient on it.
Finally we save the result into another file.
We save the view converted to 8-bit unsigned, because JPEG I/O does not support signed char.
-->

この例の中では、8ビットRGBか16ビットRGBかグレイスケールのImageになることができるImage variantを作成しています。
それから、ファイルに記録されているColor SpaceとChannel深度のまま画像を読み込む、GILのI/O extensionを使っています。
用意したImageの型と読み込んだ画像がいずれも一致しなかった場合には、例外が投げられます。
そして、算出するgradientを記憶するために8ビット符号付き(すなわち、char) Imageを構築し、`x_gradient`を呼び出します。
最後に、その結果をもうひとつのファイルに保存します。
JPEG I/Oが符号付きcharをサポートしていないことから、8ビット符号無しViewに変換してから保存します。

<!--
Note how free functions and methods such as jpeg_read_image, dimensions, view and const_view work on both templated and variant types.
For templated images view(img) returns a templated view, whereas for image variants it returns a view variant.
For example, the return type of view(runtime_image) is any_image_view<Views> where Views enumerates four views corresponding to the four image types.
const_view(runtime_image) returns a any_image_view of the four read-only view types, etc.
-->

`jpeg_read_image`、`dimensions`、`view`、`const_view`といった、テンプレート化された型とvariantな型の両方で動作する関数やメソッドがどのように振る舞うか注目しましょう。
variantなImageであれば`view(img)`はvariantなViewを返す一方で、テンプレート化されたImageであれば`view(img)`はテンプレート化されたViewを返します。
例を挙げると、`view(runtime_image)`の戻り値の型は`any_image_view<Views>`です。(ここの`Views`は上記4種類のImageに対応する4種類のViewを並べたリストです。)
また、`const_view(runtime_image)`は、4種類のread-onlyなViewの型による`any_image_view`を返します。

<!--
A warning about using variants: instantiating an algorithm with a variant effectively instantiates it with every possible type the variant can take.
For binary algorithms, the algorithm is instantiated with every possible combination of the two input types!
This can take a toll on both the compile time and the executable size.
-->

variantの使用に関して、ひとつ注意があります。
ひとつのvariantを引数に取るアルゴリズムをインスタンス化するとき、アルゴリズムとvariantが取りうるあらゆる型との組み合わせがインスタンス化されます。
ふたつのvariantを引数に取るアルゴリズムでは、ふたつの入力型が取りうる全ての組み合わせでアルゴリズムがインスタンス化されます！
これは、コンパイル時間と実行ファイルのサイズの両方に多大な影響を与える可能性があります。
