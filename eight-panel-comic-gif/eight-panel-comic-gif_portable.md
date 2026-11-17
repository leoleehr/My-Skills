---
name: eight-panel-comic-gif
description: 把原子筆排線風的八格漫畫單頁（生圖模型產生的無字底圖＋Python 組版疊字）做成 1080 寬、10 fps、6 秒無縫循環、5 MB 以內的動態 GIF。用在：替新一篇人物漫畫做動態版、調整既有漫畫 GIF 的動作、換格或鏡像版面後重算動畫位置。不適用於從零畫漫畫、或要生圖模型直接產生動畫。
---

# 原子筆八格漫畫 → 動態 GIF（可攜版，單一檔案）

> 本檔可以直接貼給任何 LLM（Claude、GPT、Gemini 等）。內容整理自一次實際製作（2026-11-17，人物篇「Maggie」，五輪修改後定稿）。
> 已移除本機路徑、帳號、session 等資訊；檔名都用相對名稱。查不到的項目標「未確認」。

**核心原則**：生圖模型只負責**無字靜態底圖**（以及必要時的局部變體，例如閉眼）。動作全部由程式把「部位」整塊旋轉或平移。文字放在獨立圖層，每一幀都鎖定。

---

## 1. 模型與工具

| 角色 | 當時實際使用 | 備註 |
|---|---|---|
| 寫程式、調度 | Claude Code headless（`claude -p`，acceptEdits 權限模式）；調度者為 Claude Opus 5.5（`claude-opus-5-5`） | 執行 GIF 任務的子 Agent 模型 ID 沒有記錄（**未確認**，推定為帳號預設模型） |
| 生圖 | Codex CLI：`codex exec --skip-git-repo-check "<指示>"`，sandbox read-only，只拿來生圖 | 生圖結果存在 Codex 的 generated_images 資料夾，再由調度 Agent 複製出來 |
| Codex 主模型 | 閉眼變體：`gpt-6.1-sol`（reasoning low，codex-cli 0.169.0）；HR 重繪底圖：`gpt-6-luna`（reasoning medium，0.166.0） | 取自 Codex log 檔頭 |
| 生圖工具／模型 | Codex 內建的 image generation 工具 | **實際生圖模型名稱未確認**（log 沒有印出） |
| Python 套件 | Python 3.12.10、Pillow 12.3.0、numpy 2.5.3 | 查詢當下的版本 |
| ffmpeg | 9.0.2（只有第一版拿來出 MP4，定稿不需要） | |
| 字型 | 微軟正黑體 Bold（`msjhbd.ttc`） | 可換成任何支援繁中的粗體字型 |

換成別的模型也可以，只要遵守：生圖模型不畫中文字、不出動畫幀；程式模型負責動作與驗證。

---

## 2. 規格

