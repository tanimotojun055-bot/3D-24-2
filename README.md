# 3D-24-2


Geometric Cryptographic Memory II

実装アーキテクチャと数値実験プロトコル

From Mathematical Theory to Executable 3D Cryptographic Memory

---

Abstract

本稿では、Geometric Cryptographic Memory（GCM）および Dynamic 3D Hash の理論モデルを、実際のソフトウェアとして実装可能な計算モデルへ変換するための中間仕様を定義する。

第1稿では、情報を単なるビット列ではなく、

$$\mathcal{M}(t)={P,g,\phi,E,Q}$$

という動的幾何状態として表現し、空間状態そのものを暗号学的記憶状態として扱うモデルを提案した。

本稿ではこれをさらに進め、

- 3次元格子への離散化
- 非線形場の時間発展
- 幾何状態の更新
- 六方向投影
- 内部構造ハッシュ
- 固有モード解析
- 状態軌道ハッシュ
- 決定論的シリアライズ
- KDFとの結合
- 再現可能な数値実験

を統合し、実装前のアルゴリズム仕様として定式化する。

---

1. Objective

GCMの実装上の目的は、入力データから単純に一度だけハッシュ値を計算することではない。

本モデルでは、

Data
→ Initial 3D State
→ Nonlinear Evolution
→ Geometric Transformation
→ Stable / Metastable State
→ State Encoding
→ Cryptographic Digest

という過程を計算する。

基本状態は、

$$\mathcal{M}_t=(P_t,g_t,\phi_t,E_t,Q_t)$$

である。

ここで、

- P_t：離散3D内部構造
- g_t：時刻 t の幾何状態
- \phi_t：暗号場
- E_t：エネルギー情報
- Q_t：固有モード情報

を表す。

---

2. Discrete 3D Space

連続空間を直接計算する代わりに、まず有限格子へ離散化する。

$$\Omega_h={(i,j,k)\mid 0\le i<N_x,\ 0\le j<N_y,\ 0\le k<N_z}$$

暗号場を、

$$\phi_{ijk}^{(n)}$$

とする。

ここで n は時間ステップである。

3D構造は、

$$P^{(n)}={\phi_{ijk}^{(n)}}$$

として保持する。

---

3. Initial-State Generator

入力データを、

$$D\in{0,1}^{*}$$

とする。

まず標準ハッシュを用いて初期シードを生成する。

$$S_0=H(D\Vert N_0)$$

ここで N_0 はnonceまたは乱数である。

このシードから決定論的擬似乱数生成器を用いて、

$$P^{(0)}=F_{\mathrm{init}}(S_0)$$

を生成する。

これにより、同じ

- 入力データ
- nonce
- パラメータ

を使用した場合には同じ初期3D状態を再現できる。

---

4. Geometric State

幾何状態を離散パラメータ集合として、

$$g^{(n)}={g_{ij}^{(n)}}$$

とする。

初期実装では、完全な一般計量を直接扱う必要はない。

最初の実験では、

$$g_{ij}=a^2(t)\delta_{ij}$$

のような等方的スケール因子から開始する。

これにより、

- 膨張
- 収縮

を単一パラメータ a(t) で表現できる。

---

5. Nonlinear Potential

基本ポテンシャルとして、

$$V(\phi)=\frac{\lambda}{4}(\phi^2-v^2)^2$$

を採用する。

その微分は、

$$V'(\phi)=\lambda\phi(\phi^2-v^2)$$

である。

したがって連続モデルでは、

$$\frac{\partial\phi}{\partial t}=\Gamma[\alpha\Delta_g\phi-\lambda\phi(\phi^2-v^2)+J]$$

となる。

---

6. Discrete Evolution Equation

時間刻みを \Delta t とする。

陽的Euler近似では、

$$\phi_{ijk}^{(n+1)}=\phi_{ijk}^{(n)}+\Delta t,\Gamma[\alpha(\Delta_g^h\phi){ijk}^{(n)}-\lambda\phi{ijk}^{(n)}((\phi_{ijk}^{(n)})^2-v^2)+J_{ijk}^{(n)}]$$

とする。

\Delta_g^h は離散Laplace–Beltrami演算子である。

平坦格子の第1実装では、

$$\Delta_h\phi_{ijk}=\frac{\phi_{i+1,j,k}+\phi_{i-1,j,k}+\phi_{i,j+1,k}+\phi_{i,j-1,k}+\phi_{i,j,k+1}+\phi_{i,j,k-1}-6\phi_{ijk}}{h^2}$$

を利用できる。

---

7. Energy Evaluation

各時間ステップで全エネルギーを計算する。

