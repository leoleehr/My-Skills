---
name: eight-panel-comic-gif
description: 把「人才 VS 咖啡」系列的原子筆排線八格漫畫單頁（Codex 無字底圖＋Python 組版疊字）做成 1080 寬、10 fps、6 秒無縫循環、5 MB 以內的動態 GIF。用在：要替某位人物篇（Maggie 之後的 Avery 等）的八格漫畫做動態版、調整既有漫畫 GIF 的動作、換格或鏡像版面後重算動畫位置、或要讓其他 LLM 重現同樣品質的 GIF 時。不適用於從零畫漫畫、或要生圖模型直接產生動畫。
---

# 原子筆八格漫畫 → 動態 GIF（eight-panel-comic-gif）

依據：2026-11-17 Maggie 八格漫畫動態 GIF 的實際製作過程（任務 T-20261117-05／07／08／09／11／12），定稿為
`<漫畫資料夾>/<人物>_codex/gif_v5/Maggie原子筆版v13b_動態_定稿.gif`。
本檔所有數字都從程式、log、驗證輸出讀出；查不到的寫「未確認」。下文相對路徑以 `<漫畫資料夾>/<人物>_codex/` 為根。

> 一句話原則：**生圖模型只負責無字靜態底圖（以及必要時的局部變體，例如閉眼）；動作全部由程式把「部位」整塊旋轉／平移，文字是獨立圖層、每幀鎖定。**

附檔：
- `references/prompts.md`：Codex 生圖 prompt 原文摘錄（底圖、HR 重繪、閉眼變體）
- `references/parameters.md`：規格、逐格動作參數表（含 v1→v5 演變）、臉部檢查表
- `references/code.md`：原始程式路徑地圖＋核心函式節錄
- `eight-panel-comic-gif_portable.md`：合併成單檔、去除本機路徑與私人資訊，直接貼給其他 LLM

---

## 1. 模型與工具（附出處）

| 角色 | 實際使用 | 出處 |
|---|---|---|
| 寫程式、調度的 LLM | Claude Code headless（`claude -p`，`--permission-mode acceptEdits`，max-turns 200），在專案根目錄執行。COS 本身為 Claude Opus 5.5（`claude-opus-5-5`）。 | `_cos_prompts/環境與模型.txt` |
| 做 GIF 的子 Agent（T-05～T-12） | 帳號預設模型，**未確認**（各任務回報與 MEMORY 都沒有記錄模型 ID）。 | 環境檔只寫「子 Agent 使用帳號預設模型」 |
| 寫本 SKILL 的 Agent | Claude Opus 5.5（`claude-opus-5-5`），Claude Code 2.1.326。 | 本 Agent 自身模型資訊；`claude --version` |
| 繪圖：Codex CLI | `codex exec --skip-git-repo-check "<指示>"`，ChatGPT 帳號登入，sandbox read-only（Windows 上無法執行指令，只拿來生圖），圖存在 `~/.codex/generated_images/<session id>/`，再由 Agent 複製出來。 | 環境檔；各 `codex_*.log` 檔頭 |
| Codex 主模型 | **11/17（閉眼變體）**：`gpt-6.1-sol`，reasoning effort low，codex-cli v0.169.0（和 config 摘錄一致）。**11/14～11/15（pen_v10、pen_v12 HR 重繪）**：`gpt-6-luna`，effort medium，v0.166.0。 | `gif_v3/_scratch/codex_blink.log`；`pen_v12/codex_img05.log`、`pen_v10/codex_img05.log`；環境檔 config 摘錄 |
| 生圖工具／模型 | 工具：Codex 內建的 image generation 工具（prompt 寫「Use the image generation tool」；環境檔稱 `image_generation`；Avery 篇的 log 顯示 Codex 另會讀 `~/.codex/skills/imagegen/SKILL.md`）。**實際生圖模型名稱：未確認**（所有 log 都沒有印出）。 | 各 `codex_*.log` |
| Python | 3.12.10（arm64） | `python --version` |
| Pillow | 12.3.0 | `PIL.__version__` |
| numpy | 2.5.3 | `numpy.__version__` |
| ffmpeg | 9.0.2-full_build（gyan.dev）。只有 gif_v1 用來出 MP4，定稿流程不需要。 | `ffmpeg -version`；`gif_v1/animate_v12.py` 檔頭 |
| codex-cli（目前） | 0.169.0 | `codex --version` |
| 字型 | 微軟正黑體 Bold `C:/Windows/Fonts/msjhbd.ttc` | `compose_brand_v13b.py` `FONT` |

