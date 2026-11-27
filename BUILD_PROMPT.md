# Build prompt: Pegasus 43 mobile product page

Build a mobile-first product detail page prototype for the Nike Pegasus 43, designed for a 375px-wide phone. The goals are **zero layout shift while loading** and **key information and actions within thumb reach**.

## Inputs and constraints

- Stock data is in `pegasus-43-stock.json`: 26 rows with name, colorway, price, size labels, stock level, display order and launch date. Read every value from it and don't hardcode stock.
- One `index.html` with plain HTML, CSS and JavaScript. No build step or dependencies. It runs from a local server.
- System font stack only, so no web font swaps in after first paint.
- Don't invent product copy, specs, photos or reviews. Mark empty slots as "Photo needed" or "Copy needed".
- The file has no currency. Assume US dollars.

## Frames and tokens

- Frames: 375 × 812 (safe areas 47 top, 34 bottom) and 375 × 667 (20 top, 0 bottom). Use `viewport-fit=cover` and safe-area insets.
- 16px side margins, 343px content width, spacing in steps of 4.
- Colours: text `#111111`, secondary `#6B6B6B`, button stroke `#767676`, sold-out fill `#F0F0F0` with text `#8A8A8A`, low stock `#9A4D00`, hero background `#F5F5F5`, background white.

## Page parts, in order

### 0. Top controls
Back and cart buttons, 44 × 44, laid over the hero 8px below the safe area. They add no height. The cart has a count badge.

### 1. Hero image: `40svh`
325px at 812, 267px at 667. Use `svh`, not `vh` or `dvh`. Three swipeable photo placeholders (side, top, outsole) with dots over the image.

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
