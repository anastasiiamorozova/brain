# Mózg na pełnych obrotach
An Onet special project: a 3D brain built from the Neurotorium atlas models, followed by the article "24 godziny z życia mózgu".

| Folder | Contents |
|---|---|
| `models/neurotorium/` | 27 structure models from the Neurotorium atlas, Draco-compressed, named by Neurotorium's structure code |
| `textures/matcaps/` | The two matcap images the Neurotorium atlas uses |
| `assets/` | Logos, the author's photo and the food photos for the infographics (`.webp`) |

## Page order

1. **Title screen.** Onet and Medonet logos, the title logo with an animated gauge, the lead, the scroll button and the content partner. The brain spins behind it, fast at first, then slowly.
2. **Introduction**, then a **card for the test** with a short scroll lock. Both keep the title screen's look.
3. **The brain through the day.** A clock at the top, and the brain turns and lights up as the steps scroll by.
4. **The article** with hour clocks, infographics and pull quotes, the expert's bio and the footer.

The title screen's look comes from the `--s1-*` CSS variables. `?s1=pomarancz` shows the earlier orange version, and `?cta=` switches the scroll button.

## Edit the story

Each `<section class="step">` is one card and one keyframe.

| Attribute | Effect |
|---|---|
| `data-time` | Time of day when the card is centred, as `HH:MM`. The clock runs through 24 hours from the first card's time. Times after midnight continue the same night. |
| `data-highlight` | What lights up, separated by spaces. Add a strength after a colon, for example `prefrontal:0.55`. The structure keeps its full shape in a paler colour, and its word in the text turns paler too. In steps where the cortex turns see-through, a strength makes the structure fainter instead. |
| `data-yaw` | Turn in degrees; 0 means the face points at the reader. Keep values going one way for a continuous turn. |
| `data-pitch` | Tilt in degrees. Positive shows the top, negative the underside. |
| `data-zoom` | 1 fits the screen; 1.2 is 20% closer. |

Highlight names: `prefrontal`, `premotor`, `motor`, `parietal`, `temporal`, `occipital`, `visual`, `cingulate`, `pcc`, `cerebellum`, `brainstem`, `hypothalamus`, `scn`, `pineal`, `accumbens`, `hippocampus`, `amygdala`. Groups: `dmn` is the medial prefrontal cortex plus the posterior cingulate cortex, and `neocortex` is the whole cortex. `glymphatic` runs a fluid wave over the brain. Structures inside the brain make the cortex see-through automatically.

In the text, wrap a structure's name in `<mark data-token="hippocampus">…</mark>`. The word gets the same colour as the structure on the model, which is how readers match the two.

Colours are CSS variables at the top of the file, such as `--c-hippocampus`. Structures shown in the same step need clearly different hues. The current pairs are amber and slate blue for the SCN and hypothalamus, teal and peach for the hippocampus and neocortex, and red and blue-violet for the amygdala and visual cortex.

## The models

The list of parts is `ATLAS.parts` in the script. Each entry names a file, the highlight it belongs to, and whether it is drawn only while highlighted.

- **Adding a structure.** Neurotorium serves every structure at `https://neurotorium.org/wp-content/themes/neurotorium/brain_atlas/models/<code>.glb`. The codes and names are in their `data.json`. Download the file, compress it with `npx @gltf-transform/cli draco in.glb out.glb`, save it under `models/neurotorium/`, add an entry to `ATLAS.parts`, and add its code to the preload list in the `<head>`.

Compression shrank the 27 files from about 4 MB to 552 KB. The geometry is unchanged apart from tiny rounding.

## How it works

- **Embed.** Ring Pages height, the paywall and `ASSET_V` cache-busting follow `sample/lalka.html`. Everything is sized from `--screen` in px, never `100vh`. Bump `ASSET_V` whenever a model, texture or image changes.
- **Loading order.** The brain's files are preloaded first. The title screen's content fades in once the brain is ready (or after 8 s), and the photos further down wait for `window.s1Gotowe`.
- **Sticky brain.** `.sticky-visual` is sticky and one `--screen` tall, and `.steps` is pulled up by the same amount, so the cards scroll over it. No parent element may use `overflow: hidden`.
- **Clock header.** The clock sits in its own sticky layer above the story text, which slides under it and fades at the bottom edge. On the last step it moves up with the text. The premium banner hides once the introduction has scrolled past.
- **Scroll to rotation.** The script turns the scroll position into a continuous step index; 2.5 means halfway between the third and fourth card. Rotation, colours and the time are interpolated from it and smoothed every frame.
- **Rendering.** The brain is drawn in one pass straight to a transparent canvas, with the browser's own antialiasing. The page background shows through behind it.
- **Look.** As in the Neurotorium atlas, surfaces use matcap shading, where lighting comes from an image of a lit sphere. Like the atlas, the page reads the matcap colours as linear light and applies ACES filmic tone mapping. Together these give the soft, creamy, low-saturation look. Highlighted structures use a grey matcap tinted with a muted region colour.
- **See-through cortex.** The outer brain is 18 separate parts. Fading them all at once would show every part through every other part, which looks like layers piling up. While the cortex is see-through, a depth-only pass first marks the outermost surface, so only that surface is blended and the brain fades as one shell. Highlighted cortex on the inner walls, such as the DMN hubs, is drawn before that pass, so it stays visible through the shell.
- **Grain.** Off for now. `GRAIN` in the script turns it back on, in the brain's shader and on the page background together.
- **Fallback.** Part 3 of the script generates a simplified brain in code. It is used when the models cannot load, with a matcap painted in code.
