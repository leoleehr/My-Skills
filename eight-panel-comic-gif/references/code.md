# 程式路徑地圖＋核心函式節錄

根目錄：`C:\Agent\LinkedIn\output\人才VS咖啡_八格漫畫\Maggie_codex\`（以下用相對路徑）。
程式不複製整份；要改或重用請直接開原始檔。下列節錄只保留核心邏輯（註解精簡過）。

## 1. 路徑地圖

| 用途 | 檔案 | 說明 |
|---|---|---|
| 無字底圖 | `pen_v12/img01～08.png` | 第 N 格來源底圖；`img05_v10.png`、`img06_v10.png` 為重繪前的備份 |
| 生圖紀錄 | `pen_v12/prompt_hr.txt`、`pen_v12/codex_img05.log`、`codex_img06.log`；`pen_v10/`、`pen_v9/`、`pen_v6/prompt_img01.txt`；`../Mia_codex/pen_v5/prompt_img07/08.txt` | prompt 原文見 `prompts.md` |
| 組版（定稿） | `compose_brand_v13b.py` | `RAW/POLYS/ORDER/LINES/CROP_X/PAGE_WHERE`、`cover()`、`caption()`、`page_captions()`、`number_badge()`、`header()`、`footer()`、`layout_v13()`（別名 `layout_v13b`） |
| 組版（前版） | `compose_brand_v12.py`（原版面）、`compose_brand_v13.py`（第三列鏡像） | |
| 腳本 | `腳本_Maggie八格漫畫_v13.md` | 角色外型表、逐格畫面與對白 |
| 動作（核心） | `gif_v3/animate_v3.py` | 所有工具與 `panel(i, p)`；v4、v5 只 import 這支 |
| 眨眼 | `gif_v3/eyes_v3.py` | `Eye`、`closure`、`apply_blinks`、`BLINK_CENTERS`；`python eyes_v3.py` 出放大校準圖 |
| 閉眼對位 | `gif_v3/_scratch/align_codex.py`、`codex_align.json`、`codex_closed.png`、`codex_blink.log` | |
| 舊眨眼草稿 | `gif_v3/blink.py` | **已不使用** |
| 定稿組版＋編碼 | `gif_v5/animate_v5.py` | `paste()`、`static_bg()`、`text_layer()`、`down()`、`main()`、`write_gif()` |
| 驗證 | `gif_v3/check_frames.py`（文字層、接縫）、`gif_v3/check_p5.py`（保護區、整數平移）、`gif_v3/check_v3.py`（臉部扣眼睛、手臂、放大圖）、`gif_v4/check_v4.py`、`gif_v5/check_v5.py`（定稿：靜態一致、文字層、出格、IoU、接縫、放大圖） | |
| 臉部檢查 | `_v13b_check/face_check.py`（`python face_check.py compose_brand_v13b.py Maggie原子筆版v13b`）→ `*_臉部檢查.png/.csv` | |
| 第一版（位移場，**反例**） | `gif_v1/animate_v12.py` | 直線會變波浪；MP4 的 ffmpeg 指令寫在檔頭 |
| 被 Codex 改壞的資料夾 | `gif_v2/`（只剩 README 與 14:5x 的驗證檔） | **不要參考** |

執行順序（新的一篇）：
```
python compose_brand_<人物>.py pen_vXX <前綴>          # 靜態單頁＋輪播
python face_check.py compose_brand_<人物>.py <前綴>      # 臉部檢查（先過這關）
python eyes_<人物>.py                                    # 眨眼校準圖（有眨眼才需要）
python animate_<人物>.py --debug                         # 遮罩圖＋單格測試幀
python animate_<人物>.py                                 # 約 2.5 分鐘（Maggie 實測）
python check_<人物>.py                                   # Q1～Q11
```

## 2. 節錄：時間函式（`gif_v3/animate_v3.py`）

```python
def smoothstep(e0, e1, x):
    t = np.clip((x - e0) / (e1 - e0), 0, 1)
    return t * t * (3 - 2 * t)

def pulse(p, center, width):          # 循環座標上的高斯脈衝
    d = (p - center + 0.5) % 1 - 0.5
    return float(np.exp(-(d / width) ** 2))

