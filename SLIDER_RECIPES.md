# Slider recipes (bones + shapekeys)

Source of truth for how each morph control behaves. UI labels only show drive type:

| Tag | Meaning |
|-----|---------|
| **(Bn)** | Bone / CTRL overlay (scale, translate, or rotate after mixer) |
| **(Sk)** | Shapekey axis (morph target influence) |

Internal amp spans (`small`/`big`/`custom`) stay in code for math only — they are **not** shown in the UI.

Canonical defs live in `assets/viewer/index.html`: `PROP_DEFS`, `FORM_CTRL_DEFS`, `MORPH_CATEGORIES`.

---

## Conventions

### Slider domain

| Drive | Typical slider | 0 means |
|-------|----------------|---------|
| Bn prop / form | −1 … +1 | bind rest |
| Bn height / head | −1 … +1 | mid / recipe baseline |
| Sk bipolar | −1 … +1 | both morphs off |
| Sk unipolar | 0 … 1 | morph off |

### Bone prop scale (`PROP_DEFS`)

`propRangeMul(v)`:

- **small**: ×(1 − 0.1·\|…\|) → ×0.9…1.1
- **big**: ×0.8…1.2
- **amp**: ×(1 − amp) … ×(1 + amp)
- **minMul / maxMul**: −1 → minMul, 0 → 1, +1 → maxMul

Applied on listed local axes of CTRL bones after height/stature.

### Form CTRL (`FORM_CTRL_DEFS`)

Applied after mixer + height, from captured bind rest:

| mode | amp | min/max or minMul/maxMul |
|------|-----|--------------------------|
| `scl` | rest × (1 + v·amp·sign) | or −1→minMul … +1→maxMul |
| `pos` | rest + v·amp·sign (meters) | or −1→min … +1→max |
| `rot` | Euler radians on axis | or −1→min … +1→max |

`axis`: `x` \| `y` \| `z` \| `xyz`. `sign` mirrors L/R when needed.

### Shapekey axes (`axisCtrl`)

- Positive targets at +1 (each key capped by `_0.N` suffix → max influence).
- Negative targets at −1 (same).
- Unipolar: only positive list, slider 0…1.
- Multi-key: `KeyA)+(KeyB` blends several keys on one side.

---

## Height (Bn)

Mode: **parents** — scale CTRL overlay bones after mixer (not Height Action bake).

| Channel | Formula (−1…+1) |
|---------|-----------------|
| Stature isotropic helper | `1 + 0.2 · H` |
| Torso length (local Y) | `1 + 0.12 · H` |
| Stature width (local X/Z) | `1 + 0.04 · H` |
| Leg length (thigh/shin Y) | `1 + 0.322 · H` |
| Ground offset | follows stature isotropic |

Neck length (Bn): ×(1 + 0.2 · neckLength) on neck CTRLs, plus head lift along local +Y by `(N−1)×neckWorldLength`.

Head size (Bn): visual base `HEAD_BASE = 1.2` at slider 0; then ×`(1 + 0.2 · headSize)`; above 0, X/Z grow at 70% of Y (throat damp).

---

## Body bones (Bn) — `PROP_DEFS`

| Control | Bones | Axes | Span |
|---------|-------|------|------|
| Shoulder width | `CTRL-ORG-shoulder.L/R` | Y | small ×0.9…1.1 |
| Chest width | `CTRL-tweak_spine.003` | X | ×0.9…1.3 |
| Chest thickness | `CTRL-tweak_spine.003` | Z | ×0.9…1.3 |
| Waist width | `CTRL-tweak_spine.002` | X | ×0.9…1.3 |
| Waist thickness | `CTRL-tweak_spine.002` | Z | ×0.9…1.3 |
| Hip width | `CTRL-tweak_spine.001` | X | amp 0.3 → ×0.7…1.3 |
| Hip thickness | `CTRL-tweak_spine.001` | Z | ×0.9…1.3 |
| Lats width | `CTRL-DEF-spine.003` | X | big ×0.8…1.2 |
| Arm size | `CTRL-DEF-upper_arm.L/R` | XZ | big ×0.8…1.2 |
| Leg thickness | `CTRL-DEF-thigh.L/R` | X | small ×0.9…1.1 |
| Hip bone size | `CTRL-tweak_spine` | X | big ×0.8…1.2 |

---

## Form CTRLs (Bn) — `FORM_CTRL_DEFS`

### Eyes

