# Build prompt: Pegasus 43 mobile product page

Build a mobile-first product detail page prototype for the Nike Pegasus 43, designed for a 375px-wide phone. The goals are **zero layout shift while loading** and **key information and actions within thumb reach**.

## Inputs and constraints

- Stock data is in `pegasus-43-stock.json`: 26 rows with name, colorway, price, size labels, stock level, display order and launch date. Read every value from it and don't hardcode stock.
- One `index.html` with plain HTML, CSS and JavaScript. No build step or dependencies. It runs from a local server.
- System font stack only, so no web font swaps in after first paint.
- Don't invent product copy, specs, photos or reviews. Mark empty slots as "Photo needed" or "Copy needed".
- The file has no currency. Assume US dollars.
- `speed-budget.json` at the repo root is the speed budget. It has exactly these three keys, and the page must meet them on a slow phone (see "Slow phone check"). Don't add keys, remove keys or loosen a value. A value may only be made stricter.

  | Key | Value | Meaning |
  |---|---|---|
  | `performance_score_min` | 90 | Lighthouse mobile performance score must be at least 90 (stricter is higher) |
  | `size_grid_shift_px_max` | 0 | The size grid's top edge may move 0 CSS px from first paint to fully loaded (0 is the only allowed value) |
  | `size_button_min_px` | 44 | The smaller of width and height of every size button, sold out included, must be at least 44 CSS px (stricter is higher) |

- The speed check finds five elements by `data-testid`. If one is missing, the check fails.

  | `data-testid` | Put it on |
  |---|---|
  | `size-grid` | The page's size grid box (the 464px grid, not the size sheet's copy) |
  | `size-button` | Every available or low-stock size button |
  | `size-button-sold-out` | Every sold-out size button. These stay in the grid at full size |
  | `main-photo` | The first hero slide's photo: the "Photo needed" placeholder now, the `<img>` once a photo is supplied |
  | `price` | The `$140` price text |

## Frames and tokens

- Frames: 375 × 812 (safe areas 47 top, 34 bottom) and 375 × 667 (20 top, 0 bottom). Use `viewport-fit=cover` and safe-area insets.
- 16px side margins, 343px content width, spacing in steps of 4.
- Colours: text `#111111`, secondary `#6B6B6B`, button stroke `#767676`, sold-out fill `#F0F0F0` with text `#8A8A8A`, low stock `#9A4D00`, hero background `#F5F5F5`, background white.

## Page parts, in order

### 0. Top controls
Back and cart buttons, 44 × 44, laid over the hero 8px below the safe area. They add no height. The cart has a count badge.

### 1. Hero image: `40svh`
325px at 812, 267px at 667. Use `svh`, not `vh` or `dvh`. Three swipeable photo placeholders (side, top, outsole) with dots over the image.

The hero's height comes only from `40svh`, never from the photo. When real photos arrive, each `<img>` fills its slide (`width: 100%; height: 100%; object-fit: contain`) and has `width` and `height` attributes. The first photo is `data-testid="main-photo"`, loads eagerly with `fetchpriority="high"`, and is the largest thing on the first screen. A photo that sizes its own box pushes the size grid down when it arrives.

### 2. Name, colorway and price: 108px
- 16 top + 32 name + 20 colorway + 4 gap + 32 price + 4 bottom.
- "Nike Pegasus 43" at 28px bold, the largest text on the page.
- Colorway label under the name, 13px.
- Price on its own line, 24px semibold, with "Tax and shipping at checkout" beside it.
- Price and colorway appear only here, never on a size button.

### 3. Size selector: 524px
- 60px header: "Select size (US)", a "Low stock" key with an amber dot, and a "Size guide" button 44px high.
- 464px grid: 4 columns × 7 rows, buttons 76.75 × 56, 12px gaps, in `display_order`.
- Each button shows "M 9.5" (16px semibold) over "W 11" (13px), and only its size and status.

| State | Rule | Look |
|---|---|---|
| Available | In stock, more than 5 units | White, 1.5px stroke |
| Low stock | 1–5 units | Stroke plus an 8px amber dot |
| Sold out | 0 units | Grey, diagonal line, disabled, kept in place |
| Selected | One at a time | Solid black, white text |
| Pressed | | Scales to 97% |

- Native radio group. Each option reads in full, for example "Men's 9.5, women's 11, 2 left".
- Gaps are not clickable. Selection happens on release, so scrolling never selects. Tapping a size never adds to the cart.
- With this layout, two full rows show above the cart bar and the third peeks out.

### 4. Details: 324px
Title, a description held to 3 lines (72px), and three 56px expandable rows: "Cushioning and tech", "Fit and sizing", "Materials and care". Content moves only when the user taps.

### 5. Reviews
Before first paint, choose between two states:
- Default: "Reviews open at launch on 10 Dec."
- Loading skeleton: title, rating line, three 120px photo squares, two 144px cards and a 48px button.

### 6. Sticky cart bar
80px plus the bottom safe area, with a 343 × 56 pill button, a top divider and matching page padding.

| State | Button text | Second line | Tap |
|---|---|---|---|
| Loading | Loading sizes | | Disabled |
| Load failed | Try again | Sizes didn't load | Retries |
| Before launch | Notify me · Launches 10 Dec | Chosen size, if any | Without a size, opens the size sheet. With one, sets an alert |
| Alert set | Alert set · 10 Dec | Size · Tap a size to change | |
| Launched, no size | Select size · $140 | | Opens the size sheet |
| Launched, size chosen | Add to Cart · $140 | M 9.5 / W 11 · Only 2 left | Adds to cart |
| Working | Spinner | | Ignored |
| Added | Added · View cart | Size | Announces that the cart page isn't built |
| Add or alert failed | Try again | What failed | Retries |