def wave(p, cycles, phase=0.0):       # 0～1 平滑起伏，p=0 時為 0
    return 0.5 - 0.5 * np.cos(TAU * (cycles * p + phase))

def pop(p, c, w=0.10):                # 彈出：放大略過頭 → 回落 → 淡出
    d = (p - c + 0.5) % 1 - 0.5
    if d < -0.04 or d > w + 0.12: return 0.0
    if d < 0: return float(smoothstep(-0.04, 0.0, d)) * 1.15
    return float(1.15 - 0.15 * smoothstep(0, 0.03, d)) * float(1 - smoothstep(w, w + 0.12, d))
```

## 3. 節錄：剛性圖層 `Mover.apply`

```python
class Mover:
    """poly 圈出的部位整塊以 pivot 旋轉 deg（正＝順時針）、縮放、平移。
    wfn(X,Y)：1＝整塊剛性，0＝不動，中間是和身體相接的過渡區。
    fill=("streak", 方向[, 雙側]) 或 ("offset", fn)：部位離開後露出的縫怎麼補。
    refine=(允許的顏色類別, 強制納入多邊形)：只帶人物像素，背景不跟著動。"""
    def apply(self, arr, src, deg=0.0, t=(0.0, 0.0), scale=1.0):
        x0, y0, x1, y1 = self.box
        W = self.W
        th = np.radians(deg) * W
        s = 1 + (scale - 1) * W
        px, py = self.pivot
        dx = (self.X - t[0] * W - px) / s
        dy = (self.Y - t[1] * W - py) / s
        c, sn = np.cos(th), np.sin(th)
        sx = px + c * dx + sn * dy            # 反向取樣
        sy = py - sn * dx + c * dy
        layer = bilinear(src, sx, sy)
        alpha = bilinear(self.maskf, sx, sy)[..., None]
        under = np.where(self.mloc[..., None], self.plate, src[y0:y1, x0:x1])   # 補底
        arr[y0:y1, x0:x1] = under * (1 - alpha) + layer * alpha
```

要點：
- 整數平移（例如點白板）時呼叫端先 `round()`，bilinear 在整數位移下等於原像素搬移，所以 Q7 能驗到差 0。
- `streak_fill`：遮罩內每點沿方向（與反方向）走到遮罩外取像素，兩側依距離內插——門框、牆面直條紋接得起來。
- `offset_fill`：從固定偏移的乾淨區域複製（白板）。
- `color_classes`：3×3 平均後分類 `Y` 黃毛衣（r−b>40）、`s` 皮膚／暖色（r−b>12）、`#` 很暗（lum<80）、`n` 藍、`b` 淺藍、`h` 中暗、`w` 中性亮。**換角色主色時要重調**。

## 4. 節錄：以關節為軸 `RotArm.apply`

```python
def apply(self, arr, src, deg):
    x0, y0, x1, y1 = self.box
    th = np.radians(deg) * self.W            # W：手肘附近漸變到 0
    px, py = self.pivot
    dx, dy = self.X - px, self.Y - py
    c, sn = np.cos(th), np.sin(th)
    layer = bilinear(src, px + c * dx + sn * dy, py - sn * dx + c * dy)
    alpha = bilinear(self.maskf, px + c * dx + sn * dy, py - sn * dx + c * dy)[..., None]
    gx, gy = self.gap                          # 只往一個方向轉，縫都在同一側
    plate = src[y0 + gy:y1 + gy, x0 + gx:x1 + gx]   # 每幀從當幀偏移處取白板排線，色光一致
    under = np.where(self.mloc[..., None], plate, src[y0:y1, x0:x1])
    arr[y0:y1, x0:x1] = under * (1 - alpha) + layer * alpha
```

## 5. 節錄：光效

```python
def screen(sub, col, g):     # 濾色：只會變亮，不蓋筆觸
    return 255 - (255 - sub) * (1 - g[..., None] * np.asarray(col, np.float32) / 255)

# glow：高斯半徑 radius，外圈 3.0～4.8σ² 收邊，用 screen 疊
# tint：同樣的高斯，但用正片疊底 arr*(1-g+g*col/255)——白底上看得到、不洗白圖示
# shadow_band：沿 (X + Y*angle) 的高斯暗帶，位置 pos = -2.5w + (W+5w)*p，p=0/1 都在畫面外 → 無縫
```