| Control | Bone(s) | Mode · axis | Span |
|---------|---------|-------------|------|
| Eye forward | `CTRL-DEF-eye_master` | scl · Y | ×1.0…1.2 |
| Eye distance | `CTRL-eye_master.L/R` | pos · X | amp 0.012 · slider −0.2…0.3 (L+ / R− → wider) |
| Eye size horizontal | `CTRL-eye_master.L/R` | scl · X | amp 0.1 |
| Eye size vertical | `CTRL-eye_master.L/R` | scl · Z | amp 0.2 |
| Lid root scale Z | `CTRL-DEF-lid.L/R` | scl · Z | amp 0.1 |
| Lid root rotate Y | `CTRL-DEF-lid.L/R` | rot · Y | amp 0.1 (L− / R+) |
| Lid B / B.001…003 loc Z | matching `CTRL-DEF-lid.B.*` | pos · Z | amp 0.001 |
| Lid T / T.001…003 loc Z | matching `CTRL-DEF-lid.T.*` | pos · Z | amp 0.001 |

### Brows

| Control | Bone | Mode · axis | Span |
|---------|------|-------------|------|
| Eyebrows vertical | `CTRL-DEF-brow_master` | pos · Z | amp 0.002 |

### Nose

| Control | Bone | Mode · axis | Span |
|---------|------|-------------|------|
| Upper nosebridge vertical | `CTRL-DEF-NosebridgeUP` | scl · Z | ×0.5…1.1 |
| Upper nosebridge width | `CTRL-DEF-NosebridgeUP` | scl · X | ×0.6…1.2 |
| Lower nose width | `CTRL-ORG-nose_master` | scl · X | ×0.72…1.5 |
| Lower nose angle | `CTRL-ORG-nose_master` | rot · X | −0.15…0.1 rad |
| Nose ridge width | `CTRL-DEF-nose_ridge` | scl · X | ×0.25…1.6 |
| Nose volume | `CTRL-DEF-nose_master` | scl · Z | ×0.8…1.2 · slider **−0.4…0.4** |
| Nose vertical position | `CTRL-DEF-nose_master` | pos · Z | amp 0.002 |

### Cheeks

| Control | Bones | Mode · axis | Span |
|---------|-------|-------------|------|
| Upper cheekbone horizontal | `CTRL-DEF-cheek.T.L/R` | pos · Z | amp 0.004 |
| Upper cheekbone vertical | `CTRL-DEF-cheek.T.L/R` | pos · X | amp 0.004 (L− / R+) |
| Lower cheekbone horizontal | `CTRL-DEF-cheek.B.L/R.001` | pos · Z | amp 0.004 |
| Lower cheekbone vertical | `CTRL-DEF-cheek.B.L/R.001` | pos · X | amp 0.004 (L− / R+) |

### Jaw / mouth / lips

| Control | Bone(s) | Mode · axis | Span |
|---------|---------|-------------|------|
| Jaw up/down tilt | `CTRL-DEF-jaw_master` | rot · X | amp 0.02 |
| Mouth up/down | `CTRL-DEF-jaw_master` | pos · Z | amp 0.003 |
| Mouth side tilt | `CTRL-DEF-jaw_master` | rot · Y | amp 0.02 |
| Jaw forward/backward | `CTRL-DEF-jaw_master` | scl · Y | ×0.9…1.1 |
| Jaw and chin width | `CTRL-DEF-jaw_master` | scl · X | ×0.9…1.1 |
| Chin vertical size | `CTRL-DEF-jaw_master` | scl · Z | ×0.8…1.05 |
| Upper lip width | `CTRL-DEF-upperlip` | scl · X | ×1.0…1.5 · slider **0…0.3** |
| Lower lip width | `CTRL-DEF-lowerlip` | scl · X | ×1.0…1.5 · slider **0…0.3** |
| Lips forward/backward | upper + lower lip | scl · Y | ×1.0…1.35 · slider **0…1.3** |

### Body fatty

| Control | Bones | Mode · axis | Span |
|---------|-------|-------------|------|
| Breast size | `CTRL-DEF-breast_size.L/R` | scl · xyz | ×0.9…1.6 |
| Breast volume | `CTRL-DEF-breast_volume.L/R` | scl · xyz | ×0.9…1.4 |
| Glute size | `CTRL-DEF-glute.L/R` | scl · xyz | ×0.9…1.3 |

---

## Shapekey axes (Sk) — by category

Suffix `_0.N` = max influence at that end of the slider. `null` negative = unipolar 0…1.

### Head shapes

| Control | + | − |
|---------|---|---|
| Gender expression | Face gender expression @ mid 0.5 | — (0…1) |
| Face width | Wide Head @0.5 | Skinny Head @0.2 |
| Face height | Tall Head @0.4 | Short Head @0.4 |
| Face fullness | Large Face @0.5 | Skinny Face @0.5 |
| Neck size | Large Neck @0.5 | Skinny Neck @0.5 |
| Head cranium size | Big Cranium | Small Cranium @0.4 |
| Head vertical tilt | Lower Jaw Protrusion @0.2 | Forehead Protrusion @0.2 |
| Head shape morph | Taper Bottom Wide @0.4 | Taper Top Wide @0.4 |
| Temporal bone width | Head Side Width Out @0.5 | — |
| Lower face length | Lower Face Long @0.3 | Lower Face Short @0.3 |

