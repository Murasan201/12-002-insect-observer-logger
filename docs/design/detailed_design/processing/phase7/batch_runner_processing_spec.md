# batch_runner.py 処理説明書

**文書番号**: 12-002-PROC-013  
**プロジェクト名**: 昆虫自動観察＆ログ記録アプリ  
**文書名**: batch_runner.py 処理説明書  
**対象ファイル**: `batch_runner.py`  
**バージョン**: 1.0  
**作成日**: 2025-07-29  
**作成者**: 開発チーム  

---

## 📋 ファイル概要

### 目的
cronやタスクスケジューラからの定期実行、バッチ処理スクリプトの実行をサポートし、システムの自動化・運用管理を実現する。

### 主要機能
- 定期実行スケジューリング（間隔・日次・週次実行）
- 複数ジョブの並列実行・ジョブキュー管理
- エラーハンドリング・リトライ・タイムアウト処理
- 実行ログ記録・結果追跡・通知機能
- CLI ベースのジョブ管理・設定変更
- cron統合・外部スケジューラ連携

---

## 🔧 関数・メソッド仕様

### BatchRunner.__init__(config_path)

**概要**: バッチ処理実行エンジンの初期化

**処理内容**:
1. ロガー設定・設定ファイルパスの保存
2. ジョブ辞書・実行状態フラグの初期化
3. スケジューラースレッド管理の初期化
4. ジョブ設定読み込み（_load_job_config）
5. シグナルハンドラー設定（SIGINT, SIGTERM）

**入力インターフェース**:
```python
def __init__(self, config_path: str = "./config/batch_config.json"):
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| config_path | str | × | バッチ設定ファイルパス（デフォルト: batch_config.json） |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| なし | None | インスタンス初期化 |

---

### _load_job_config()

**概要**: ジョブ設定読み込み

**処理内容**:
1. 設定ファイルの存在確認・JSON読み込み
2. ジョブデータの解析・BatchJobインスタンス作成
3. ジョブ辞書への登録・ログ出力
4. 設定ファイル未存在時のデフォルト設定作成

**入力インターフェース**:
```python
def _load_job_config(self) -> None:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| なし | None | 設定読み込み完了 |

---

### _create_default_config()

**概要**: デフォルト設定作成

**処理内容**:
1. デフォルトジョブリストの定義
2. 設定辞書の作成・ディレクトリ作成
3. JSON形式での設定ファイル保存
4. 設定再読み込み（_load_job_config）

**デフォルトジョブ**:
- `hourly_detection`: 1時間ごとの検出実行
- `daily_analysis`: 深夜2時の日次分析
- `weekly_cleanup`: 日曜日の週次クリーンアップ
- `daily_backup`: 深夜3時のバックアップ（無効）

**入力インターフェース**:
```python
def _create_default_config(self) -> None:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| なし | None | デフォルト設定作成完了 |

---

### add_job(job)

**概要**: ジョブ追加

**処理内容**:
1. ジョブ辞書への追加・設定保存
2. スケジューラー動作中時の動的スケジュール設定
3. 追加完了ログの出力

**入力インターフェース**:
```python
def add_job(self, job: BatchJob) -> None:
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| job | BatchJob | ○ | 追加するバッチジョブオブジェクト |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| なし | None | ジョブ追加完了 |

**使用例**:
```python
new_job = BatchJob(
    name="custom_task",
    command="python custom_script.py",
    schedule_type="daily",
    schedule_time="12:00"
)
runner.add_job(new_job)
```

---

### remove_job(job_name)

**概要**: ジョブ削除

**処理内容**:
1. ジョブ存在確認・辞書からの削除
2. 設定保存・スケジューラーからの削除
3. 削除結果の返却・ログ出力

