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

入力は8-bit符号なしグレイスケール画像、出力は8-bit符号ありグレイスケール画像として始めましょう。
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

`gray8c_view_t`は入力画像の型です。Pixelがread-only ("c"で表されています)の8-bitグレイスケールViewです。
出力は8-bit符号あり("s"で表されています)整数型のグレイスケール画像です。
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
上記のコードは読みやすいのですが、それほど高速ではありません。というのも、実行ファイル内の`operator()`が2次元格子上の座標を算出する際に和と積を用いているからです。
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
Image Viewと同じように、Locatorは現在位置を基準にした相対的オフセットで指定されるPixelへの参照を返す`operator()`を提供します。
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

### ジェネリックなGILアルゴリズムの作成

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

```cpp
template <typename SrcView, typename DstView>
void x_gradient(const SrcView& src, const DstView& dst) {
    gil_function_requires<ImageViewConcept<SrcView> >();
    gil_function_requires<MutableImageViewConcept<DstView> >();
    gil_function_requires<ColorSpacesCompatibleConcept<
                                typename color_space_type<SrcView>::type,
                                typename color_space_type<DstView>::type> >();

    ... // compute the gradient
}
```

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

```cpp
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
```

<!--
Having an explicit loop for each channel could be a performance problem.
GIL allows us to abstract out such per-channel operations:
-->

単純な各Channelのループは、パフォーマンスの問題になる可能性があります。
GILは、各Channelの操作を次のように抽象化することができます。

```cpp
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
```

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

```cpp
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
```

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