> 版本是 2026-11-17 查詢當下的本機版本。製作當下的 Python／Pillow／numpy／ffmpeg 版本沒有另外記錄，推定相同，**未確認**。

---

## 2. 規格

| 項目 | 值 | 出處 |
|---|---|---|
| 單頁（靜態組版）尺寸 | 1996×3014（`S=5.5`，版面單位 363×548） | `compose_brand_v13b.py` |
| 底圖尺寸 | img01～04、07、08：1536×1024；img05：1671×941；img06：1670×942；閉眼變體 1536×1024 | 實測 |
| 輸出 GIF | 1080×1631（`round(3014×1080/1996)`） | `gif_v5/animate_v5.py`、`_check_log.txt` |
| fps／幀數／循環 | 10 fps（每幀 100 ms）、60 幀、6.0 秒、`loop=0`（無限） | `animate_v3.py` `FPS_CS=10`、`N=60` |
| 大小上限 | 5 MB（COS 指示；LinkedIn 官方 GIF 限制**未查證**，T-05 網路權限被拒；Leo 表示之前發過 GIF 沒問題）。定稿 4.47 MB（4,472,941 bytes）。 | T-07、MEMORY T-05 節、實測 |
| 調色盤 | 全片共用 1 組：靜態圖＋第 12、20、45 幀垂直拼接，MEDIANCUT 取 255 色；索引 255 保留給透明 | `write_gif()` |
| 抖色 | 靜態像素：Floyd–Steinberg 量化一次、全片沿用；動態像素：4×4 Bayer 有序抖色（振幅 ±5）；時間穩定 `STAB=10`；和上一幀相同的索引寫成透明；`disposal=1`、`optimize=False` | `write_gif()` |
| 品牌色 | Navy `#283568`、Slate `#596579`、Periwinkle `#7B92E8`、Mist `#B8C2D2`、紙色 `#F5F4F2`、文字 `#2F2B36`；說話者標籤：Maggie 芥末黃 `(176,120,20)`、HR Navy、法遵／主管 Slate | `compose_brand_v13b.py` |
| 字型與字級（單頁） | 微軟正黑體 Bold；標題 56、副標 0.52 倍、對白 46（說話者標籤 0.72 倍）、時間標籤 0.66 倍、格號 40、頁尾 34、LOGO 高 135；框線 Navy 寬 7；頂部 Navy 色條高 14 | 同上 |
| 斜線分格 | 4 列×2 格，格間是斜線（例：第三列 v13b 左格右邊 186→197、右格左邊 193→205，版面單位）；`POLYS` 多邊形同時當遮罩與框線 | `RAW` |
| 遮罩 | 每格：`cover()` 等比放大填滿外框（`cx` 控制水平裁切點，0 靠左、0.5 置中、1 靠右），再用多邊形 L 遮罩 `paste` | `layout_v13()` |
| 文字層 | 格號、對白框、說話者標籤、框線、標題、頁尾、LOGO 畫在**同一張透明層**，原尺寸畫一次、premultiplied（RGBa）LANCZOS 縮一次；alpha ≥ 235 的像素每幀直接換成靜態圖像素（`lock`） | `text_layer()`、`main()` |

---

## 3. 流程

### 3.1 無字底圖（Codex 生圖）

