# UPVN Universal Intel Collector - 全事業部情報自動化搜集與分析

## 描述
本技能旨在自動化搜集 Uni-President Vietnam (UPVN) 各事業部相關的越南即時情報，將其轉化為結構化資料寫入 `UPVN_INTEL_HUB` 資料庫，並產出主管級的深度分析日報。

## 業務範圍 (Categories)
本技能支持以下 `category_id` 的對接：
- **1**: 水產事業與出口
- **2**: 畜產事業與防疫
- **3**: 大宗原物料採購
- **4**: 食品加工與零售
- **5**: 動保藥規與食安
- **6**: 貿易物流與關稅
- **7**: 經營合規與勞政
- **8**: 永續與智慧化
- **9**: 總體經濟與匯率
- **10**: 其他

## 🛠️ 操作指南 (SOP)

### Step 1: 資源配置 (根據 `category_id` 選擇)

**1. 目標網站清單 (Site List)**
- **若 `id=1` (水產)**：
  `https://thuysanvietnam.com.vn/`, `https://contom.vn/`, `https://seafood.vasep.com.vn/`, `https://vietfishmagazine.com/`, `https://vietnamnet.vn`, `https://www.vietnamplus.vn/`, `https://vietnamnews.vn/`, `https://vneconnews.com/`, `https://nhandan.vn/`, `https://tuoitre.vn/`, `https://thoibaonganhang.vn/`
- **若 `id=2` (畜產)**：
  `https://nhachannuoi.vn/`, `https://nongnghiepmoitruong.vn/chan-nuoi/`, `https://zh.vietnamplus.vn/`, `https://cn.nhandan.vn/`, `https://www.vnfeednews.com/`, `https://www.viv.net/`, `https://www.agripost.cn/`, `https://www.yuanzhezixun.com/`, `https://www.mard.gov.vn/`, `https://freshdi.com/`
- **若 `id` 為其他值**：
  `https://vnexpress.net/`, `https://tuoitre.vn/`, `https://baomoi.com/`, `https://nld.com.vn/`, `https://congan.com.vn/`, `https://thuvienphapluat.vn/`, `https://dichvucong.gov.vn/`

**2. 搜尋關鍵字建議 (越南文)**
- **id=1**: `nuôi trồng thủy sản`, `thị trường thủy sản`, `xuất khẩu thủy sản`
- **id=2**: `chăn nuôi`, `dịch bệnh gia súc`, `vắc xin thú y`, `thị trường chăn nuôi`
- **id=3**: `nguyên liệu đại trà`, `giá ngô`, `giá đậu tương`, `nhập khẩu nguyên liệu`
- **id=4**: `chế biến thực phẩm`, `bán lẻ thực phẩm`, `chuỗi cung ứng thực phẩm`
- **id=5**: `quy định thuốc thú y`, `an toàn thực phẩm`, `kiểm dịch động vật`
- **id=6**: `logistics thương mại`, `thuế quan`, `vận tải biển`, `thủ tục hải quan`
- **id=7**: `tuân thủ kinh doanh`, `luật lao động Việt Nam`, `quản trị nhân sự`
- **id=8**: `phát triển bền vững`, `nông nghiệp thông minh`, `chuyển đổi số nông nghiệp`
- **id=9**: `kinh tế vĩ mô Việt Nam`, `tỷ giá hối đoái`, `lạm phát`, `GDP Việt Nam`
- **id=10**: `tin tức doanh nghiệp Việt Nam`, `thị trường tiêu dùng`

### Step 2: 情報搜集與摘要
1. **執行動作**：
   - 使用 `agent-reach` 或 `multi-search-engine` 搜尋指定日期（參數日期）的相關新聞。
   - **排除項**：嚴格排除與該分類無關的訊息（如 id=1 時排除農業種植）。
   - **數量要求**：搜集 20-30 則單篇新聞（含標題、短摘要、原文連結）。
   - **語言要求**：摘要必須翻譯為繁體中文。

### Step 3: 資料庫寫入 - `portal_news`
- **資料庫**：`postgresql://USER:PASSWORD@HOST:PORT/UPVN_INTEL_HUB (set via DATABASE_URL / env; do not commit credentials)`
- **目標表**：`public.portal_news`
- **寫入策略**：若 `publish_date` 與 `category_id` 已存在對應紀錄，請先刪除該紀錄後再寫入，無需詢問。
- **欄位映射**：
  - `publish_date`: 參數日期
  - `category_id`: 對應的 ID
  - `title`: 基於該分類新聞重點生成具吸引力的標題
  - `content`: 完整 Markdown 格式，每則新聞間隔一行並加上數字標號。

### Step 4: 深度分析與寫入 - `portal_analysis`
1. **分析來源**：結合 Step 2 的新聞與針對該分類 `description` 進行的當日深度檢索。
2. **分析結構 (主管版本日報)**：
   - **趨勢概述**：以 Markdown 格式撰寫。若有多個重點，請以項目編號（1.、2.、3.⋯）呈現，**每一項內容至少需 300 字以上 (繁體中文)**。
   - **決策點 (Decision Points)**：針對主管的具體建議。
   - **風險點 (Risk Factors)**：潛在威脅或市場波動。
   - **可行動的待辦 (Actionable Items)**：接下來 24-48 小時建議採取的行動。
3. **目標表**：`public.portal_analysis`
- **寫入策略**：若 `publish_date` 與 `category_id` 已存在對應紀錄，請先刪除該紀錄後再寫入，無需詢問。
- **欄位映射**：
  - `publish_date`: 參數日期
  - `category_id`: 對應的 ID
  - `title`: 專業分析報告標題 (例如：[日期] [分類名稱] 深度觀察)
  - `content`: 上述分析之完整內容。

## 💾 技術實現細節

### 資料庫寫入建議
建議使用 Python 虛擬環境執行 `psycopg2-binary` 進行寫入，以避免 SQL 注入並確保特殊字元正確處理。

**執行範例**：
```bash
python3 -m venv ~/db_env
~/db_env/bin/pip install psycopg2-binary
~/db_env/bin/python3 insert_script.py
```

## 📌 注意事項
- **資料潔癖**：來源連結必須準確，去除末尾的 `\n` 字元。
- **內容質量**：分析部分禁止空洞的描述，必須基於事實數據（如具體金額、百分比、法規號碼）。
- **適配性**：在執行前，必須先確認 `category_id` 並對應正確的網站清單與關鍵字。
