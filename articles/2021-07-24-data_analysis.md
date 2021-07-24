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

# 自己回帰過程(autoregressive AR過程)のパワースペクトル密度(power spectrum density PSD)

m次自己回帰過程
$$
\begin{aligned}
x(t) &= \sum_{m=1}^{M} a_m x(t-m\Delta t) + \varepsilon(t)\\
\end{aligned}
$$
式変形を行う．
$$
\begin{aligned}
y(t) &= x(t) - \sum_{m=1}^{M} a_m x(t-m\Delta t) = \varepsilon(t)\\
\end{aligned}
$$
フーリエ変換を行う．
$$
\begin{aligned}
Y(f) &= X(f) - \left(\sum_{m=1}^{M} a_m e^{-2i\pi fm\Delta t} X(f)\right)\\
&= \left(1 - \sum_{m=1}^{M} a_m e^{-2i\pi fm\Delta t} \right)X(f)\\
&= A(f)X(f)
\end{aligned}
$$
ここで$A(f)$を次のように定義する．
$$
\begin{aligned}
A(f) &= {1 - \sum_{m=1}^{M} a_m e^{-2i\pi fm\Delta t}} \\
\end{aligned}
$$
パワースペクトルの関係は次のように表される．
$$
\begin{aligned}
S_y(f) &= |A(f)|^2 S_x(f) = S_\varepsilon (f)\\
\end{aligned}
$$
したがって，
$$
\begin{aligned}
S_x(f) &= \frac{ S_\varepsilon (f) }{ |A(f)|^2 }\\
&= \frac{ \sigma^2 \Delta t }{|1 - \sum_{m=1}^{M} a_m e^{-2i\pi fm\Delta t}|^2}
\end{aligned}
$$

## $\varepsilon$のPSD

横軸$f$，縦軸$PSD$のグラフにおいて，ナイキスト周波数$\pm \frac{ 1 }{ 2\Delta t }の範囲で$\varepsikon$の$PSD$は一定．

このときグラフの面積は分散$\sigma^2$と等しい．(パーセバルの等式)

面積は$\sigma^2$であり，幅は$\frac{ 1 }{ \Delta t} = \frac{ 1 }{ 2 \Delta t } - (-\frac{ 1 }{ 2 \Delta t })$より，高さ(PSD)は$\sigma^2\Delta t$

## 1次自己回帰過程のパワースペクトル

1次自己回帰過程
$$
\begin{aligned}
x(t) = ax(t-1) + \varepsilon(t)\\
\end{aligned}
$$
パワースペクトルは$M=1,\:\Delta t = 1$として，
$$
\begin{aligned}
S(f) &= \frac{ \sigma^2 \Delta t }{|1 - \sum_{m=1}^{M} a_m e^{-2i\pi fm\Delta t}|^2}\\
&= \frac{ \sigma^2 \Delta t }{|1 -  a e^{-2i\pi f}|^2}\\
&= \frac{ \sigma^2 \Delta t }{|1 -  a (\cos(2\pi f) - i \sin(2\pi f))|^2}\\
&= \frac{ \sigma^2 \Delta t }{(1 -  a \cos(2\pi f) + i a\sin(2\pi f))(1 -  a \cos(2\pi f) - i a\sin(2\pi f))}\\
&= \frac{ \sigma^2 \Delta t }{(1 -  a \cos(2\pi f))^2 + (a\sin(2\pi f))^2}\\
&= \frac{ \sigma^2 }{ 1 + a^2 - 2a \cos(2\pi f) }\\
\end{aligned}
$$