| 項目 | 值 |
|---|---|
| 底圖 | 每格一張 3:2 橫圖（例 1536×1024；個別重繪的圖可能是 1671×941 之類），檔名 `img01～img08.png` |
| 單頁靜態組版 | 1996×3014（版面單位 363×548，×5.5） |
| 輸出 GIF | 1080×1631 |
| fps／幀數／循環 | 10 fps（每幀 100 ms）、60 幀、6 秒、無限循環（loop=0） |
| 大小上限 | 5 MB（自訂目標；LinkedIn 官方 GIF 限制**未查證**，過去實際上傳 GIF 沒問題）。定稿 4.47 MB |
| 調色盤 | 全片共用 255 色（MEDIANCUT，樣本＝靜態圖＋第 12、20、45 幀），第 256 色保留給透明 |
| 抖色 | 靜態像素 Floyd–Steinberg 一次全片沿用；動態像素 4×4 Bayer 有序抖色（±5）；時間穩定門檻 10；沒變的像素寫成透明；disposal=1 |
| 品牌色 | Navy `#283568`、Slate `#596579`、Periwinkle `#7B92E8`、Mist `#B8C2D2`、紙色 `#F5F4F2`、文字 `#2F2B36`；主角說話者標籤用角色色（例：芥末黃 `#B07814`） |
| 字級（單頁像素） | 標題 56、副標 ×0.52、對白 46（說話者標籤 ×0.72）、時間標籤 ×0.66、格號 40、頁尾 34、LOGO 高 135；框線 Navy 寬 7；頂部 Navy 色條高 14 |
| 對白框 | 白底 alpha 242、圓角 14、Navy 外框寬 4、右下 Navy 半透明陰影 |
| 斜線分格 | 4 列×2 格，格與格之間是斜線；每格一個四邊形 `POLYS`，同時當遮罩與框線 |
| 每格貼圖 | `cover()`：等比放大填滿外框，`cx` 控制水平裁切點（0 靠左、0.5 置中、1 靠右），再用四邊形遮罩貼上 |
| 文字層 | 格號、對白、框線、標題、頁尾、LOGO 畫在同一張透明層；原尺寸畫一次、premultiplied（RGBa）LANCZOS 縮一次；alpha ≥ 235 的像素每幀直接換成靜態圖像素 |

---

## 3. 流程

### 3.1 無字底圖（生圖）
一格一次，附參考圖。prompt 結構：
- 新圖：`Generate ONE new image with your image generation tool (do not write code, do not draw with scripts).`
- 改圖：`Use the image generation tool to create an edited version of the attached illustration (landscape, same aspect ratio as the attached image).` → `Keep EVERYTHING identical: …` → `Change ONLY …`
- 畫風：`hand-drawn colored ballpoint-pen illustration, dense blue/indigo cross-hatching on cream paper, with warm mustard-yellow and skin tones as accent colours.`
- 構圖：`Keep all important faces and action in the CENTRAL 60% of the frame (the image will be cropped to near-square).`、`Leave the lower-left quarter relatively calm (a caption box will be placed there later).`
- 必加：`absolutely no text, letters, numbers, Chinese characters …`、`… is a fictional character and must not resemble any real person.`、`Do not run any shell commands; only generate the image.`
- 想保留構圖、又不想讓模型照抄舊臉：附**高斯模糊過的舊圖**當構圖參考，另附一張寫實度標準的臉部特寫。

完整 prompt 原文見附錄 A。

### 3.2 組版疊字
所有中文由程式（Pillow）疊上，生圖模型一個中文字都不畫。組版程式要提供：`POLYS`、`LINES`（每格對白）、`cover()`、`caption()`、`number_badge()`、`header()`、`footer()`，以及一個 `layout()` 函式，回傳每個位置的 `slot / src（來源底圖）/ poly / bbox / mask / box（對白定位框）/ cx`。

### 3.3 動畫分層（每一幀由下到上）
1. 紙色底＋頂部色條（靜態）。
2. 八格底圖：`panel(src, p)` 在**底圖原圖座標**算出這一幀，`p=f/60`；`cover()`＋遮罩貼到單頁。
3. 整頁縮到 1080 寬。
4. 疊上已縮好的文字層；alpha ≥ 235 的像素換成靜態圖。

### 3.4 動作做法

工具：
- **剛性圖層 Mover**：多邊形圈出部位（可用顏色類別篩選，只帶人物像素），整塊旋轉／縮放／平移。權重 1 的地方整塊動，只在和身體相接處（手腕、袖口、腰）漸變到 0。部位離開後露出的縫要補底：沿背景紋理方向延伸（門框、牆面），或從固定偏移的乾淨區域複製（白板）。
- **關節旋轉 RotArm**：以手肘等關節為軸心剛性旋轉，權重在關節附近漸變到 0；只往一個方向轉，縫都在同一側，每幀從當幀的偏移位置複製補底。
- 光效：濾色光暈（只變亮）、正片疊底色光（白底上看得到、不洗白圖示）、四芒星閃光、斜向淡雲影帶。
- 時間：`wave(p,cycles,phase)=0.5-0.5cos(2π(cycles·p+phase))`；`pulse(p,c,w)` 循環高斯；`pop()` 放大略過頭再回落。**所有週期都是 6 秒的整數倍**，循環才會無縫。