1. 每格一張 3:2 橫圖 `imgNN.png`（第 N 格的「來源底圖」），放在 `pen_vXX/`。附上參考圖（角色或前一張底圖），一格一次 `codex exec`。
2. prompt 固定結構（原文見 `references/prompts.md`）：
   - 開頭：`Generate ONE new image with your image generation tool (do not write code, do not draw with scripts).`（新圖），或 `Use the image generation tool to create an edited version of the attached illustration (landscape, same aspect ratio as the attached image).`（改圖）。
   - 畫風：`hand-drawn colored ballpoint-pen illustration, dense blue/indigo cross-hatching on cream paper, with warm mustard-yellow and skin tones as accent colours`。
   - 構圖：`Keep all important faces and action in the CENTRAL 60% of the frame (the image will be cropped to near-square)`；`Leave the lower-left quarter relatively calm (a caption box will be placed there later)`。
   - 改圖：先 `Keep EVERYTHING identical: …` 列出不能變的東西，再 `Change ONLY …`。
   - 必加：`absolutely no text, letters, numbers, Chinese characters…`；`…is a fictional character and must not resemble any real person.`；`Do not run any shell commands; only generate the image.`
3. 從 `~/.codex/generated_images/<session id>/` 把圖複製出來；log 存成 `codex_imgNN.log`、prompt 存成 `prompt_*.txt`，舊底圖改名保留（例 `img05_v10.png`）。
4. Avery 篇的經驗：要保留構圖但不想讓 Codex 照抄舊臉，附**高斯模糊過的舊圖**當構圖參考，另附寫實度標準的臉部特寫（MEMORY 2026-11-15 Avery v4 節）。

### 3.2 組版疊字（中文字不給生圖模型畫）

- `python compose_brand_v13b.py pen_v12 Maggie原子筆版v13b` → 單頁 PNG、8 頁輪播 PNG（1080×1350）與 PDF。
- 所有中文（標題、旁白、對白、格號、頁尾）都由 Pillow 用微軟正黑體 Bold 疊上。對白框白底 alpha 242、Navy 外框寬 4、右下 Navy 半透明陰影。超框時程式會印 `[超框]`。
- `layout_v13()` 回傳每個位置的 `slot / src / poly / bbox / mask / box / cx / dx / dy`，GIF 端直接 import 使用。

### 3.3 動畫分層

每一幀由下到上：
1. **紙色底＋頂部 Navy 色條**（靜態）。
2. **八格底圖**：每格 `panel(src, p)` 在**底圖原圖座標**（cover 前）算出這一幀的陣列，`p = f/60 ∈ [0,1)`；再 `cover()`＋多邊形遮罩貼到單頁。
3. 整頁 LANCZOS 縮到 1080 寬。
4. **文字層**（已縮好的同一張）alpha 合成；alpha ≥ 235 的像素整塊換成靜態圖（保證每幀差 0）。

動畫全部寫在原圖座標，換版面時動畫程式一行都不用改（gif_v4、gif_v5 都是 import `gif_v3/animate_v3.py` 的 `panel()`，只換版面）。

### 3.4 動作做法與參數範圍（詳細數值見 `references/parameters.md`）

核心工具（`gif_v3/animate_v3.py`，節錄見 `references/code.md`）：
- **`Mover`（剛性圖層）**：多邊形圈出部位（可加顏色類別篩選 `refine`），整塊旋轉／縮放／平移；權重 `wfn` 為 1 的地方整塊動，只有和身體相接處（手腕、袖口、腰）漸變到 0。部位離開後露出的縫用「補底」：`streak`（沿背景紋理方向延伸，適合門框、牆面）或 `offset`（從乾淨的白板區複製）。
- **`RotArm`**：以關節為軸心的剛性旋轉，縫每幀從當幀的偏移位置複製（色光一致）。
- 光效：`glow`（濾色，只變亮、不蓋筆觸）、`tint`（正片疊底，白底上看得到、不洗白圖示）、`sparkle`（四芒星）、`shadow_band`（斜向淡雲影）。
- 時間函式：`wave(p, cycles, phase)`（0 起的平滑起伏）、`pulse(p, center, width)`（循環高斯脈衝）、`pop()`（彈出：放大略過頭再回落）。**所有週期都是 6 秒的整數倍**，所以循環無縫。

