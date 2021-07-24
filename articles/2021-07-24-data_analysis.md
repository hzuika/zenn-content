---
title: "データ解析勉強ノート"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---
# Wiener-Khintchine(ウィナーヒンチン)の定理

弱定常過程では，自己相関関数(autocorrelation function) $R(t)$のフーリエ変換がパワースペクトル密度(power spectral density) $P(f)$になる．
また，パワースペクトル密度の逆フーリエ変換が自己相関関数になる．

$$
\begin{aligned}
P(f) &= \sum_{t=-\infty}^{\infty} R(t)e^{-2i\pi ft}\\
R(t) &= \int_{-\tfrac{1}{2}}^{\tfrac{1}{2}} P(f) e^{2i\pi ft} df
\end{aligned}
$$