| 動作 | 做法 | 實測可用參數（原圖座標） |
|---|---|---|
| 揮手 | 以手腕為軸旋轉；多邊形沿手掌輪廓描，不用顏色篩選（牆和皮膚同色會一起被帶走） | ±14°、3 次／循環；長輩 0～−18°。±3.5° 完全看不出來 |
| 呼吸 | 頭到腰整塊上移，權重由胸口往腰漸變到 0 | 10～13 px、2 次／循環；抽泣 11 px |
| 杯子晃動 | 以杯底為軸旋轉，兩杯相位錯開 | ±6°、3 次 |
| 光暈、掃光 | 光暈半徑 60～150，強度 0.18～0.55；箭頭掃光只在箭頭方框內、依筆畫深色遮罩由左到右變橘＋淡黃光暈，方框邊羽化 6 px | 橘 `(236,112,18)`、光暈 `(255,226,120)`×0.55 |
| 依序彈出 | 圓形區域整塊放大＋同色正片疊底光 | 放大 24%、頂點時間 0.15／0.35／0.55；脈動 16% |
| 手臂以手肘為軸 | RotArm；多邊形從手指描到手肘 | 0～3.5°、3 次（指尖約 15 px） |
| 點白板 | 手臂沿手臂方向整塊平移，**平移量取整數像素**；白板圖示、箭頭設保護區 | 18 px，三次點擊 |
| 邀請手勢 | 旋轉＋前後平移 | ±3.5°、5 px |
| 眨眼 | 見 3.4.1 | 每人每 2 秒一次、多人錯開；3 幀：半閉→全閉→半閉 |
| 蒸氣 | 唯一允許局部變形（位移場）的地方：蒸氣區沒有直線 | x 7 px、y 3 px 正弦 |
| 水柱 | 沿水柱中心線的高斯條紋往下流（濾色） | 條紋週期 26 px、6 次／循環、強度 0.45～0.55 |
| 雲影 | 斜向淡暗帶從左掃到右，p=0 與 p=1 都在畫面外 | 強度 0.07、寬 240、斜率 0.35，只掃遠景那一帶 |

#### 3.4.1 眨眼
1. 請生圖模型依原底圖生「同構圖、指定的人閉眼、無字」變體（附錄 A-C）。
2. 對位：每張臉在「臉部扣掉眼睛」的區域用高通灰階做正規化相關，搜尋縮放 0.90～1.10、位移 ±24 px，再細搜。
3. 只取眼周：每隻眼標出上睫毛線中心點與下眼瞼點，外擴 6 px 成橢圓；橢圓內只保留「模糊 1.5 px 後和原圖平均色差 > 16」的像素，外擴 3 px、羽化 1.2 px；橢圓外一律用原圖。
4. 全閉＝遮罩內換成變體像素；半閉＝上睫毛線到半閉線之間蓋變體的眼瞼皮膚，再把原圖的上睫毛線逐欄往下搬，以「變暗」混合畫上（保留原筆觸）。
5. 戴眼鏡的人：鏡框一律用原圖（變體的鏡框常偏 1～2 px）；鏡片內太平滑時，疊上附近臉頰的高頻排線紋理。
6. 同格的光效用羽化臉部橢圓擋掉，臉部除了眼睛之外每幀等於原圖。

### 3.5 換格、鏡像版面時的座標換算
- 動畫一律寫在**原圖座標**；換版面只換 `bbox / mask / cx`，動畫程式不用改。
- 原圖座標 → 單頁座標（和 `cover()` 同一套）：
  ```
  r = max(bw/iw, bh/ih); sw, sh = int(iw*r+1), int(ih*r+1)
  ox = int((sw-bw)*cx); oy = (sh-bh)//2
  X = bx0 + x*sw/iw - ox; Y = by0 + y*sh/ih - oy
  GIF 座標 = 單頁座標 × 1080/1996
  ```