$$E^{(n)}=\sum_{ijk}\left[\frac{\alpha}{2}|\nabla_h\phi_{ijk}^{(n)}|^2+V(\phi_{ijk}^{(n)})-J_{ijk}^{(n)}\phi_{ijk}^{(n)}\right]\Delta V$$

この値を、

$$E^{(0)},E^{(1)},E^{(2)},\ldots$$

として保存する。

静的なエネルギー地形では、基本的に安定化過程で、

$$E^{(n+1)}\le E^{(n)}$$

となることを確認する。

動的外場を含む場合には、この単調減少条件を必須とはしない。

---

8. Convergence Condition

安定状態への収束判定を、

$$|\phi^{(n+1)}-\phi^{(n)}|_2<\varepsilon$$

とする。

または、

$$|E^{(n+1)}-E^{(n)}|<\varepsilon_E$$

を使用する。

収束した状態を、

$$\phi_\ast$$

とする。

最大反復数、

$$n_{\max}$$

も必ず設定する。

---

9. Spherical Reference Model

最初の検証モデルとして球対称構造を使用する。

$$\phi(r)\approx v\tanh\left(\frac{r-R}{\xi}\right)$$

ここで、

$$\xi=\frac{\sqrt{2\alpha}}{v\sqrt{\lambda}}$$

である。

この解析近似と3D格子数値解を比較する。

これにより、

- 数値解の妥当性
- 格子サイズ依存性
- 時間刻み依存性
- 境界条件依存性

を検証する。

---

10. Six-Directional Projection

収束した3D状態から六方向投影を計算する。

$$\Pi_{+x},\Pi_{-x},\Pi_{+y},\Pi_{-y},\Pi_{+z},\Pi_{-z}$$

第1実装では各方向について、

- maximum projection
- sum projection
- weighted projection

を比較する。

各面のハッシュを、

$$h_f=H(\operatorname{Encode}(\Pi_f))$$

とする。

---

11. Surface Digest

六面情報を正規順序で連結する。

$$H_{\mathrm{surface}}=H(h_{+x}\Vert h_{-x}\Vert h_{+y}\Vert h_{-y}\Vert h_{+z}\Vert h_{-z})$$

順序は必ず仕様として固定する。

---

12. Internal-State Digest

内部構造全体も独立にエンコードする。

$$H_{\mathrm{internal}}=H(\operatorname{EncodeCanonical}(P_\ast))$$

Canonical Encoding が重要である。

同じ数学状態が、浮動小数点表現やプラットフォーム差によって異なるbyte列になってはならない。

---

13. Canonical Quantization

連続値を直接ハッシュへ入力せず、固定量子化する。

例えば、

$$q_{ijk}=\operatorname{round}(s\phi_{ijk})$$

とする。

ここで s はスケール定数である。

さらに固定幅整数、

$$q_{ijk}\in\mathrm{int32}$$

などへ変換する。

これによってCPUやOSによる浮動小数点差を抑える。

---

14. Canonical Serialization

最終状態を、

$$B_\phi=\operatorname{SerializeCanonical}(q_{000},q_{001},\ldots)$$

としてbyte列へ変換する。

シリアライズ順序は、

$$i\rightarrow j\rightarrow k$$

など、仕様で一意に固定する。

endiannessも固定する。

例えば、

Little Endian

または、

Big Endian

のどちらかを仕様として採用する。

---

15. Mode Analysis

球形モデルについては、

$$\phi(r,\theta,\varphi)=\sum_{\ell,m}q_{\ell m}(r)Y_{\ell m}(\theta,\varphi)$$

としてモード解析を行う。

有限個のモード、

$$0\le\ell\le\ell_{\max}$$

だけを取り出し、

$$Q={q_{\ell m}}$$

として状態情報に加える。

---

16. External Rotating Field

外部回転場を、

$$\mathbf{E}_{\mathrm{rot}}(t)=E_0(-\sin\Omega t,\cos\Omega t,0)$$

とする。

これを外部駆動、

$$J(\mathbf{x},t)$$

または結合エネルギー、

$$E_{\mathrm{coupling}}=-\gamma\mathbf{P}\cdot\mathbf{E}_{\mathrm{rot}}$$

として導入する。

これにより、

$$\ell=0$$

だけでなく、

$$\ell>0$$

のモード励起を観測する。

---

17. Dynamic Geometry

次の段階では、

$$a=a(t)$$

を導入する。

例えば膨張モデル、

$$a(t)=a_0e^{Ht}$$

および収縮モデル、

$$a(t)=a_0e^{-Ht}$$

を比較する。

このときGCM状態は、

$$\mathcal{M}(t)=\mathcal{M}[\phi(t),a(t)]$$

となる。

---

18. Trajectory Digest

GCMでは最終状態だけでなく、その状態へ至る軌道も利用する。

