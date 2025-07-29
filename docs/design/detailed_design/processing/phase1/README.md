# Phase 1: 基盤モジュール 処理説明書

**フェーズ概要**: Phase 1 - 基盤モジュール  
**作成日**: 2025-07-28  
**更新日**: 2025-07-28  

---

## 📋 フェーズ概要

### 目的
昆虫自動観察システムの基盤となるデータモデル、設定管理、ユーティリティ機能を提供する。

### 対象モジュール
Phase 1では以下の6つのモジュールの処理説明書を作成しています：

---

## 📁 処理説明書一覧

### 1. データモデル系（models/）

#### 1.1 detection_models.py
- **文書番号**: 12-002-PROC-001
- **ファイル**: [detection_models_processing_spec.md](detection_models_processing_spec.md)
- **概要**: YOLOv8検出結果・CSV出力用レコード・個別検出詳細のデータクラス定義

#### 1.2 activity_models.py
- **文書番号**: 12-002-PROC-002
- **ファイル**: [activity_models_processing_spec.md](activity_models_processing_spec.md)
- **概要**: 活動量指標・日次統計・時間別統計のデータクラス定義

#### 1.3 system_models.py
- **文書番号**: 12-002-PROC-003
- **ファイル**: [system_models_processing_spec.md](system_models_processing_spec.md)
- **概要**: システム設定・ログレコードのデータクラス定義

### 2. 設定管理系（config/）

#### 2.1 config_manager.py
- **文書番号**: 12-002-PROC-004
- **ファイル**: [config_manager_processing_spec.md](config_manager_processing_spec.md)
- **概要**: システム設定の読み込み・保存・検証・管理機能

### 3. ユーティリティ系（utils/）

#### 3.1 data_validator.py
- **文書番号**: 12-002-PROC-005
- **ファイル**: [data_validator_processing_spec.md](data_validator_processing_spec.md)
- **概要**: データ検証・品質管理・クリーニング機能

#### 3.2 file_naming.py
- **文書番号**: 12-002-PROC-006
- **ファイル**: [file_naming_processing_spec.md](file_naming_processing_spec.md)
- **概要**: ファイル命名規則・ディレクトリ管理・クリーンアップ機能

---

## 🔗 モジュール間依存関係

```
┌─────────────────────────────────────────────────────────────┐
│                  Phase 1: 基盤モジュール                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  データモデル    │  │   設定管理      │  │  ユーティリティ  │  │
│  │    (models/)    │  │  (config/)     │  │   (utils/)      │  │
│  │                 │  │                │  │                 │  │
│  │ ┌─────────────┐ │  │ ┌─────────────┐ │  │ ┌─────────────┐ │  │
│  │ │ detection_  │ │  │ │ config_     │ │  │ │ data_       │ │  │
│  │ │ models      │ │◄─┼─│ manager     │ │  │ │ validator   │ │  │
│  │ └─────────────┘ │  │ └─────────────┘ │  │ └─────────────┘ │  │
│  │                 │  │        ▲       │  │        ▲        │  │
│  │ ┌─────────────┐ │  │        │       │  │        │        │  │
│  │ │ activity_   │ │◄─┼────────┘       │  │        │        │  │
│  │ │ models      │ │  │                │  │        │        │  │
│  │ └─────────────┘ │  │                │  │ ┌─────────────┐ │  │
│  │                 │  │                │  │ │ file_       │ │  │
│  │ ┌─────────────┐ │  │                │  │ │ naming      │ │  │
│  │ │ system_     │ │◄─┼────────────────┼──┼─│             │ │  │
│  │ │ models      │ │  │                │  │ └─────────────┘ │  │
│  │ └─────────────┘ │  │                │  │                 │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 依存関係説明
- **config_manager** → **system_models**: システム設定データクラスを使用
- **data_validator** → **detection_models, activity_models**: 各データモデルの検証を実行
- **file_naming** → **system_models**: ログファイル名生成時にシステム設定を参照

---

## 📊 Phase 1 実装状況

| モジュール | 処理説明書 | 実装状況 | テスト状況 |
|-----------|-----------|---------|----------|
| detection_models.py | ✅ 完了 | ✅ 実装済み | ✅ テスト済み |
| activity_models.py | ✅ 完了 | ✅ 実装済み | ✅ テスト済み |
| system_models.py | ✅ 完了 | ✅ 実装済み | ✅ テスト済み |
| config_manager.py | ✅ 完了 | ✅ 実装済み | ✅ テスト済み |
| data_validator.py | ✅ 完了 | ✅ 実装済み | ✅ テスト済み |
| file_naming.py | ✅ 完了 | ✅ 実装済み | ✅ テスト済み |

---

## 🔧 主要機能サマリー

### データモデル機能
- YOLOv8検出結果の構造化データ管理
- 活動量・統計データの体系的管理
- システム設定・ログの一元管理

### 設定管理機能
- JSON形式設定ファイルの読み書き
- 設定値の検証・バックアップ
- デフォルト設定の自動生成

### ユーティリティ機能
- 各種データの検証・クリーニング
- 統一的なファイル命名規則
- ディレクトリ構造の自動管理

---

## 📝 使用例

### 基本的な使用パターン

```python
# 1. 設定管理
from config.config_manager import ConfigManager
config_manager = ConfigManager()
config = config_manager.load_config()

# 2. 検出結果データ作成
from models.detection_models import DetectionResult
detection = DetectionResult(100.0, 150.0, 50.0, 40.0, 0.85, 0, "2025-07-28T10:30:00")

# 3. データ検証
from utils.data_validator import DataValidator
validator = DataValidator()
is_valid, errors = validator.validate_detection_result(detection)

# 4. ファイル名生成
from utils.file_naming import FileNamingConvention
from datetime import date
filename = FileNamingConvention.generate_detection_log_filename(date.today())
```

---

## 🔄 更新履歴

| 日付 | 更新者 | 更新内容 |
|------|--------|----------|
| 2025-07-28 | 開発チーム | Phase 1処理説明書一式作成 |

---

## 📚 関連文書

- [設計書作成標準規約](../../design_document_standards.md)
- [プロジェクトドキュメント管理標準規約](../../../document_management_standards.md)
- [システムアーキテクチャ設計書](../../basic_design/architecture/system_architecture_design.md)