- 只有外框寬高不變時，才能用「整格平移 dx」換算（例：第三列左右鏡像時兩格分別 −693、+1057 單頁像素）；外框尺寸一變就只能走原圖座標。
- 格號、對白、框線用新版面重畫，不搬舊文字層。

### 3.6 GIF 壓縮
1. 靜態圖＋3 張樣本幀 → MEDIANCUT 255 色；索引 255＝透明。
2. 靜態圖 Floyd–Steinberg 量化一次，全片的靜止像素沿用。
3. 每幀動態像素＝和靜態圖差 > 5、外擴 2 px、扣掉文字鎖定區；加 4×4 Bayer（±5）後無抖色量化。
4. 上一幀索引的誤差 ≤ 本幀最佳 + 10 就沿用上一幀索引（防閃爍）。
5. 和上一幀相同的索引寫成透明；`duration=100, loop=0, disposal=1, transparency=255, optimize=False`。
6. 超過 5 MB：先降到 8 fps 或縮寬度，文字層不動。

---

## 4. 品質關卡（全部通過才交件）

| # | 關卡 | 做法 | 通過標準 |
|---|---|---|---|
| Q1 | 靜態重組一致 | 動作全為 0 的那一幀（原尺寸）vs 組版程式輸出的單頁 PNG | 最大差 0（1080 寬比對時約 7% 像素差 > 2，是「先縮再疊字」的反鋸齒，正常） |
| Q2 | 文字層每幀差 0 | 解碼 GIF，文字層 alpha ≥ 235 的像素逐幀 vs 第 0 幀 | 最大差 0 |
| Q3 | 循環接縫 | 末幀→首幀平均差 vs 相鄰幀平均差 | 接縫較小（實測 0.239 < 0.316） |
| Q4 | 動態不出格 | 框線外擴 2 px 以外的變動像素；每格動態是否跑到別格 | 接近 0；跑到別格 0 |
| Q5 | 臉不能被切到 | 每張臉人工標註原圖範圍（髮際到下巴、含耳）→ 換算到單頁 → 是否完整落在「格子內縮 6 px」內、是否被文字層蓋到；手、道具、圖示也列進去 | 每項切到 0、蓋到 0；輸出標框檢查圖（綠＝通過、紅＝不通過）人工逐格看 |
| Q6 | 背景直線不變波浪 | 放大看門框、窗框、白板邊、桌緣在動作最大幀 | 直線仍直 |
| Q7 | 部位整塊平移 | 整數平移的部位平移回去逐像素比；保護區（圖示）逐幀比 | 皆差 0 |
| Q8 | 臉只動眼睛 | 眨眼格：臉部扣掉眼睛遮罩，所有幀 vs 原圖 | 最大差 0 |
| Q9 | 手臂沒有切口 | 擺幅最大幀並排放大；手臂範圍外差 0 | 袖口、手肘看不出接縫 |
| Q10 | 放大比對圖 | 每個有動作的格子：原圖 vs 最不利幀並排（2～6 倍），加 GIF 解碼後同區域 | 人工看過並寫進回報 |
| Q11 | 大小與格式 | ≤ 5 MB、60 幀、1080 寬、100 ms、loop=0 | 全部符合 |

驗證程式的完整輸出要存檔（上次定稿的部分驗證數字沒有寫進 log，事後查不到）。

---

## 5. 業主偏好與踩過的坑

