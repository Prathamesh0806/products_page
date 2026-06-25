# PDP Quantity-Discount Demo

**Part B deliverable** for the Decode Age PDP Volume Discount task.
A working, self-contained demo of the redesign proposed in Part A.

## 1. How to run this

This is a **HTML file** with no dependencies, no build step, and no
backend. Three ways to view it:

**Option A just open it**
Double-click `product_page.html`, or drag it into any browser tab. Works fully
offline.

**Option B CodePen / StackBlitz**
Paste the contents into a new HTML pen/project. Since everything (CSS + JS)
is inlined in one file, no separate CSS/JS panes are needed.

**Option C GitHub Pages**
Push `product_page.html` to a repo, enable GitHub Pages on the `main` branch, and
it's live at `https://<username>.github.io/<repo>/`.

**To test mobile responsiveness:** open Chrome DevTools → Toggle device
toolbar → select "iPhone SE" (375px width) or set a custom width to 375px.
No horizontal scroll should appear at any point, and the Add to Cart button
should remain fully visible without clipping.

---

## 2. What this demonstrates

This demo implements the redesign from the Part A ideation document:
replacing the original "pack-as-variant" pricing model (which caused
inconsistent totals when bundle tiles were mixed — see Part A, Friction
Point 1) with a **single quantity stepper + live quantity-based discount
engine**.

| Feature | Where to find it |
|---|---|
| Size selector (250mg/500mg/1000mg) | Pills below product title |
| Quantity stepper | `− [qty] +` control |
| Live discount tier ladder | 4-box row showing 1 / 2-3 / 4-5 / 6+ with current tier highlighted |
| Live savings + total | Green summary box, updates on every quantity/size change |
| Add to Cart | Adds the current size+quantity+price to the cart drawer |
| Cart drawer | Slide-in panel, per-line breakdown, grand total, total saved |

---

## 3. Design decisions

**Why a single quantity stepper instead of separate bundle tiles?**
This directly fixes the bug found during live-site research: with separate
"Pack of 1/2/3" variants, a customer could reach the same total quantity
(e.g. 4 bottles) through different combinations of tiles, each producing a
*different price* for the same number of bottles. A single stepper makes
quantity the only input, so there's exactly one price for any given
quantity no ambiguity, no hidden combinations.

**Why a persistent tier ladder instead of just showing the current discount?**
Showing only the active discount tells the customer what they're getting,
but not what they're missing. The ladder shows all four tiers at once with
a "YOU ARE HERE" flag, so the customer can see before adding to cart —
exactly how many more bottles would unlock the next discount level. This
turns the pricing table into a visible incentive rather than a static label.

**Why does adding the same size twice update the existing cart line instead
of creating a duplicate?**
This is a direct fix for the "mixing tiles" inconsistency. If a customer
adds 2 bottles of 500mg, then comes back and adds 2 more, the cart
correctly merges this into one line of 4 bottles at the 4-5 tier rate (18%
off) rather than two separate lines at the 2-3 tier rate (10% off) each,
which would silently overcharge them relative to what the tier ladder
promised.

**Why is the pricing logic written as pure functions (`getTierForQty`,
`calculatePricing`)?**
These functions take plain data in and return plain data out no DOM
access inside them. This was verified independently with a standalone
Node.js script before wiring them into the UI (all 24 combinations across
3 sizes × 8 quantities were checked for correct math). Keeping pricing
logic separate from rendering means it could be swapped for a real backend
call (e.g. a Shopify Function or Storefront API price calculation) without
touching any UI code — only the `PRODUCTS` data object and the calculation
functions would need to change.

## 4. Known simplifications (intentional, for a demo scope)

- **Dummy pricing data**, not pulled from a live Storefront API — per the
  brief's explicit allowance ("dummy/mock data is fine").
- **No real checkout** the Checkout button shows a toast confirming this
  is a demo, since the brief doesn't require payment integration.
- **No persistence** cart state resets on page reload (no
  localStorage/sessionStorage used, since those aren't reliable in all
  embedded/sandboxed environments; state lives in memory for the session).
- **Single product family** only NMN is modeled, since that's the
  product the task is scoped around.

---

## 5. If extending this toward a real Shopify implementation

This was deliberately built so the swap to production is small:

1. Replace the `PRODUCTS` object with data fetched from the Storefront API
   (variant prices, available sizes).
2. Replace the in-memory `TIERS` array with values read from the merchant's
   actual discount configuration (e.g. metafields, or a Shopify Function on
   Plus plans see the separate Shopify Function writeup from earlier
   project discussion for that approach).
3. Replace the local `state.cart` array with real Shopify cart API calls
   (`/cart/add.js`, `/cart/change.js`) so the cart drawer reflects the
   actual Shopify cart object instead of local state.
4. Everything else the tier ladder UI, live summary, stepper can stay
   as-is, since it only depends on a `(sizeKey, qty) → pricing` function
   shape, regardless of where that data ultimately comes from.

---

*Files: `product_page.html` (this demo) · `PDP-Ideation-Proposal.md` (Part A)*
