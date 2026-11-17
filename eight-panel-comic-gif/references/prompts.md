# Codex 生圖 prompt 原文摘錄

全部逐字從本機檔案複製（英文原文不改）。路徑相對於 `<漫畫資料夾>/`。
指令：`codex exec --skip-git-repo-check "<下列 prompt>"`，並附上參考圖（附圖用的參數 log 沒有記錄，**未確認**）。

> 底圖 img02～img06 最早的生圖 prompt（Mia 篇 pen_v1～v4 時期）**本機沒有保存，未確認**。下列是找得到的全部。

---

## A. 新圖（無字底圖）

### A1. 第 1 格：童年搬家（`<人物>_codex/pen_v6/prompt_img01.txt`；對應 log 未保存，模型未確認）

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

### A2. 第 7 格：一個月後演練（`<人物>_codex/pen_v5/prompt_img07.txt`；主角當時叫 Mia，後改名 Maggie；log 未保存）

```
Generate ONE new image with your image generation tool (do not write code, do not draw with scripts). Save nothing else.

Style: exactly match the attached reference images - hand-drawn colored ballpoint-pen illustration, dense blue/indigo cross-hatching on cream paper, with warm mustard-yellow and skin tones as accent colours. Landscape 3:2. Keep all important faces and action in the CENTRAL 60% of the frame (the image will be cropped to near-square).

Scene (comic panel 7, "one month later"): a bright office in daytime. A stand-up drill for handling an urgent client request. Characters:
- Mia (from the references): shoulder-length black bob with side-swept bangs, small gold earrings, white inner blouse, mustard-yellow knit cardigan, teal/blue lanyard ID badge, navy trousers. She stands calmly beside a whiteboard, smiling with confidence, holding a tablet.
- The four project teammates from reference image 1 (same faces, hair and clothes: young man with wavy dark hair in light-blue shirt; woman with long brown hair in grey blazer and white turtleneck; man with glasses in navy sweater; woman with hair bun in lavender blouse). They take turns: the man with glasses steps forward holding a phone and a checklist clipboard, the others watch attentively, one holds a stopwatch-style timer (no digits needed).
- Their department manager: a man in his fifties with short grey hair, NO glasses, charcoal-grey suit and a red tie (he must look clearly different from any other man). He holds a tablet and gives a calm nod, decisive.
- On the wall behind: a board with three coloured rows (green check icon row, amber clock icon row, blue check icon row) - icons only, absolutely NO letters.
- A simple wall clock reads about 2 o'clock afternoon.

IMPORTANT: absolutely no text, letters, numbers, Chinese characters, signs or speech bubbles anywhere in the image. No watermark. Leave the lower-left quarter relatively calm (a caption box will be placed there later).
After generating, reply only with the saved image path.
```

### A3. 第 8 格：手沖＋高架棚床（`<人物>_codex/pen_v5/prompt_img08.txt`；log 未保存）

```
Generate ONE new image with your image generation tool (do not write code, do not draw with scripts). Save nothing else.

Style: exactly match the attached reference images - hand-drawn colored ballpoint-pen illustration, dense blue/indigo cross-hatching on cream paper, with warm mustard-yellow, coffee-brown and berry-red accents. Landscape 3:2. Keep the key subjects in the CENTRAL 60% of the frame (it will be cropped to near-square).

Scene (comic panel 8, ending): a dreamy two-layer composition.
- Foreground (lower centre): a pour-over coffee setup on a wooden counter - gooseneck kettle pouring a thin, steady spiral stream of water into a cone dripper over a glass server; the hands and sleeves belong to Mia (mustard-yellow knit cardigan, as in the reference). A small open bowl of roasted coffee beans and a few dried red-purple coffee cherries beside it.
- Background (upper half, fading softly into the steam rising from the cup like a memory): the highlands of Guji, Ethiopia - rows of raised wooden African drying beds on stilts covered with red and dark-purple coffee cherries drying in the sun; several local workers (respectful, natural depiction, East African, wearing simple work clothes and head wraps) take turns gently raking and turning the cherries by hand; a light shade net stretched over part of the beds for the midday sun; green hills and tall shade trees under a big sky.

IMPORTANT: absolutely no text, letters, numbers, labels, Chinese characters, signs, logos on bags or speech bubbles anywhere. No watermark. Leave the lower-left and lower-right corners relatively calm (a caption box will go at the bottom).
After generating, reply only with the saved image path.
```

---

## B. 改圖（以舊底圖為基礎重繪局部）

### B1. HR 年輕化（`<人物>_codex/pen_v9/prompt_img05.txt`；log `pen_v9/codex_img05.log`）

```
Use the image generation tool to create an edited version of the attached illustration (landscape, same aspect ratio as the attached image, about 1672x941).
Keep EVERYTHING identical: composition, camera angle, blue ballpoint-pen cross-hatching style with colored-pencil tints, background, furniture, plants, lighting, the woman in the mustard-yellow cardigan (her face, hair, pose unchanged), the man's pose, hand position, navy suit, striped navy shirt and glasses.
Change ONLY the man in the navy suit: make his face look clearly younger, around 28 to 32 years old. Smooth youthful skin, no wrinkles, no crow's feet, no nasolabial folds, fuller thick black hair with a modern short cut, brighter eyes, slimmer jawline. He is a fictional character and must not resemble any real person.
No text, no letters, no numbers anywhere in the image.
Do not run any shell commands; only generate the image.
```