| 動作 | 做法 | 定稿參數（建議範圍） |
|---|---|---|
| 揮手 | `Mover` 以手腕為軸旋轉；多邊形沿手掌輪廓描、**不用顏色篩選**（牆和皮膚同色會被一起帶走）；補底 `streak (0,-1)` 單側 | Maggie ±14°×3 次（`sin`）；奶奶 0～−18°×3 次。v1 的 ±3.5° 看不出來；建議 ±10～18° |
| 呼吸 | `Mover` 頭到腰整塊上移，權重由胸口往腰漸變到 0；`refine="#hsY"` 只帶人物色 | 第 3 格 13 px、新第 5 格 10 px、第 4 格抽泣 11 px，各 2 次／循環。建議 8～14 px（原圖座標） |
| 杯子晃動 | `Mover` 以杯底為軸旋轉，兩杯相位錯開 0.8 rad | ±6°×3 次 |
| 光暈、掃光 | `glow`／`tint`；箭頭掃光只在箭頭方框內、依筆畫深色遮罩由左到右變橘＋淡黃光暈，方框邊羽化 6 px | 窗光 0.18、檯燈 0.2～0.28、手機光 0.55、紅叉紅光 0.55；箭頭橘 `(236,112,18)` |
| 依序彈出 | `Mover` 圓形區域整塊縮放＋`tint` 同色光 | 放大 24%（紅叉脈動 16%）；頂點時間 0.15／0.35／0.55 |
| 手臂以手肘為軸 | `RotArm`：多邊形從拇指描到手肘、下緣停在物件之上；權重在手肘附近漸變到 0；**只往單一方向轉**，縫都在同一側，用偏移 (−55,+25) 的白板排線補 | 0～3.5°×3 次（拇指尖約 15 px） |
| 點白板 | `Mover` 手臂沿手臂方向整塊平移，**平移量取整數像素**（筆和手每幀都是原圖整數平移、不重新取樣）；人形圖示＋箭頭方框設 `protect` | 收 18 px（v2 的 44 px 太大），三次點擊 0.20／0.42／0.64 |
| 邀請手勢 | `Mover` 旋轉＋前後平移 | ±3.5°、5 px（v2 的 ±7°、10 px 太大） |
| 眨眼 | 見 3.4.1 | 每人每 2 秒（20 幀）一次、五人錯開；3 幀：半閉→全閉→半閉 |
| 蒸氣 | 第 8 格蒸氣區（沒有直線的區域）用小位移場 `warp_roi`，**只有這裡允許局部變形** | x 7 px、y 3 px 正弦 |
| 水柱 | 沿水柱中心線的高斯條紋往下流（濾色） | 條紋週期 26 px、6 次／循環、強度 0.45～0.55 |
| 雲影 | `shadow_band` 斜向淡暗帶從左掃到右，p=0 與 p=1 都在畫面外 | 強度 0.07、寬 240、斜率 0.35，只掃棚床那一帶（y 170～470） |

#### 3.4.1 眨眼（Codex 閉眼變體，只取眼周）

1. 請 Codex 依原底圖生「同構圖、指定的人全部閉眼、無字」的變體（prompt 見 `references/prompts.md` C 節）。
2. **對位**：每張臉在「臉部區域扣掉眼睛橢圓」用高通灰階做正規化相關，搜尋縮放 0.90～1.10、位移 ±24 px，再細搜（`gif_v3/_scratch/align_codex.py`）。Maggie 這次五張臉最佳解都是原位，所以直接用原座標。
3. **只取眼周**：每隻眼睛標出上睫毛線中心點（`up`）與下眼瞼點（`lo`）（原圖座標，放大格線圖量出），外擴 6 px 成橢圓；橢圓內只保留「模糊 1.5 px 後與原圖平均色差 > 16」的像素，外擴 3 px、羽化 1.2 px；橢圓外 alpha 一律 0。
4. **全閉**＝遮罩內換成 Codex 像素；**半閉**＝上睫毛線到半閉線之間蓋 Codex 眼瞼皮膚，再把原圖的上睫毛線逐欄往下搬到半閉線，以「變暗」混合畫上去（仍是原筆觸）。
5. 戴眼鏡的人：Codex 鏡框會偏 1～2 px，睫毛帶以外的深色線（鏡框）一律用原圖；Codex 在鏡片內畫得太平滑，加上附近臉頰的高頻排線紋理。
6. 同一格的窗光、手機光用羽化臉部橢圓擋掉，臉部像素逐幀等於原圖（只有眼睛會變）。

### 3.5 換格、鏡像版面時的座標換算

