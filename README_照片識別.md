# 🎯 網頁拍照識別功能使用指南

## ✨ 已完成的功能

✅ 在網頁中添加了拍照/上傳照片按鈕（橙色相機按鈕）
✅ 集成 TensorFlow.js 圖像識別功能
✅ 支持自定義模型或使用 MobileNet 預訓練模型
✅ 照片預覽和識別結果顯示
✅ 自動將識別結果填入添加藥品表單

---

## 📱 如何使用（用戶視角）

### 步驟 1：打開網頁
訪問 `wangya/index.html`，在"我的藥品"頁面

### 步驟 2：點擊拍照識別按鈕
右下角有兩個浮動按鈕：
- **綠色 +** = 手動添加藥品
- **橙色 📷** = 拍照識別藥品（新功能！）

### 步驟 3：拍攝或上傳照片
1. 點擊"拍攝或上傳照片"按鈕
2. 選擇：
   - 📷 使用相機拍照（手機）
   - 🖼️ 從相簿選擇（電腦/手機）

### 步驟 4：開始識別
1. 照片上傳後，點擊"開始識別"
2. 等待模型加載（首次使用約 5-10 秒）
3. 查看識別結果和可信度

### 步驟 5：使用結果
1. 如果識別正確，點擊"使用此結果"
2. 系統會自動填入藥品名稱和照片
3. 完善其他信息（劑量、頻率等）
4. 保存藥品

---

## 🔧 技術實現方案

### 方案 A：使用 MobileNet（目前默認，無需配置）

**優點：**
- ✅ 立即可用，無需額外配置
- ✅ 輕量級，加載快速
- ✅ 支持 1000+ 種常見物體

**缺點：**
- ⚠️ 不是專門的藥品識別模型
- ⚠️ 識別結果是通用物體名稱（如 "pill", "bottle"）

**適用場景：**
- 快速原型測試
- 通用物體識別
- 作為備用方案

### 方案 B：使用自定義模型（推薦）

**優點：**
- ✅ 專門針對你的藥品訓練
- ✅ 識別準確度高
- ✅ 可以識別具體藥品名稱

**缺點：**
- ⚠️ 需要轉換 Core ML 模型
- ⚠️ 首次配置需要一些步驟

**適用場景：**
- 生產環境
- 需要高準確度
- 有特定藥品列表

---

## 🚀 如何轉換 Core ML 模型為 TensorFlow.js

### 方法 1：使用轉換腳本（推薦）

我已經創建了 `convert_model.py` 腳本。

#### 步驟：

```bash
# 1. 安裝依賴
pip install coremltools tensorflow tensorflowjs onnx onnx-tf

# 2. 將 MyImageClassifier3.mlmodel 複製到項目根目錄

# 3. 運行轉換腳本
python convert_model.py

# 4. 轉換成功後，模型文件會保存在 wangya/tfjs_model/
```

### 方法 2：使用 Teachable Machine（最簡單）

如果轉換困難，可以使用 Google 的 Teachable Machine 重新訓練：

#### 步驟：

1. **訪問網站**  
   https://teachablemachine.withgoogle.com/

2. **創建項目**  
   選擇 "Image Project" → "Standard image model"

3. **上傳訓練圖片**  
   - 為每種藥品創建一個類別
   - 每個類別上傳 10-50 張照片
   - 建議不同角度、光線的照片

4. **訓練模型**  
   點擊 "Train Model"，等待訓練完成（約 5-10 分鐘）

5. **導出模型**  
   - 點擊 "Export Model"
   - 選擇 "TensorFlow.js"
   - 選擇 "Download" 或使用雲端 URL
   - 下載後解壓到 `wangya/tfjs_model/`

6. **更新標籤**  
   編輯 `index.html`，找到第 4033 行：
   ```javascript
   const labels = ['藥品A', '藥品B', '藥品C']; // 替換為你的實際標籤
   ```
   改為：
   ```javascript
   const labels = ['普拿疼', '維他命C', '感冒藥']; // 你的藥品名稱
   ```

### 方法 3：手動轉換（進階）

如果你熟悉 Python：

```python
import coremltools as ct
import tensorflow as tf
import tensorflowjs as tfjs

# 1. 載入 Core ML 模型
coreml_model = ct.models.MLModel('MyImageClassifier3.mlmodel')

# 2. 獲取模型規格
spec = coreml_model.get_spec()
print("輸入:", spec.description.input)
print("輸出:", spec.description.output)

# 3. Core ML → TensorFlow (需要額外工具)
# 建議使用 ONNX 作為中間格式

# 4. TensorFlow → TensorFlow.js
tfjs.converters.save_keras_model(tf_model, 'wangya/tfjs_model')
```

---

## 📁 文件結構

轉換成功後，你的目錄應該是：

