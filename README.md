# Exotic Manifolds and the Parameter Space of Machine Learning
# 機械学習のパラメータ空間とエキゾチック多様体

**What if the parameter space were ℝ⁴ - or 7-dimensional?**
**パラメータ空間の「滑らかさ」は誰が選んだのか**

A research note for differential geometers and AI research scientists
AI Research Scientist 向け研究ノート

Author / 著者: Étale Cohomology
Published / 公開: April 2026 / 2026 年 4 月

License: Creative Commons Attribution 4.0 International (CC BY 4.0)

---

## 📖 Read the paper / 論考を読む

| Language | File |
|----------|------|
| 🇯🇵 日本語版 | [`ja/exotic_r4_and_differential_structures_in_ml.pdf`](ja/exotic_r4_and_differential_structures_in_ml.pdf) |
| 🇬🇧 English version | [`en/exotic_r4_and_differential_structures_in_ml_en.pdf`](en/exotic_r4_and_differential_structures_in_ml_en.pdf) |

---

## Abstract (English)

The parameter space of a deep neural network is almost always treated as ℝ^N with N ≫ 1, and optimization proceeds by backpropagation together with some descendant of gradient descent. In this note we pursue a thought experiment: suppose the parameter space were homeomorphic to ℝ⁴ or to S⁷ — dimensions in which a topological manifold admits more than one smooth structure in the sense of differential topology. On which smooth structure is gradient-based optimization actually being performed, and does the choice of structure alter the optima that training converges to?

We argue the following. **(i)** SGD, Adam, and the natural gradient method all presuppose a single smooth chart on which the update rule is written, so the learner has implicitly committed to one diffeomorphism class before training begins. **(ii)** In principle the critical-point structure of a loss function depends on the smooth structure carried by its domain, so two smoothings of the same topological space could in principle disagree on minima. **(iii)** In practice, floating-point arithmetic, bounded architectures, and standard computational graphs conspire to force the standard smooth structure, so exotic structures are not expected to surface as observable phenomena in current systems.

We also clarify that the invariance of Amari's natural gradient — invariance under reparametrization — is a statement about charts within one smooth structure, and is a strictly weaker notion than invariance under change of smooth structure.

---

## 概要 (日本語)

深層ニューラルネットワークを訓練するとき、我々はパラメータ空間を ℝ^N と書き、SGD・Adam・自然勾配法といった勾配ベースの最適化アルゴリズムを走らせる。ここで一つ問いを立てたい。「パラメータ空間 = ℝ^N」と書いた瞬間、我々はどのような数学的選択を黙って行っているのだろうか。

本稿の出発点は、この一見当然に見える前提の中に、**「計量の選択」と「滑らかさの選択」という二段階の暗黙的コミットメント**が潜んでいるという観察である。計量の選択 (ユークリッド計量か、Fisher 計量か、その他か) は機械学習の文献で広く議論されてきたが、その下層にある「滑らかさそのものの選択」は、パラメータ空間が **4 次元の場合に限り、原理的に非可算無限の選択肢**を持つ。これが**エキゾチック微分構造 (exotic smooth structure)** として知られる、4 次元位相幾何学の極めて特異な現象である。

本稿では、AI Research Scientist を主たる読者として、

1. 我々が標準実装で何を選んでいるか
2. なぜ現代の自動微分・浮動小数点演算の枠組みではエキゾチック構造が「不可視」になるか
3. その不可視性自体が機械学習の最適化理論にとってどのような認識論的意味を持つか

を論じる。

---

## Origin of this note — GI Theory Vol.1, Section 11.2
## 本論考の起源 — GI 理論 Vol.1 第 11.2 節

The subject of this note originates from a passage in **GI Theory Vol.1** (Étale Cohomology, March 2026), Section 11.2 "Exotic manifolds and optimal transport":