| 問題 | 之後一律照做 |
|---|---|
| 第一版動作太小，業主說「完全看不出來」 | 目標：手機上滑動態消息一眼看得出每格在動，但仍是同一張漫畫，不卡通化、不走樣 |
| 「箭頭依序出現」用淡出遮罩，連帶破壞了筆和白板人形圖示 | 不隱藏原圖，只在箭頭方框內加亮或掃光；圖示設保護區，筆和手只做整數平移 |
| 位移場（局部變形）一加大，門框等直線變波浪 | 部位整塊移動＋補底；位移場只留給蒸氣 |
| 講電話時搖頭、點頭被否決 | 不搖頭，改成眨眼 |
| HR 點白板 44 px、邀請手勢 ±7° 太大 | HR 的手部動作要小：18 px、±3.5° |
| 比讚只圈手，手臂被切開 | 前臂＋手以手肘為軸一起轉 |
| 眨眼不能像貼色塊 | 閉眼變體只取眼周、對位、羽化；半閉用原睫毛線下移 |
| 業主請生圖代理（Codex）直接改動畫程式，效果很差，整批作廢 | 生圖代理只生靜態圖；動畫程式由程式型 LLM 寫並驗證 |
| 換格（鏡像）後斜邊切到半張臉，業主：「不能切到臉」 | 每版都跑臉部檢查；可調斜線位置／斜度、裁切點、等比縮放、搬對白框，不能非等比變形 |
| 每版都輸出到新資料夾 | 版本遞增，不覆蓋舊版 |

定稿仍接受的小瑕疵：白板手臂收回時指尖旁 1～2 px 殘點；某位同事全閉時眼瞼略亮；低頭的人眨眼很含蓄。

---

## 6. 提示詞範本（直接貼給別的模型）

```
# 任務：把八格漫畫單頁做成動態 GIF

## 輸入
- 無字底圖 img01.png～img08.png（第 N 格的來源底圖；若有換格，以 layout() 的 src 為準）
- 組版程式 compose_brand_<人物>.py：POLYS、LINES、cover()、caption()、number_badge()、header()、footer()、layout()
- 靜態單頁 <人物>_單頁.png（1996×3014）、漫畫腳本 .md（每格畫面與對白）
- （選用）生圖模型產生的閉眼變體圖 closed.png（和 img02 同尺寸）

## 輸出（全部放新資料夾，不覆蓋任何舊檔）
- <人物>_動態.gif：1080×1631、10 fps、60 幀（6 秒）、loop=0、≤ 5 MB
- animate.py（支援 --debug：輸出每個部位的遮罩疊色圖與單格測試幀）
- check.py（Q1～Q11 的數字）、關鍵幀 2～4 張、每個有動作格子的放大比對圖、臉部檢查表 CSV＋標框圖

## 做法規則
1. 文字（標題、對白、格號、框線、頁尾、LOGO）畫在獨立透明層，原尺寸畫一次、縮一次；alpha ≥ 235 的像素每幀換成靜態圖。
2. 動畫寫在底圖原圖座標：panel(src, p) → cover() → 遮罩貼到單頁 → 整頁縮到 1080 → 疊文字層。
3. 動作一律用「多邊形圈出部位、整塊旋轉或平移、露出的縫補底」，權重只在和身體相接處漸變。
   不用位移場（只有蒸氣區例外）。需要像素完全不變的部位（手、筆）用整數平移。
4. 臉除了眼睛之外，每幀等於原圖；眨眼用閉眼變體只取眼周、對位、羽化。
5. 不要請生圖模型產生動畫幀或修改動畫程式。中文字一律由程式疊上。
6. 所有週期都是 6 秒的整數倍。
7. 偏好：每格動作在手機上一眼看得出來（揮手約 ±14°、呼吸 10～13 px）；但 HR 的手部動作要小（約 18 px、±3.5°）；
   不搖頭（改眨眼）；手臂不能被切開（以手肘為軸轉）；任何臉都不能被格線、斜邊或對白框切到。
8. GIF 編碼：全片共用 255 色調色盤＋透明索引；靜態像素 Floyd–Steinberg 一次；動態像素 4×4 Bayer 有序抖色；
   時間穩定門檻 10；沒變的像素寫透明；disposal=1。

## 驗收（逐項回報數字）
Q1 靜態重組 vs 單頁 PNG 原尺寸最大差 0 ｜ Q2 文字層每幀最大差 0 ｜ Q3 接縫 < 相鄰幀平均 ｜
Q4 格外變動接近 0、跑到別格 0 ｜ Q5 臉部檢查表全部通過 ｜ Q6 直線不變波浪（放大圖） ｜
Q7 整數平移部位與保護區差 0 ｜ Q8 臉部扣眼睛差 0 ｜ Q9 手臂無切口（放大圖） ｜
Q10 每格放大比對圖 ｜ Q11 ≤ 5 MB、60 幀、1080 寬、100 ms、loop=0
做不到的項目要明講原因，不要省略。
```

