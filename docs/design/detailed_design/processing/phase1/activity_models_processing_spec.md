# activity_models.py 処理説明書

**文書番号**: 12-002-PROC-002  
**プロジェクト名**: 昆虫自動観察＆ログ記録アプリ  
**文書名**: activity_models.py 処理説明書  
**対象ファイル**: `models/activity_models.py`  
**バージョン**: 1.0  
**作成日**: 2025-07-28  
**作成者**: 開発チーム  

---

## 📋 ファイル概要

### 目的
昆虫の活動量算出・分析に関連するデータ構造を定義し、活動量指標や日次・時間別統計データの管理を提供する。

### 主要機能
- 活動量指標データクラス定義
- 日次活動統計データクラス定義
- 時間別活動統計データクラス定義
- データ検証・変換・統計計算機能

---

## 🔧 関数・メソッド仕様

### ActivityMetrics.__post_init__()

**概要**: ActivityMetricsデータクラスの初期化後検証処理

**処理内容**:
1. total_detectionsが非負数であることをチェック
2. total_distanceが非負数であることをチェック
3. activity_durationが非負数であることをチェック

**入力インターフェース**:
```python
def __post_init__(self) -> None:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| なし | None | 検証完了後に処理終了 |

**例外**:
| 例外名 | 発生条件 |
|-------|---------|
| ValueError | total_detections, total_distance, activity_durationが負数の場合 |

### ActivityMetrics.to_dict()

**概要**: ActivityMetricsオブジェクトを辞書形式に変換

**処理内容**:
1. 全属性を辞書のキー・値ペアに変換
2. 辞書オブジェクトを返却

**入力インターフェース**:
```python
def to_dict(self) -> Dict[str, Any]:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| metrics_dict | Dict[str, Any] | 活動量指標の辞書表現 |

**使用例**:
```python
metrics = ActivityMetrics(50, 1250.5, 25.5, "14:00-15:00", 180.0)
metrics_dict = metrics.to_dict()
```

### ActivityMetrics.calculate_activity_rate()

**概要**: 時間当たりの活動率を計算

**処理内容**:
1. 総検出回数を活動継続時間（時間単位）で割る
2. 1時間当たりの平均検出回数を算出

**入力インターフェース**:
```python
def calculate_activity_rate(self) -> float:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| activity_rate | float | 時間当たりの活動率（検出回数/時間） |

**例外**:
| 例外名 | 発生条件 |
|-------|---------|
| ZeroDivisionError | activity_durationが0の場合 |

**使用例**:
```python
metrics = ActivityMetrics(50, 1250.5, 25.5, "14:00-15:00", 180.0)
rate = metrics.calculate_activity_rate()
```

### DailyActivitySummary.add_hourly_data()

**概要**: 時間別データを日次サマリーに追加

**処理内容**:
1. HourlyActivitySummaryオブジェクトを受け取る
2. hourly_summariesリストに追加
3. 統計データの再計算を実行

**入力インターフェース**:
```python
def add_hourly_data(self, hourly_data: HourlyActivitySummary) -> None:
```

| 引数名 | 型 | 必須 | 説明 |
|-------|---|------|------|
| hourly_data | HourlyActivitySummary | ○ | 追加する時間別データ |

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| なし | None | データ追加完了後に処理終了 |

**使用例**:
```python
daily_summary = DailyActivitySummary(...)
hourly_data = HourlyActivitySummary(...)
daily_summary.add_hourly_data(hourly_data)
```

### DailyActivitySummary.calculate_peak_hours()

**概要**: ピーク活動時間帯を計算

**処理内容**:
1. 各時間別データの検出回数を比較
2. 最も活動量の多い時間帯を特定
3. ピーク時間帯のリストを生成

**入力インターフェース**:
```python
def calculate_peak_hours(self) -> List[str]:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| peak_hours | List[str] | ピーク活動時間帯のリスト |

**使用例**:
```python
daily_summary = DailyActivitySummary(...)
peak_hours = daily_summary.calculate_peak_hours()
```

