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
