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

* インストール
* 例題 - Gradient画像の算出
    * インタフェースとグルーコード
    * はじめてのインプリメンテーション
    * Locatorの使い方
    * ジェネリックなGILアルゴリズムの作成
    * Image Viewの変換
    * 1次元Pixel Iterator
    * STL-Styleアルゴリズム
    * 色変換
    * Image
    * Virtual Image View
    * 実行時に型を指定するImageとImage View
    * まとめ
* 付録
    * GILが定める型の命名規則
