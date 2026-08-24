確認した正式なパスは`motor_simulation_base/`です。Git管理されているトップレベルフォルダは11個あり、加えてローカル生成フォルダがあります。

## トップレベル構成

| フォルダ | 主な内容 | 位置づけ |
|---|---|---|
| [`.agents/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/.agents) | Sim-no8テストパターン用のrepository-local skill | AI作業・定型実験の支援 |
| [`archive/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/archive) | 過去のC++・Rust実装やプロット | 履歴参照用。現行実装ではない |
| [`cpp/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/cpp) | C++モータモデル、flux map、CMake実行ハーネス | C++基準シミュレーション |
| [`design/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/design) | 設計経緯、実装計画、判断理由 | アーカイブ。現行仕様は`docs/spec/`を優先 |
| [`docs/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/docs) | 利用説明、現行仕様、共有資料 | 現在メンテナンスする文書の入口 |
| [`plant/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/plant) | モータ定数、磁束map、controller table、plant manifest | plantデータのsource of truth |
| [`python/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/python) | データ変換、table生成、比較、解析、plot | シミュレーション補助ツール |
| [`report/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/report) | ベース制御の評価結果・検証報告 | 過去評価と判断根拠 |
| [`rust/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/rust) | plant、controller wrapper、runner、export、sweep | 現在の主要シミュレーション実行基盤 |
| [`simulation/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/simulation) | scenario、sweep、実行script、結果 | 実験の実行・入出力領域 |
| [`viewer/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/viewer) | Rust GUI viewer | 補助可視化ツール。base本体とは別扱い |

全体像は次のようになります。

```text
motor_simulation_base/
├── .agents/
├── archive/
├── cpp/
├── design/
├── docs/
├── plant/
├── python/
├── report/
├── rust/
├── simulation/
└── viewer/
```

## `.agents/`

| サブフォルダ | 内容 |
|---|---|
| `.agents/skills/` | repository-local skillの格納場所 |
| `.agents/skills/sim-no8-test-patterns/` | 標準トルク指令patternの作成・実行・成果物整理手順 |

`.skills/`もローカルに存在しますが、現在は空の互換用フォルダです。正は`.agents/skills/`です。

## `archive/`

| サブフォルダ | 内容 |
|---|---|
| `archive/cpp/` | 過去のC++実装・結果 |
| `archive/rust/` | 過去のRust実装・結果 |
| `archive/rust/plots/` | 過去のplot成果物 |

新規実装の追加先ではなく、過去状態を調べるための場所です。

## `cpp/`

| ファイル・構成 | 内容 |
|---|---|
| `motor_model.{h,cpp}` | C++モータモデル |
| `flux_map.{h,cpp}` | 2次元磁束mapの読込み・補間 |
| `main.cpp` | C++検証ハーネス |
| `CMakeLists.txt` | CMakeビルド定義 |

Rust実装との比較や、C++側の基準動作確認に使います。

## `design/`

| サブフォルダ | 内容 |
|---|---|
| `design/cpi/` | standalone C PI、Phase7 C controllerの設計・工程 |
| `design/rust_plan/` | Rust化、runner、export、sweepの工程記録 |
| `design/startup/` | ベースシミュレーション初期立ち上げ計画 |
| `design/variable_control_period/` | 可変carrier時の制御周期対応 |
| `design/*.md` | PWM同期、plant環境、model前提などの設計経緯 |

ここは「なぜ現在の構造になったか」を調べる場所です。現在の仕様判断には使いすぎず、`docs/spec/`を優先します。

## `docs/`

| サブフォルダ | 内容 |
|---|---|
| [`docs/spec/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/docs/spec) | 現在のアーキテクチャ、利用方法、C呼び出し構造 |
| [`docs/flyer/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/docs/flyer) | 説明資料・共有用資料 |
| [`docs/README.md`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/docs/README.md) | ドキュメント全体の入口 |

主な現行仕様は次です。

| 文書 | 内容 |
|---|---|
| `sim_rust_architecture.md` | Rust workspaceとcrate責務 |
| `sim_controller_no8_user_manual.md` | No8 controllerの利用方法 |
| `sim_controller_no8_c_call_structure.md` | Rust→C→各制御関数の呼び出し構造 |
| `sim_torque_command_test_patterns.md` | 標準トルク指令pattern |

今回の制御モード仕様は、ここへ次の名前で追加するのが自然です。

```text
docs/spec/sim_controller_no8_app_modes.md
```

## `plant/`

| サブフォルダ | 内容 | 状態 |
|---|---|---|
| `plant/private/` | private IPMの実行用データ | active/default |
| `plant/prius/` | Prius IPMの候補plantデータ | candidate/generating |
| `plant/im_minimal/` | 最小線形Gamma IMパラメータ | IM評価用 |
| `plant/generator/` | plantデータ・controller table生成ツール | 生成補助 |
| `plant/catalog.toml` | plant名からmanifestへの解決 | runnerの入口 |

IPM plantの基本構成は次です。

```text
plant/<variant>/
├── plant.toml
├── motor_params.csv
├── inverse_flux_map_id.csv
├── inverse_flux_map_iq.csv
└── controller/
    ├── table1_idqcmd.csv
    ├── table2_vdq_cmd.csv
    ├── matrix_trqest.csv
    ├── max_motoring_torque_map_Nm.csv
    └── max_regeneration_torque_abs_map_Nm.csv
```

| データ | 内容 |
|---|---|
| `motor_params.csv` | `Rs`、`Ld`、`Lq`、極対数など |
| `inverse_flux_map_*` | 磁束から電流を求める逆磁束map |
| `table1_idqcmd.csv` | トルク指令から`Id/Iq`指令を得るtable |
| `table2_vdq_cmd.csv` | Vdq feed-forward table |
| `matrix_trqest.csv` | トルク推定table |
| `max_*_torque_map` | 力行・回生最大トルクtable |

## `python/`

| 構成 | 内容 |
|---|---|
| `generate_controller_tables.py` | plantデータからC const tableを生成 |
| `generate_torque_command_scenario.py` | 標準トルク指令scenarioを生成 |
| `compare_cpp_rust.py` | C++/Rust出力比較 |
| `convert_flux_map.py` | 磁束map変換 |
| `csv_to_viewer_parquet.py` | viewer用Parquet変換 |
| `plot_*.py` | 波形・sweep結果のplot |
| `estimate_params.py` | パラメータ推定補助 |
| `python/tests/` | Python toolのテスト |
| `requirements.txt` | base固有のPython依存 |

Pythonは`motor_simulation_base/.venv`を使用します。

## `report/`

| レポート | 内容 |
|---|---|
| `c_pi_trqcmd_lpf_report.md` | torque command LPF評価 |
| `c_pi_pwm_duty_boundary_step8_9_report.md` | Duty境界化評価 |
| `c_pi_pwm_ff_evaluation_report.md` | Vdq feed-forward評価 |
| `README.md` | レポート索引と追加方針 |

`report/`は評価結果、`design/`は設計経緯、`docs/spec/`は現在仕様という役割分担です。

## `rust/`

Rust workspaceの構成です。

| フォルダ | package | 内容 |
|---|---|---|
| `rust/src/` | `motor_sim_rust` | `motor_sim_core`の互換re-export facade |
| `rust/tests/` | `motor_sim_rust` | facade境界のテスト |
| `rust/crates/sim-core/` | `motor_sim_core` | plant、PWM、INV、sensor、型、座標変換 |
| `rust/crates/sim-controller-c/` | `sim-controller-c` | standalone C PIのbuild・FFI wrapper |
| `rust/crates/sim-controller-no8/` | `sim-controller-no8` | No8 Phase7 C controllerのbuild・FFI wrapper |
| `rust/crates/sim-runner/` | `motor_sim_runner` | CLI、scenario、plant解決、simulation loop |
| `rust/crates/sim-export/` | `sim-export` | CSV→Parquet、summary、debug merge |
| `rust/crates/sim-sweep/` | `sim-sweep` | 複数条件runの実行管理 |
| `rust/target/` | — | Cargoビルド生成物。Git管理外 |

### `sim-core/`

| 主なmodule | 内容 |
|---|---|
| `types` | 電流、電圧、角度、トルクなどの型 |
| `transforms` | 電力不変Clarke/Park変換 |
| `motor_model` | IPM/IM plant |
| `pwm` | carrier、Duty、event、latch |
| `inverter` | inverter bridge/leg/gate |
| `sensor` | 電流・Vdc・角度sample |
| `current_pi` | Rust PI参照実装 |
| `drive` | `MotorDrive`とcontroller trait |
| `flux_map` | 逆磁束mapの読込み・補間 |

### `sim-controller-no8/`

| サブフォルダ | 内容 |
|---|---|
| `src/` | Rust FFI定義、safe wrapper、API・単体テスト |
| `c/` | No8 Phase7 C controller本体 |
| `c/const/` | plant別のコンパイル時定数・controller table |
| `c/common/` | 共通制御部品 |
| `c/ipm/` | 標準No8 IPM親制御 |
| `c/im-cvc/` | No8 IM CVC |
| `c/ipm-ekf/` | IPM EKF monitor |
| `c/ipm-kf/` | IPM KF monitor |
| `c/sync-pwm/` | synchronized PWM monitor |
| `c/custom/` | No8 IPM custom wrapperとユーザーhook |

### `sim-controller-no8/c/common/`

| モジュール | 内容 |
|---|---|
| `current_control` | dq電流PI |
| `feedback_transform` | 三相電流からdq電流への変換 |
| `torque_cmd_lpf` | トルク指令LPF |
| `torque_reference` | トルク指令から`Id/Iq`指令生成 |
| `torque_estimation` | 電流からトルク推定 |
| `vdq_feed_forward` | Vdq feed-forward |
| `voltage_limit` | 電圧制限・anti-windup |
| `pwm_duty` | VdqからDuty生成 |
| `carrier_command` | carrier周波数・制御周期管理 |
| `flux_map` | forward flux map参照 |
| `maximum_torque_limit` | 最大力行・回生トルク制限 |

### `sim-controller-no8/c/`直下

| ファイル | 内容 |
|---|---|
| `main.c` | Rustから呼ばれる公開C ABI |
| `controller_app_phase7.c` | `app_mode`によるモード分岐 |
| `controller_phase7.h` | 公開ABIとモード定数 |
| `periodic_task_phase7.c` | 100 µs～10 ms周期scheduler |
| `rte_phase7.c` | Rust/C公開入出力と内部信号の変換 |
| `phase7_types.h` | C内部state/input/output型 |
| `controller_phase7_optional_debug.h` | debug scratchpad ABI |

## `simulation/`

| サブフォルダ・ファイル | 内容 |
|---|---|
| `simulation/scenarios/` | 単一runの入力scenario |
| `simulation/sweeps/` | 複数条件sweep定義 |
| `simulation/presets/` | viewer・解析preset |
| `simulation/results/` | 実行結果、基準波形、plot |
| `run_scenario.sh` | 通常scenario実行の入口 |
| `run_sweep.sh` | sweep実行の入口 |
| `run_torque_pattern.sh` | 標準トルクpattern実行 |
| `check_phase6_pi_csv_parity.sh` | Phase6 parity確認 |
| `check_phase7_three_way_parity.sh` | Phase7 parity確認 |

`simulation/results/`は、Git管理された基準結果とローカル生成結果が混在する領域です。

## `viewer/`

| 構成 | 内容 |
|---|---|
| `viewer/src/` | Rust GUI実装 |
| `viewer/Cargo.toml` | viewer単独のCargo設定 |
| `viewer/README.md` | 利用方法 |
| `viewer/target/` | ローカルビルド生成物 |

panel-y由来の補助GUIであり、現在のbase制御実装の正ではありません。

## ローカル生成フォルダ

| フォルダ | 内容 | Git管理 |
|---|---|---|
| `.venv/` | base用Python仮想環境 | 対象外 |
| `rust/target/` | Rust workspaceビルド成果物 | 対象外 |
| `viewer/target/` | viewerビルド成果物 | 対象外 |
| `build/` | CMakeビルド成果物。現在は未作成 | 対象外 |
| `.skills/` | 現在空のローカル互換フォルダ | 対象外 |

## ルート直下の主要ファイル

| ファイル | 内容 |
|---|---|
| [`AGENTS.md`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/AGENTS.md) | baseのcanonical作業契約 |
| [`AGENT.md`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/AGENT.md) | `AGENTS.md`への互換入口 |
| [`progress.md`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/progress.md) | トラックごとの現在地 |
| [`history.md`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/history.md) | 完了工程の詳細履歴 |

今回作成する制御モード文書は、実装コードではなく現在の利用・保守仕様なので、[`docs/spec/`](/Users/ochiba/newspace/life/factory/motor-control-ai/motor_simulation_base/docs/spec)へ置くのが適切です。