### DailyActivitySummary.generate_summary_stats()

**概要**: 日次統計サマリーを生成

**処理内容**:
1. 時間別データから平均・最大・最小値を算出
2. 標準偏差を計算
3. 統計サマリー辞書を生成

**入力インターフェース**:
```python
def generate_summary_stats(self) -> Dict[str, float]:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| stats | Dict[str, float] | 統計サマリー辞書 |

**使用例**:
```python
daily_summary = DailyActivitySummary(...)
stats = daily_summary.generate_summary_stats()
```

### HourlyActivitySummary.calculate_detection_density()

**概要**: 時間当たりの検出密度を計算

**処理内容**:
1. 検出回数を時間（60分）で割る
2. 分当たりの検出密度を算出

**入力インターフェース**:
```python
def calculate_detection_density(self) -> float:
```

**出力インターフェース**:
| 戻り値 | 型 | 説明 |
|-------|---|------|
| density | float | 分当たりの検出密度 |

**使用例**:
```python
hourly_summary = HourlyActivitySummary(...)
density = hourly_summary.calculate_detection_density()
```

---

## 📊 データ構造

### ActivityMetrics

**概要**: 活動量指標を表現するデータクラス

**属性**:
| 属性名 | 型 | 説明 |
|-------|---|------|
| total_detections | int | 総検出回数 |
| total_distance | float | 総移動距離（ピクセル） |
| avg_activity_per_hour | float | 時間別平均活動量 |
| peak_activity_time | str | ピーク活動時間帯 |
| activity_duration | float | 活動継続時間（分） |
| movement_patterns | List[str] | 移動パターンリスト |

### DailyActivitySummary

**概要**: 日次活動統計を表現するデータクラス

**属性**:
| 属性名 | 型 | 説明 |
|-------|---|------|
| target_date | date | 対象日付 |
| total_detections | int | 日次総検出回数 |
| total_distance | float | 日次総移動距離 |
| active_hours_count | int | 活動時間数 |
| peak_activity_hours | List[str] | ピーク活動時間帯リスト |
| hourly_summaries | List[HourlyActivitySummary] | 時間別サマリーリスト |

### HourlyActivitySummary

**概要**: 時間別活動統計を表現するデータクラス

**属性**:
| 属性名 | 型 | 説明 |
|-------|---|------|
| hour | int | 対象時間（0-23） |
| detections_count | int | 時間内検出回数 |
| distance_moved | float | 時間内移動距離 |
| avg_confidence | float | 平均信頼度 |
| detection_intervals | List[float] | 検出間隔リスト（分） |

---

## 🔄 処理フロー

### メイン処理フロー
```
1. ActivityMetricsオブジェクト生成
   ↓
2. __post_init__による自動検証
   ↓
3. to_dict()による辞書変換
   ↓
4. calculate_activity_rate()による活動率算出
```

### 日次サマリー処理フロー
```
1. DailyActivitySummaryオブジェクト生成
   ↓
2. add_hourly_data()で時間別データ追加
   ↓
3. calculate_peak_hours()でピーク時間算出
   ↓
4. generate_summary_stats()で統計生成
```

### エラー処理フロー
```
データ検証エラー発生
   ↓
ValueError例外をスロー
   ↓
呼び出し元でエラーハンドリング
```

---

## 📝 実装メモ

### 注意事項
- field(default_factory=list)によりリスト属性の初期化を適切に行う
- statisticsモジュールを使用して統計計算を実行
- 時間計算では分単位と時間単位の変換に注意

### 依存関係
- dataclasses（標準ライブラリ）
- datetime（標準ライブラリ）
- typing（標準ライブラリ）
- statistics（標準ライブラリ）

---

## 🔄 更新履歴

| バージョン | 更新日 | 更新者 | 更新内容 |
|-----------|--------|--------|----------|
| 1.0 | 2025-07-28 | 開発チーム | 初版作成 |