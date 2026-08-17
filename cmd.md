# W4-B 実行指示プロンプト — 指定フルパスによるNF2-R0 full-result receipt／intake

> 対象環境: source Gitと断線したフォルダコピー先の別PC
> 開始条件: W4-A pre-acquisition freeze完了後に、46,369行full result CSVを受領済み
> 入力方法: ユーザーが対象CSVの絶対パスを別途1本だけ指定する
> 非対象: JMAG-RT再取得、policy再設計、tolerance調整、production／intake module変更

以下を、full result CSV受領後の実行指示として使用する。

---

あなたは、移設先PCでNF2-R0のW4-B receipt／strict intakeを実行する。source branch、source commit ancestry、
source origin同期、現リポジトリへの証跡返却は要求しない。移設先GitのW4-A commitとjournal、固定artifact、
ユーザー指定CSVだけを根拠にする。

## 実行時入力

ユーザーは次の値を別メッセージまたは本プロンプトへの追記で指定する。

```text
【移設先PC・ユーザー記入（必須）】
FULL_RESULT_CSV_ABSOLUTE_PATH = "<ここに受領済みfull result CSVの絶対パスを記入>"
```

- placeholderのまま、相対path、file不在、複数候補の指定では開始しない。
- `find`、glob、更新日時、ファイル名の類似度で対象CSVを探索・自動選択しない。
- 指定CSVをrename、copy、文字コード変換、改行変換、Excel等で再保存しない。
- 指定CSVはGit worktree外のraw受領領域に置く。worktree内なら、作業者判断で移動せず報告して止まる。
- CLIとjournalには指定された同じ絶対pathを使用し、configの`expected_filename`にはそのbasenameだけを記録する。

## 最初に読むもの

1. `AGENTS.md`、`features/AGENTS.md`、`features/calibration/AGENTS.md`
2. `features/calibration/report/den_master_point_selection/coenergy_forward_fit_validation_plan_v2.md`
3. `features/calibration/report/den_master_point_selection/handoff_20260810/README.md`
4. `features/calibration/report/den_master_point_selection/handoff_20260810/prompt_w4_nf2_r0_policy_freeze_intake.md`
5. `features/calibration/simulation/run_cases/den_motor/den_master_forward_fit_validation/nf2_dense_jmag_rt_full_result_intake/README.md`
6. 同run caseの`policy.toml`、`config.toml`と、移設先journalのW4-A取得記録

## 固定契約

- result schema: `jmag_rt_data_col_v1`
- encoding: `utf-8-sig`
- data rows: `46,369`
- full queue SHA-256:
  `02c6a5e06f7d6da90797b0b550d3d916821e6ff9c364d204969df1fc93ede945`
- raw列と列順:

```text
STPNO_COL,PairID,DCV_COL,RPM_COL,ACAV_COL,ACAA_COL,TQ_COL,ID_COL,IQ_COL,ID_SOFT,IQ_SOFT,VD_SOFT,VQ_SOFT
```

- `STPNO_COL = sequence_id + 1`
- `PairID`はnonblankの診断列であり、mirror identityへ使用しない
- `ID_SOFT/IQ_SOFT`はreference、`ID_COL/IQ_COL`はachieved current
- H25／regular H12.5だけをdesign公開し、offsetは`SEALED_UNTIL_NF2_C0`
- raw CSVはGitへ追加しない。filename、exact SHA、行数、schema、取得・export metadataだけを記録する

## 手順

### 1. rawを解釈する前にW4-Aを確認する

移設先worktreeで次を確認する。

- worktreeのbranch、HEAD、statusを記録する
- W4-A pre-acquisition freeze commitが移設先remoteへpush済み
- `policy.toml`の`policy.status = "FROZEN"`
- `policy.toml`の`flux_identification.status = "FROZEN"`
- measurement-plan provenance、全tolerance、`Rs`／抵抗温度／source SHA、`pole_pairs=4`、
  `omega_e`、`Iq=0` flux式が取得前に固定済み
- full receipt `config.toml`の`[policy].sha256`が現在の`policy.toml` exact bytesと一致
- `[result].input_status = "HOLD_INPUT_NOT_RECEIVED"`で、他のresult identity fieldが
  `TO_BE_RECEIVED`のまま
- preflight QAが`READY_FOR_FULL_QUEUE`かつ`full_queue_authorized=true`
- full queueが46,369データ行で固定SHAと一致
- W4-A時点の専用testと全`code_flux` regressionがPASS

source Git lineageが確認できないことは停止理由にしない。上記の移設先W4-A証跡または技術的Hard fenceが
不足する場合は、指定CSVをpandas、Excel、plot等で開かず、欠落項目を報告して止まる。

### 2. ユーザー指定pathと取得記録を受け付ける

`FULL_RESULT_CSV_ABSOLUTE_PATH`が絶対pathであり、readableな単一fileであることだけを確認する。
basenameをreceiptの`expected_filename`候補として記録し、exact bytesのSHA-256を計算する。

移設先journalの取得記録から、次を取得する。値をfilename、file timestamp、CSV内容から推測しない。

- `acquisition_timestamp`: timezone付きISO-8601
- `export_method`
- `export_tool_version`
- JMAG-RT model ID／revision、MILS case、温度がW4-A policyと一致すること

