# 24 godziny z życia mózgu: 3D scrollytelling graphic

`index.html` is the graphic. A 3D brain built from the Neurotorium atlas models stays pinned while the article cards scroll past. As the reader scrolls, the brain turns, the structures named in the text light up, and a sun moves along a 24-hour progress bar at the top.

## View it

The models are loaded over the network, so serve the folder instead of opening the file directly:

```
python3 -m http.server 8000
```

Then open http://localhost:8000. Opened straight from disk, the page shows a simplified brain generated in code instead of the atlas models.

## Files

| Path | Contents |
|---|---|
| `index.html` | Page, styles and script |
| `models/neurotorium/` | 26 structure models from the Neurotorium atlas, Draco-compressed, named by Neurotorium's structure code |
| `textures/matcaps/` | The two matcap images the Neurotorium atlas uses |

## Edit the story

Each `<section class="step">` is one card and one keyframe.

| Attribute | Effect |
|---|---|
| `data-time` | Time of day when the card is centred, as `HH:MM`. The bar spans 24 hours from the first card's time. Times after midnight continue the same night. |
| `data-highlight` | What lights up, separated by spaces. Add a strength after a colon, for example `prefrontal:0.45`. |
| `data-yaw` | Turn in degrees; 0 means the face points at the reader. Keep values going one way for a continuous turn. |
| `data-pitch` | Tilt in degrees. Positive shows the top, negative the underside. |
| `data-zoom` | 1 fits the screen; 1.2 is 20% closer. |

Highlight names: `prefrontal`, `premotor`, `motor`, `parietal`, `temporal`, `occipital`, `visual`, `cingulate`, `pcc`, `cerebellum`, `brainstem`, `hypothalamus`, `scn`, `accumbens`, `hippocampus`, `amygdala`. Groups: `dmn` is the medial prefrontal cortex plus the posterior cingulate cortex, and `neocortex` is the whole cortex. `glymphatic` runs a fluid wave over the brain. Structures inside the brain make the cortex see-through automatically.

In the text, wrap a structure's name in `<mark data-token="hippocampus">…</mark>`. The word gets the same colour as the structure on the model, which is how readers match the two.

Colours are CSS variables at the top of the file, such as `--c-hippocampus`. Structures shown in the same step need clearly different hues. The current pairs are amber and slate blue for the SCN and hypothalamus, teal and peach for the hippocampus and neocortex, and red and blue-violet for the amygdala and visual cortex.

## The models

The list of parts is `ATLAS.parts` in the script. Each entry names a file, the highlight it belongs to, and whether it is drawn only while highlighted.

- **Splits.** Two parts are split by position. The inner face of the prefrontal cortex is the medial prefrontal cortex. The back 35% of the cingulate gyrus is the posterior cingulate cortex.
- **Enlarged nucleus.** The suprachiasmatic nucleus is about a millimetre across, so it also gets a glowing dot of 2.8 mm radius. The page no longer says that small nuclei are enlarged.
- **Adding a structure.** Neurotorium serves every structure at `https://neurotorium.org/wp-content/themes/neurotorium/brain_atlas/models/<code>.glb`. The codes and names are in their `data.json`. Download the file, compress it with `npx @gltf-transform/cli draco in.glb out.glb`, save it under `models/neurotorium/`, and add an entry to `ATLAS.parts`.

Compression shrank the 26 files from about 4 MB to 544 KB. The geometry is unchanged apart from tiny rounding.

## Credits and licence

The models and matcaps come from Neurotorium (© Lundbeck Foundation) and are used under Onet's agreement. Neurotorium's public terms only allow personal and educational use, so publishing relies on that agreement. Make sure the credit wording on the page matches what the agreement requires. The credit sits in the bottom-left corner of the graphic. On phones, where the story text covers that corner, it sits under the timeline instead.

Two of the matcap images also appear in the free nidorx/matcaps collection on GitHub, which says their original authors are unknown.

## How it works

- **Sticky brain.** `.sticky-visual` uses `position: sticky; top: 0; height: 100vh`. `.steps` has `margin-top: -100vh`, so the cards scroll over it. No parent element may use `overflow: hidden`.
- **Timeline header.** The timeline sits in its own sticky layer above the story text, so the text slides under it and never reaches the top of the screen. The header uses the same background as the brain's layer and fades at its bottom edge, so it doesn't show as a box. On phones it also carries the model credit.
- **Scroll to rotation.** The script turns the scroll position into a continuous step index; 2.5 means halfway between the third and fourth card. Rotation, colours and the time are interpolated from it and smoothed every frame.
- **Rendering.** The brain is drawn in one pass straight to a transparent canvas, with the browser's own antialiasing. The page background shows through behind it.
- **Look.** As in the Neurotorium atlas, surfaces use matcap shading, where lighting comes from an image of a lit sphere. Like the atlas, the page reads the matcap colours as linear light and applies ACES filmic tone mapping. Together these give the soft, creamy, low-saturation look. Highlighted structures use a grey matcap tinted with a muted region colour.
- **See-through cortex.** The outer brain is 18 separate parts. Fading them all at once would show every part through every other part, which looks like layers piling up. While the cortex is see-through, a depth-only pass first marks the outermost surface, so only that surface is blended and the brain fades as one shell. Highlighted cortex on the inner walls, such as the DMN hubs, is drawn before that pass, so it stays visible through the shell.
- **Grain.** Film grain is added in the brain's shader, at the strength Neurotorium uses. A noise tile with the same strength is painted onto the page background, so the graphic and the article match. `GRAIN` in the script sets both.
- **Fallback.** Part 3 of the script generates a simplified brain in code. It is used when the models cannot load, with a matcap painted in code.

## Performance

Measured with a Chrome trace while scrolling through the whole story at 1440×900, 2× pixel density, on an Apple M5 laptop:

| Version | Main thread per frame | GPU process per frame |
|---|---|---|
| With a post-processing chain for FXAA and grain | about 3 ms | about 7.5 ms |
| Current | about 3 ms | about 2 ms |

On a 12 Mbit/s connection with an empty cache, the brain is ready about 2.6 s after the page starts loading, while the reader is still on the intro.

When a laptop runs on battery below 20%, or when Chrome's Energy Saver is on, Chrome limits every page to 30 frames per second. Animations then look less smooth, and a page cannot turn this off. In testing, the same scroll ran at 30 frames per second on battery at 11% and at 60 when plugged in.