ただし全時間ステップをそのまま保存するとデータ量が大きいため、逐次ハッシュを使う。

初期値を、

$$T_0=H(\text{domain separator})$$

とする。

各ステップで、

$$T_{n+1}=H(T_n\Vert H(\operatorname{EncodeCanonical}(\mathcal{M}_n)))$$

と更新する。

最終的に、

$$H_{\mathrm{trajectory}}=T_N$$

とする。

これなら軌道全体を保持する必要がない。

---

19. Geometric State Digest

最終的な幾何状態ダイジェストを、

$$H_{\mathrm{GCM}}=H(H_{\mathrm{surface}}\Vert H_{\mathrm{internal}}\Vert H_{\mathrm{trajectory}}\Vert H_{\mathrm{mode}}\Vert H_{\mathrm{parameter}})$$

と定義する。

ここで、

$$H_{\mathrm{parameter}}$$

には、

- 格子サイズ
- \Delta t
- \alpha
- \lambda
- v
- 境界条件
- 幾何パラメータ
- アルゴリズムバージョン

を含める。

---

20. Domain Separation

他の用途のハッシュと混同しないため、固定文字列を使用する。

例えば、

GCM-V1

をdomain separatorとして用いる。

形式的には、

$$H_{\mathrm{GCM}}=H(\text{"GCM-V1"}\Vert B)$$

とする。

---

21. Cryptographic Key Derivation

GCMダイジェスト単独を秘密鍵にしない。

秘密鍵、

$$K_s$$

および新規乱数、

$$N_t$$

を使用する。

$$K_t=\operatorname{HKDF}(K_s,H_{\mathrm{GCM}}\Vert N_t)$$

とする。

これによって、

GCM = dynamic state layer

標準暗号 = security layer

という役割分担を維持する。

---

22. Determinism Requirement

暗号アルゴリズムとして使用する場合、同じ入力に対して実装間で同一結果を得られる必要がある。

したがって、

$$F(D,\Theta)=H_{\mathrm{GCM}}$$

に対して、

$$F_A(D,\Theta)=F_B(D,\Theta)$$

が必要である。

ここで A,B は異なる計算機または実装である。

このため、

- 固定精度演算
- 固定量子化
- 固定シリアライズ
- 固定境界条件
- 固定乱数生成方式

が重要になる。

---

23. Avalanche Experiment

入力を1bitだけ変更する。

$$D'=D\oplus2^k$$

そして、

$$H_{\mathrm{GCM}}(D)$$

と、