```
yao/
├── wangya/
│   ├── index.html              ✅ 已更新，包含識別功能
│   ├── tfjs_model/             ⬅️ 放置轉換後的模型
│   │   ├── model.json          ⬅️ 模型架構
│   │   ├── group1-shard1of1.bin ⬅️ 模型權重
│   │   └── metadata.json       ⬅️ 元數據（可選）
│   └── README_照片識別.md      ✅ 本說明文件
├── MyImageClassifier3.mlmodel  ⬅️ 原始 Core ML 模型
└── convert_model.py            ✅ 轉換腳本
```

---

## 🧪 測試功能

### 在本地測試：

```bash
# 1. 進入目錄
cd wangya

# 2. 啟動本地服務器
python -m http.server 8080
# 或使用 Python 2
python -m SimpleHTTPServer 8080

# 3. 打開瀏覽器訪問
http://localhost:8080/index.html
```

### 在手機測試：

```bash
# 1. 確保手機和電腦在同一 WiFi
# 2. 獲取電腦 IP（例如 192.168.1.100）
# 3. 在手機瀏覽器訪問
http://192.168.1.100:8080/index.html
```

---

## 🎨 自定義藥品名稱映射

如果使用 MobileNet，可以自定義藥品名稱映射。

編輯 `index.html`，找到第 3865-3873 行：

```javascript
const medicineNameMapping = {
    'pill': '藥丸',
    'tablet': '藥片',
    'capsule': '膠囊',
    'bottle': '藥瓶',
    'medicine': '藥品',
    'drug': '藥物',
    // 添加更多映射
    'aspirin': '阿斯匹靈',
    'painkiller': '止痛藥',
    'vitamin': '維他命'
};
```

---

## ❓ 常見問題

### Q1: 模型加載很慢？
**A:** 首次加載需要下載模型文件（約 5-20MB），之後會緩存。可以：
- 使用更小的模型
- 預加載模型（在頁面載入時）
- 使用 CDN 加速

### Q2: 識別準確度低？
**A:** 
- 使用自定義模型而不是 MobileNet
- 確保訓練圖片多樣化
- 拍攝時確保光線充足、對焦清晰
- 增加訓練樣本數量

### Q3: 手機上無法使用相機？
**A:**
- 確保使用 HTTPS 或 localhost
- 檢查瀏覽器相機權限
- 嘗試不同的瀏覽器（推薦 Chrome/Safari）

### Q4: 識別結果是英文的？
**A:**
- 這是 MobileNet 的默認行為
- 使用 `medicineNameMapping` 添加中文映射
- 或使用自定義模型直接輸出中文名稱

### Q5: 如何添加更多藥品？
**A:**
- 使用 Teachable Machine 重新訓練
- 或在現有模型基礎上增量訓練
- 更新 `labels` 數組

---

## 📊 性能優化建議

### 1. 預加載模型
在頁面加載時就開始加載模型：

```javascript
// 在 init() 函數中添加
init() {
    loadData();
    loadProfile();
    renderMedicines();
    changeHealthType(currentHealthType);
    
    // 預加載識別模型
    loadTensorFlowModel(); // ⬅️ 添加這行
}
```

### 2. 壓縮圖片
在識別前壓縮圖片可以加快速度：

```javascript
// 已在代碼中實現，無需額外配置
```

### 3. 使用 Web Worker
對於大型模型，可以在 Web Worker 中運行：

```javascript
// 進階功能，需要額外配置
```

---

## 🎓 下一步建議

1. **測試基本功能**  
   先使用 MobileNet 測試功能是否正常

2. **轉換自定義模型**  
   使用 Teachable Machine 或轉換腳本

3. **收集訓練數據**  
   為每種藥品拍攝多張照片

4. **訓練和優化**  
   根據識別結果調整模型

5. **部署到生產環境**  
   將模型文件放在穩定的服務器上

---

## 💡 提示

- 📷 拍攝時確保藥品清晰、光線充足
- 🎯 每種藥品至少需要 10-20 張訓練圖片
- 🔄 定期更新模型以提高準確度
- 📱 手機端體驗最佳（支持直接拍照）
- 🌐 需要 HTTPS 才能在生產環境使用相機

---

## 📞 需要幫助？

如果遇到問題：

1. 檢查瀏覽器控制台（F12）查看錯誤信息
2. 確保所有文件路徑正確
3. 驗證模型文件完整性
4. 查看本文檔的"常見問題"部分

---

## 🎉 祝你使用愉快！

現在你的網頁已經具備完整的拍照識別功能了！

**立即測試：**
1. 打開 `wangya/index.html`
2. 點擊橙色相機按鈕 📷
3. 上傳藥品照片
4. 查看識別結果！

---

**更新日期:** 2025-10-19  
**版本:** 1.0  
**作者:** AI Assistant  