---

## 7. 下一篇套用清單

**直接重用**：GIF 編碼函式、文字層鎖定與分層組版、Mover／RotArm／補底／光效／時間函式、眨眼演算法與對位程式、驗證程式骨架、臉部檢查程式、規格與品牌色。

**一定要換**
- 底圖與組版程式；若組版程式沒有 `layout()`，先補一個回傳 `slot/src/poly/bbox/mask/box/cx` 的函式。
- 每格部位多邊形、軸心、權重範圍、補底方向（先跑 `--debug` 看遮罩有沒有圈到臉或背景直線）。
- 顏色篩選門檻（原本依芥末黃毛衣調；主角換色就要重調）。
- 閉眼變體、對位、每隻眼的標點、眨眼時間表。
- 臉部檢查表（每張臉、手、道具、圖示的原圖範圍）。
- 保護區座標、輸出資料夾與檔名、參考單頁 PNG。

**先決定**：每格要動什麼（照第 5 節偏好）；有沒有換格／鏡像（有的話照 3.5 換算）。

---

## 附錄 A：生圖 prompt 原文（英文原文照錄）

### A-1 新圖範例（第 1 格，童年回憶）
```
Generate ONE new image with your image generation tool (do not write code, do not draw with scripts). Save nothing else.

Style: exactly match the attached reference images - hand-drawn colored ballpoint-pen illustration, dense blue/indigo cross-hatching on cream paper, with warm mustard-yellow and skin tones as accent colours. Landscape 3:2. Keep all important faces and action in the CENTRAL 60% of the frame (the image will be cropped to near-square).

Scene (comic panel 1, a childhood memory of moving house): a quiet residential street in a Taiwanese neighbourhood on a sunny late afternoon, low apartment buildings, potted plants, a scooter, a small moving truck with its back open and a few stacked cardboard moving boxes on the pavement (boxes are plain, no writing).
- Main character: the SAME woman from the references (Maggie), but as a girl of about 9 years old - clearly recognisable as her younger self: black bob haircut with side-swept bangs, same face shape and eyes, wearing a small mustard-yellow knit cardigan over a white top, a little backpack. She stands in the centre holding a small notebook with a simple hand-drawn map of streets (lines and squares only, NO writing), pencil in hand, looking around carefully to memorise the way.
- She is smiling and giving a polite little wave to a friendly elderly neighbour (grandmother with grey hair in a bun, cardigan) who is watering plants at her doorway and waving back; a neighbour child of similar age peeks curiously from behind a gate.
- In the background, her parents (seen from behind / small) carry a box into the new building.
- Soft nostalgic feeling, slightly vignetted edges like a memory.

IMPORTANT: absolutely no text, letters, numbers, Chinese characters, house numbers, street signs, shop signs, logos or speech bubbles anywhere in the image. No watermark. Leave the lower-left quarter relatively calm (a caption box will be placed there later).
After generating, reply only with the saved image path.
```