"Only N left" shows only for low-stock sizes. Compare today with `launch_date` to pick the launch state.

## Sheets

- **Size sheet:** 56px header, the same 464px grid and room for the cart bar (616px plus the bottom safe area). It stays in sync with the page grid. The cart bar stays usable. It closes 600ms after a successful add.
- **Size guide sheet:** a table of US men's, US women's and EU sizes from the file, with a note that fit advice and foot lengths are still needed.
- Both close with the X, a backdrop tap, Escape or a downward swipe on the header. The page behind is inert, and focus returns on close.

## Loading and layout shift

- Name, colorway, price and the cart bar are in the HTML for first paint.
- Show 26 size skeletons in the final positions. Every skeleton matches the box that replaces it.
- If stock fails, show the retry message inside the 464px grid box.
- Fix every layout choice before first paint, and insert nothing above existing content.
- Fade content in over 150ms. Turn off shimmer and transitions for reduced motion.
- Text that the stock file rewrites after first paint (name, colorway, price) must not change its box. The name row is 32px tall: if a name from the file would wrap to a second line, it pushes the grid down 32px when stock arrives.
- The cart bar's second line must not move the main label when it appears, for example "Sizes didn't load". Keep the label in the same place with or without the second line.

## Slow phone check

Judge layout shift on a slow phone only. Don't judge the size grid from a desktop trace or an unthrottled trace.

**Settings.** Use these for every trace and Lighthouse run:

- 375 × 812 CSS px viewport, 3x pixel density, mobile and touch on
- Slow 4G: 150 ms round trip, 1638.4 Kbps (DevTools throttling: 562.5 ms request latency, 1474.56 Kbps down, 675 Kbps up)
- 4x CPU slowdown
- Cold cache, page served from the local server

**Trace (Chrome DevTools MCP).**

1. Emulate the viewport as `375x812x3,mobile,touch`, the network as `Slow 4G` and the CPU at `4`.
2. Before the page loads, add a script (`navigate_page` with `initScript`) that:
   - logs every `layout-shift` entry with its time, value and sources (node, previous rect, current rect);
   - logs the top edge of `[data-testid="size-grid"]` (`getBoundingClientRect().top + scrollY`) on every DOM change and every 10 ms, from first paint until a few seconds after `load` and the stock file have both finished.
3. Record a performance trace with reload and cache bypass, save it, and count its `LayoutShift` events.
4. Repeat steps 2 and 3 for `?stock=slow` and `?stock=fail` (then tap "Try again").

**Lighthouse.** The DevTools MCP's own Lighthouse tool leaves out performance, so run the Lighthouse CLI with this config, unchanged. Run it at least three times. A single low score with a large "Unattributable" task is the test machine, so rerun before judging.

```json
{
  "extends": "lighthouse:default",
  "settings": {
    "onlyCategories": ["performance"],
    "formFactor": "mobile",
    "throttlingMethod": "simulate",
    "throttling": { "rttMs": 150, "throughputKbps": 1638.4, "requestLatencyMs": 562.5, "downloadThroughputKbps": 1474.56, "uploadThroughputKbps": 675, "cpuSlowdownMultiplier": 4 },
    "screenEmulation": { "width": 375, "height": 812, "deviceScaleFactor": 3, "mobile": true, "disabled": false }
  }
}
```

**Report.** List every element that moves the size grid while the page loads: the photo, a font, the price, or anything else. For each one, give how many px it pushes the grid, when it happens (ms from navigation), and the cause. If nothing moves it, say so, and give the grid's top edge with when it was measured. List shifts elsewhere on the page separately. Then compare the results with each `speed-budget.json` key.

## Prototype controls

A dashed, collapsible panel at the bottom, clearly not part of the design, with a layout-shift meter. Each setting is also a URL parameter.

| Parameter | Values |
|---|---|
| `launch` | `auto`, `pre`, `live` |
| `price` | `b` (default), `a` (price right of the name, 72px block) |
| `stock` | `normal` (0.4s), `slow` (2.5s), `fail` (first load fails) |
| `add` | `ok`, `fail` (first try fails) |
| `reviews` | `launch`, `skeleton` |
| `frame` | `812`, `667`, or the device's own |
| `controls` | `0` hides the panel |

## Done when

Check these in a browser at both frames and report the results, including anything that fails.

1. Layout shift is 0 during normal and slow loading.
2. 26 size buttons appear in order: 4 sold out, 5 low stock, 17 available.
3. No horizontal page scroll, and every control in the design (not the prototype panel's menus) is at least 44px high.
4. Every cart bar state can be reached with the URL parameters.
5. Keyboard use works for the sizes and both sheets.
6. The console shows no errors in the normal flow.
7. Under the slow phone check, the size grid moves 0 px in the normal, slow and failed-stock loads, and the trace has no `LayoutShift` events.
8. Lighthouse performance is at least 90 with the config above.
9. Every size button, sold out included, is at least 44px on its smaller side.
10. All five `data-testid` elements are on the page, and `speed-budget.json` has exactly its three keys and values.