`policy_frozen_at <= acquisition_timestamp`を確認する。不成立またはmetadata不足なら、intakeへ進まない。

### 3. receipt configの`[result]`だけをfreezeする

W4-A後の`policy.toml`を変更しない。full receipt `config.toml`の`[result]`だけを、受領事実に合わせて更新する。

```toml
[result]
input_status = "FROZEN"
expected_filename = "<FULL_RESULT_CSV_ABSOLUTE_PATHのbasename>"
expected_sha256 = "<指定CSV exact bytesのSHA-256>"
encoding = "utf-8-sig"
schema_id = "jmag_rt_data_col_v1"
expected_rows = 46369
acquisition_timestamp = "<取得記録のtimezone付きISO-8601>"
export_method = "<取得記録の値>"
export_tool_version = "<取得記録の値>"
receipt_frozen_at = "<受領freeze時点のtimezone付きISO-8601>"
receipt_git_commit = "<移設先W4-A pre-acquisition freeze commit>"
```

次を満たすことを確認する。

```text
policy_frozen_at <= acquisition_timestamp <= receipt_frozen_at
```

`[policy].sha256`、`policy.toml`、measurement-plan、preflight成果物、full queue、command membership、
dataset manifest、analysis-view contractを変更しない。CSV結果を見てpolicy値、tolerance、`Rs`を変更しない。

### 4. 空の一時出力先でstrict intakeを実行する

tracked `output/`へ直接実行しない。存在しないか空の一時出力先を用意し、ユーザー指定の絶対pathを
第2引数へそのまま渡す。

```bash
cd features/calibration/code_flux

../.venv/bin/python run_dense_jmag_rt_full_result_intake.py \
  ../simulation/run_cases/den_motor/den_master_forward_fit_validation/nf2_dense_jmag_rt_full_result_intake/config.toml \
  "<FULL_RESULT_CSV_ABSOLUTE_PATH>" \
  <empty-output-dir>
```

Windowsでは同じcalibration venvの`..\.venv\Scripts\python.exe`を使用する。system Pythonへfallbackしない。

成功時は次を要求する。

- status: `READY_DESIGN_INPUT_OFFSET_SEALED`
- physical rows: `46,369`
- 7成果物のatomic publish
- H25／regular H12.5だけがvalue-bearing成果物へ含まれる
- offset値、統計、score、plot、最大位置が出力されない

HOLD／FAIL／ERRORでは再実行前にreasonだけを報告する。offsetやFAIL rawの値を調査せず、policyを緩和せず、
旧schema、preflight 9行、synthetic、master3へfallbackしない。

### 5. 成果物を確認し、W4-Bを固定する

一時出力で成功を確認した後だけ、正規run caseのtracked成果物を更新する。raw CSVは移動・追加しない。

```bash
cd features/calibration/code_flux

../.venv/bin/python -m unittest discover \
  -s tests -p 'test_dense_jmag_rt_full_result_intake.py' -q

../.venv/bin/python -m unittest discover \
  -s tests -p 'test_*.py' -q

../.venv/bin/python -m py_compile \
  dense_jmag_rt_full_result_intake.py \
  run_dense_jmag_rt_full_result_intake.py
```

repository rootで`git diff --check`を実行する。test件数は固定値と比較せず、実測件数と結果をjournalへ記録する。
state-lock testは新tracked stateをexactに要求し、旧HOLD経路を一時fixtureで維持する。

次だけを移設先W4-B commitへ含める。

- full receipt `config.toml`の`[result]`更新
- strict intakeの正規成果物
- state-lock testのreceipt期待値更新
- 移設先journalとREADMEの状態更新

raw CSV、一時出力、venvをcommitしない。W4-B commitを移設先remoteへpushする。source repositoryはpush先にせず、
現リポジトリへ同commitや後工程証跡を戻すことは要求しない。

## 完了報告

値を推測せず、次を報告する。

- 使用した`FULL_RESULT_CSV_ABSOLUTE_PATH`
- basename、exact SHA-256、46,369データ行、schema検査結果
- policy SHA、W4-A commit、取得・receipt時系列
- intake statusと公開成果物名
- offsetがsealedであること
- 専用test、全`code_flux` regression、compile、diff checkの結果
- 移設先W4-B commit／remote
- source Git lineageと現リポジトリ側証跡が存在しない承認済み制約

## 停止条件

- `FULL_RESULT_CSV_ABSOLUTE_PATH`が未記入、相対path、file不在、または複数指定
- 指定CSVがGit worktree内にある、または受領後にbytesが変更された
- W4-A policy／preflight／queue／test／commit／pushの前提不足
- acquisition metadataまたはprovenanceを独立記録から確定できない
- policy SHA bindingまたはfreeze時系列が不正
- filename、SHA、13列の名前・順序、46,369行、step identity、finite、tracking Gateの不一致
- intakeがHOLD／FAIL／ERRORを報告する
- W4-A後のpolicy変更、production／intake module変更、offset開封が必要になる

source branch／commit ancestry／origin同期を確認できないこと、および現リポジトリへ証跡を戻せないことだけでは
停止しない。