- **動畫寫在原圖座標（一律這樣寫）**：完全不用改，換版面只換 `bbox / mask / cx`：
  `page.paste(cover(panel(src, p), bw, bh, cx), (bx0, by0), mask)`。
- **原圖座標 → 單頁座標**（和 `cover()` 同一套算法，見 `_v13b_check/face_check.py` 的 `to_page()`）：
  ```
  r  = max(bw/iw, bh/ih);  sw, sh = int(iw*r+1), int(ih*r+1)
  ox = int((sw-bw)*cx);    oy = (sh-bh)//2
  X  = bx0 + x*sw/iw - ox; Y = by0 + y*sh/ih - oy
  GIF 座標 = 單頁座標 × 1080/1996
  ```
- **只有外框寬高不變時**才能用單純平移：v12→v13 鏡像時 img06 `dx=−693`、img05 `dx=+1057`（單頁像素，y 不變）；img05 裁切點 0.5→0.35，格內座標 x 約 +28 px。v13b 第三列外框尺寸變了（縮放比例改變），`dx/dy` 設 `None`，只能走原圖座標。
- 格號、對白、框線一律用新版面的 `text_layer()` 重畫，不搬舊文字層。

### 3.6 GIF 壓縮（`write_gif()`）

1. 調色盤：靜態圖＋三張樣本幀垂直拼接 → MEDIANCUT 255 色，無抖色；索引 255 = 透明。
2. 靜態圖用該調色盤 Floyd–Steinberg 量化一次（`sq`），所有幀的靜止像素直接沿用。
3. 每幀找動態像素：和靜態圖差 > 5，外擴 2 px，扣掉文字鎖定區。動態像素加 4×4 Bayer 抖色（±5）後無抖色量化（同顏色每幀抖成同樣的點，不會像誤差擴散那樣閃）。
4. 時間穩定：上一幀的索引顏色誤差 ≤ 本幀最佳誤差 + 10，就沿用上一幀索引。
5. 和上一幀索引相同的像素寫成透明（255），`disposal=1`、`duration=100`、`loop=0`、`optimize=False`。
6. 超過 5 MB 時（T-07 記錄的選項）：改 8 fps 或縮寬度；文字層不動。

---

## 4. 品質關卡（逐項照做，全部通過才交件）

| # | 關卡 | 做法 | 通過標準 | 參考程式 |
|---|---|---|---|---|
| Q1 | 靜態重組一致 | 所有動作為 0 的那一幀（原尺寸）vs compose 輸出的單頁 PNG | 最大差 0。1080 寬比對時差 > 2 的像素約 7.2%，來源是「先縮再疊字」的反鋸齒，屬正常 | `gif_v5/animate_v5.py` 自檢、`check_v5.py [1]` |
| Q2 | 文字層每幀差 0 | 解碼 GIF，文字層 alpha ≥ 235 的像素逐幀和第 0 幀比 | 最大差 0（定稿 332,404 像素） | `gif_v3/check_frames.py`、`check_v5.py [2]` |
| Q3 | 循環接縫 | 末幀→首幀平均差 vs 相鄰幀平均差 | 接縫 < 相鄰幀平均（v4：0.239 < 0.316） | 同上 |
| Q4 | 動態不出格 | 框線外擴 2 px 以外有變動的像素；每格動態有沒有跑到別格 | 接近 0（定稿 92 px，邊緣反鋸齒）；跑到別格 0 | `check_v5.py [3][4]` |
| Q5 | 臉部不能被切到 | 每張臉人工標註原圖範圍（髮際到下巴、含耳）→ 換算到單頁 → 檢查是否完整落在「格子內縮 6 px」內、是否被文字層蓋到 | 每列「切到格外 0、被文字層蓋到 0」；另輸出標框檢查圖，人工逐格看 | `_v13b_check/face_check.py`；表見 `references/parameters.md` |
| Q6 | 背景直線不能變波浪 | 放大看門框、窗框、白板邊、桌緣在動作最大幀 | 直線仍直；只有蒸氣區允許位移場 | 放大比對圖 |
| Q7 | 部位整塊平移 | 用整數平移的部位（筆、手）：平移回去逐像素比對；保護區（人形圖示）逐幀比對 | 皆差 0 | `gif_v3/check_p5.py` |
| Q8 | 臉部只動眼睛 | 眨眼格：臉部橢圓扣掉眼睛遮罩，60 幀 vs 原圖 | 最大差 0 | `gif_v3/check_v3.py (1)` |
| Q9 | 手臂沒有切口 | 旋轉手臂：擺幅最大幀並排放大；手臂範圍以外差 0 | 袖口、手肘看不出接縫 | `gif_v3/check_v3.py (2)` |
| Q10 | 放大比對圖 | 每個有動作的格子：原圖 vs 最不利幀並排（2～6 倍），加 GIF 解碼後同區域 | 人工看過、寫進回報 | `p2_blink_compare.png`、`p5_zoom_compare.png`、`p7_arm_max.png`、`zoom_新第*格_*.png` |
| Q11 | 大小與格式 | ≤ 5 MB、60 幀、1080 寬、100 ms、loop=0 | 全部符合 | `check_v5.py` 開頭 |

