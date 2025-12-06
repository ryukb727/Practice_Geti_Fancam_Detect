| [Korean 🇰🇷](#korean) | [Japanese 🇯🇵](#japanese) | [Geti](#Geti) |
| :---: | :---: | :---: |

</div>

---

<div id="korean">

### 🇰🇷 Korean Version
# 🔬 Intel Geti를 활용한 AI Classification & Detection 실습

본 문서는 **Intel Edge AI SW Academy 과정** 중 진행한  
**Intel Geti 플랫폼 기반 AI 실습(Classification / Object Detection)** 내용을 정리한 것입니다.

본 실습을 통해 모델 학습 전 과정과 함께,  
**데이터의 양·품질·모델 아키텍처 선택이 성능에 미치는 영향**을 분석했습니다.

---

## 🎯 실습 개요 (Overview)

- **플랫폼:** Intel Geti Platform
- **실습 목표**
  - 데이터셋 구성 및 라벨링
  - Classification / Detection 모델 학습 및 검증
  - 데이터·모델 변경에 따른 성능 변화 분석

---

## 🛠 기술 스택 (Tech Stack)

- **Platform:** Intel Geti
- **Task:** Classification / Object Detection
- **Models**
  - EfficientNet-B0 (Classification)
  - EfficientNet-V2-S (Classification)
  - MobileNetV2-AT55 (Detection)
- **Workflow**
  - Dataset 구성 → Labeling → Training → Validation → 성능 비교

---

## 🔎 실습 1. Classification — *Guess Who Am I*

특정 인물 분류(Classification)를 목표로 진행한 실습으로,  
**데이터셋 크기 증가**와 **모델 아키텍처 변경**이 Accuracy에 미치는 영향을 비교했습니다.

<br>

### ✔ 실험 구성 (Experiment Setup)

- **사용 모델**
  - EfficientNet-B0
  - EfficientNet-V2-S

- **변경 요소**
  - 데이터셋 이미지 수 (37 → 67 → 119)
  - 모델 아키텍처 변경 (B0 ↔ V2-S)

> 동일한 데이터 조건에서 모델별 성능 차이를 비교하여  
> 데이터와 모델 선택이 정확도에 미치는 영향을 검증

<br>

### ✔ 실험 결과 ① — EfficientNet-B0

| Version | 데이터 수 | Accuracy |
|--------|----------|----------|
| Version 1 | 37 Images | 56% |
| Version 2 | 67 Images | 89% |
| Version 3 | 119 Images | **75%** |


### ✔ 실험 결과 ② — EfficientNet-V2-S

| Version | 데이터 수 | Accuracy |
|--------|----------|----------|
| Version 1 | 37 Images | 67% |
| Version 2 | 67 Images | 78% |
| Version 3 | 119 Images | **100%** |

<br>

### ✔ 비교 분석

- 데이터 수 증가에 따라 두 모델 모두 Accuracy 향상 경향 확인
- **EfficientNet-V2-S는 데이터 증가에 따라 성능이 안정적으로 상승**
- EfficientNet-B0는 Version 3에서 Accuracy 하락 →  
  **데이터 구성의 편향으로 인한 일반화 한계 가능성** 확인
  

### ✅ Classification 결론

> Classification 성능 향상을 위해서는  
> **충분한 데이터 확보 + 문제 특성에 맞는 모델 선택**이 핵심임을 확인했습니다.

---

## 🎥 실습 2. Object Detection — *Fan Cam*

영상 내 특정 인물의 얼굴 영역을 탐지하는 Object Detection 실습으로,  
**입력 데이터 해상도와 품질, Bounding Box 설계 Detection 성능에 미치는 영향**을 분석했습니다.

<br>

### ✔ 실험 구성 (Experiment Setup)

- **사용 모델:** MobileNetV2-AT55  
- **목표:** 영상 내 특정 인물의 얼굴 영역 탐지  
- **변경 요소**
  - 입력 영상 해상도 (360p → 1080p)
  - **데이터셋 구성**
    - 단일 직캠 캡처 이미지 →  
      의상 색·배경 색이 다른 여러 직캠 캡처 이미지 추가
  - **Bounding Box 범위**
    - 전신 → 얼굴 영역으로 축소

<br>

### 🚨 문제 상황 (Version 1 ~ Version 3)

#### ✔ 개선 과정

- 저해상도(360p) 영상 기반 데이터셋으로 학습
- 동일 조건에서 **고해상도 영상(1080p)** 데이터셋으로 변경
- 해상도와 데이터 수만 개선한 상태에서 모델 재학습

#### ✔ 정량 평가 결과

| 조건 | Detection Score |
|-----|----------------|
| 360p 영상 | 16% |
| 1080p 영상 | **81%** |

- 수치상 Detection Score 약 **400% 이상 향상**

<br>

### ⚠️ 실제 성능 검증 실패

- Version 3 모델을 실제 성능 검사(Test)에 적용한 결과  
  → **Inference Score: 0점**
- 학습 데이터와 유사한 영상에서는 탐지 가능  
- 그러나 **조금이라도 다른 영상 입력 시 전혀 탐지되지 않는 문제 발생**
  → 정량 지표와 실제 일반화 성능 간의 큰 괴리 확인

<br>

### 🧠 원인 분석 (Root Cause Analysis)

#### 1. 데이터 편향 및 Overfitting

- 아이돌 음악방송 특유의 **화려한 배경**
- 컨셉·의상 개성이 강한 대상의 특성
- 단일 직캠 기반 데이터셋 구성

→ 특정 영상 스타일에 과도하게 최적되어  
**학습 데이터 외 환경에 취약한 Overfitting 발생**

#### 2. Bounding Box 설정 문제

- Version 1~3:
  - 전신 영역 Bounding Box 사용
  - 얼굴 탐지 목적 대비 불필요한 특징 다수 포함
- 1080p 영상이라도 얼굴 영역의 실질 해상도는 낮아  
  **Bounding Box 크기와 유효 해상도가 Detection 성능에 직접적 영향**

<br>

### 🔧 개선 시도 (Version 4)

- **데이터 다양성 확보**
  - 의상·배경이 다른 직캠 영상 추가
- **Bounding Box 재설계**
  - 전신 → 얼굴 중심으로 범위 축소
- 단순 Score 상승이 아닌  
  **실제 성능 검사 통과를 목표로 방향 전환**

<br>

### ✅ 최종 결과 (Version 4)

| 항목 | 결과 |
|----|----|
| Detection Score | **78%** |
| Inference Score | **95%** |

- 수치상 Detection Score는 Version 3 대비 **3%p 감소**
- 그러나 실제 검증에서는
  - 안정적인 탐지 성능 확보
  - 일반화 성능 대폭 개선

<br>

### ✔ 비교 분석

- Version 1~3:
  - Detection Score는 상승했으나 실제 성능 검증 실패
- Version 4:
  - Detection Score는 소폭 하락했으나, **일관된 탐지 성능 확보**
- Detection에서는
  **수치 지표(Detection Score)보다 데이터 다양성과 일반화 성능이 더 중요함을 확인**


### ✅ Detection 결론

> Object Detection 성능은  
> **해상도·데이터 다양성·Bounding Box 설계가 함께 작용**하며,  
> **정량 지표보다 실제 성능 검증이 더 중요함**을 체감했습니다.

---

## 📝 실습을 통해 배운 점

- **Data-Centric AI 관점 강화**
  - 모델 변경 없이도 데이터 설계 개선만으로 성능이 크게 달라짐을 경험
- **Classification vs Detection 차이 명확화**
  - Classification: 데이터 수 + 모델 아키텍처 중요
  - Detection: 데이터 품질·다양성·Bounding Box 설계 중요
- **정량 지표의 한계 인식**
  - 학습 Score ≠ 실제 성능
- **Edge AI 실무 관점 확보**
  - 배포 환경에서의 입력 조건 변화가 성능에 미치는 영향 이해

---


<div align="center">
<a href="#japanese">⬇️ 日本語バージョンへ移動 (Go to Japanese Version) ⬇️</a>
</div>

</div>

---

<div id="japanese">

### 🇯🇵 Japanese Version
# 🔬 Intel Getiを活用した AI 分類・検出実習

本ドキュメントは、**Intel Edge AI SW Academy コース**にて実施した  
**Intel Geti プラットフォームを用いた AI 実習（分類 / オブジェクト検出）**の内容をまとめたものです。

本実習では、モデル学習の一連の流れを通じて、  
**データの量・品質・モデルアーキテクチャの選択が性能に与える影響**を分析しました。

---

## 🎯 実習概要 (Overview)

- **プラットフォーム:** Intel Geti Platform  
- **実習目標**
  - データセットの構築およびラベリング
  - 分類 / 検出モデルの学習・検証
  - データおよびモデル変更に伴う性能変化の分析

---

## 🛠 技術スタック (Tech Stack)

- **Platform:** Intel Geti  
- **Task:** Classification / Object Detection  
- **Models**
  - EfficientNet-B0（Classification）
  - EfficientNet-V2-S（Classification）
  - MobileNetV2-AT55（Detection）
- **Workflow**  
  データセット構築 → ラベリング → トレーニング → 検証 → 性能比較

---

## 🔎 実習 1. 分類 (Classification) — *Guess Who Am I*

特定人物の分類を目的とした実習で、  
**データセットサイズの増加**および**モデルアーキテクチャの違い**が Accuracy に与える影響を比較しました。

<br>

### ✔ 実験構成 (Experiment Setup)

- **使用モデル**
  - EfficientNet-B0
  - EfficientNet-V2-S

- **変更要素**
  - データセット画像数（37 → 67 → 119）
  - モデルアーキテクチャの変更（B0 ↔ V2-S）

同一データ条件下でモデルごとの性能差を比較し、  
データ設計およびモデル選択が性能に与える影響を検証しました。

<br>

### ✔ 実験結果① — EfficientNet-B0

| Version | データ数 | Accuracy |
|--------|----------|----------|
| Version 1 | 37 Images | 56% |
| Version 2 | 67 Images | 89% |
| Version 3 | 119 Images | 75% |


### ✔ 実験結果② — EfficientNet-V2-S

| Version | データ数 | Accuracy |
|--------|----------|----------|
| Version 1 | 37 Images | 67% |
| Version 2 | 67 Images | 78% |
| Version 3 | 119 Images | **100%** |

<br>

### ✔ 比較分析

- データ数の増加に伴い、両モデルとも Accuracy は全体的に向上
- **EfficientNet-V2-S** はデータ増加に対して安定した性能向上を示し、問題特性に適合
- **EfficientNet-B0** は Version 3 にて Accuracy が低下  
  → データ構成の偏りや一般化性能の限界が要因として考えられる


### ✅ 分類（Classification）の結論

分類性能の向上には、  
**十分なデータ量の確保**と**タスク特性に適したモデル選択**が重要であることを確認しました。

---

## 🎥 実習 2. オブジェクト検出 (Object Detection) — *Fan Cam*

映像内の特定人物の**顔領域検出**を目的とした実習で、  
**入力データの解像度・多様性・バウンディングボックス設計**が検出性能に与える影響を分析しました。

<br>

### ✔ 実験構成 (Experiment Setup)

- **使用モデル:** MobileNetV2-AT55  
- **目的:** 映像内の特定人物の顔領域検出  
- **主な変更要素**
  - 入力映像解像度（360p → 1080p）
  - データセット構成  
    - 単一ファンカム映像からのキャプチャ  
    - 衣装・背景色が異なる複数ファンカム映像を追加
  - バウンディングボックス範囲  
    - 全身 → 顔領域へ変更

<br>

### 🚨 問題発生と一次改善（Version 1〜3）

#### ✔ 改善プロセス

- 360p 映像を用いたデータセットで学習
- 同一条件下で 1080p 映像データセットへ変更
- 解像度およびデータ数のみを改善した状態で再学習を実施

#### ✔ 定量評価結果

| 条件 | Detection Score |
|-----|----------------|
| 360p 映像 | 16% |
| 1080p 映像 | **81%** |

- 定量指標上では約 **400%以上の性能向上**を確認

<br>

### ⚠️ 実運用テストでの問題

- Version 3 モデルを実際の検証環境で評価した結果  
  → **Inference Score: 0 点**
- 学習データに類似した映像以外では検出不能
  → 定量指標と実際の一般化性能との乖離を確認

<br>

### 🧠 原因分析 (Root Cause Analysis)

**① データ偏りによる過学習（Overfitting）**

- 音楽番組特有の派手な背景
- キャラクター性・衣装変化が大きい対象特性
- 単一ファンカム基準のデータ構成

→ 特定映像に過度に適応し、汎化性能が大きく低下

**② バウンディングボックス設計の問題**

- Version 1〜3 では全身領域を検出対象と設定
- 顔検出目的に対して不要な特徴量が含まれていた
- 1080p 映像でも、顔領域の実質的な解像度不足が影響

<br>

### 🔧 改善施策（Version 4）

- 衣装・背景が異なるファンカム映像を追加し、データ多様性を確保
- バウンディングボックスを **顔中心** に設計変更
- 目標をスコア向上から **実用的な性能検証通過**へ変更

<br>

### ✅ 最終結果（Version 4）

| 項目 | 結果 |
|----|----|
| Detection Score | 78% |
| Inference Score | **95%** |

- 定量 Detection Score は Version 3 比で 3%p 低下
- しかし実運用環境では
  - 安定した検出性能を確保
  - 一般化性能が大幅に改善

<br>

### ✔ 比較分析
- Version 1〜3:
  - Detection Scoreは上昇したが、実際の性能検証に失敗
- Version 4:
  - Detection Scoreは小幅に下落したが、一貫した検出性能を確保
- 検出においては、Detection Scoreよりもデータ多様性と一般化性能がより重要であることを確認

### ✅ 検出（Detection）の結論

オブジェクト検出では、  
**解像度・データ多様性・バウンディングボックス設計**が相互に影響し合い、  
定量指標だけでなく、**実際の性能検証が重要**であることを学びました。

---

## 📝 実習を通じて学んだこと (What I Learned)

- モデル変更よりも **データ設計の重要性**を強く認識
- 分類と検出における設計観点の違いを理解
  - Classification：データ量 + モデル構造
  - Detection：データ品質・多様性・検出範囲設計
- 学習スコアは実性能を保証しないという現実的な知見を獲得
- Edge AI 環境における入力条件変動の影響を実感

---

<div align="center">
<a href="#korean">⬆️ 한국어 버전으로 돌아가기 (Go back to Korean Version) ⬆️</a>
</div>

</div>

---

<div align="center">
<a href="#Geti">⬇️ Go to Geti README.md ⬇️</a>
</div>

</div>

---

<div id="Geti">

# Code deployment
## Table of contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation](#Installation)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [Package contents](#package-contents)


## Introduction

This code deployment .zip archive contains:

1. Inference model(s) for your Intel® Geti™ project.

2. A sample image or video frame, exported from your project.   

3. A very simple code example to get and visualize the result of inference for your 
   project, on the sample image.
   
4. Jupyter notebooks with instructions and code for running inference locally for your project.

The deployment holds one model for each task in your project, so if for example 
you created a deployment for a `Detection -> Classification` project, it will consist of
both a detection, and a classification model. The Intel® Geti™ SDK is used to run 
inference for all models in the project's task chain.

This README describes the steps required to get the code sample up and running on your 
machine.

## Prerequisites

- [Python](https://www.python.org/downloads/) (>=3.9, <=3.12)

## Installation

1. Install [prerequisites](#prerequisites). You may also need to 
   [install pip](https://pip.pypa.io/en/stable/installation/). For example, on Ubuntu 
   execute the following command to install Python and pip:

   ```
   sudo apt install python3-dev python3-pip
   ```
   If you already have installed pip before, make sure it is up to date by doing:

   ```
   pip install --upgrade pip
   ```

2. Create a clean virtual environment: <a name="virtual-env-creation"></a>

   One of the possible ways for creating a virtual environment is to use `virtualenv`:

   ```
   python -m pip install virtualenv
   python -m virtualenv <directory_for_environment>
   ```

   Before starting to work inside the virtual environment, it should be activated:

   On Linux and macOS:

   ```
   source <directory_for_environment>/bin/activate
   ```

   On Windows:

   ```
   .\<directory_for_environment>\Scripts\activate
   ```

   Please make sure that the environment contains 
   [wheel](https://pypi.org/project/wheel/) by calling the following command:

   ```
   python -m pip install wheel
   ```

   > **NOTE**: On Linux and macOS, you may need to type `python3` instead of `python`.

3. In your terminal, navigate to the `example_code` directory in the code deployment 
   package.

4. Install requirements in the environment:

   ```
   python -m pip install -r requirements.txt
   ```

5. (Optional) Install the requirements for running the `demo_notebook.ipynb` Juypter notebook:

   ```
   python -m pip install -r requirements-notebook.txt
   ```
   
## Usage
### Local inference
Both `demo.py` script and the `demo_notebook.ipynb` notebook contain a code sample for:

1. Loading the code deployment (and the models it contains) into memory.

2. Loading the `sample_image.jpg`, which is a random image taken from the project you 
   deployed. 

3. Running inference on the sample image.

4. Visualizing the inference results.

### Inference with OpenVINO Model Server
Inference with OpenVINO Model Server (OVMS) is deprecated in Intel® Geti™ SDK.

To use OVMS, create a new model deployment in Intel® Geti™ and select "OpenVINO Model Server deployment". 
After downloading the deployment package, follow the included README instructions to run OVMS.

### Running the demo script

In your terminal:

1. Make sure the virtual environment created [above](#virtual-env-creation) is activated.

2. Make sure you are in the `example_code` directory in your terminal.

3. Run the demo using:
  
   ```
   python demo.py
   ```

The script will run inference on the `sample_image.jpg`. A window will pop up that 
displays the image, and the results of the inference visualized on top of it.

### Running the demo notebooks

In your terminal:

1. Make sure the virtual environment created [above](#virtual-env-creation) is activated.

2. Make sure you are in the `example_code` directory in your terminal.

3. Start JupyterLab using:
   
   ```
   jupyter lab
   ```
   
4. This should launch your web browser and take you to the main page of JupyterLab.

Inside JuypterLab:

5. In the sidebar of the JupyterLab interface, double-click on `demo_notebook.ipynb` open one of the notebooks.
   
6. Execute the notebook cell by cell to view the inference results. 


> **NOTE** The `demo_notebook.ipynb` is a great way to explore the `AnnotationScene` 
> object that is returned by the inference. The demo code only has very basic 
> visualization functionality, which may not be sufficient for all use case. For 
> example if your project contains many labels, it may not be able to visualize the 
> results very well. In that case, you should build your own visualization logic 
> based on the `AnnotationScene` returned by the `deployment.infer()` method.

## Troubleshooting

1. If you have access to the Internet through a proxy server only, please use pip 
   with a proxy call as demonstrated by the command below:

   ```
   python -m pip install --proxy http://<usr_name>:<password>@<proxyserver_name>:<port#> <pkg_name>
   ```

2. If you use Anaconda as environment manager, please consider that OpenVINO has 
   limited [Conda support](https://docs.openvino.ai/2021.4/openvino_docs_install_guides_installing_openvino_conda.html). 
   It is still possible to use `conda` to create and activate your python environment, 
   but in that case please use only `pip` (rather than `conda`) as a package manager 
   for installing packages in your environment.

3. If you have problems when you try to use the `pip install` command, please update 
   pip version as per the following command:
   ```
   python -m pip install --upgrade pip
   ```

## Package contents

The code deployment files are structured as follows:

- deployment
    - `project.json`
    - "<title of task 1>"  
        - model
          - `model.xml`
          - `model.bin`
          - `config.json`
        - python
          - model_wrappers
            - `__init__.py`
            - model_wrappers required to run demo
          - `README.md`
          - `LICENSE`
          - `demo.py`
          - `requirements.txt`
    - "<title of task 2>" (Optional)
        - ...
- example_code
    - `demo.py`
    - `demo_notebook.ipynb`
    - `demo_ovms.ipynb`  
    - `README.md`
    - `requirements.txt`
    - `requirements-notebook.txt`
- `sample_image.jpg`
- `LICENSE`

---

<div align="center">
<a href="#korean">⬆️ 한국어 버전으로 돌아가기 (Go back to Korean Version) ⬆️</a>
</div>

</div>

---
