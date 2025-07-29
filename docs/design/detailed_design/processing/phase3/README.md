# Phase 3: 検出機能 処理説明書

**フェーズ概要**: Phase 3 - 検出機能  
**作成日**: 2025-07-28  
**更新日**: 2025-07-28  

---

## 📋 フェーズ概要

### 目的
YOLOv8を使用した昆虫検出処理の実行と制御を行い、検出結果の後処理・品質管理・モデル管理を統合的に提供する。

### 対象モジュール
Phase 3では以下の3つのモジュールの処理説明書を作成しています：

---

## 📁 処理説明書一覧

### 1. 昆虫検出処理

#### 1.1 insect_detector.py
- **文書番号**: 12-002-PROC-201
- **ファイル**: [insect_detector_processing_spec.md](insect_detector_processing_spec.md)
- **概要**: YOLOv8を使用した昆虫検出処理の実行・カメラ制御連携・バッチ処理対応

### 2. 検出結果処理

#### 2.1 detection_processor.py
- **文書番号**: 12-002-PROC-202
- **ファイル**: [detection_processor_processing_spec.md](detection_processor_processing_spec.md)
- **概要**: 検出結果の後処理・フィルタリング・品質評価・統計分析・CSV出力

### 3. モデル管理

#### 3.1 model_manager.py
- **文書番号**: 12-002-PROC-203
- **ファイル**: [model_manager_processing_spec.md](model_manager_processing_spec.md)
- **概要**: YOLOv8モデルの管理・Hugging Faceダウンロード・検証・変換・最適化

---

## 🔗 モジュール間依存関係

```
┌─────────────────────────────────────────────────────────────┐
│                 Phase 3: 検出機能                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  昆虫検出処理    │  │  検出結果処理    │  │  モデル管理      │  │
│  │                 │  │                │  │                 │  │
│  │ ┌─────────────┐ │  │ ┌─────────────┐ │  │ ┌─────────────┐ │  │
│  │ │ insect_     │ │──┼─│ detection_  │ │  │ │ model_      │ │  │
│  │ │ detector    │ │  │ │ processor   │ │  │ │ manager     │ │  │
│  │ │             │ │  │ │             │ │◄─┼─│             │ │  │
│  │ │ ・YOLO推論   │ │  │ │ ・フィルタ   │ │  │ │ ・HF管理     │ │  │
│  │ │ ・画像撮影   │ │  │ │ ・品質評価   │ │  │ │ ・検証      │ │  │
│  │ │ ・バッチ処理 │ │  │ │ ・CSV出力   │ │  │ │ ・変換      │ │  │
│  │ └─────────────┘ │  │ └─────────────┘ │  │ └─────────────┘ │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│           │                      │                      ▲        │
│           │                      │                      │        │
│  ┌─────────┼──────────────────────┼──────────────────────┼──────┐  │
│  │         ▼                      ▼                      │      │  │
│  │      外部依存関係                                     │      │  │
│  │                                                       │      │  │
│  │ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌──────┴────┐ │  │
│  │ │ Phase 2     │ │ Phase 1     │ │ ultralytics │ │ huggingface│ │  │
│  │ │ ハードウェア │ │ 基盤モジュール│ │ (YOLOv8)    │ │ _hub      │ │  │
│  │ └─────────────┘ └─────────────┘ └─────────────┘ └───────────┘ │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 依存関係説明
- **insect_detector** → **model_manager**: YOLOモデルの読み込み・管理
- **insect_detector** → **detection_processor**: 検出結果の後処理・品質管理
- **detection_processor** → **Phase 1 models**: DetectionResult・DetectionRecord データクラス使用
- **insect_detector** → **Phase 2 hardware**: カメラ・LED制御との連携
- **model_manager** → **huggingface_hub**: Hugging Faceからのモデルダウンロード
- 全モジュール → **ultralytics**: YOLOv8ライブラリ使用

---

## 📊 Phase 3 実装状況

| モジュール | 処理説明書 | 実装状況 | テスト状況 |
|-----------|-----------|---------|----------|
| insect_detector.py | ✅ 完了 | ✅ 実装済み | ✅ テスト済み |
| detection_processor.py | ✅ 完了 | ✅ 実装済み | ✅ テスト済み |
| model_manager.py | ✅ 完了 | ✅ 実装済み | ✅ テスト済み |

---

## 🔧 主要機能サマリー

### 昆虫検出処理（insect_detector.py）
- YOLOv8モデルによる高精度昆虫検出
- カメラ・IR LED制御との統合連携
- バッチ処理・リアルタイム処理対応
- 検出品質評価・統計収集

### 検出結果処理（detection_processor.py）
- 信頼度・サイズ・重複による高度フィルタリング
- IoU計算による重複検出除去
- 時系列パターン分析・活動量算出
- CSV形式での検出ログ出力

### モデル管理（model_manager.py）
- Hugging Face Hubからの自動モデルダウンロード
- モデル検証・バージョン管理
- PyTorch → ONNX変換機能
- 性能ベンチマーク・推奨モデル選定

---

## 🤖 AI・機械学習仕様

### 使用モデル
| 項目 | 仕様 |
|------|------|
| **モデルアーキテクチャ** | YOLOv8 (You Only Look Once v8) |
| **学習済みモデル** | Murasan/beetle-detection-yolov8 |
| **入力サイズ** | 640×640 pixels |
| **検出クラス** | beetle (カブトムシ) |
| **性能** | mAP@0.5: 97.63%, mAP@0.5:0.95: 89.56% |

### 検出パラメータ
| パラメータ | デフォルト値 | 説明 |
|-----------|-------------|------|
| **信頼度閾値** | 0.5 | 検出結果採用の最小信頼度 |
| **NMS閾値** | 0.4 | 重複検出除去の閾値 |
| **最大検出数** | 10 | 1画像あたりの最大検出数 |
| **入力解像度** | 640×640 | YOLO入力画像サイズ |

### 後処理設定
| 設定項目 | デフォルト値 | 説明 |
|---------|-------------|------|
| **最小サイズ** | 10×10 pixels | フィルタリング最小サイズ |
| **最大サイズ** | 500×500 pixels | フィルタリング最大サイズ |
| **重複IoU閾値** | 0.7 | 重複検出判定閾値 |
| **品質閾値** | 0.5 | 検出品質スコア閾値 |

---

## 📝 使用例

### 基本的な使用パターン

```python
# 1. モデル管理・ダウンロード
from model_manager import ModelManager