本論考の主題は、**GI 理論 Vol.1** (Étale Cohomology, 2026 年 3 月) 第 11.2 節「Exotic 多様体と最適輸送」で言及された以下の問題意識に起源を持つ。

> 同相であるが微分同相でない多様体の存在は、同一の大まかな構造を持つ経営環境であっても微分構造が異なりうることを意味する。最適輸送理論との組み合わせによりパンデミック等の早期検出に応用。
>
> *The existence of manifolds that are homeomorphic but not diffeomorphic implies that two business environments with the same coarse structure can carry different differential structures. Combined with optimal transport theory, this idea has potential applications such as early detection of pandemics.*

GI Theory Vol.1 introduced this perspective as a single item in its list of geometric extensions. The present note develops the seed into an independent research note for AI research scientists.

GI 理論 Vol.1 ではこの観点を、幾何学的拡張の 1 項目として簡潔に提示するにとどめた。本論考は、この種を **AI Research Scientist 向けの独立した研究ノートとして展開**するものである。

---

## Three questions and conclusions
## 3つの問いと結論

| Question / 問い | Conclusion / 結論 |
|---|---|
| **(Q1)** Which smooth structure does the learning algorithm actually run on? / 学習アルゴリズムが「走る」微分構造は何によって決まるのか | The implementation chooses; the algorithm does not. / 実装が選ぶ。アルゴリズム (SGD/Adam/自然勾配法) は選ばない |
| **(Q2)** Can the choice of smooth structure change the optimum? / 微分構造の違いは、到達する最適パラメータを変えるか | Yes in principle, no in current implementations. / 原理的には変わりうるが、現代の標準実装では事実上変わらない |
| **(Q3)** Do different optimization algorithms differ in their sensitivity to the smooth structure? / SGD と自然勾配法のような異なる最適化アルゴリズムは、微分構造に対する感度を異にするか | No; all current methods are equally insensitive. / 両者とも微分構造の違いを検出する機構を持たない |

---

## Central claims (four conclusions)
## 中心的主張 (4 つの結論)

1. **(C1)** The smooth structure on which a learning algorithm runs is **not chosen by the algorithm**. It is fixed by the implementation — by IEEE 754 floating-point arithmetic and automatic differentiation — and is the standard smooth structure on ℝ^N.
   機械学習の最適化アルゴリズムが「走る」微分構造は、**アルゴリズムではなく実装によって**選ばれる。

2. **(C2)** In dimension four, ℝ⁴ admits **uncountably many** distinct smooth structures (Freedman, Donaldson, Taubes, Gompf, DeMichelis–Freedman). This is a feature of dimension four alone.
   N = 4 では、原理的に**非可算無限個**の異なる微分構造が存在する。4 次元という次元の絶対的な特異性を示している。

3. **(C3)** Modern machine learning does not see this multiplicity, not because the alternatives are uninteresting but because they **cannot be expressed in the finite computational vocabulary** of the implementation.
   現代の機械学習はこのエキゾチック構造を「見ない」が、それは『見ないことを選んだ』のではなく、『**見る手段を持たない**』状態である。

4. **(C4)** **This invisibility is itself a substantive observation.** The standard smooth structure on ℝ^N is not a self-evident default but a tacit choice embedded in the computational substrate.
   **この不可視性そのものが、認識論的な論点である**。

---

## Structure of the note
## 本論考の構成

The note is designed so that AI research scientists can read it through. Rigorous formulations for specialists in differential topology and information geometry are isolated in Appendices A and B.

本論考は、AI Research Scientist が読み通せるよう設計されている。微分位相幾何と情報幾何の専門家向けの厳密な定式化は、Appendix A・B に分離した。

