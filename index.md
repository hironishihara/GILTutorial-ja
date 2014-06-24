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


# Generic Image Library Tutorial 日本語訳

この文章は[Generic Image Library Tutorial](http://stlab.adobe.com/gil/html/giltutorial.html)の日本語訳です。
この日本語訳は原文の著者や権利者とまったく関係がありません。
また、原文と訳文の正確さに関して、訳者は一切の責任を負いません。

#### 原文:
Generic Image Library Tutorial  
<http://stlab.adobe.com/gil/html/giltutorial.html>

#### 原文のVersion:
2.1

#### 原文のライセンス:
Boost Software License, Version 1.0 (2008)  
MIT License (2005-2007)

#### 訳者:
Hiroaki Nishihara (<hiro.nishihara@gmail.com>)

#### 訳文のLicense:
Boost Software License, Version 1.0

#### 翻訳日時:
2014年6月22日〜

#### Repository:
<https://github.com/hironishihara/GILTutorial-ja>

***

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
ViewのコピーコンストラクションはPixelのディープコピーではありません
Image Viewに付加されたconst性はPixelまで伝播しないので、常にconst参照で使用すべきです。
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
GILのImageはSTLにおける`std::vector`などといったコンテナに相当し、GILのImage ViewはSTLにおけるRange(しばしば、`begin()`と`end()`のようなIteratorの組で表現されています)に対応します。
STLアルゴリズムがRangeの上で動作するのと同じ様に、GILアルゴリズムはImage Viewの上で動作します。

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

このグルーコードはとても高速であり、2つのViewはとても軽量(上記の例では16バイトです)です。
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

Image Viewの`operator(x,y)`を使用して与えられた座標のPixel参照を取得し、その両隣のPixelの差分の1/2をそこに代入します。
`operator()`はグレイスケールPixelの参照を返します。
グレイスケールPixelはそのChannel (`src`における`unsigned char`)と変換可能であり、Channelからのコピーコンストラクションが可能です。(これが可能なのはグレイスケールPixelだけです。)
上記のコードは読みやすいけれど、それほど高速ではありません。というのも、実行ファイル内の`operator()`が2次元格子上の座標を算出する際に和と積を用いているからです。
上記のコードをより高速にしたバージョンを次に示します。

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
GILのIteratorはランダムアクセス走査Iteratorです．
もしランダムアクセスIteratorに詳しくないのであれば、ひとまずポインタだと考えておけば大丈夫でしょう。
実際に、上記の例における2個のIteratorはCポインタであり、`operator[]`はポインタの高速なインデクシングを行う演算子です。

<!--
The code to compute gradient in the vertical direction is very similar:
-->

垂直方向のgradientを計算するコードはとてもよく似ています。

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

各行のループを回すかわりに、各列のループを回してその中で垂直方向に移動するIteratorである`y_iterator`を作成します。
このとき、垂直方向に隣接するPixel間の距離はその画像の1行分のバイト数と等しくなっており、シンプルなポインタを使用することはできません。
ここでGILはサイズが8バイトの特別なステップIterator (Cポインタとステップ幅の値を含んでいます)を使用します。
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