> 注意：`gif_v5/_check_log.txt` 只記到 `[4]` 標題就結束，[4]～[6] 的數字沒有寫進 log（`check_v5.py` 該段的輸出**未確認**）。下一篇請把完整輸出存檔。

---

## 5. Leo 的偏好與踩過的坑（逐輪回饋）

| 輪次 | 回饋／問題 | 結論（之後一律照做） |
|---|---|---|
| v1（T-05） | 動作太小，「現在完全看不出來」（揮手只有 ±3.5°） | 目標是**手機上滑 LinkedIn 動態消息時一眼看得出每格在動**，但仍是同一張漫畫、不卡通化、不走樣。揮手 ±14°、呼吸 10～13 px 才看得出來 |
| v1 | 第 5 格「箭頭依序出現」用淡出／遮罩，**連帶影響 HR 的筆和白板人形圖示** | 不隱藏原圖；改成只在箭頭方框內加亮／掃光，圖示設保護區，筆和手只做整數平移，放大逐幀比對 |
| v1 | 位移場（局部變形）一加大，門框等**背景直線就變波浪** | 改用剛性圖層：部位整塊旋轉／平移＋補底；位移場只留給蒸氣 |
| v2（T-07） | 第 2 格 Maggie 講電話**不要搖頭、點頭** | 改成五人眨眼，每人約每 2 秒一次、彼此錯開 |
| v2 | 第 5 格 HR 手臂收 44 px、第 6 格邀請手勢 ±7°／10 px **太大** | HR 的手部動作要小：收 18 px、±3.5°／5 px。看得出在點就好 |
| v2 | 第 7 格主管比讚只圈手和袖口，**手臂被切開** | 前臂＋手以手肘為軸心一起轉，權重在手肘漸變，只往一個方向轉 |
| v2→v3 | 眨眼不能像貼色塊 | Codex 閉眼圖只取眼周、對位、羽化；半閉用原睫毛線下移 |
| 15:33–15:43 | Leo 另請 **Codex 直接改動畫**（改 `animate_v2.py` 參數、重出 GIF、加了對調第 5／6 格的 `finalize_edits.py`），效果很差，已由 COS 清除 | **Codex 只拿來生靜態圖**；動畫程式由 Claude 寫、驗證。`gif_v2/` 那段不是正確流程 |
| v13（T-09） | 原文改成「先非正式溝通、再開會」→ 第 5、6 格對調 | 第三列左右鏡像，底圖不重新裁切；寫成 `layout_v13()` 讓 GIF 直接套 |
| v13b（T-12） | 鏡像後新第 6 格斜邊**切到 Maggie 半張臉**，Leo：「不能切到臉」 | 全頁每張臉都要做臉部檢查表；可調斜線位置／斜度、裁切點、等比縮放、搬對白框；不能非等比變形 |
| 全程 | 每版輸出到新資料夾（gif_v1～v5），舊版不動 | 版本資料夾遞增，不覆蓋；不 push、不發文；專案不是 git repo |