### A-2 新圖範例（第 8 格，結尾雙層構圖）
```
Generate ONE new image with your image generation tool (do not write code, do not draw with scripts). Save nothing else.

Style: exactly match the attached reference images - hand-drawn colored ballpoint-pen illustration, dense blue/indigo cross-hatching on cream paper, with warm mustard-yellow, coffee-brown and berry-red accents. Landscape 3:2. Keep the key subjects in the CENTRAL 60% of the frame (it will be cropped to near-square).

Scene (comic panel 8, ending): a dreamy two-layer composition.
- Foreground (lower centre): a pour-over coffee setup on a wooden counter - gooseneck kettle pouring a thin, steady spiral stream of water into a cone dripper over a glass server; the hands and sleeves belong to Mia (mustard-yellow knit cardigan, as in the reference). A small open bowl of roasted coffee beans and a few dried red-purple coffee cherries beside it.
- Background (upper half, fading softly into the steam rising from the cup like a memory): the highlands of Guji, Ethiopia - rows of raised wooden African drying beds on stilts covered with red and dark-purple coffee cherries drying in the sun; several local workers (respectful, natural depiction, East African, wearing simple work clothes and head wraps) take turns gently raking and turning the cherries by hand; a light shade net stretched over part of the beds for the midday sun; green hills and tall shade trees under a big sky.

IMPORTANT: absolutely no text, letters, numbers, labels, Chinese characters, signs, logos on bags or speech bubbles anywhere. No watermark. Leave the lower-left and lower-right corners relatively calm (a caption box will go at the bottom).
After generating, reply only with the saved image path.
```
（主角當時暫名 Mia，後改名 Maggie。）

### A-3 改圖範例（只改一個人的畫法，定稿底圖用的就是這份）
```
Use the image generation tool to create an edited version of the attached illustration (landscape, same aspect ratio as the attached image).
Keep EVERYTHING identical: composition, camera angle, background, furniture, plants, lighting, colors, the woman in the mustard-yellow cardigan (unchanged), the man's pose, hands, navy suit, striped navy shirt, his face shape, features, expression, age (around 30), no glasses, and his neat slicked-back side-parted hair.
Change ONLY the rendering technique of the man's face, neck and hair: they currently look too smooth and airbrushed, almost photographic. Redraw them in exactly the same hand-drawn colored ballpoint-pen style as the woman's face and the rest of the picture: visible blue ballpoint-pen line work, fine cross-hatching and diagonal hatching strokes for shading, slightly sketchy outlines, warm colored-pencil skin tints with visible pencil grain, paper texture showing through. No smooth gradients, no glossy highlights, no photorealism on his face or hair.
He is a fictional character and must not resemble any real person.
No text, no letters, no numbers anywhere in the image.
Do not run any shell commands; only generate the image.
```

### A-C 動畫用變體：五人閉眼
```
Use the image generation tool to create an edited version of the attached illustration (landscape, same aspect ratio and same framing as the attached image).
Keep EVERYTHING identical: composition, camera angle, every person's position, pose, head angle, face shape, hair, clothes, hands, phones, background, office, windows, colors and the hand-drawn colored ballpoint-pen style with hatching.
Change ONLY the eyes of all five people: the woman in the mustard-yellow cardigan on the phone (center), the woman top-left, the young man bottom-left, the man with glasses top-right, and the woman bottom-right. Each of them has their eyes gently CLOSED, as in the middle of a natural blink: upper eyelids fully lowered, a thin curved eyelash line drawn with ballpoint pen where the eye was, skin of the eyelid drawn with the same pen hatching and warm skin tint. Eyebrows unchanged, expressions otherwise unchanged.
No text, no letters, no numbers anywhere in the image.
Do not run any shell commands; only generate the image.
```
寫法重點：`same framing`、逐一點名每個人、描述閉眼的筆觸。這次五張臉對位結果都在原位，但仍只取眼周。

---

## 附錄 B：核心程式節錄（Python／numpy／Pillow）