**入力インターフェース**:
```python
def remove_job(self, job_name: str) -> bool:
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| job_name | str | ○ | 削除対象ジョブ名 |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| success | bool | 削除成功可否 |

---

### enable_job(job_name, enabled)

**概要**: ジョブ有効/無効化

**処理内容**:
1. ジョブ存在確認・有効フラグの更新
2. 設定保存・有効時のスケジュール設定
3. 無効時のスケジュール削除
4. 状態変更結果の返却・ログ出力

**入力インターフェース**:
```python
def enable_job(self, job_name: str, enabled: bool = True) -> bool:
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| job_name | str | ○ | 対象ジョブ名 |
| enabled | bool | × | 有効フラグ（デフォルト: True） |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| success | bool | 状態変更成功可否 |

---

### _setup_job_schedule(job)

**概要**: ジョブスケジュール設定

**処理内容**:
1. ジョブ有効性確認・既存スケジュールクリア
2. ジョブ実行関数の定義（lambda）
3. スケジュールタイプ別の設定：
   - interval: 秒間隔での実行
   - daily: 指定時刻での日次実行
   - weekly: 指定曜日での週次実行
4. スケジュール登録・ログ出力

**入力インターフェース**:
```python
def _setup_job_schedule(self, job: BatchJob) -> None:
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| job | BatchJob | ○ | スケジュール設定対象ジョブ |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| なし | None | スケジュール設定完了 |

---

### _run_job(job)

**概要**: ジョブ実行

**処理内容**:
1. 実行開始時刻記録・BatchResult初期化
2. subprocess によるコマンド実行・タイムアウト設定（30分）
3. 実行結果の判定（成功・エラー・タイムアウト）
4. ジョブ状態更新・設定保存
5. 実行結果記録・エラー通知（3回連続失敗時）

**入力インターフェース**:
```python
def _run_job(self, job: BatchJob) -> BatchResult:
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| job | BatchJob | ○ | 実行対象ジョブ |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| result | BatchResult | ジョブ実行結果 |

**実行状態**:
- `running`: 実行中
- `success`: 実行成功
- `error`: 実行エラー
- `timeout`: タイムアウト

---

### run_scheduler()

**概要**: スケジューラー実行

**処理内容**:
1. 実行フラグ設定・スケジューラー開始ログ
2. 全ジョブのスケジュール設定
3. スケジューラーループ（1秒間隔でのrun_pending）
4. 例外処理・終了時のクリーンアップ

**入力インターフェース**:
```python
def run_scheduler(self) -> None:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| なし | None | スケジューラー終了時 |

**使用例**:
```python
runner = BatchRunner()
runner.run_scheduler()  # スケジューラー開始（ブロッキング）
```

---

### run_job_immediately(job_name)

**概要**: ジョブ即時実行

**処理内容**:
1. ジョブ存在確認・エラー時のNone返却
2. ジョブ実行（_run_job）・結果返却

**入力インターフェース**:
```python
def run_job_immediately(self, job_name: str) -> Optional[BatchResult]:
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| job_name | str | ○ | 実行対象ジョブ名 |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| result | Optional[BatchResult] | 実行結果（ジョブなし時None） |

**使用例**:
```python
result = runner.run_job_immediately("daily_analysis")
if result and result.status == "success":
    print("分析が正常に完了しました")
```

---

### get_job_status()

**概要**: 全ジョブ状態取得

**処理内容**:
1. 全ジョブの状態情報収集
2. 次回実行時刻の取得（schedule.jobs 検索）
3. ジョブ状態リストの作成・返却