| Section / 章 | Content / 内容 |
|----|------|
| §1 | Introduction / 動機: 機械学習で我々は何を黙って選んでいるのか |
| §2 | Mathematical preliminaries / 必要な数学的道具 (短期速習) |
| §3 | The peculiarity of dimension four / 4 次元の特異性 |
| §4 | Why exotic structures are invisible to ML / なぜ機械学習はエキゾチック構造を「見ない」のか |
| §5 | The Casson handle construction (optional deep dive) / Casson handle の構成 (詳細・任意) |
| §6 | SGD versus the natural gradient: two notions of invariance / 自然勾配法と微分構造: 二つの「不変性」を区別する |
| §7 | Discussion: the significance of invisibility / では、だからどうする: 不可視性の意味を考える |
| §8 | Conclusion / 結論 |
| Appendix A | Corrections for specialists in differential topology / 微分トポロジー専門家向け補足・修正 |
| Appendix B | Corrections for specialists in information geometry / 情報幾何学専門家向け補足・修正 |

§5 (the Casson handle construction) is an optional chapter for readers seeking detailed mathematical discussion. §3 and §4 alone suffice to reach the main claims of the note.

§5 (Casson handle の構成) は、詳細な数学的議論を求める読者向けの任意の章である。§3 と §4 のみで主要な主張に到達できるよう構成されている。

---

## Intended readers
## 想定読者

The primary intended reader is an **AI Research Scientist** comfortable with linear algebra, multivariable calculus, and probability, who has heard of Riemannian metrics and Fisher information matrices, and whose familiarity with topology and manifold theory does not extend much beyond the notions of continuous map and topological space.

本論考は、線形代数・微積分・確率論を当然の前提とし、Riemann 計量や Fisher 情報行列といった概念を聞いたことはあるものの、多様体論やトポロジーは「連続写像」「位相空間」あたりまでしか踏み込んでいない **AI Research Scientist** を主たる対象としている。

---

## Academic positioning
## 学術的位置づけ

This note is positioned as a **thought experiment**, given that to the author's knowledge no peer-reviewed paper has directly addressed exotic smooth structures on machine-learning parameter spaces.

本論考は、著者の知る限り「機械学習のパラメータ空間上のエキゾチック微分構造」を直接の主題として扱った査読論文が存在しないことを踏まえ、**半ば思弁的な思考実験**として位置づけられる。

The note does **not** claim:

本論考は以下のような主張をするものではない。

- ❌ A solution to any concrete technical problem in machine learning / 機械学習における具体的な技術的問題への解決策の提示
- ❌ A method for "using" exotic structures in ML / エキゾチック構造を機械学習に「使う」方法の提案
- ❌ A direct alternative to existing optimization theories / 既存の最適化理論への直接的な代替案

The note **does** offer:

本論考が示すのは以下である。

- ✅ Explicit articulation of tacit mathematical commitments in ML optimization algorithms / 機械学習の最適化アルゴリズムが暗黙に行っている数学的選択の明示化
- ✅ The epistemological meaning of exotic structures in dimension four for the ML context / 4 次元におけるエキゾチック構造の存在が、機械学習の文脈で持つ認識論的意味
- ✅ A precise discussion of the limits of the natural gradient's reparametrization invariance / 自然勾配法とパラメータ化不変性の限界の精密な議論

---

## Related works by Étale Cohomology
## 関連する Étale Cohomology の仕事

### Theoretical foundations / 理論基盤

- **GI Theory Vol.1**: Geometric Intelligence — A differential-geometric decision framework on data-driven differentiable manifolds (March 2026)
  - GI 理論 Vol.1 第 11.2 節「Exotic 多様体と最適輸送」が本論考の起源
  - Section 11.2 "Exotic manifolds and optimal transport" is the origin of this note
- **GI Theory Vol.2**: Riemannian World Models — Deep reinforcement learning on data-driven differentiable manifolds (March 2026)

### Related notes / 関連論考

