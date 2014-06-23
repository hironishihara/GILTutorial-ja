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

## インストール

<!--
The latest version of GIL can be downloaded from GIL's web page, at http://opensource.adobe.com/gil.
GIL is approved for integration into Boost and in the future will be installed simply by installing Boost from http://www.boost.org.
GIL consists of header files only and does not require any libraries to link against.
It does not require Boost to be built.
Including boost/gil/gil_all.hpp will be sufficient for most projects.
-->

最新バージョンのGILは、GILのWebサイト <http://opensource.adobe.com/gil> からダウンロードすることができます。
GILはBoostへの統合が承認されており、近い将来、<http://www.boost.org>からBoostをインストールする際にGILも同時にインストールされるようになるでしょう。
GILはヘッダーファイルだけで構成されており、他のライブラリのリンクは不要です。
また、Boostのビルドも必要ありません。
ほとんどのプロジェクトでは、`boost/gil/gil_all.hpp`をインクルードするだけで十分です。

<!--
Example - Computing the Image Gradient
-->

## 例題 - Gradient画像の算出

<!--
This tutorial will walk through an example of using GIL to compute the image gradients.
We will start with some very simple and non-generic code and make it more generic as we go along.
Let us start with a horizontal gradient and use the simplest possible approximation to a gradient - central difference.
The gradient at pixel x can be approximated with the half-difference of its two neighboring pixels: D[x] = (I[x-1] - I[x+1]) / 2
For simplicity, we will also ignore the boundary cases - the pixels along the edges of the image for which one of the neighbors is not defined.
The focus of this document is how to use GIL, not how to create a good gradient generation algorithm.
-->

このチュートリアルは、Gradient画像を算出するというGILの使用例を通じて進めていくことにしましょう。
まずは極めてシンプルでジェネリックでないコードからスタートし、それを少しずつジェネリックなコードにしていきましょう。
水平方向Gradientから始めることとし、Gradientの最もシンプルな近似と思われる中心差分を使うことにしましょう。
位置xにあるPixelのGradientは、その両隣のPixelの差分の1/2で近似されます。

D[x] = (I[x-1] - I[x+1]) / 2

簡単のために、境界ケース(すなわち、対象のPixelが画像の端にあって隣接Pixelの一報が定義されていないケース)は無視することにしましょう。
この文章のフォーカスは、GILの使い方であり、上質なGradient画像生成アルゴリズムの作り方ではないのです。

<!--
Interface and Glue Code
-->

### インタフェースとグルーコード

<!--
Let us first start with 8-bit unsigned grayscale image as the input and 8-bit signed grayscale image as the output.
Here is how the interface to our algorithm looks like:
-->

8-bit符号なしグレイスケール画像をインプット、8-bit符号ありグレイスケール画像をアウトプットとして始めましょう。
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

`gray8c_view_t`はインプット画像の型です。Pixelがread-only ("c"で表されています)の8-bitグレイスケールViewです。
アウトプットは8-bit符号あり("s"で表されています)整数型のグレイスケール画像です。
GILが定める型の命名規則については、付録を参照ください。

<!--
GIL makes a distinction between an image and an image view.
A GIL image view, is a shallow, lightweight view of a rectangular grid of pixels.
It provides access to the pixels but does not own the pixels. Copy-constructing a view does not deep-copy the pixels.
Image views do not propagate their constness to the pixels and should always be taken by a const reference.
Whether a view is mutable or read-only (immutable) is a property of the view type.

A GIL image, on the other hand, is a view with associated ownership.
It is a container of pixels; its constructor/destructor allocates/deallocates the pixels, its copy-constructor performs deep-copy of the pixels and its operator== performs deep-compare of the pixels.
Images also propagate their constness to their pixels - a constant reference to an image will not allow for modifying its pixels.

Most GIL algorithms operate on image views; images are rarely needed.
GIL's design is very similar to that of the STL.
The STL equivalent of GIL's image is a container, like std::vector, whereas GIL's image view corresponds to STL's range, which is often represented with a pair of iterators.
STL algorithms operate on ranges, just like GIL algorithms operate on image views.

GIL's image views can be constructed from raw data - the dimensions, the number of bytes per row and the pixels, which for chunky views are represented with one pointer.
Here is how to provide the glue between your code and GIL:
-->

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