(+ Head size is **Bn**, see height/head above.)

### Body shapes

| Control | + | − |
|---------|---|---|
| Gender expression | Body gender expression @ mid 0.5 | — |
| Torso size | Large Torso | Skinny Torso | slider −0.2…0.2 |
| Breast size roundness | Large Breasts | Skinny Breasts | slider −0.2…0.2 |
| Butt size roundness | Large Butt | Skinny Butt | slider −0.4…0.4 |
| Muscular morph | Muscles | Simplify Body |
| Big belly | Big belly | — |

Body shapes order: Gender → Torso → Breast size (Bn) → Breast volume (Bn) → Breast size roundness (Sk) → Glute size (Bn) → Butt size roundness (Sk) → Muscular → Big belly.

### Eyes (Sk)

| Control | + | − |
|---------|---|---|
| Eye size | Big Eyes @0.5 | Small Eyes @0.3 |
| Eye monolid | Monolid | — |

### Brows (Sk)

| Control | + | − |
|---------|---|---|
| Brow depth | Brow Forward | Brow Back @0.5 |
| Brow horizontal | Brow Close | Brow Apart |
| Brow vertical | Brow Up @0.5 | Brow Down @0.5 |
| Inner brow horizontal | Inner Brow Up @0.8 | Inner Brow Down |

### Jaw (Sk)

| Control | + | − |
|---------|---|---|
| Jaw size | Square Jaw | Thin Jaw |
| Jaw position | Jaw Up | Jaw Down |
| Jaw Forward | Jaw Forward | — |
| Jaw width | Jowls Large | Jowls Small @0.3 |
| Chin FW/BW | Chin Forward | Chin Back |
| Chin shape | Square Chin | Narrow Chin |
| Chin length | Long Chin | Short Chin |
| Cleft chin | Butt Chin | — |
| Cheek fullness | Full Cheeks @0.3 | Sunken Cheeks @0.5 |
| Cheekbones position | Cheekbones Up @0.5 | Cheekbones Down @0.5 |
| Cheekbones size | Pronounced Cheekbones @0.7 | Flat Cheekbones @0.7 |

### Mouth (Sk)

| Control | + | − |
|---------|---|---|
| Mouth width | Wide Mouth @0.7 | Narrow Mouth @0.7 |
| Mouth position | Mouth Down @0.6 | Mouth Up @0.6 |
| Mouth corner shape | Mouth Corners Up | Mouth Corners Down @0.4 |
| Upper lip size | Full Top Lip @0.4 | Thin Top Lip @0.4 |
| Bottom lip size | Full Bottom Lip @0.2 | Thin Bottom Lip @0.7 |
| Cupids Bow shape | Remove Cupids Bow | — |

### Nose (Sk)

| Control | + | − |
|---------|---|---|
| Nose width | Small Nose @1 | Small Nose @−1 |
| Nose scale | Large Nose @0.5 | Large Nose @−0.5 |
| Nose vertical | Long Nose @0.5 | Short Nose @0.5 |
| Nose Bridge | Bridge Out | Bridge In · slider **±0.3** |
| Nose tip shape | Button Nose @0.8 | Pointy Nose @0.8 |
| Nose tip length | Pinocchio @0.6 | — |
| Nose depth | Nose Out @0.2 | Nose In @0.2 |
| Flatten nose | Flatten Nose @0.5 | — |
| Nostrils | Big Nostrils @0.7 | Small Nostrils @0.7 |

### Ears (Sk)

| Control | + | − |
|---------|---|---|
| Ear size | Big Ears | Small Ears @0.5 |
| Ear vertical | Ears Up @0.4 | Ears Down @0.4 |
| Ear forward | Ears Forward @0.6 | — |
| Ear out | Ears Protruding + Ears Side Tilt | — |
| Pointy ear | Pointy Ears @0.5 | — |
| Round ear | Round Ears | — |

### Specials (Sk)

| Control | + | Anim |
|---------|---|------|
| Fangs | Vampire Teeth + Lower Fangs | idle_mouthopen |
| Split tongue | Forked Tongue | idle_mouthopen_tongueout |

---

## Materials (untagged)

| Control | Behavior |
|---------|----------|
| Iris hue | 0…1 hue on iris composite; pupil stays black |

---

## Randomizer

Category **Randomizer** shows one **Gender expression** slider (0 = feminine, 1 = masculine) that sets both `face_gender_expression` and `body_gender_expression`.

**Randomize** rolls every other control using `RANDOM_BIAS` / `RANDOM_CORRELATE` in `index.html`:

| Lean | Examples | Notes |
|------|----------|-------|
| `female` | **breasts (strong)**; glutes/lips (soft) | Breasts stay tightly female-leaning |
| `male` | muscle, arms, jaw (soft) | Women can still roll big arms |
| `neutral` | belly, torso, most face, iris, body width/thickness | Full / correlated random |
| `skip` | gender pair, **height**, fangs, tongue | Unchanged |

**Face soft:** head/eyes/brows/jaw/ears/specials sample bipolar Sk inside **±0.32** and face Bn forms inside **±0.35**. **Mouth + nose** use full control range (harder), except upper/lower lip width (Bn) and nose scale (Sk) which stay soft. Materials category is not randomized.

**Unipolar 0…1 shapekeys** (no negative twin — e.g. Pointy ear, monolid, flatten nose, big belly):
- Usually **0…0.10** (not mid 0.5)
- ~8% of rolls can reach up to **0.70**

**Correlated torso:** `chestWidth` / `waistWidth` / `hipWidth` share a base and stay within **±0.20** of it (same for chest/waist/hip **thickness**). Shoulders/lats/hipBone/legs use a looser ±0.16 group.

**Muscle vs fat:** if `muscular_morph` **> 0.40**, then `big_belly` ≤ **0.05** and chest/waist/hip thickness ≤ **+0.20**.

**Gender expression floors** (after sampling; `g` = Gender expression 0 fem … 1 masc):

| When | Floors / caps |
|------|----------------|
| Always | Glute size (Bn) ≥ **0.20** |
| Near full female (`g` ≤ 0.15) | Hip bone ≥ **0.30**; Chest thickness ≤ **0.10**; Breast size ≥ **0.10**; Breast volume ≥ **0.50**; Butt roundness (Sk) ≥ **0.10** |
| Near full male (`g` ≥ 0.85) | Head shape / Lower face length ≥ **−0.20**; Shoulder width ≥ **0**; Chest width ≥ **0.20**; Chest thickness ≥ **0**; Lats width ≥ **0**; Hip bone ≤ **0.20**; Arm size ≥ **−0.30**; Muscular morph ≥ **−0.70**; Breast roundness (Sk) = **0** |

**Head size:** ±0.12 only. Height stays fixed.

---

## Materials textures

Pack under `assets/textures/`:

| Slot | Path pattern | Notes |
|------|--------------|-------|
| Skin diffuse | `body/main/diffuse/*`, `face/main/diffuse/*` | Caucasian / Black baked, or **White + SkinDepth OKLCH** (luminance detail) |
| Body rough / normal | `body/main/roughness`, `body/main/normalmap` | Always on body — **not** used for redness |
| Head rough | `face/main/roughness` | Head has no normal in pack |
| Face overlays | `face/overlays/{makeup,extra,eyebrows}` | Makeup/extra = alpha; eyebrows = white-key |
| Body overlays | `body/overlays/extra`, `body/overlays/underwear/<Item>/` | Underwear: `*_D` composited; `*_N`/`*_R` available |
| Eyes | `eyes/{TOON,REAL,SNAKE}_*` + shine | Iris hue tints chosen style |

**Skin params** (Custom White base):

| Param | Range | Role |
|-------|-------|------|
| Skin depth | 0…1 | OKLCH anchors (fair→deep); interpolated |
| Warmth | −1…1 | Hue += Warmth × 18° |
| Redness | −1…1 | Hue −= Redness × 10°; Chroma += max(Redness,0)×0.015 |
| Saturation | 0…1.5 | Multiplies chroma |
| Lip pink / strength | 0…1 | Head only, via `face/main/diffuse/Lip_Mask.png` (same UV as White_Head_D) |

White albedo is **luminance detail**, not RGB multiply:

`FinalL = SkinL + (TextureL − 0.85) × 0.35` then `OKLCH(FinalL, SkinC, FinalH)` → sRGB.

**Lips:** prefer a grayscale **lip mask** (same head UV; white = lips) + Lip pink/strength. You do **not** need a full separate lip albedo — mask + OKLCH is enough. Fair Skin depth auto-boosts lip influence.

---

## Blink overlay (`anim_blink`)

App-driven shapekey blink (not a bone Action):

- Key name: **`anim_blink`** (0 = open, 1 = closed)
- Applied every frame after mixer + form CTRLs so lids follow eye bones
- Auto interval ~1.6–5s with occasional double-blink
- Console: `[blink] overlay on "anim_blink"` when the key exists in the GLB
- Debug: `sandboxBlinkNow()`, `sandboxSetBlink(false)`, `sandboxGetBlink()`

---

## When editing recipes

1. Change numbers in `PROP_DEFS` / `FORM_CTRL_DEFS` / `axisCtrl(...)` in `index.html`.
2. Update this file to match.
3. UI will keep showing only `(Bn)` / `(Sk)` — do not reintroduce Big/Small tags in labels.