## 6. 節錄：眨眼狀態（`gif_v3/eyes_v3.py`）

```python
def closure(f, centers, n=60):   # 全閉 1、前後一幀 0.5、其他 0
    for c in centers:
        d = (f - c + n // 2) % n - n // 2
        if d == 0: return 1.0
        if abs(d) == 1: return 0.5
    return 0.0

# Eye.__init__ 的遮罩：
diff = blur(|orig - closed|.mean(2), 1.5)
m = (diff > THR) & ellipse              # 只留真的不同的像素
alpha = blur(maxfilter(m, 7) ∩ ellipse, 1.2) * ellipse   # 橢圓外一律 0
# 全閉：out = orig*(1-a) + closed*a
# 半閉：逐欄，U(上睫毛中心)～M=U+0.5*(D-U-t/2) 蓋 Codex 眼瞼；原睫毛線搬到 M，以 min()（變暗）混合
```

## 7. 節錄：逐幀組版（`gif_v5/animate_v5.py`）

```python
LAYOUT = cb.layout_v13()                       # 每格 bbox / mask / cx / src
def paste(page, c, arr):
    bx0, by0, bx1, by1 = c["bbox"]
    img = Image.fromarray(np.round(arr).astype(np.uint8))
    page.paste(cb.cover(img, bx1 - bx0, by1 - by0, c["cx"]), (bx0, by0), c["mask"])

top = down(text_layer())                       # 文字層：畫一次、縮一次（RGBa premultiplied LANCZOS）
static_small = alpha_composite(down(static_bg()), top)
lock = np.asarray(top)[..., 3] >= 235
for f in range(60):
    page = bg0.copy()
    for c in LAYOUT:
        paste(page, c, av3.panel(c["src"] + 1, f / 60))   # 動畫在原圖座標
    fr = np.asarray(alpha_composite(down(page), top).convert("RGB"))
    frames.append(Image.fromarray(np.where(lock[..., None], static_arr, fr)))   # 文字每幀差 0
```

## 8. 節錄：GIF 編碼 `write_gif()`

```python
sample = vstack([static_small] + [frames[k] for k in (12, 20, 45)])
pal = sample.quantize(colors=255, method=MEDIANCUT, dither=NONE)
palette = pal.getpalette()[:255*3] + [255, 0, 255]          # 索引 255 = 透明
sq = static_small.quantize(palette=pimg, dither=FLOYDSTEINBERG)   # 靜態像素量化一次
bayer = ((4x4 Bayer) + 0.5) / 16 - 0.5 ; dith = tile(bayer) * 10
for fr in frames:
    ch = |fr - static|.max(2) > 5 ; chd = dilate(ch, 2) & ~lock
    q = quantize(fr + dith, palette, dither=NONE)
    cur = where(chd, q, sq)
    if prev is not None:                                      # 時間穩定
        keep = chd & (err(prev) <= err(cur) + 10)
        cur = where(keep, prev, cur)
for k: enc = cur.copy(); enc[cur == idx[k-1]] = 255            # 沒變的像素透明
out[0].save(path, save_all=True, append_images=out[1:], duration=100, loop=0,
            disposal=1, transparency=255, optimize=False)
```

## 9. 節錄：原圖座標 → 單頁座標（`_v13b_check/face_check.py` `to_page()`）

```python
def to_page(c, box):
    bx0, by0, bx1, by1 = c["bbox"]; w, h = bx1 - bx0, by1 - by0
    iw, ih = 底圖尺寸
    r = max(w / iw, h / ih)
    sw, sh = int(iw * r + 1), int(ih * r + 1)
    ox, oy = int((sw - w) * c["cx"]), (sh - h) // 2
    rx, ry = sw / iw, sh / ih
    x0, y0, x1, y1 = box
    return (bx0 + x0*rx - ox, by0 + y0*ry - oy, bx0 + x1*rx - ox, by0 + y1*ry - oy)
# GIF 座標 = 單頁座標 × 1080 / 1996
```
