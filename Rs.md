はい、可能です。しかも既存の`pair_grid`実装に、ほぼ同じ診断があります。

ただし役割は分ける必要があります。

| 値 | 目的 | 決定時期 |
|---|---|---|
| policyの固定`Rs` | `Iq=0` single点の磁束補正、取得契約 | full結果を見る前 |
| pairから推定する`Rs_diag / Vdt_diag` | モデル設定値との整合診断 | full受領後 |

## ペア和差による磁束

正負Iqの鏡像ペアについて、achieved currentと`VD_SOFT/VQ_SOFT`を使います。

\[
I_d=\frac{I_{d+}+I_{d-}}{2},\qquad
I_q=\frac{I_{q+}-I_{q-}}{2}
\]

\[
\psi_d=\frac{V_{q+}+V_{q-}}{2\omega_e}
\]

\[
\psi_q=\frac{V_{d-}-V_{d+}}{2\omega_e}
\]

この和差式では、鏡像対称性が成立していれば`Rs`項が相殺されるため、policyの`Rs`とは独立してpair磁束を計算できます。これは[既存pair_grid方式](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/identification.py:230)と同じ式です。

## Rs／Vdt診断

ペア磁束から逆起電力相当を除き、各側の残差電圧ベクトルを作れます。

```text
Vdef_pos_d = -Vd_pos - omega_e*psi_q
Vdef_pos_q =  Vq_pos - omega_e*psi_d

Vdef_neg_d =  Vd_neg - omega_e*psi_q
Vdef_neg_q =  Vq_neg - omega_e*psi_d
```

各側について、

\[
|V_{def}|=\sqrt{V_{def,d}^2+V_{def,q}^2}
\]

を計算し、複数の電流振幅に対して、

\[
|V_{def}|=R_s^{diag}|I_{dq}|+V_{dt}^{diag}
\]

をロバストfitできます。現行コードも[soft-L1で同じ診断fit](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/identification.py:154)を行っています。

重要なのは、1 pairだけでは`Rs`と`Vdt`を分離できないことです。異なる電流振幅のpair群が必要です。今回のdense H25/H12.5には複数の電流半径があるため、診断条件としては適しています。

## 今回の扱い

推奨する位置づけは次のとおりです。

- `Rs_policy`
  - JMAG-RT設定値
  - full取得前に固定
  - `Iq=0` single点の補正に使用
- `Rs_diag`
  - full結果のpair群から推定
  - JMAG設定値との差を診断
  - policy値の置換には使用しない
- `Vdt_diag`
  - Rs以外の等価誤差電圧intercept
  - JMAG-RTにdeadtimeモデルがなければ、数値誤差、制御誤差、電圧channel差などを含む「等価残差」
  - 物理的なdeadtime電圧と断定しない

また、C0より前はsealed offsetを使用せず、H25とregular H12.5だけで診断する必要があります。

現在のW4 full-intakeはpair groupingとachieved currentを生成しますが、磁束・Rs/Vdt診断そのものはまだ実行しません。後工程へ追加するなら、W4 intakeを変更せず、W5のreport-only診断として次を出す構成が自然です。

- pair別の`psi_d / psi_q`
- pair別の`Vdef`ベクトルと大きさ
- `Rs_diag`
- `Vdt_diag`
- fit residual RMS
- 使用pair数・電流振幅範囲
- `Rs_diag - Rs_policy`
- mirror current／speed mismatchとの関係

つまり、**固定Rsは取得前契約、pair推定Rs/Vdtは取得後診断**として併存できます。これはデータスヌーピングを避けながら、JMAG設定値の妥当性も確認できる構成です。
