# SIM1Racing Website: Design System & Build Guide

> **How to use this document:** paste all of it into a new Claude chat, then describe the page you want built or redesigned (and paste that page's current code blocks if it has any). Everything Claude needs to match the homepage's look, animations and technical setup is below.

---

## 0. Brief for Claude

You are designing pages for **SIM1Racing** (sim1racing.com), the largest sim-racing center in America, in Paramus, NJ. The site runs on **Squarespace**. Every page you build must look and feel like the homepage described here: dark, bold, motorsport-inspired, with a single brand red, big condensed italic headlines, and smooth, purposeful animations.

**Deliverable format:**
- Build each page as **one self-contained HTML snippet** that the owner pastes into **a single Squarespace Code Block** (full-width section, no padding, "Display Source" off).
- Put all HTML, `<style>` and `<script>` in that one snippet. Load Google Fonts with a `<link>` tag at the top.
- **No external build tools, frameworks or libraries.** Use plain HTML, CSS and vanilla JavaScript only.

**Working preferences of the site owner (follow these):**
- The owner designs and reviews on **desktop first**. Don't change the mobile layout unless they explicitly say they are reviewing mobile, but keep mobile working and responsive.
- Make only the changes they ask for. If something they ask for would break or look bad, say so and suggest the closest good alternative.
- Never invent facts, reviews, prices or claims. Use only facts the owner gives you or that are listed in section 9.
- Keep copy punchy and avoid repeating the same word across nearby headings. The word "racing" is easy to overuse.
- Explain what changed in plain language, without jargon.

---

## 1. Squarespace technical rules (critical)

1. **Scope everything** under one root class so nothing leaks into the Squarespace header, footer or other blocks:
   - Wrap the whole page in `<div class="s1" id="s1-root"> … </div>`.
   - Every class name starts with `s1-` (e.g. `.s1-hero`, `.s1-btn`).
   - **Never** write global selectors such as `h1 {}`, `p {}`, `a[href=…]`, `div[style*=…]`. Earlier versions of the site broke because separate code blocks each had global `h2`/`p` rules that overrode one another.
2. **Reset with zero specificity** so component classes always win:
   `.s1 :where(h1,h2,h3,p,ul,dl,dd){margin:0;padding:0}` and `.s1 :where(a){color:inherit;text-decoration:none}`.
   Using `.s1 p {margin:0}` instead breaks things, because it outranks `.s1-lead` margins.
3. **One code block per page.** Don't split a page across several blocks.
4. **JavaScript** runs in an IIFE: `(function(){ … })();`. Add the class `s1-js` to the root first. All "hidden before animating" CSS is written as `.s1-js .thing {…}`, so content stays visible if scripts don't run.
5. **Respect reduced motion.** Read `matchMedia('(prefers-reduced-motion: reduce)')` and add the `@media (prefers-reduced-motion:reduce)` block from section 6.
6. **Images:** use real `<img>` tags with `object-fit:cover`, not CSS `background-image` (easier to swap, lazy-loads, and supports an error fallback). Image sources:
   - Squarespace-hosted: `https://static1.squarespace.com/static/…/IMAGE.jpg`. Use the clean URL only, never pasted twice.
   - Repo-hosted (`michaelrpajaro/Home`, public): `https://cdn.jsdelivr.net/gh/michaelrpajaro/Home@<COMMIT_SHA>/images/<file>.jpg`, **pinned to a commit SHA** so it never changes.
7. **Video:** the hero uses this exact Vimeo embed. Do **not** add `background=1` or `quality=…` (those caused stuttering and pausing):
   `https://player.vimeo.com/video/1177535811?badge=0&autopause=0&player_id=0&app_id=58479&autoplay=1&muted=1&loop=1&controls=0&title=0&byline=0&portrait=0`
   with `allow="autoplay; fullscreen; picture-in-picture; clipboard-write; encrypted-media; web-share"`.
   **Never animate, filter or scale the video element itself.** Animate overlay layers above it instead. Dim it with `opacity:.8` on its wrapper plus a gradient overlay.

8. **Desktop scaling (looks identical on every laptop/monitor).** The design is built at a **2560px-wide reference** (the owner's Windows laptop, 2560×1271). Narrower desktop screens show the *same* layout scaled down, not a bigger-looking version: a 1728px MacBook shows it at 67.5%. Phones and tablets (≤1024px) are not scaled. Implement it exactly like this:
   - Put this script **first inside** `#s1-root`:
     ```html
     <script>
     (function(){
       var REF = 2560, MIN = 0.675, el = document.getElementById('s1-root');
       function fit(){
         var w = document.documentElement.clientWidth, h = window.innerHeight;
         var z = w > 1024 ? Math.min(1, Math.max(MIN, w / REF)) : 1;
         el.style.setProperty('--z', z);
         el.style.setProperty('--vwu', (w / z / 100) + 'px');
         el.style.setProperty('--vhu', (h / z / 100) + 'px');
       }
       fit(); window.addEventListener('resize', fit);
     })();
     </script>
     ```
   - Add `zoom:var(--z,1);` to the `.s1` rule.
   - **Never write raw `vw`/`vh`/`svh` units.** Inside a zoomed element they come out the wrong size. Write `calc(N * var(--vwu,1vw))` and `calc(N * var(--vhu,1vh))` instead. Container units (`cqi`) and `px` are fine.
   - Media queries still use the real screen width (1024px / 768px breakpoints are unaffected).
9. **Edge to edge on phones/tablets.** Squarespace adds side margins around blocks on mobile, which shows up as thin borders on the left and right. Inside `@media (max-width:1024px)` add:
   `.s1{width:100vw;max-width:100vw;margin-left:calc(50% - 50vw);margin-right:calc(50% - 50vw)}`

---

## 2. Brand tokens

```css
.s1{
  --red:#ED1C24;      /* the ONE brand red: buttons, accents, highlighted words */
  --red-dk:#C8161D;   /* red hover state */
  --ink:#000;         /* page background */
  --ink-2:#0A0A0B;    /* subtle section tint */
  --ink-3:#141417;    /* cards / panels / image placeholders */
  --line:rgba(255,255,255,.1);  /* hairline dividers */
  --muted:#A1A1AA;    /* secondary text */
  --cond:'Barlow Condensed',sans-serif;  /* headings, labels, buttons */
  --body:'Barlow',sans-serif;            /* paragraphs */
  --ease:cubic-bezier(.2,.7,.2,1);       /* the house easing curve */
}
```

- **Text colors:** headings `#fff`; body copy `#C9C9CF` to `#E4E4E7`; secondary `var(--muted)`.
- **Red glow** for shadows: `rgba(237,28,36,.6)`.
- Don't introduce other accent colors. Status colors are only allowed inside a data widget.

**Fonts** (first lines of every snippet):
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Barlow+Condensed:ital,wght@0,600;0,700;0,800;0,900;1,800;1,900&family=Barlow:wght@400;500;600&display=swap" rel="stylesheet">
```

---

## 3. Typography

| Role | Style |
|---|---|
| **Hero headline** | Barlow Condensed 900 *italic*, UPPERCASE, `line-height:.9`, `letter-spacing:.03em`, 4.5px black outline (`-webkit-text-stroke:9px #000; paint-order:stroke fill`). Key phrase in red. Each line is its own block that never wraps; size it to the block width (section 5). |
| **Section heading (`.s1-h2`)** | Barlow Condensed 900 upright, UPPERCASE, `clamp(42px,5.4vw,76px)`, `line-height:.96`, white, with the second phrase in red: `More than <span class="s1-red">just racing</span>` |
| **XL heading (`.s1-h2--xl`)** | Same, `clamp(50px,8vw,120px)`, `line-height:.9`. Often two lines, with the red second line in *italic*: `Your next party<br><span class="s1-italic s1-red">starts here</span>` |
| **Eyebrow / label (`.s1-eyebrow`)** | Barlow Condensed **800**, `clamp(17px,1.3vw,20px)`, `letter-spacing:.2em`, UPPERCASE, **white**, preceded by a 38×4px slanted **red** bar (`skewX(-30deg)`). Sits above every section heading. |
| **Lead paragraph (`.s1-lead`)** | Barlow 400, `clamp(17px,1.35vw,19px)`, `line-height:1.75`, `#C9C9CF`, max-width ~580px. |
| **Small caps labels** (stat labels, list keys like Hours / Address / Phone) | Barlow Condensed **800**, **16–20px** (list keys 18px), `letter-spacing:.14–.2em`, UPPERCASE, **white or `#D4D4D8`**. |
| **Big numbers** | Barlow Condensed 900 *italic*, up to 150px, white; units like `"` or `+` in red at 50% size. |
| **Outlined numbers** (01, 02…) | Barlow Condensed 900 italic, transparent fill, `-webkit-text-stroke:1.5px var(--red)`. |

**Black outline on all titles.** Every section heading (`.s1-h2`, including XL) and every group title (01/02/03 column headers) gets the same black outline as the hero headline: `-webkit-text-stroke:max(4px,.06em) #000; paint-order:stroke fill;`. Half the stroke sits behind the letters, so they keep their weight.

**Readability rule: never use red for small text.** Thin, small red text on black is hard to read, and the owner flagged it. Red is only for large headline words, buttons, bars, underlines, outlined numbers and big stat accents. Any text under about 24px (eyebrows, labels, list keys, small links) is **white or light gray, weight 800**, with red only as an accent shape (a bar before it or an underline beneath it).

**No periods at the end of headings.** The owner dislikes the "Title." style. Write "Come see it for yourself", not "Come see it for yourself." The same goes for eyebrows, column titles and button labels.

---

## 4. Layout & spacing

- **Containers:** `.s1-wrap{max-width:1240px;margin:0 auto;padding:0 24px}`. For edge-to-edge desktop sections use `.s1-wrap--wide{max-width:1760px;padding-left:clamp(24px,4vw,72px);padding-right:clamp(24px,4vw,72px)}`. The owner prefers using the **full width** on desktop rather than leaving empty side space.
- **Section rhythm:** `.s1-section{padding:120px 0}`, or 84px on mobile.
- **Radii:** buttons 6px; cards/photos 12px; pills 99px.
- **Dividers:** 1px `var(--line)` hairlines; red 3px bars for emphasis under column titles.
- **Breakpoints:** `max-width:1024px` (tablet: stack two-column layouts) and `max-width:768px` (phone). Desktop-only rules use `min-width:1025px`.
- **Desktop hero must fit one screen:** hours, headline, subtitle and buttons visible at 1366×657 and up. Hide the "Explore" scroll cue on screens shorter than 960px.

---

## 5. Components (copy these patterns)

### Base CSS (always include)
```css
.s1{ /* tokens from section 2 */
  background:var(--ink);color:#F4F4F5;font-family:var(--body);font-size:17px;line-height:1.6;
  overflow:hidden;position:relative;-webkit-font-smoothing:antialiased}
.s1 *,.s1 *::before,.s1 *::after{box-sizing:border-box}
.s1 :where(h1,h2,h3,p,ul,dl,dd){margin:0;padding:0}
.s1 :where(a){color:inherit;text-decoration:none}
.s1 svg{width:1em;height:1em;fill:none;stroke:currentColor;stroke-width:2;stroke-linecap:round;stroke-linejoin:round;flex-shrink:0}
.s1-red{color:var(--red)} .s1-italic{font-style:italic}
.s1-eyebrow{font-family:var(--cond);font-weight:800;font-size:clamp(17px,1.3vw,20px);letter-spacing:.2em;text-transform:uppercase;color:#fff;display:flex;align-items:center;gap:14px;margin-bottom:18px}
.s1-eyebrow::before{content:"";width:38px;height:4px;background:var(--red);transform:skewX(-30deg)}
.s1-eyebrow--light{color:#fff} .s1-eyebrow--center{justify-content:center}
.s1-h2{font-family:var(--cond);font-weight:900;font-size:clamp(42px,5.4vw,76px);line-height:.96;text-transform:uppercase;color:#fff}
.s1-h2--xl{font-size:clamp(50px,8vw,120px);line-height:.9}
.s1-lead{color:#C9C9CF;font-size:clamp(17px,1.35vw,19px);line-height:1.75;max-width:580px;margin-top:24px}
```

### Buttons
- **Primary (`.s1-btn--red`):** solid red, white text, red glow shadow, a white **shine sweep** every 4s, a trailing arrow icon that slides right on hover, and a 3px lift on hover. Use it for the main action (Book Now, Plan Your Event).
- **Ghost (`.s1-btn--ghost`):** 2px white border, transparent background, turns solid white with black text on hover. Over photos or video, give it a frosted background: `background:rgba(0,0,0,.35); backdrop-filter:blur(14px) saturate(1.2)`.
- All button labels are Barlow Condensed 800, UPPERCASE, `letter-spacing:.12em`, 19px, padded 19×38px (22×52px in the desktop hero).
```css
.s1-ctas{display:flex;gap:14px;flex-wrap:wrap;margin-top:38px}
.s1-btn{position:relative;display:inline-flex;align-items:center;justify-content:center;gap:12px;font-family:var(--cond);font-weight:800;font-size:19px;letter-spacing:.12em;text-transform:uppercase;line-height:1.1;padding:19px 38px;border-radius:6px;overflow:hidden;isolation:isolate;cursor:pointer;transition:transform .3s var(--ease),box-shadow .3s var(--ease),background .3s,color .3s,border-color .3s}
.s1-btn svg{font-size:20px;transition:transform .3s var(--ease)} .s1-btn:hover svg{transform:translateX(5px)}
.s1-btn--red{background:var(--red);color:#fff;box-shadow:0 10px 30px -8px rgba(237,28,36,.6)}
.s1-btn--red::after{content:"";position:absolute;inset:0;z-index:-1;background:linear-gradient(110deg,transparent 30%,rgba(255,255,255,.4) 50%,transparent 70%);transform:translateX(-120%);animation:s1-shine 4s ease-in-out infinite}
.s1-btn--red:hover{background:var(--red-dk);transform:translateY(-3px);box-shadow:0 16px 36px -8px rgba(237,28,36,.8)}
.s1-btn--ghost{border:2px solid rgba(255,255,255,.75);color:#fff;background:rgba(255,255,255,.05)}
.s1-btn--ghost:hover{background:#fff;color:#000;border-color:#fff;transform:translateY(-3px)}
@keyframes s1-shine{0%,55%{transform:translateX(-120%)}85%,100%{transform:translateX(120%)}}
```
Arrow icon: `<svg viewBox="0 0 24 24" aria-hidden="true"><path d="M5 12h14M13 6l6 6-6 6"/></svg>`

### Hero (homepage pattern; reuse a shorter version for inner-page banners)
- Full-screen video or photo background, an `opacity:.8` dimmer, and a gradient overlay that fades to black at the bottom.
- **Red speed streaks:** six thin 2px red-gradient lines shoot left→right across the hero on a loop (`transform` + `opacity` only).
- **Opening iris:** a black radial vignette over the media scales up from the center and fades out over 2.6s on load, then is **removed from the page** (`animationend` → `remove()`).
- **Hours line** above the headline: Barlow Condensed 800, `clamp(22px,2.5vw,40px)`, white with a text-shadow, `Open 7 Days a Week · 11AM – 11PM` (spacing only, no dash).
- **Headline:** each line slides up into place, staggered 0.15s, from inside an `overflow:hidden` line wrapper.
- **Headline sizing** (always fits on one line per row, based on the block's real width):
  ```css
  .s1-hero{container-type:inline-size}
  .s1-hero__title{font-size:min(12cqi,18vh,calc((min(100cqi,1760px) - 64px) / 8.45),210px)}
  .s1-line{display:block;overflow:hidden;white-space:nowrap;padding:.1em .25em}
  .s1-line+.s1-line{margin-top:-.2em}
  ```
  The divisor (8.45) is the longest line's width in ems plus padding. Re-measure it if the wording changes.
- Subtitle, then two buttons (primary + frosted ghost), then an "Explore" scroll cue (a small label over a 2px line with a red drip animating down).

### Stats band
Four columns. Huge italic numbers **count up** from 0 with an ease-out over 1.6s when scrolled into view. A 36×4px slanted red bar sits above each label, and there are faint vertical dividers between columns. Mobile: 2×2.

### Three themed columns (used for "More than just racing")
Three columns separated by hairlines. Each has an outlined red number (01/02/03) plus a big italic uppercase title on one line, a 3px red underline, then a list of items: a bold condensed uppercase title (22–30px) with a one-line muted description. Hover: the title turns red and nudges 6px right. **On phones show titles only** (no descriptions or item links), as quick-scan tiles with a red left bar, two per row; desktop keeps the descriptions. Optional small links under an item (`View events →`): white, 800 weight, 16px, with a 2px red underline that turns red on hover. The columns fade up one after another (0.12s stagger).

### Photo gallery
A grid with one large photo on the left (spans two rows, ~0.88:1) and two wide photos stacked on the right (**2.5:1**). 16px gaps, 12px radius. Each photo reveals with a **black curtain** sliding off (staggered), its image settling from `scale:1.15` to 1. Hover: 1.06 zoom plus a red 2px inner outline.
- Upload sizes: right tiles **2500×1000** (or 2000×800); large tile **1400×1600**.

### Feature / event-type cards
Small dark glass panels: `rgba(0,0,0,.45)`, blur, a 3px red left border, a condensed uppercase title and a muted one-line description.

### Full-bleed photo band (e.g. Private Events)
A background photo with a left-to-right black gradient for legibility, a slow parallax as the page scrolls, and an XL two-line heading (red italic second line), lead paragraph, cards and two buttons.

### Visit / contact block
Two columns, **centered as a group** (`justify-content:center`): the info on the left and the map on the right. No photo.
- **Left (`max-content`):** eyebrow, heading `clamp(42px,4.4vw,84px)` that never wraps, a definition list (bold white 18px keys Hours / Address / Phone; values are underlined links that turn red on hover), and two buttons kept on one row.
- **Right (`minmax(0, 60vh)` via `--vhu`):** a grayscale-inverted Google Map embed that runs from the **top of the heading** to the **bottom of the buttons**. Use `.s1-visit__info{display:contents}` with grid rows (eyebrow row 1, heading row 2, list row 3, buttons row 4) and put the map at `grid-row:2/5`.
- A checkered-flag strip runs along the top edge. Below 1024px it stacks, with the map square.

### Mobile sticky bar (phones only)
Appears after scrolling past the hero: a frosted dark bar pinned to the bottom with a phone-icon button and a full-width red Book Now button.

---

## 6. Animation system

**House rules:** every animation has a purpose (reveal content, signal speed or draw the eye to a call to action). Use the `var(--ease)` curve. Animate only `transform`, `opacity` and `scale`, never layout properties. Nothing animates the video itself.

| Animation | Where | How |
|---|---|---|
| **Intro slide-up** | Hero items on load | `.s1-js .s1-intro{opacity:0;transform:translateY(40px);animation:s1-in .9s var(--ease) forwards;animation-delay:var(--d)}`, with `--d` staggered .1s–.9s inline. **On `animationend`, add `.is-done`** (sets `animation:none;opacity:1;transform:none`). A lingering animation blocks `backdrop-filter` blur on anything inside it. |
| **Scroll reveal** | Most content | `.s1-js .s1-reveal{opacity:0;transform:translateY(36px);transition:opacity .9s var(--ease),transform .9s var(--ease);transition-delay:var(--rd,0s)}` → `.is-in` resets it. Triggered by IntersectionObserver (threshold .15, rootMargin `0px 0px -40px 0px`), unobserving after the first trigger. Stagger siblings with `--rd:.08s/.12s/.24s`. |
| **Curtain reveal** | Photos | `::before` black panel at `z-index:2`, `transform-origin:right`, transition 1.2s `cubic-bezier(.7,0,.2,1)` to `scaleX(0)` on `.is-in`, while the `img` goes from `scale:1.15` to 1. **Never use `clip-path` to hide the observed element:** IntersectionObserver treats a fully clipped element as never visible, so it never reveals. |
| **Count-up** | Stat numbers | rAF loop, ease-out quart (`1-(1-p)^4`), 1.6s, `toLocaleString` for thousands separators. |
| **Speed streaks** | Hero | 6 × `<i>` lines, `@keyframes` translateX(0→620%) with opacity in/out, 3.2s linear infinite, staggered delays. |
| **Iris open** | Hero load | Radial-gradient layer scales 1→5 and fades over 2.6s, then is removed. |
| **Button shine** | Primary buttons | White gradient sweep across every 4s. |
| **Ken Burns** | Feature photos | 24s ease-in-out infinite alternate, `scale(1)→scale(1.08) translate(-1.5%,-1%)`. |
| **Parallax** | Full-bleed backgrounds | On scroll (rAF-throttled), translateY by `(distance from viewport center) × -0.08`. |
| **Hover** | Interactive items | Lift 3px (buttons), zoom 1.05–1.06 (photos), a red text shift (list items), a red outline (photos). |

**Reduced motion block (required):**
```css
@media (prefers-reduced-motion:reduce){
  .s1 *,.s1 *::before,.s1 *::after{animation:none!important;transition:none!important}
  .s1-js .s1-intro,.s1-js .s1-reveal{opacity:1!important;transform:none!important}
  .s1-js .s1-curtain::before{display:none}
  .s1-js .s1-curtain img{scale:1}
}
```

**Core script skeleton:**
```js
(function(){
  var root = document.getElementById('s1-root'); if (!root) return;
  root.classList.add('s1-js');
  var reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  root.querySelectorAll('.s1-intro').forEach(function(el){
    el.addEventListener('animationend', function(){ el.classList.add('is-done'); });
  });

  function countUp(el){ /* see section 6 table: 1.6s ease-out quart */ }

  var targets = root.querySelectorAll('.s1-reveal, .s1-curtain, [data-count]');
  if ('IntersectionObserver' in window) {
    var io = new IntersectionObserver(function(entries){
      entries.forEach(function(en){
        if (!en.isIntersecting) return;
        var el = en.target;
        if (el.hasAttribute('data-count')) countUp(el); else el.classList.add('is-in');
        io.unobserve(el);
      });
    }, {threshold: 0.15, rootMargin: '0px 0px -40px 0px'});
    targets.forEach(function(el){ io.observe(el); });
  } else { targets.forEach(function(el){ el.classList.add('is-in'); }); }

  // Photos: if a URL fails, retry once with Squarespace's resized version, then hide
  root.querySelectorAll('img').forEach(function(img){
    img.addEventListener('error', function(){
      if (!img.dataset.retried) { img.dataset.retried = '1'; img.src = img.src.split('?')[0] + '?format=1500w'; return; }
      img.style.display = 'none';
    });
  });
})();
```

---

## 7. Voice & copy

- **Tone:** confident, energetic, short. Headlines are 2–6 words, often split across two lines with the punchline in red.
- **No trailing periods on any heading** (see section 3).
- **Pattern:** an eyebrow names the topic ("Private Events", "Visit Us", "The Experience"), then the heading makes the claim.
- Vary vocabulary, and don't stack "racing / race / racer" in consecutive headings.
- **Calls to action:** "Book Now", "Plan Your Event", "Email Us", "Call Us", "View Events →", "Group Coaching →".
- **Headings already used on the homepage** (don't reuse verbatim): "The Largest Sim-Racing Center in America", "More than just racing", "Your next party starts here", "Come see it for yourself"

---

## 8. Quality checklist before handing back code

- [ ] Everything is scoped under `.s1` / `s1-` classes, with no global selectors.
- [ ] It's a single snippet that works pasted into one Squarespace Code Block.
- [ ] Each headline line stays on one row at 1280, 1366, 1440, 1920 and 2560px wide, and inside narrower 1000–1200px containers.
- [ ] Desktop hero content fits on the first screen at 1366×657 and above.
- [ ] Scroll reveals, count-ups and curtains all trigger. Elements hidden for animation are scoped under `.s1-js`.
- [ ] Reduced-motion users see everything, static.
- [ ] No sideways scrolling on any width. Mobile is still usable even if not the focus.
- [ ] All links are real (section 9). `mailto:` and `tel:` links work.
- [ ] No invented facts, reviews or prices.

---

## 9. Business facts & links

- **Name:** SIM1Racing. **Tagline:** The Largest Sim-Racing Center in America.
- **Address:** 145 Route 4, Paramus, NJ 07652. Directions link: `https://www.google.com/maps/dir/?api=1&destination=145+Route+4+Paramus+NJ+07652`
- **Hours:** open 7 days a week, 11AM–11PM (write "7 days a week", not "walk-ins welcome daily"). Walk-ins welcome.
- **Phone:** (201) 988-1329 → `tel:201-988-1329`
- **Email:** info@sim1racing.com → `mailto:info@sim1racing.com` (add `?subject=…` where helpful)
- **Facility:** 20 full-motion simulators · 8,000 sq ft · 230" main screen · 250+ parking spaces · BYOB · high-speed WiFi / remote work space
- **Offerings:**
  - Race with friends: walk in or book, and race **up to 20 people at the same time** while seeing each other on track.
  - Memberships: discounted racing, exclusive benefits, a free issue of **Esses Magazine**, and more.
  - Competitions: weekly & monthly qualifiers, monthly Time Attack Championship, championship events with prizes. **No leagues currently**, so don't mention leagues or teams.
  - Group classes & **private coaching** (adults and kids, all levels).
  - **The Lounge:** hang out with ping pong, foosball, poker, and a **wet bar** (fridge, microwave, sink, small counter). It is **not a bar** and no drinks are served; pairs with BYOB.
  - Watch parties: **Formula 1, IMSA, WEC** and other major races on the 230" main screen or the TVs throughout the facility (not NASCAR).
  - **Car meets** (mainly) and Cars & Coffee: **not daily**; dates are posted on Instagram.
  - Birthday parties, bachelor parties, corporate events & team building, meetings, remote work space, BYOB.
- **Positioning:** speak to everyone, not only motorsport fans. Frame a visit as a **one-of-a-kind experience for all ages and skill levels**, for friends, families, dates and teams. Don't tie intro copy to "Paramus, NJ" in a way that sounds local-only; people travel to visit (the address belongs in the visit/contact block). Avoid jargon like "professional motorsport simulation" in headline or intro copy.
- **Site links:**
  - **Book Now (all booking buttons):** `https://sim1racing.as.me/`
  - Daily rates page: `https://www.sim1racing.com/daily-rates`
  - Pricing: `/pricing`
  - Group packages / events: `/group-packages`
  - Group classes: `/group-classes`
  - Private coaching: `/private-coaching`
  - Events calendar: `https://www.sim1racing.com/events`
- **Hero video:** see section 1, item 7.
- **Venue photos:**
  - `https://static1.squarespace.com/static/675f613ae652273e9723cfd3/t/69c5cffcb41fbc6a94e62da2/1774571517828/CTA06637.jpg`
  - `https://static1.squarespace.com/static/675f613ae652273e9723cfd3/t/69c5da63a814f34287109d52/1774574183404/CTA06863.jpg`
  - `https://cdn.jsdelivr.net/gh/michaelrpajaro/Home@2cdde9d455db1f8d0ba405f6ca76d95876310a08/images/front-desk.jpg` (front desk + race car + main screen)
  - `https://cdn.jsdelivr.net/gh/michaelrpajaro/Home@527b21516be3555be45e47e11ebaf56bade7dbb7/images/sim-rigs.jpg` (row of simulators in use)
  - `https://cdn.jsdelivr.net/gh/michaelrpajaro/Home@1cc9949bcb7db1cc84f7679e3f28f82867fc02fb/images/car-meet.jpg` (indoor car meet, 2.5:1; homepage gallery bottom-right)
  - `https://cdn.jsdelivr.net/gh/michaelrpajaro/Home@f6c504c9b6163a7ae40ca85c5eea69a9a072c7c4/images/exterior.jpg` (outside of the building with cars, 16:9; not currently used on the homepage)

**Reference implementation:** the full homepage code is `sim1racing-home.html` in the GitHub repo `michaelrpajaro/Home`. If you can read it, treat it as the source of truth for exact values.