- **Note 2**: Algebraic structure of neural networks via quiver representations
  - GitHub: [https://github.com/EtaleCohomology/neural-networks-via-quiver-representations](https://github.com/EtaleCohomology/neural-networks-via-quiver-representations)
  - Zenn: [https://zenn.dev/etalecohomology/articles/f282eb793f97a8](https://zenn.dev/etalecohomology/articles/f282eb793f97a8)

---

## Repository structure
## ファイル構成
exotic-r4-and-differential-structures-in-ml/
├── ja/
│   └── exotic_r4_and_differential_structures_in_ml.pdf  (Japanese version / 日本語版)
├── en/
│   └── exotic_r4_and_differential_structures_in_ml_en.pdf  (English version / 英訳版)
└── README.md

---

## Key references
## 主要参考文献

A complete list of references is in the References section of the note. The following are excerpts of the most important ones.

完全な参考文献リストは論考本文の References セクションを参照のこと。以下は主要なもののみ抜粋。

### Differential topology and exotic structures / 微分トポロジー・エキゾチック構造

- Milnor, J. (1956). On manifolds homeomorphic to the 7-sphere. *Annals of Mathematics*, 64(2), 399–405.
- Freedman, M. H. (1982). The topology of four-dimensional manifolds. *J. Differential Geometry*, 17(3), 357–453.
- Donaldson, S. K. (1983). An application of gauge theory to four-dimensional topology. *J. Differential Geometry*, 18(2), 279–315.
- Taubes, C. H. (1987). Gauge theory on asymptotically periodic 4-manifolds. *J. Differential Geometry*, 25(3), 363–430.
- DeMichelis, S., & Freedman, M. H. (1992). Uncountably many exotic ℝ⁴'s in standard 4-space. *J. Differential Geometry*, 35(1), 219–254.
- Gompf, R. E., & Stipsicz, A. I. (1999). *4-Manifolds and Kirby Calculus*. AMS.

### Information geometry and the natural gradient / 情報幾何・自然勾配法

- Amari, S. (1998). Natural gradient works efficiently in learning. *Neural Computation*, 10(2), 251–276.
- Amari, S., & Nagaoka, H. (2000). *Methods of Information Geometry*. AMS.
- Watanabe, S. (2009). *Algebraic Geometry and Statistical Learning Theory*. Cambridge University Press.

---

## Author / 著者

**Étale Cohomology**

- GitHub: [https://github.com/EtaleCohomology](https://github.com/EtaleCohomology)
- Zenn: [https://zenn.dev/etalecohomology](https://zenn.dev/etalecohomology)
- Email: etalecohomology@proton.me

---

## License / ライセンス

License: Creative Commons Attribution 4.0 International (CC BY 4.0)

This note may be cited, redistributed, and adapted freely, provided that proper attribution is given to the source.

本論考の引用・再配布・改変は、出典を明示すれば自由に行うことができる。

---

## Citation / 引用形式

If you wish to cite this note, the following format is recommended.

本論考を引用する場合は、以下の形式を推奨する。

Étale Cohomology. (2026). Exotic Manifolds and the Parameter Space of Machine Learning:
What if the parameter space were ℝ⁴ - or 7-dimensional?
GitHub: https://github.com/EtaleCohomology/exotic-r4-and-differential-structures-in-ml

---

## Comments and corrections / コメント・誤りの指摘

Comments, corrections, and suggestions for extension are welcomed via GitHub Issues.

本論考へのコメント、誤りの指摘、拡張提案は GitHub Issue にて歓迎する。

The author particularly welcomes feedback from:

特に以下のような連絡を歓迎する:

- Specialists in differential topology (especially 4-dimensional topology and exotic structures) on rigorous corrections / 微分位相幾何 (特に 4 次元トポロジー、エキゾチック構造) の専門家からの厳密な訂正
- Specialists in information geometry and statistical learning theory on methodological critique / 情報幾何学・統計的学習理論の専門家からの方法論的批判
- Specialists in machine learning and optimization theory on potential applications / 機械学習・最適化理論の専門家からの応用可能性の指摘
- Pointers to related prior work / 関連する先行研究の指摘
