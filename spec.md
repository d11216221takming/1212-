# 軟體開發規格（SDS）— 單檔塔羅抽牌（index.html）

## 1. 文件資訊
- 文件名稱：spec.md
- 系統名稱：Single-file Tarot Draw（單檔塔羅抽牌）
- 版本：v1.0
- 交付物：
  - `index.html`（單一檔案，包含 HTML + CSS + JS）
  - `spec.md`（本規格文件）

---

## 2. 目標與背景
塔羅牌系統透過「牌面象徵」提供占卜或自我反思的提示。本 v1 以**純前端、單檔**形式實作最小可行版本（MVP），提供：
- 基本 UI（輸入問題、選擇牌陣、抽牌按鈕）
- 隨機抽牌（無放回）
- 正位/逆位（可選）
- 依牌陣位置產生「提示式」簡單敘述（不做絕對預言）

---

## 3. 範圍（Scope）

### 3.1 In Scope（v1 必做）
1. 單頁面 UI（無路由、無多頁）
2. 牌陣：
   - 單張牌（今日提示）
   - 三張牌（過去/現在/未來）
3. 問題輸入（可留空）、類別選擇（可選）
4. 抽牌演算法：
   - 洗牌
   - 抽取 N 張（無放回，不重複）
   - 逆位開關（啟用時每張牌 50% 機率逆位）
5. 顯示抽牌結果：
   - 位置（如：過去/現在/未來）
   - 牌名
   - 正位/逆位
   - 關鍵詞
   - 簡短牌義
6. 產生並顯示總結敘述（規則/模板型）
7. 操作：
   - 開始抽牌
   - 重新抽牌
   - 清除結果

### 3.2 Out of Scope（v1 不做）
- 會員系統、登入/註冊
- 後端 API、資料庫、雲端儲存
- 付費功能、真人老師預約、聊天
- 78 張完整牌庫（可用精簡示例牌庫替代；但要保留可擴充結構）
- AI 自由文本生成（僅規則模板合成）

---

## 4. 使用者故事（User Stories）
- US-01：作為使用者，我想輸入一個問題並抽牌，以得到一段提示式解讀。
- US-02：作為使用者，我想選擇單張或三張牌陣，以符合我的需求。
- US-03：作為使用者，我想可選擇是否啟用逆位，以符合我習慣的讀牌方法。
- US-04：作為使用者，我想可以重新抽牌或清除結果，以重新開始。

---

## 5. 功能需求（Functional Requirements, FR）
### UI 與輸入
- FR-01 系統提供問題輸入欄位（可空白）。
- FR-02 系統提供牌陣選擇（至少：單張、三張-過去/現在/未來）。
- FR-03 系統提供逆位開關（啟用/停用）。
- FR-04 系統提供抽牌按鈕，點擊後生成一次抽牌結果。
- FR-05 系統提供重新抽牌按鈕（沿用當前設定重新抽取）。
- FR-06 系統提供清除按鈕，清空結果並回到未抽牌狀態。

### 抽牌邏輯
- FR-10 系統抽牌必須「無放回」，同一次抽牌不得重複牌卡。
- FR-11 系統須先洗牌再取前 N 張，N = 牌陣位置數。
- FR-12 逆位啟用時，每張牌以 50% 機率為逆位；停用時全部正位。

### 顯示與敘述
- FR-20 系統顯示每張牌的：
  - 位置（position label）
  - 牌名（name）
  - 正位/逆位（orientation）
  - 關鍵詞（keywords）
  - 簡短牌義（meaning）
- FR-21 系統需生成「總結敘述」：
  - 由每張牌的（位置 + 牌義 + 關鍵詞）組合而成
  - 採提示/反思語氣，避免絕對命運斷言
- FR-22 系統需顯示倫理提醒（disclaimer）：不取代醫療/法律/財務等專業建議。

---

## 6. 非功能需求（Non-functional Requirements, NFR）
- NFR-01 單檔可攜性：`index.html` 可在無伺服器情境下直接開啟使用（file://）。
- NFR-02 相容性：支援主流現代瀏覽器（Chrome / Edge / Firefox / Safari 近期版本）。
- NFR-03 效能：抽牌與渲染在一般裝置上 1 秒內完成（以精簡牌庫為基準）。
- NFR-04 安全性：不傳輸資料到網路；畫面輸出需避免注入（例如 HTML escape）。
- NFR-05 可維護性：牌庫與牌陣以資料結構（JSON-like）呈現，可擴充。

