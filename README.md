# C-Space Placement Visualizer
![License](https://img.shields.io/badge/license-MIT-blue.svg) ![Language](https://img.shields.io/badge/language-JavaScript-yellow.svg)

Robotic Pick & Place における配置（Placing）アルゴリズムの比較シミュレーターです。
従来の「幾何学的探索（Geometric Search）」と、提案手法である「距離場とC-Spaceを用いた探索（Distance Field & C-Space Search）」の挙動の違いを可視化し、デッドスペースの発生推移を検証するために使用します。

## 🔗 Live Demo
<img width="928" height="717" alt="Simulation Demo" src="https://github.com/user-attachments/assets/b4c5b4de-0b2d-4587-9adc-e82585bbdbcf" />

[View Demo on GitHub Pages](https://funmatu.github.io/c-space-placement-visualizer/)


## 📖 Overview

産業用ロボットによるパレタイジングやビンパッキングにおいて、アイテムをどのように配置するかはサイクルタイムと充填率（容積効率）に直結します。本プロジェクトでは、以下の2つのアプローチを比較します。

1. **Current Method (Geometric Search):** 空間の空き領域を探索し、安全マージンが最大となる（広い場所の真ん中）位置に配置するアプローチ。

2. **New Method (Distance Field + C-Space):** 距離変換を用いてConfiguration Space（C-Space）を算出し、デッドスペースが最小となる（隅に詰める）位置に配置するアプローチ。

## 🛠 Algorithms

### 1. Current Method: Geometric / Voxel Search

現在運用されている標準的な手法の簡易モデルです。

* **Concept:**
  * 障害物点群から空き領域（Free Space）を特定し、そこに物体（矩形）が収まるかを判定します。
  * 安全性（壁からのクリアランス）を重視する傾向があります。

* **Process:**
  1. **Grid Scanning:** 配置エリアをグリッド（またはボクセル）として走査します。
  2. **Collision Check:** 各候補点において、物体の矩形領域内に障害物が存在しないかを確認します（単純な包含判定）。
  3. **Selection Strategy:** 衝突しない候補の中で、**「最も壁から遠い点（クリアランス最大）」**または「最初に見つかった点」を選択します。

* **Pros & Cons:**
  * ✅ 実装が直感的。
  * ⚠️ 「広い場所の真ん中」に置く傾向があり、周囲に中途半端な隙間（Dead Space）を生み出しやすい。
  * ⚠️ 充填率（Packing Density）が上がりにくい。

### 2. New Method: Distance Field & C-Space Search

画像処理的アプローチ（距離変換）を応用し、幾何学的に厳密かつ高密度な配置を実現する手法です。

* **Concept:**
  * **Distance Transform (DT):** バイナリマップ上の各画素に対し、最も近い障害物までの距離を計算します。
  * **Configuration Space (C-Space):** ロボット（物体）の中心が物理的に存在可能な領域を算出します。

* **Process:**
  1. **Binary Map Generation:** 障害物（1）と空き空間（0）の二値画像を作成します。
  2. **Distance Transform:** 全画素に対してユークリッド距離（またはマンハッタン距離）変換を適用します。
     * `Map(x, y) = min_distance_to_obstacle(x, y)`
  3. **C-Space Masking:** 物体の半径（`r`）に基づき、配置可能領域（Valid Region）を抽出します。
     * `ValidRegion = { (x,y) | Map(x,y) >= r }`
     * これにより、形状が凹多角形やドーナツ型であっても、物体が内接できる領域が数学的に保証されます。
  4. **Optimization (Dead Space Minimization):** `ValidRegion` 内の座標群から、充填戦略に基づき最適な1点を選択します。
     * Cost Function: `Cost = w1 * x + w2 * y` (例: 左奥から詰める場合)
     * `Target = argmin(Cost) in ValidRegion`

* **Pros & Cons:**
  * ✅ **凹形状・不定形に対応:** 複雑な境界形状でも正確に配置可能。
  * ✅ **デッドスペース最小化:** 安全性を担保したまま、壁際や既存荷物の隣に整列させることが可能。
  * ✅ **高速性:** FFTやGPU並列化（PyTorch/CUDA）との相性が良く、計算量が定数時間またはO(N)に収束する。

## 💻 Technical Details of Simulation

本シミュレーションは、以下の技術スタックで実装されています。

* **Language:** HTML5 / JavaScript (ES6)
* **Rendering:** HTML5 Canvas API
* **Math:**
  * **Distance Transform:** Two-pass algorithm (O(N) approx. Manhattan/Euclidean) implementation in JS.
  * **Heatmap Visualization:** Mapping distance values to RGB color space.

## 🚀 How to Run

1. リポジトリをクローンまたはダウンロードします。
2. `index.html` (または `PlaceAlgorithmComparison.html`) をブラウザで開くだけで実行可能です。
3. Canvas上でマウスドラッグを行い、障害物を描画した後に「配置計算実行」を押してください。

## 📂 Project Structure

```
.
├── index.html # Main simulation logic & UI   
└── README.md # Documentation
```

## 🔍 Future Work (Production Implementation)

実環境（3D点群/ロボット）への適用時は、以下の拡張を想定しています。

* **Input:** Ensensoカメラ等からの3D点群 → 2D高さマップへの投影。
* **Processing:** OpenCV (`cv2.distanceTransform`) または PyTorch (`Conv2d` / FFT) による高速化。
* **Rotation:** テンプレートマッチング（FFT畳み込み）を用いた、複数回転角の同時探索。

---