**入力インターフェース**:
```python
def get_job_status(self) -> List[Dict[str, Any]]:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| status_list | List[Dict[str, Any]] | 全ジョブ状態リスト |

**戻り値構造**:
```python
[{
    "name": str,              # ジョブ名
    "enabled": bool,          # 有効フラグ
    "schedule_type": str,     # スケジュールタイプ
    "schedule_time": str,     # スケジュール時刻
    "last_run": str,          # 最終実行時刻
    "last_status": str,       # 最終実行状態
    "error_count": int,       # エラー回数
    "next_run": str           # 次回実行時刻
}]
```

---

### _save_job_result(result)

**概要**: ジョブ実行結果保存

**処理内容**:
1. ログディレクトリの作成・確認
2. 日付別JSONLファイルの生成
3. 実行結果のJSONL形式追記保存
4. エラー時のログ出力

**入力インターフェース**:
```python
def _save_job_result(self, result: BatchResult) -> None:
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| result | BatchResult | ○ | 保存対象実行結果 |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| なし | None | 結果保存完了 |

**保存先**: `./logs/batch/batch_YYYYMMDD.jsonl`

---

### create_cron_entry(job_name, schedule_time, python_path)

**概要**: cronエントリ生成

**処理内容**:
1. スクリプトパスの取得
2. スケジュール時刻のcron形式変換：
   - 時刻形式（HH:MM）: 日次実行
   - 数値形式: 間隔実行（分単位変換）
3. cronエントリ文字列の生成・返却

**入力インターフェース**:
```python
def create_cron_entry(job_name: str, schedule_time: str, python_path: str = "python") -> str:
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| job_name | str | ○ | ジョブ名 |
| schedule_time | str | ○ | スケジュール時刻 |
| python_path | str | × | Pythonパス（デフォルト: "python"） |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| cron_entry | str | cron設定エントリ |

**使用例**:
```python
entry = create_cron_entry("daily_analysis", "02:00")
print(entry)  # "0 2 * * * cd /path && python batch_runner.py run-job daily_analysis"
```

---

## 🖥️ CLI機能仕様

### main()

**概要**: メイン関数・CLI エントリーポイント

**処理内容**:
1. ArgumentParser による引数解析・サブコマンド定義
2. ログ設定・BatchRunner インスタンス作成
3. サブコマンド分岐処理・結果出力
4. 例外処理・終了コード返却

**サブコマンド**:
- `scheduler`: スケジューラー起動
- `run-job <job_name>`: ジョブ即時実行
- `list`: ジョブ一覧表示
- `add <name> <command>`: ジョブ追加
- `remove <job_name>`: ジョブ削除
- `enable/disable <job_name>`: ジョブ有効/無効化
- `cron <job_name>`: cronエントリ生成

**共通オプション**:
- `--config, -c`: バッチ設定ファイル
- `--log-level, -l`: ログレベル

**使用例**:
```bash
# スケジューラー起動
python batch_runner.py scheduler

# ジョブ即時実行
python batch_runner.py run-job daily_analysis

# ジョブ一覧表示
python batch_runner.py list

# ジョブ追加
python batch_runner.py add backup_task "python backup.py" --type daily --time "03:00"

# cronエントリ生成
python batch_runner.py cron daily_analysis
```

---

## 📊 データ構造

### BatchJob

**概要**: バッチジョブ定義

**属性**:
| 属性名 | 型 | 説明 |
|-------|---|------|
| name | str | ジョブ名（一意識別子） |
| command | str | 実行コマンド |
| schedule_type | str | スケジュールタイプ（interval/daily/weekly） |
| schedule_time | str | スケジュール時刻（"10:30"/"60"/"sunday"） |
| enabled | bool | 有効フラグ |
| last_run | Optional[str] | 最終実行時刻（ISO形式） |
| last_status | Optional[str] | 最終実行状態 |
| error_count | int | エラー回数 |

### BatchResult

**概要**: バッチ実行結果

**属性**:
| 属性名 | 型 | 説明 |
|-------|---|------|
| job_name | str | ジョブ名 |
| start_time | str | 実行開始時刻（ISO形式） |
| end_time | str | 実行終了時刻（ISO形式） |
| status | str | 実行状態（success/error/timeout） |
| output | str | 標準出力 |
| error | Optional[str] | エラーメッセージ |

---

## 🔄 処理フロー

### スケジューラー実行フロー
```
スケジューラー起動
   ↓
