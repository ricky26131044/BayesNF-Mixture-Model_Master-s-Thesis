# 應用結合貝氏神經場與混合模型於肺癌死亡率之時空分析
**Spatio-Temporal Analysis of Lung Cancer Mortality Rates Using Bayesian Neural Fields with Mixture Models**

 **國立成功大學 統計學研究所 碩士論文 (2026)**  
 **作者：** 張芮萁 (Jui-Chi Chang)  
 **指導教授：** 李國榮 博士 (Dr. Kuo-Jung Lee)

---

##  專案簡介 (Introduction)

肺癌長年位居台灣癌症死因首位。在鄉鎮層級的肺癌死亡率資料中，存在著高度複雜的統計特性：死亡率為受限於 $[0,1]$ 的比例資料，且伴隨極嚴重的「零值膨脹 (Zero-Inflation)」與「右端邊界集中」現象。傳統基於高斯過程 (Gaussian Process) 的時空模型在處理長年期、大樣本資料時面臨龐大的運算負擔 $\mathcal{O}(N^3)$，且單一常態誤差假設難以準確擬合此類特殊分布。

本專案提出一基於 **貝氏神經場 (Bayesian Neural Fields, BayesNF)** 框架之「膨脹常態混合模型 (Inflated-Normal Mixture Model)」。模型不依賴預先定義的共變異核函數 (Covariance Kernel)，透過三輸出頭架構同步學習時空潛在平均場 ($F_{it}$)、膨脹發生機率 ($p_{it}$) 與異質性變異數 ($\sigma_{it}$)，有效分離固定邊界事件與連續風險波動，並結合迴歸模型探討各項環境與臨床協變數對肺癌死亡率的真實影響。

##  研究流程與方法 (Methodology)

本專案之程式碼與分析流程涵蓋以下四大核心步驟：

1. **資料前處理與邊界值校正**
   * 採用 Complementary log-log (cloglog) 轉換，將 $[0,1]$ 比例資料映射至連續實數空間。
   * 針對右端極端觀測值 (死亡率=1) 進行穩健化處理 (Z-score based truncated normal imputation)，結合局部時空特徵降低對右尾分布的過度干擾，同時保留潛在高風險訊號。

2. **四階段時空相依性檢定**
   * **Stage 1:** 年度別全域空間相依性 (Global Moran's I)
   * **Stage 2:** 年度別局部空間異質性 (Local Moran's I / LISA 搭配 FDR 校正)
   * **Stage 3:** 跨年度時空延續性 (Bivariate Moran's I & Differential Moran's I)
   * **Stage 4:** 協變數調整後之殘差空間相依性 (OLS Residual Moran's I)

3. **BayesNF 混合觀測模型訓練**
   * 採用具膨脹結構之三頭 BayesNF 架構，以小批量隨機梯度上升與變分推論 (MFVI) 進行參數優化。
   * 模型輸入特徵包含：標準化經緯度、年度、經緯度交互項 ($lat \times lon$) 及空間中心距離。

4. **混合迴歸分析 (Mixture Regression)**
   * 將 BayesNF 萃取之潛在平均場 ($F_{it}$) 納入一般線性迴歸 (M1) 與膨脹常態混合迴歸 (MIX-XF) 中。
   * 評估環境變數 (如 $PM_{2.5}$)、人口結構與臨床特徵對連續死亡風險的解釋力。

##  核心成果展示 (Key Findings)

本專案 Jupyter Notebook (`.ipynb`) 之核心執行結果摘要如下：

* **精準的膨脹點分離能力**
  實際資料中，零值膨脹點約佔整體觀測之 32.7%。BayesNF 模型估計之膨脹機率 ($p_{it}$) 對真實膨脹點的鑑別力 (AUC) 達到 1.0，近乎完美地區分了膨脹點與非膨脹點，避免模型對邊界值產生過度擬合 (Overfitting)。
* **預測誤差顯著降低**
  在模擬與實證資料中，Inflated-Normal BayesNF 模型對比一般常態模型 (Normal) 及傳統階層式時空模型 (GP-based)，其 MAE 與 RMSE 皆有大幅度下降。
* **潛在平均場 ($F_{it}$) 的強大解釋力**
  將 $F_{it}$ 納入迴歸模型 (MIX-XF) 後，模型的 AIC/BIC 指標顯著下降，且 $F_{it}$ 呈現極高度顯著 ($p < 0.001$)，證明其能有效捕捉一般協變數未能解釋的時空潛在異質結構。
* **顯著的環境與臨床風險因子**
  在控制時空背景趨勢後，混合迴歸模型顯示：**$PM_{2.5}$ 濃度、性別 (男性比例)、仍吸菸比例、以及特定的用藥組合與癌症期別**，在非膨脹連續風險成分中均維持高度的統計顯著性，具備明確的流行病學解釋價值。

##  檔案結構 (Repository Structure)

```text
├── data/                    # 模擬與實證資料集 (或資料範例)
├── notebooks/               # 核心 Jupyter Notebooks
│   ├── 01_Data_Preprocessing.ipynb        # 資料前處理、cloglog 轉換與離群值處理
│   ├── 02_Spatiotemporal_EDA.ipynb        # 四階段時空自相關檢定 (Moran's I / LISA)
│   ├── 03_BayesNF_Model_Training.ipynb    # BayesNF 膨脹常態模型訓練與評估
│   └── 04_Mixture_Regression.ipynb        # 結合 BayesNF 潛在場之混合迴歸分析
├── src/                     # 演算法與自定義函數原始碼
│   ├── bayesnf_utils.py     # BayesNF 相關工具函數
│   ├── spatial_stats.py     # 空間統計運算函數
│   └── data_transform.py    # 資料轉換邏輯
├── output/                  # 包含視覺化地圖 (LISA maps)、QQ-plots 及結果表格
└── README.md                # 專案說明文件
```

##  開發環境與套件依賴 (Requirements)

本專案主要使用 Python 進行資料處理與模型建立。關鍵套件包含：

* `numpy`, `pandas`, `scipy`: 數值運算與資料操作
* `geopandas`, `matplotlib`: 地理資訊處理與時空視覺化 (LISA Maps)
* `esda`, `libpysal`: 空間自相關與權重矩陣計算 (Moran's I, KNN, Queen)
* `statsmodels`, `scikit-learn`: 統計檢定、FDR 校正與混合迴歸模型擬合
* `jax`, `bayesnf`: 貝氏神經場模型訓練與後驗推論

---
*此專案為國立成功大學統計學研究所碩士學位論文之輔助程式碼與成果展示庫。詳細模型推導與數學證明請參閱完整學位論文。*
