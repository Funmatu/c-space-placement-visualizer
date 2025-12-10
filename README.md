# C-Space Placement Visualizer
![License](https://img.shields.io/badge/license-MIT-blue.svg) ![Language](https://img.shields.io/badge/language-JavaScript-yellow.svg) ![Library](https://img.shields.io/badge/Three.js-r160-black)

Robotic Pick & Place における配置（Placing）アルゴリズムの比較シミュレーターです。
従来の「幾何学的探索（Geometric Search）」と、提案手法である「FFTと距離場を用いたハイブリッド探索（FFT & Distance Field Search）」の挙動の違いを可視化し、配置戦略による効率や安全性の変化を検証するために使用します。

## 🔗 Live Demo
<img width="1389" height="835" alt="image" src="https://github.com/user-attachments/assets/273b191b-061b-458d-96e1-428a2103096a" />

[View Demo on GitHub Pages](https://funmatu.github.io/c-space-placement-visualizer/)

## 📖 Overview

産業用ロボットによるパレタイジングやビンパッキングにおいて、アイテムをどのように配置するかはサイクルタイムと充填率（容積効率）に直結します。本プロジェクトでは、以下の2つのアプローチを比較します。

1. **Current Method (Geometric Search):** グリッドスキャンと衝突判定を行い、最初に見つかった空き領域（または特定のルールに基づく場所）に配置するベースライン手法。

2. **New Method (Hybrid: FFT + Distance Field):** 高速フーリエ変換（FFT）を用いて厳密なConfiguration Space（C-Space）を生成し、さらに距離変換（EDT）を適用することで、「安全性」と「充填率」を動的に制御する高度な手法。

## 🛠 Algorithms

### 1. Current Method: Geometric Search

現在運用されている標準的な手法のモデルです。単純なグリッド探索を行います。

* **Process:**
  1. **Grid Scanning:** コンテナ底面をグリッドとして走査します。
  2. **Collision Check:** 各候補点において、物体の矩形領域内に障害物が存在しないかを確認します（単純な包含判定）。
  3. **Selection:** 衝突しない候補が見つかり次第、配置位置として採用します（First Fit）。

* **Characteristics:**
  * ✅ 実装が単純で、障害物が少ない場合は高速。
  * ⚠️ 障害物が増えると計算コストが増加する。
  * ⚠️ 「壁際」や「隙間」を数値的に評価しているわけではないため、最適な詰め込みが保証されない。

### 2. New Method: FFT & Distance Field Search (Hybrid)

画像処理的アプローチを応用し、幾何学的に厳密かつ柔軟な配置を実現する手法です。

* **Core Concept:**
  * **FFT Convolution:** 障害物マップと物体形状（カーネル）の畳み込み積分をFFTで行い、物理的に配置可能な領域（C-Space）をピクセル単位で厳密に特定します。
  * **Euclidean Distance Transform (EDT):** C-Space上の各点に対し、最も近い「衝突境界」までのユークリッド距離を計算し、ヒートマップ化します。

* **Process:**
  1. **C-Space Generation (FFT):** 
     * `OverlapMap = IFFT( FFT(Obstacles) * FFT(ItemKernel) )`
     * これにより、物体の中心が配置可能な領域（Overlap ≈ 0）を高速に算出します。
  2. **Safety Scoring (EDT):** 
     * 配置可能領域に対して距離変換を適用します。
     * `Score(x, y) = Distance_to_Nearest_Wall(x, y)`
  3. **Placement Strategy:** 算出されたスコアマップに基づき、用途に応じた座標を選択します。
     * **Top-Left (Density):** 配置可能な中で、最も左上（原点）に近い座標を選択。 → **充填率優先**
     * **Max Safety (Gap):** 壁からの距離が最大となる座標を選択。 → **安全性優先**

* **Pros:**
  * ✅ **幾何学的厳密性:** ピクセル単位で干渉ゼロを保証。
  * ✅ **戦略の柔軟性:** 同じ計算結果から「詰め込み」も「安全重視」も即座に選択可能。
  * ✅ **可視化:** ヒートマップにより、どこが安全でどこが危険かが直感的に分かる。

## 💻 Technical Details

本シミュレーションは、以下の技術スタックで実装されています。

* **Language:** HTML5 / JavaScript (ES6 Modules)
* **Rendering:** **Three.js (WebGL)** による3D可視化
* **Math:**
  * **FFT:** Optimized 2D Fast Fourier Transform for convolution.
  * **EDT:** Meijster's algorithm for linear-time Euclidean Distance Transform.
  * **Visuals:** Custom shader-like mapping for distance field heatmaps using RGB gradients.

## 🚀 How to Run

1. リポジトリをクローンまたはダウンロードします。
2. `index.html` をブラウザで開くだけで実行可能です（ローカルサーバー推奨）。
3. **操作方法:**
   * **Scene Buttons:** "Random Scene" または "Tight Gap Test" で障害物を生成。
   * **Sliders:** アイテムのサイズ（幅・奥行き）を調整。
   * **Strategy Radio:** "Top-Left"（詰め込み）か "Max Safety"（安全性）を選択。
   * **RUN COMPARISON:** 両アルゴリズムを実行し、結果と処理時間を比較。

## 📂 Project Structure

```
.
├── index.html # Main application logic & UI
└── README.md  # Documentation
```

## 🔍 Future Work (Production Implementation)

実環境（3D点群/ロボット）への適用時は、以下の拡張を想定しています。

* **Input:** Ensenso/Realsense等のデプスカメラからの3D点群 → 2D高さマップへの投影。
* **Processing:** Python (OpenCV / PyTorch / CuPy) によるGPU高速化。
  * `torch.fft.rfft2` や `cv2.distanceTransform` を使用することで、数ミリ秒での計算が可能。
* **Rotation:** テンプレートマッチング（FFT畳み込み）を複数の回転角度に対して並列実行し、最適な角度を探索。

---