設定読み込み・ジョブ登録
   ↓
全ジョブのスケジュール設定
   ├─ interval: 秒間隔実行
   ├─ daily: 指定時刻実行
   └─ weekly: 指定曜日実行
   ↓
スケジューラーループ（1秒間隔）
   ├─ schedule.run_pending()
   ├─ 実行対象ジョブの検出
   └─ ジョブ実行・結果記録
   ↓
シグナル受信時の終了処理
```

### ジョブ実行フロー
```
ジョブ実行開始
   ↓
BatchResult初期化・開始時刻記録
   ↓
subprocess でコマンド実行
   ├─ 標準出力・エラー出力キャプチャ
   ├─ 30分タイムアウト設定
   └─ 戻り値チェック
   ↓
実行結果判定
   ├─ 戻り値0: success
   ├─ 戻り値非0: error
   └─ タイムアウト: timeout
   ↓
ジョブ状態更新・結果保存
   ├─ エラー回数更新
   ├─ 最終実行時刻記録
   ├─ 設定ファイル保存
   └─ JSONLログ出力
   ↓
連続エラー時の通知処理（3回以上）
```

### ジョブ管理フロー
```
CLI コマンド実行
   ↓
引数解析・サブコマンド分岐
   ├─ add: ジョブ追加・設定保存
   ├─ remove: ジョブ削除・スケジュール削除
   ├─ enable/disable: 有効性変更・スケジュール更新
   ├─ list: ジョブ状態一覧表示
   ├─ run-job: 即時実行・結果表示
   └─ cron: cronエントリ生成
   ↓
実行結果・エラーメッセージ出力
   ↓
終了コード返却
```

---

## ⚙️ 設定ファイル仕様

### batch_config.json 構造
```json
{
  "jobs": [
    {
      "name": "hourly_detection",
      "command": "python main.py --mode single",
      "schedule_type": "interval",
      "schedule_time": "3600",
      "enabled": true,
      "last_run": null,
      "last_status": null,
      "error_count": 0
    },
    {
      "name": "daily_analysis",
      "command": "python main.py --mode analysis",
      "schedule_type": "daily",
      "schedule_time": "02:00",
      "enabled": true,
      "last_run": null,
      "last_status": null,
      "error_count": 0
    }
  ]
}
```

### ログファイル仕様

**ファイル名**: `./logs/batch/batch_YYYYMMDD.jsonl`  
**形式**: JSONL（1行1レコードのJSON）

**レコード例**:
```json
{"job_name": "daily_analysis", "start_time": "2025-07-29T02:00:00", "end_time": "2025-07-29T02:05:30", "status": "success", "output": "Analysis completed successfully", "error": null}
```

---

## 🔗 外部連携機能

### cron 統合
```bash
# 生成されたcronエントリの例
0 2 * * * cd /path/to/project && python batch_runner.py run-job daily_analysis
*/60 * * * * cd /path/to/project && python batch_runner.py run-job hourly_detection
```

### systemd 統合
```ini
[Unit]
Description=Insect Observer Batch Scheduler
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 /path/to/batch_runner.py scheduler
Restart=always
User=observer

[Install]
WantedBy=multi-user.target
```

---

## 📝 実装メモ

### 注意事項
- subprocess による安全なコマンド実行・タイムアウト制御
- ジョブ並列実行時のリソース競合回避
- エラー回数制限・通知機能による障害対応
- 設定ファイル・ログファイルの適切な管理

### 依存関係
- schedule: Pythonスケジューリングライブラリ
- subprocess: 外部コマンド実行
- argparse: CLI引数解析
- threading: スケジューラースレッド管理
- pathlib: ファイルパス操作

---

## 🔄 更新履歴

| バージョン | 更新日 | 更新者 | 更新内容 |
|-----------|--------|--------|----------|
| 1.0 | 2025-07-29 | 開発チーム | 初版作成 |