$$H_{\mathrm{GCM}}(D')$$

を比較する。

Hamming distanceを、

$$A=\frac{d_H(H,H')}{n}$$

とする。

理想的なハッシュに近い場合、

$$A\approx\frac{1}{2}$$

になることが期待される。

これは実験によって検証する。

---

24. Geometry Sensitivity Experiment

入力データを固定したまま、

$$R\rightarrow R+\delta R$$

または、

$$a(t)\rightarrow a(t)+\delta a(t)$$

とする。

そして、

$$d_H(H_{\mathrm{GCM}},H'_{\mathrm{GCM}})$$

を測定する。

これによって幾何状態に対する感度を評価する。

---

25. Energy-Landscape Experiment

異なる初期状態、

$$P_0^{(1)},P_0^{(2)},\ldots,P_0^{(N)}$$

を用意する。

各初期状態がどの安定点、

$$\mathcal{M}_\ast^{(k)}$$

へ到達するかを調べる。

これによって、

- 吸引域
- 局所極小
- 準安定状態
- 初期値依存性

を数値的に解析する。

---

26. Security Experiments

少なくとも以下を測定する。

Collision Search

異なる入力、

$$D_1\neq D_2$$

について、

$$H_{\mathrm{GCM}}(D_1)=H_{\mathrm{GCM}}(D_2)$$

となる例を探索する。

Preimage Experiment

与えられた、

$$Y=H_{\mathrm{GCM}}(D)$$

に対し、D を探索する。

Second-Preimage Experiment

既知入力 D に対して、

$$D'\neq D$$

かつ、

$$H_{\mathrm{GCM}}(D')=H_{\mathrm{GCM}}(D)$$

となる入力を探索する。

これらの実験は安全性証明そのものではないが、初期アルゴリズムの弱点検出に利用する。

---

27. Reference Implementation Architecture

初期実装を以下のモジュールに分離する。

gcm/core

3D格子および状態管理。

gcm/field

非線形場の時間発展。

gcm/geometry

球、楕円体、円錐、トーラスなどの幾何。

gcm/projection

六方向投影。

gcm/modes

球面調和関数および固有モード解析。

gcm/energy

エネルギー汎関数計算。

gcm/serialize

canonical encoding。

gcm/hash

state digestおよびtrajectory digest。

gcm/crypto

HKDF等の標準暗号との接続。

experiments

数値実験。

tests

決定論性および回帰テスト。

---

28. Suggested Repository Structure

gcm/
├── README.md
├── THEORY.md
├── PAPER-II.md
├── pyproject.toml
├── Dockerfile
├── src/
│   └── gcm/
│       ├── core.py
│       ├── field.py
│       ├── geometry.py
│       ├── energy.py
│       ├── projection.py
│       ├── modes.py
│       ├── serialize.py
│       ├── hashing.py
│       └── crypto.py
├── experiments/
│   ├── sphere_static.py
│   ├── sphere_dynamic.py
│   ├── rotating_field.py
│   └── avalanche_test.py
├── tests/
│   ├── test_determinism.py
│   ├── test_serialization.py
│   ├── test_projection.py
│   └── test_hashing.py
└── docs/
    └── figures/

---

29. Phase-1 Implementation

最初から全理論を実装しない。

Phase 1では以下だけを実装する。

1. 3D立方格子
2. \phi^4 非線形場
3. 球形初期条件
4. 勾配流
5. エネルギー計算
6. 六方向投影
7. canonical integer encoding
8. SHA-3またはSHA-2による状態ハッシュ
9. trajectory hash
10. avalanche test

これをGCM Reference Model 1とする。

---

30. Phase-2 Implementation

Phase 2では、

- 膨張・収縮
- 時間依存計量
- 外部回転場
- 球面調和モード
- 楕円体
- 円錐

を追加する。

---

31. Phase-3 Implementation

Phase 3では、

- トーラス
- トポロジカル状態
- 多数局所極小
- 分岐解析
- 非線形共振
- 量子シミュレーション

を扱う。

---

32. Cloud Experimental Architecture

計算環境はローカル計算とクラウド計算を分離する。

GitHub
→ Source Code / Version Control

Google Cloud Build
→ Build / Test

Artifact Registry
→ Container Image

Cloud Run
→ GCM API / Experimental Service

Cloud Storage
→ Experiment Results

Secret Manager
→ API Keys / Secret Parameters

Cloud Logging
→ Execution Logs

この構造ではサーバー内部へ直接秘密情報を書き込まない。

---

33. API Model

初期APIとして、

POST /simulate

POST /hash

POST /energy

POST /project

POST /verify

を定義できる。

例えば、

POST /hash

への入力を、

{
  "input": "...",
  "geometry": "sphere",
  "grid": 64,
  "steps": 1000,
  "version": "GCM-V1"
}

とする。

返値は、

{
  "gcm_digest": "...",
  "surface_digest": "...",
  "internal_digest": "...",
  "trajectory_digest": "...",
  "energy_final": 0.0,
  "iterations": 0
}

のようにする。

---

34. Reproducibility

各計算結果には必ず、

- software version
- git commit
- parameter set
- seed
- geometry version
- hash algorithm
- quantization version

を保存する。

これにより、

$$\text{Experiment}\rightarrow\text{Reproduction}$$

を可能にする。

---

35. Research Principle

GCMで最も重要なのは、複雑さそのものを安全性と同一視しないことである。

$$\text{Complexity}\neq\text{Security Proof}$$

そのため、

物理・幾何モデル
→ 新しい状態表現

標準暗号
→ 秘密性・完全性

という二層構造を維持する。

---

36. Implementation Target

最初の完成目標を、

$$D\rightarrow P_0\rightarrow\phi_t\rightarrow\phi_\ast\rightarrow H_{\mathrm{GCM}}$$

とする。

これが再現可能になった時点で、

Geometric Cryptographic Memory は概念モデルから実行可能モデルへ移行する。

---

37. Conclusion

第1稿では、

«Space is memory.»

という概念を提示した。

第2稿では、その概念を、

$$\text{Space}\rightarrow\text{Numerical State}\rightarrow\text{Canonical Bytes}\rightarrow\text{Cryptographic Digest}$$

へ変換した。

次の研究段階は、この仕様をPythonによる参照実装として構築し、

- 数値安定性
- 再現性
- avalanche effect
- geometry sensitivity
- computational cost
- cryptographic properties

を実測することである。

---

Collaboration Statement

本研究の基本着想、3Dハッシュ構想、Geometric Cryptographic Memoryの研究方向および最終的な研究判断は著者による。

本稿の数学的整理、アルゴリズム構成、実装仕様化および文章作成には、OpenAIのChatGPTを共同検討・執筆支援ツールとして使用した。

---

Keywords

Geometric Cryptographic Memory
3D Hash
Dynamic Hash
Nonlinear Field
Energy Landscape
Canonical Serialization
Geometric Cryptography
Field Cryptography
Dynamic Geometry
Post-Quantum Cryptography