定稿仍接受的已知小瑕疵：新第 6 格手臂收回時指尖右側 1～2 個紅褐小點、筆下淡藍陰影小塊被白板補底取代（GIF 尺寸約 1 px）；左下同事全閉時眼瞼略亮；眼鏡男低頭，眨眼很含蓄。

---

## 6. 給其他 LLM 的提示詞範本

完整版在 `eight-panel-comic-gif_portable.md` 第 6 節（可直接複製）。精簡版：

```
你要把一張「原子筆排線風八格漫畫」單頁做成動態 GIF，畫面、分格、文字都和靜態版一樣，只讓人物和物件動起來。
輸入：8 張無字底圖 img01～img08.png；組版程式 compose_brand_*.py（POLYS、LINES、cover、caption、header、footer、layout 函式）；
     靜態單頁 PNG；漫畫腳本；（選用）生圖模型產生的閉眼變體圖。
輸出：<人物>_動態.gif（1080 寬、10 fps、60 幀、6 秒、loop=0、≤ 5 MB）；animate.py；check.py；2～4 張關鍵幀；
     每個有動作格子的放大比對圖（原圖 vs 最不利幀）；臉部檢查表 CSV＋標框圖。
規則：文字層獨立、每幀差 0；動畫寫在底圖原圖座標；部位用多邊形整塊旋轉／平移＋補底，不用位移場（蒸氣除外）；
     臉只允許眼睛變；不用生圖模型產生動畫幀；所有週期是 6 秒的整數倍；HR 手部動作小、不搖頭、手臂不切開、臉不被切到。
驗收：Q1～Q11 全部通過，把每項數字寫進回報。
```

---

## 7. Avery 等下一篇的套用清單

**可以直接重用**
- `write_gif()`、`text_layer()`／`down()` 的分層與鎖字邏輯（`gif_v5/animate_v5.py`）。
- 動作工具：`Mover`、`RotArm`、`streak_fill`、`offset_fill`、`refine_mask`、`color_classes`、`glow`、`tint`、`sparkle`、`shadow_band`、`warp_roi`（只給蒸氣）、`wave`、`pulse`、`pop`（`gif_v3/animate_v3.py`）。
- 眨眼模組演算法（`gif_v3/eyes_v3.py` 的 `Eye`、`closure`、`apply_blinks`）與對位程式 `gif_v3/_scratch/align_codex.py`。
- 驗證骨架：`check_frames.py`、`check_v5.py`、`check_p5.py`／`check_v3.py` 的「保護區差 0」「整數平移差 0」「臉部扣眼睛差 0」寫法、`_v13b_check/face_check.py`。
- 規格（尺寸、fps、調色盤、品牌色、字型、5 MB）與閉眼 prompt 的寫法。

**一定要換**
- 底圖與組版：Avery 目前以 `<漫畫資料夾>/<人物>_codex/pen_v4/` ＋ `compose_brand_avery_v4.py` 為準。它沒有 `layout_v13()`，要先補一個回傳 `slot/src/poly/bbox/mask/box/cx` 的函式（或在 GIF 程式裡用它的 `POLYS`、`CROP_X` 自己算）。
- 每格的 `Mover` 多邊形、軸心、權重範圍、補底方向、`refine` 顏色類別：依新底圖重新描（先用 `--debug` 輸出遮罩圖，檢查有沒有圈到臉或背景直線）。
- `color_classes()` 的門檻是依 Maggie 的芥末黃調的（`Y` 類）；Avery 是鼠尾草綠，要重調。
- 眨眼：重新請 Codex 生閉眼變體、重新對位、重新量每隻眼睛的 `up/lo` 點、設定 `BLINK_CENTERS`。
- 臉部檢查表 `FACES`：依新底圖人工標註每張臉。
- 保護區座標（白板圖示、箭頭方框之類）。
- 輸出資料夾與檔名（例如 `<人物>_codex/gif_v1/`），參考靜態單頁改成 Avery 的單頁 PNG。

**先決定再動手**
- 每格挑什麼動作：照第 5 節——動作要看得出來，但 HR 的手部動作要小、不搖頭（要動就眨眼）、手臂不切開、臉不被切到、不用局部變形。
- 有沒有格子需要換格或鏡像：有的話動畫照樣寫在原圖座標，只換版面。