### B2. HR 拿掉眼鏡、改油頭（`<人物>_codex/pen_v10/prompt_img05.txt`；log：`gpt-6-luna`、effort medium、Codex v0.166.0）

```
Use the image generation tool to create an edited version of the attached illustration (landscape, same aspect ratio as the attached image).
Keep EVERYTHING identical: composition, camera angle, blue ballpoint-pen cross-hatching style with colored-pencil tints, background, furniture, plants, lighting, the woman in the mustard-yellow cardigan (her face, hair, pose unchanged), the man's pose, hands, navy suit, striped navy shirt, his young face (around 30 years old) and expression.
Change ONLY two things on the man in the navy suit:
1. Remove his glasses completely; show his eyes and eyebrows clearly, no glasses anywhere.
2. Change his hair to a neat, tidy slicked-back pompadour (classic glossy side-parted slick back, combed neatly, no loose strands).
He is a fictional character and must not resemble any real person.
No text, no letters, no numbers anywhere in the image.
Do not run any shell commands; only generate the image.
```

### B3. HR 臉部改成原子筆排線畫風（`<人物>_codex/pen_v12/prompt_hr.txt`；img05、img06 共用；log：`gpt-6-luna`、effort medium、Codex v0.166.0）——**定稿底圖 pen_v12 用的就是這份**

```
Use the image generation tool to create an edited version of the attached illustration (landscape, same aspect ratio as the attached image).
Keep EVERYTHING identical: composition, camera angle, background, furniture, plants, lighting, colors, the woman in the mustard-yellow cardigan (unchanged), the man's pose, hands, navy suit, striped navy shirt, his face shape, features, expression, age (around 30), no glasses, and his neat slicked-back side-parted hair.
Change ONLY the rendering technique of the man's face, neck and hair: they currently look too smooth and airbrushed, almost photographic. Redraw them in exactly the same hand-drawn colored ballpoint-pen style as the woman's face and the rest of the picture: visible blue ballpoint-pen line work, fine cross-hatching and diagonal hatching strokes for shading, slightly sketchy outlines, warm colored-pencil skin tints with visible pencil grain, paper texture showing through. No smooth gradients, no glossy highlights, no photorealism on his face or hair.
He is a fictional character and must not resemble any real person.
No text, no letters, no numbers anywhere in the image.
Do not run any shell commands; only generate the image.
```

log 備註：`pen_v12/codex_img05.log` 有一行 `ERROR codex_core::tools::router: error=missing field 'prompt'`，但最後仍回報生成成功；另有啟動時 `failed to install system skills … 存取被拒` 的錯誤，不影響生圖。

---

## C. 動畫用變體：五人閉眼（`<人物>_codex/gif_v3/_scratch/codex_blink.log`，2026-11-17）

log 檔頭：`OpenAI Codex v0.169.0`、`model: gpt-6.1-sol`、`sandbox: read-only`、`reasoning effort: low`。輸入 `pen_v12/img02.png`，輸出存成 `gif_v3/_scratch/codex_closed.png`（1536×1024，和原圖同尺寸）。

```
Use the image generation tool to create an edited version of the attached illustration (landscape, same aspect ratio and same framing as the attached image).
Keep EVERYTHING identical: composition, camera angle, every person's position, pose, head angle, face shape, hair, clothes, hands, phones, background, office, windows, colors and the hand-drawn colored ballpoint-pen style with hatching.
Change ONLY the eyes of all five people: the woman in the mustard-yellow cardigan on the phone (center), the woman top-left, the young man bottom-left, the man with glasses top-right, and the woman bottom-right. Each of them has their eyes gently CLOSED, as in the middle of a natural blink: upper eyelids fully lowered, a thin curved eyelash line drawn with ballpoint pen where the eye was, skin of the eyelid drawn with the same pen hatching and warm skin tint. Eyebrows unchanged, expressions otherwise unchanged.
No text, no letters, no numbers anywhere in the image.
Do not run any shell commands; only generate the image.
```

寫法重點：
- `same framing` 讓 Codex 盡量不動構圖（這次五張臉對位結果都是原位、縮放 1）。
- 逐一點名每個人的位置與衣著，避免漏掉或改錯人。
- 描述閉眼的筆觸（ballpoint-pen eyelash line、hatching、warm skin tint），避免貼色塊感。
- 就算對位是原位，仍然**只取眼周**，其他像素一律用原圖。

---

## D. 共通寫法清單

- 新圖開頭：`Generate ONE new image with your image generation tool (do not write code, do not draw with scripts).`
- 改圖開頭：`Use the image generation tool to create an edited version of the attached illustration (landscape, same aspect ratio …)`，接 `Keep EVERYTHING identical: …`、`Change ONLY …`。
- 畫風：`hand-drawn colored ballpoint-pen illustration, dense blue/indigo cross-hatching on cream paper` ＋角色色（芥末黃等）。
- 構圖：重要臉孔在中央 60%（輪播會裁成近正方形）；左下或底部留白給對白框。
- 一律：無文字（含門牌、招牌、螢幕、紙張）、虛構人物不得像真人、`Do not run any shell commands; only generate the image.`
- 要動畫變體（閉眼、張嘴等）時：`same framing`、`Change ONLY the eyes of …`，之後只取局部、對位、羽化。不要請生圖模型直接出動畫幀。