manager = ModelManager("./weights")
model_path = manager.download_from_huggingface(
    "Murasan/beetle-detection-yolov8", 
    "best.pt"
)

# 2. 検出器初期化
from insect_detector import InsectDetector, DetectionSettings
from hardware_controller import HardwareController

settings = DetectionSettings(
    model_path=model_path,
    confidence_threshold=0.6
)
detector = InsectDetector(settings, hardware_controller)
detector.initialize_model()

# 3. カメラ検出実行
detections, image = detector.detect_from_camera(
    led_brightness=0.8, 
    save_image=True
)

# 4. 検出結果後処理
from detection_processor import DetectionProcessor, ProcessingSettings

proc_settings = ProcessingSettings(min_confidence=0.7)
processor = DetectionProcessor(proc_settings)
filtered = processor.process_detections(detections)

# 5. 結果保存・分析
csv_path = processor.save_to_csv(filtered, "./logs")
quality = processor.evaluate_quality(filtered)
```

### バッチ処理例

```python
# 画像ファイル一括検出
import cv2
from pathlib import Path

image_files = list(Path("./input_images").glob("*.jpg"))
images = [cv2.imread(str(f)) for f in image_files]

# バッチ検出実行
batch_results = detector.detect_batch(images, batch_size=4)

# 結果の統合処理
all_detections = []
for results in batch_results:
    filtered = processor.process_detections(results)
    all_detections.extend(filtered)

# 時系列分析
patterns = processor.analyze_temporal_patterns(all_detections, 24)
print(f"ピーク活動時間: {patterns['peak_hours']}")
```

### モデル管理例

```python
# モデル検証・ベンチマーク
verification = manager.verify_model(model_path)
if verification['valid']:
    benchmark = manager.benchmark_model(model_path, 100)
    print(f"平均推論時間: {benchmark['avg_inference_time']:.3f}ms")

# ONNX変換
onnx_path = manager.convert_to_onnx(model_path, "./weights/model.onnx")

# 推奨モデル取得
recommended, reason = manager.get_recommended_model("accuracy")
print(f"推奨: {recommended}, 理由: {reason}")
```

---

## ⚠️ 注意事項

### パフォーマンス要件
- **CPU推論時間**: ≤5秒/画像（1920×1080）
- **GPU推論時間**: ≤1秒/画像（CUDA利用時）
- **メモリ使用量**: ≤2GB（モデル読み込み時）
- **バッチ処理**: 4画像/バッチ推奨

### 品質管理
- **信頼度範囲**: 0.5-1.0（調整可能）
- **検出精度**: True Positive Rate ≥80%
- **誤検出率**: False Positive Rate ≤10%
- **処理安定性**: 50連続画像処理対応

### システム要件
- **Python**: 3.9+
- **PyTorch**: 2.0+
- **ultralytics**: 8.0+
- **CUDA**: 11.8+ (GPU使用時)
- **メモリ**: 4GB+ RAM推奨

---

## 🔄 更新履歴

| 日付 | 更新者 | 更新内容 |
|------|--------|----------|
| 2025-07-28 | 開発チーム | Phase 3処理説明書一式作成 |

---

## 📚 関連文書

- [Phase 1: 基盤モジュール処理説明書](../phase1/README.md)
- [Phase 2: ハードウェア制御処理説明書](../phase2/README.md)
- [設計書作成標準規約](../../design_document_standards.md)
- [システムアーキテクチャ設計書](../../basic_design/architecture/system_architecture_design.md)
- [Hugging Face モデルカード](../../../references/huggingface_model_card.md)