---
# 磁気共エネルギー

はい。ただし、この第2報で扱う \(W\) は厳密には「磁気エネルギー」ではなく、電流を独立変数とする磁気共エネルギー \(W'(I_d,I_q)\) です。

\[
\psi_d=\frac{\partial W'}{\partial I_d},
\qquad
\psi_q=\frac{\partial W'}{\partial I_q}
\]

したがって、\((I_d,I_q)\) 全域の磁束鎖交数面 \((\psi_d,\psi_q)\) から、積分によって \(W'\) を構成できます。

\[
W'(I_d,I_q)
=
C+
\int_{\mathcal P}
\left(
\psi_d\,dI_d+\psi_q\,dI_q
\right)
\]

例えば、原点からd軸方向、その後q軸方向へ積分するなら、

\[
W'(I_d,I_q)
=
C+
\int_0^{I_d}\psi_d(s,0)\,ds
+
\int_0^{I_q}\psi_q(I_d,t)\,dt
\]

です。単位も、

\[
\mathrm{Wb}\cdot\mathrm{A}=\mathrm{J}
\]

となります。

ここで使うのは、通常の磁束 \(\phi\) ではなく、モータ電圧式に現れる磁束鎖交数 \(\psi\) です。もしご記載の \(\phi_d,\phi_q\) が磁束鎖交数を表しているなら、その理解で構いません。

### 1点の4値だけでは計算できない

\[
(\psi_d,\psi_q,I_d,I_q)
\]

の1点だけからWを一意に算出することはできません。原点からその点までの磁束変化を積分する必要があるため、周囲を含む磁束面が必要です。

また、一般には

\[
W'=\frac{1}{2}
(\psi_d I_d+\psi_q I_q)
\]

とはできません。この式が使えるのは、ほぼ線形・相反的な限定条件です。永久磁石磁束や磁気飽和、交差飽和があるIPMでは不十分です。

例えば線形IPMモデルが、

\[
\psi_d=\psi_{\mathrm{PM}}+L_d I_d,
\qquad
\psi_q=L_q I_q
\]

なら、共エネルギーは、

\[
W'
=
\psi_{\mathrm{PM}}I_d
+\frac{1}{2}L_dI_d^2
+\frac{1}{2}L_qI_q^2
+C
\]

です。微分すると元の磁束式へ戻ります。

### 相反性が必要

積分経路によらず同じWを得るには、

\[
\frac{\partial\psi_d}{\partial I_q}
=
\frac{\partial\psi_q}{\partial I_d}
\]

が必要です。

この条件が成立しない磁束面では、原点から

- d方向→q方向
- q方向→d方向

で積分した結果が異なり、1つのW面を厳密には定義できません。

第1報P0はこの状態を許容します。第2報E1は逆に、まず1つの \(W'\) 面をB-splineで作り、その勾配が観測された \(\psi_d,\psi_q\) に近くなるように係数をFitします。

つまり第2報では、

```text
点磁束を積分してWを後から作る
```

というより、

```text
観測したψd、ψqを勾配として最もよく説明するW面を直接Fitする
```

という実装です。

なお、磁束はWの微分なので、Wの定数 \(C\) は磁束データから決まりません。この任意定数がgaugeであり、第2報ではFit後に共通基準へ合わせています。


---
# 逆CDF


はい。FAN RINGは逆CDFを使用しています。しかも、リング半径とリング内位相の両方で使っています。

リング半径では [measurement_point_selection.py:632](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_points/measurement_point_selection.py:632) で、

1. 各半径・位相の曲率をサンプリング
2. 位相平均35%＋上位分位65%から動径スコアを作成
3. 密度床を加える
4. その密度の逆CDFでリング半径を配置

しています。

逆CDF処理本体は [measurement_point_selection.py:428](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_points/measurement_point_selection.py:428) です。

```python
cumulative = np.cumsum(radial_density)
cumulative = normalize(cumulative)

rings = np.interp(
    np.linspace(0.0, 1.0, n_rings),
    cumulative,
    radial_grid,
)
```

さらに、

- 最小電流リング
- 曲率最大位置のリング
- 定格電流リング

は必ず残すように補正しています。

リング内の位相配置でも、[measurement_point_selection.py:502](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_points/measurement_point_selection.py:502) で位相方向の曲率密度に対する逆CDFを使用しています。そのうえで、正負`Id`側の曲率最大位相を必須点として残します。

違いを整理すると以下です。

| 方式 | 逆CDFに使う密度 |
|---|---|
| Gaussian方式 | 仮定したGaussian飽和膝密度 |
| FAN RING | 実際のJMAG磁束mapから算出した曲率密度 |
| Uniform方式 | 逆CDFを使わず半径等間隔 |

---
# 外挿部分の処置

外挿の考え方は大きく異なります。

- 第1報P0：外側ではRBF補正を減衰させ、物理backboneへ戻す
- 第2報E1：明示したdomain内で共エネルギー面を滑らかに延長し、domain外は評価しない

### 第1報P0

P0は、

\[
\psi
=
\psi_{\mathrm{backbone}}
+
t(r)\,\psi_{\mathrm{RBF残差}}
\]

です。taperは [forward_fit.py:99](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/forward_fit.py:99) にあり、

- \(r\le1074\,A\)：RBF残差を100%使用
- \(1074<r<1200\,A\)：RBF残差をsmoothstepで減衰
- \(r\ge1200\,A\)：RBF残差を0にし、backboneだけを使用

となります。

したがってP0の外挿には、2種類あります。

1. training凸包外だが \(r<1074\,A\)

   RBFがそのまま外挿します。taperは半径だけを見ているため、凸包外であること自体では減衰しません。

2. 外周 \(1074～1200\,A\) 以上

   RBFの影響を減らし、最終的には物理backboneだけに戻します。

また、training凸包内外は [forward_fit.py:274](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/forward_fit.py:274) で判定しますが、凸包外の点を評価禁止にはしていません。map値は凸包外にも生成されます。

つまりP0は「どこでも一応値を返すが、外挿値の妥当性は別途評価する」方式です。

### 第2報E1

E1にはP0のような「backboneへ戻す」処理はありません。

共エネルギー面

\[
W(I_d,I_q)
\]

をB-splineで定義し、その勾配から磁束を得ます。観測凸包外でも、設定されたB-spline domain内ならW面を評価できます。

ただし、その領域はデータ補間ではなく、

- 低次共エネルギー項
- B-splineの局所連続性
- 3階微分roughness正則化
- `Iq=0`境界条件
- 共通Wによる相反拘束

によって決まる「滑らかなmodel外挿」です。

P0のような物理backboneへのフォールバックではありません。

一方、明示したB-spline domain外は評価しません。[coenergy_forward_fit.py:423](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/coenergy_forward_fit.py:423) で例外となり、B-spline自体も `extrapolate=False` です。

[coenergy_forward_fit.py:386](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/coenergy_forward_fit.py:386)

### 比較

| 観点 | 第1報 P0 | 第2報 E1 |
|---|---|---|
| 凸包外・内側領域 | global RBFが外挿 | B-spline＋正則化でW面を延長 |
| 外周処理 | RBFを減衰しbackboneへ戻す | 共エネルギー面をdomain境界まで継続 |
| model domain外 | continuous modelは実質的に値を返せる | 明示的に評価禁止 |
| 外挿時の相反性 | 保証なし | domain内では保証 |
| 外挿を決めるもの | backbone＋RBF＋半径taper | 共通W、境界条件、basis、正則化 |
| 外挿精度 | 保証なし | 同様に保証なし |

重要なのは、E1は「物理整合した外挿」ではありますが、「正しい外挿」を保証するものではないことです。特にinner hole、外周、電流円外、矩形四隅は、referenceがなければ精度を主張できません。

そのため第2報では、

- physical evaluation domain
- training凸包内外
- model補間／model外挿
- LUT三角形補間／nearest fallback

を別々に記録する計画です。[coenergy_forward_fit_validation_plan_v2.md:429](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/report/den_master_point_selection/coenergy_forward_fit_validation_plan_v2.md:429)

なお、P0とE1を公開LUTにした後は、同じ格子・同じruntime補間を使うため、artifact外のnearest fallback条件は共通です。違いが現れるのは、主に「LUT node値を外挿領域でどう生成したか」です。

現状、E1の実データ用breakpoint/domain数値を固定した正規configはまだ見当たらず、存在するのはsynthetic QA用設定です。したがってA-Q1前に、少なくともE1 domainが公開map全nodeを包含することと、outer／円外／矩形四隅をどのdomain classで扱うかを固定する必要があります。

---
# 物理モデルFit

「d/qの拘束」を分けて考える必要があります。

第1報P0にも物理的な形状拘束はあります。しかし、第2報E1が導入する「d/q相反拘束」はありません。

### 第1報P0にある拘束

P0のbackboneは、

- `ψd`が`Id`だけでなく`|Iq|`にも依存する
- `ψq`が`Iq`だけでなく`|Id|`にも依存する
- パラメータを物理的な範囲に制限する
- `ψq` backboneは`Iq=0`で0になる
- 外周でRBF残差をtaperする

という意味で、IPMらしい飽和・交差飽和の形を持っています。

ただし、`ψd`と`ψq`は別々にFitされています。

- d-backboneとq-backboneを別々に`curve_fit`  
  [forward_fit.py:150](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/forward_fit.py:150)
- d残差とq残差を別々のRBFでFit  
  [forward_fit.py:315](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/forward_fit.py:315)

つまり、

\[
\psi_d=f_d(I_d,I_q),\qquad
\psi_q=f_q(I_d,I_q)
\]

と、両方が同じ電流座標に依存しているだけで、`f_d`と`f_q`の係数を結び付ける式はありません。

そのため、

\[
\frac{\partial\psi_d}{\partial I_q}
=
\frac{\partial\psi_q}{\partial I_d}
\]

は保証されません。backbone部分だけを見ても一般には一致せず、さらに独立RBF残差が加わるため、total磁束面でも保証されません。

また、P0の`Iq=0`における`ψq=0`は、最終評価時の置換です。

[forward_fit.py:357](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/forward_fit.py:357)

```python
psi_q = np.where(iq_array == 0.0, 0.0, psi_q)
```

したがって、境界値は0でも、その近傍の傾きやd/q相反性までは拘束していません。

### 第2報E1との本質的な違い

E1では、`ψd`と`ψq`を直接別々にFitしません。まず1つの共エネルギー面をFitします。

\[
W=W(I_d,I_q)
\]

そこから、

\[
\psi_d=\frac{\partial W}{\partial I_d},\qquad
\psi_q=\frac{\partial W}{\partial I_q}
\]

として両磁束を生成します。

実装は [coenergy_forward_fit.py:117](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/coenergy_forward_fit.py:117) です。

このため混合微分が同じになり、

\[
L_{dq}
=
\frac{\partial\psi_d}{\partial I_q}
=
\frac{\partial^2W}{\partial I_q\partial I_d}
=
\frac{\partial^2W}{\partial I_d\partial I_q}
=
\frac{\partial\psi_q}{\partial I_d}
=
L_{qd}
\]

が連続model上で構造的に成立します。コードでも [coenergy_forward_fit.py:159](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/coenergy_forward_fit.py:159) で、

```python
l_dq = ...
l_qd = np.array(l_dq, copy=True)
```

となっています。

| 観点 | 第1報 P0 | 第2報 E1 |
|---|---|---|
| Fit対象 | `ψd`と`ψq`を別々にFit | 1つの共エネルギー`W`をFit |
| 交差飽和 | parametric式として表現 | 共通W面の混合微分として表現 |
| d/q係数 | 独立 | 共通係数 |
| 相反性 `Ldq=Lqd` | 保証なし | 連続modelで構造的に保証 |
| `Iq=0`の`ψq=0` | 最後に0へ置換 | Wの境界条件として成立 |
| 非保存的なデータへの追従 | 追従可能 | そのままは追従できず、保存場へ投影される |
| Fit自由度 | 比較的高い | 物理整合の分だけ制限される |

直感的には、P0は「d方向とq方向の傾斜図を別々に描く方式」、E1は「1つの地形Wを作り、そのd方向・q方向の傾斜を磁束とする方式」です。

なお、E1だから必ず磁束誤差やトルク誤差が小さくなるわけではありません。相反性を満たさない観測成分やノイズまで独立に追従できないため、点磁束残差はP0より大きくなる可能性があります。一方で、点間補間、微分、境界付近の安定性が改善する可能性があります。これを同じ600 pair・同じ高密度referenceのトルクQAで比較するのが、第2報A-Q1です。

また、E1の連続modelが相反性を満たしても、`ψd`／`ψq`を別々の公開LUTへ変換するとLUT補間で相反誤差が再発し得ます。そのため第2報では「連続model」と「公開LUT」を分けて評価する設計になっています。


したがって、FAN RINGは「実曲率に基づく逆CDF配置」、Gaussian方式は「仮定した飽和膝に基づく逆CDF配置」です。FAN RINGの方が対象磁束mapに適応した配置です。