---

## 7. 資料設計（Data Design）

### 7.1 牌卡資料結構（Tarot Card）
最小欄位（建議）：
- `id`: number|string
- `name`: string
- `keywords_upright`: string[]
- `meaning_upright`: string
- `keywords_reversed`: string[]
- `meaning_reversed`: string

> v1 允許精簡牌庫（例如 12 張），但結構需可擴充至完整 78 張。

### 7.2 牌陣資料結構（Spread）
- `id`: string
- `name`: string
- `positions`: `{ key: string, label: string }[]`

v1 必備牌陣：
- `single`: positions = 1
- `three_ppf`: positions = 3（past/present/future）

### 7.3 抽牌結果資料（Reading）
- `created_at`: ISO string
- `spread_id`: string
- `allow_reversed`: boolean
- `question`: string
- `category`: string（可選）
- `picked`: array of:
  - `card`: Tarot Card
  - `orientation`: `upright | reversed`
  - `position`: SpreadPosition

---

## 8. 商業規則（Business Rules）
- BR-01 同一個 reading 不重複牌卡（無放回抽取）。
- BR-02 逆位規則：allow_reversed = true 時，orientation 隨機決定；false 時固定 upright。
- BR-03 解讀語氣：必須使用「可能、提醒、建議、傾向」等措辭，避免肯定句斷言。
- BR-04 問題可空：若未輸入問題，系統仍輸出「泛用提示」。

---

## 9. 介面規格（UI Spec）
### 9.1 必要元件
- 下拉選單：牌陣選擇
- 開關/下拉：是否啟用逆位
- 文字輸入：問題（textarea 或 input）
- 按鈕：
  - 開始抽牌
  - 重新抽牌
  - 清除
- 結果區：
  - 顯示 N 張牌（卡片風格 list/grid）
  - 顯示總結敘述文字（段落/列表皆可）

### 9.2 互動規則
- 初始狀態：結果區顯示「尚未抽牌」。
- 抽牌後：顯示結果，並啟用重新抽牌/清除按鈕。
- 清除後：回到初始狀態。

---

## 10. 演算法規格（Algorithm Spec）
### 10.1 洗牌
- 使用 Fisher–Yates shuffle（或等效均勻洗牌方法）
- 對牌庫複本洗牌，避免破壞原始資料

### 10.2 抽牌（無放回）
- N = spread.positions.length
- shuffledDeck 取前 N 張作為抽牌結果
- 若牌庫張數 < N，須阻止抽牌並提示錯誤（v1 可簡化為「確保牌庫足夠」）

### 10.3 逆位
- allow_reversed = true：
  - 每張牌 orientation = random() < 0.5 ? reversed : upright
- allow_reversed = false：
  - orientation = upright

### 10.4 簡單敘述生成（規則模板）
依 position_key 套用模板，例如：
- past：`【過去】...（關鍵詞：...）`
- present：`【現在】...（留意：...）`
- future：`【未來】...（提醒：...）`
- single/today：`【今日提示】...`

總結由各位置段落串接，最後附上 disclaimer。

---

## 11. 測試規格（Test Spec）
### 11.1 功能測試
- TS-01 三張牌抽牌結果不得重複
- TS-02 逆位停用時，所有牌 orientation 均為 upright
- TS-03 切換牌陣後，抽到的張數必須等於位置數
- TS-04 問題空白仍可抽牌並產生總結

### 11.2 介面測試
- TS-10 抽牌後結果區可完整顯示每張牌資訊
- TS-11 清除後結果區回到初始狀態

---

## 12. 交付驗收標準（Acceptance Criteria）
- AC-01 提供 `index.html` 單檔，離線可直接開啟運行
- AC-02 UI 可完成：選牌陣、（可選）逆位、輸入問題、抽牌
- AC-03 抽牌符合無放回規則，且三張牌可顯示位置與簡短牌義
- AC-04 總結敘述為模板生成，語氣為提示式並含 disclaimer
- AC-05 提供 `spec.md` 作為開發規格文件（本文件）

---

## 13. 未來擴充（Roadmap）
- v1.1：新增「情境/行動/結果」三張牌陣
- v1.2：擴充牌庫至 22 大阿卡那或完整 78 張
- v2：加入歷史紀錄（localStorage）與匯出/分享
- v3：後端化（帳號、保存 Session、管理端牌義）