```python
TAU = 2 * np.pi
def smoothstep(e0, e1, x):
    t = np.clip((x - e0) / (e1 - e0), 0, 1); return t * t * (3 - 2 * t)
def wave(p, cycles, phase=0.0): return 0.5 - 0.5 * np.cos(TAU * (cycles * p + phase))
def pulse(p, c, w): d = (p - c + 0.5) % 1 - 0.5; return float(np.exp(-(d / w) ** 2))

def bilinear(arr, sx, sy): ...            # 雙線性取樣（整數位移時等於原像素）

class Mover:                              # 剛性圖層
    # __init__：mask = 多邊形（可再用顏色類別篩選、補洞、去小塊）；
    #           plate = 補底圖（沿方向延伸 streak_fill，或固定偏移複製 offset_fill）；
    #           W = wfn(X, Y)：1 整塊動、0 不動
    def apply(self, arr, src, deg=0.0, t=(0.0, 0.0), scale=1.0):
        x0, y0, x1, y1 = self.box; W = self.W
        th = np.radians(deg) * W; s = 1 + (scale - 1) * W
        px, py = self.pivot
        dx = (self.X - t[0] * W - px) / s; dy = (self.Y - t[1] * W - py) / s
        c, sn = np.cos(th), np.sin(th)
        sx = px + c * dx + sn * dy; sy = py - sn * dx + c * dy      # 反向取樣
        layer = bilinear(src, sx, sy)
        alpha = bilinear(self.maskf, sx, sy)[..., None]
        under = np.where(self.mloc[..., None], self.plate, src[y0:y1, x0:x1])
        arr[y0:y1, x0:x1] = under * (1 - alpha) + layer * alpha

def write_gif(frames, static_small, lock, path):
    sample = Image.fromarray(np.vstack([np.asarray(static_small)] + [np.asarray(frames[k]) for k in (12, 20, 45)]))
    pal = sample.quantize(colors=255, method=Image.Quantize.MEDIANCUT, dither=Image.Dither.NONE)
    palette = pal.getpalette()[:255 * 3] + [255, 0, 255]
    pimg = Image.new("P", (1, 1)); pimg.putpalette(palette)
    sq = np.asarray(static_small.quantize(palette=pimg, dither=Image.Dither.FLOYDSTEINBERG))
    pal_arr = np.array(palette, np.int16).reshape(-1, 3)
    bayer = (np.array([[0,8,2,10],[12,4,14,6],[3,11,1,9],[15,7,13,5]]) + 0.5) / 16 - 0.5
    ref = np.asarray(static_small, np.int16); hh, ww = ref.shape[:2]
    dith = np.tile(bayer, (hh // 4 + 1, ww // 4 + 1))[:hh, :ww, None] * 10
    idx, prev = [], None
    for fr in frames:
        a = np.asarray(fr, np.int16)
        ch = np.abs(a - ref).max(2) > 5
        chd = ch.copy()
        for s in (1, 2):
            chd[s:] |= ch[:-s]; chd[:-s] |= ch[s:]; chd[:, s:] |= ch[:, :-s]; chd[:, :-s] |= ch[:, s:]
        chd &= ~lock
        q = np.asarray(Image.fromarray(np.clip(a + dith, 0, 255).astype(np.uint8)).quantize(palette=pimg, dither=Image.Dither.NONE))
        cur = np.where(chd, q, sq).astype(np.uint8)
        if prev is not None:
            keep = chd & (np.abs(pal_arr[prev] - a).sum(2) <= np.abs(pal_arr[cur] - a).sum(2) + 10)
            cur = np.where(keep, prev, cur)
        idx.append(cur); prev = cur
    out = []
    for k, cur in enumerate(idx):
        enc = cur.copy()
        if k > 0: enc[cur == idx[k - 1]] = 255
        im = Image.fromarray(enc, "P"); im.putpalette(palette); out.append(im)
    out[0].save(path, save_all=True, append_images=out[1:], duration=100, loop=0,
                disposal=1, transparency=255, optimize=False)
```
