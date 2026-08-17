自然なのは、Gitワークスペース外にraw受領専用フォルダを作り、intakeには絶対パスで渡す構成です。現行実装も入力ファイルの配置場所は固定せず、basename・SHA・schemaを検査します。

## 推奨配置

```text
<destination-data-root>/
└── motor-control-ai-raw/
    └── den_master_forward_fit_validation/
        └── nf2_r0/
            ├── preflight/
            │   └── prefix_DATA.csv
            └── full/
                └── <receipt-id>/
                    └── nf2_dense_jmag_rt_full_DATA.csv
```

`<receipt-id>`は、上書き防止のため `20260818T153000_JST_<model-revision>` のようにします。

- `preflight/prefix_DATA.csv`
  - 現行契約でファイル名・SHAとも固定済み
  - renameやExcelでの再保存は禁止
- `full/.../nf2_dense_jmag_rt_full_DATA.csv`
  - 推奨する新しい運用名
  - JMAGのexport prefixを `nf2_dense_jmag_rt_full` に設定する想定
  - 受領後、このbasenameと実ファイルSHAを[full receipt config](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/simulation/run_cases/den_motor/den_master_forward_fit_validation/nf2_dense_jmag_rt_full_result_intake/config.toml:6)へ固定

`output/`はintake後のreport成果物用なので、raw CSVを置かない方が安全です。また、現在の`.gitignore`は任意のrawフォルダを除外していないため、リポジトリ内配置は誤コミットの危険があります。

なお、W4文書の`full_prefix_DATA.csv`とfull-intake READMEの`prefix_DATA.csv`は、どちらも現在は例示です。full側の`expected_filename`はまだ`TO_BE_RECEIVED`なので、上記の `nf2_dense_jmag_rt_full_DATA.csv` に統一するのが分かりやすいです。

## 列名契約

W4の正規schemaは `jmag_rt_data_col_v1` です。[実装上も列名と順序を完全一致で検査します](/Users/ochiba/newspace/life/factory/motor-control-ai/features/calibration/code_flux/dense_jmag_rt_preflight_result_intake.py:26)。

```csv
STPNO_COL,PairID,DCV_COL,RPM_COL,ACAV_COL,ACAA_COL,TQ_COL,ID_COL,IQ_COL,ID_SOFT,IQ_SOFT,VD_SOFT,VQ_SOFT
```

| raw列 | 正規化後 | 契約 |
|---|---|---|
| `STPNO_COL` | `step_no` | queueの`sequence_id + 1`と完全一致 |
| `PairID` | `jmag_pair_id` | 空欄不可、診断用のみ |
| `DCV_COL` | `dc_voltage_V` | DC電圧 |
| `RPM_COL` | `rpm` | 回転速度 |
| `ACAV_COL` | `ac_voltage_V` | AC電圧 |
| `ACAA_COL` | `ac_current_Arms` | AC電流monitor |
| `TQ_COL` | `torque_measured_Nm` | 正規トルクchannel |
| `ID_COL` | `id_achieved_A` | achieved Id |
| `IQ_COL` | `iq_achieved_A` | achieved Iq |
| `ID_SOFT` | `id_reference_A` | reference Id |
| `IQ_SOFT` | `iq_reference_A` | reference Iq |
| `VD_SOFT` | `vd_V` | d軸電圧 |
| `VQ_SOFT` | `vq_V` | q軸電圧 |

追加条件：

- 列順も上記どおり。余分な列も不可
- CSVはカンマ区切り、`utf-8-sig`
- `PairID`以外は数値かつfinite
- blank行不可
- preflightは9データ行
- fullは46,369データ行
- `ID_SOFT/IQ_SOFT`が指令値、`ID_COL/IQ_COL`が実到達値
- `PairID`をmirror pair identityとして使用しない
- direct fluxやcoenergy列を同じCSVへ追加しない。取得する場合は別schema・別ファイルの`FE_DIRECT_AUDIT`として扱う

特に、一般的な`code_flux`の旧 `jmag_vi_13col`（`PTN_No`, `Arms_moni`など）とは別schemaです。W4では必ず上記の`*_COL`形式を使用します。

この規約を採用する場合、次はW4文書内のfullファイル例を `nf2_dense_jmag_rt_full_DATA.csv` に統一するのが自然です。
