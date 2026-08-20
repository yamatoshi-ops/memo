以下の受領CSVを、基準JMAG-RT fullデータとして処理してください。

受領CSV:
<受領CSVの絶対パス>

目的は、features/motor_model/ に実装済みの工程を使用し、第2報の実データ評価へ進むことです。

実施内容:
1. ワークスペースと features/motor_model/ のAGENTS.md、progress.md、README.md、W4 intake計画、coenergy_forward_fit_validation_plan_v3.md、第2報を読み、現行契約を確認する。
2. 受領CSVを変更せず、行数、列名、型、欠損、重複、pair成立性、試験条件をpreflightする。
3. 列名不一致がある場合は、既存の薄いadapter／列名mappingで吸収できるか確認する。意味を推測して黙って変換しない。
4. W4 intakeを実行し、mirror pair、Iq=0 single、除外行と理由を生成・確認する。
5. 点磁束同定を実行し、identified_flux_points.csvを生成する。
6. 出力には少なくとも次を含める。
   - pair／singleの識別情報
   - id、iq
   - psi_d、psi_q
   - 各磁束成分の有効mask
   - 正側／負側／singleのsource row
   - 元の試験パターン（振幅、進角、目標電流、sequenceなど）
   - 計算不能または除外理由
7. 第2報runnerを実行し、次を評価する。
   - A-Q1: full-data Co-energy fit、残差、25 A格子の磁束map
   - A-Q2: 少数点選択探索
   - A-Q3: whole-ring baselineとの同数比較
8. 実行結果と成果物をQAし、第2報へ実測結果を記入する。結果が不足する問いは未回答として明示し、推測値で埋めない。
9. 必要な範囲のテストを実行し、最後に変更ファイル、生成成果物、評価結果、残課題を報告する。

実行環境:
features/motor_model/.venv/bin/python を明示して使用してください。

制約:
- features/calibration/ は参照のみとし、変更しない。
- 受領CSVは上書きしない。
- 既存の数式coreを再実装せず、既存adapter／runnerを使用する。
- SHAや補助metadataの不足だけを理由に解析を止めない。
- 技術判断に影響する入力不整合が見つかった場合は、その時点で事実と影響範囲を報告する。
- 第3報用のRs違いJMAGデータ、実測anchor、共通validation点は今回の入力に含まれない限り評価対象外とする。
- commit、push、PR更新は、私が明示的に依頼するまで行わない。

まず受領CSVのpreflightと、実行可能性の判定から開始してください。
