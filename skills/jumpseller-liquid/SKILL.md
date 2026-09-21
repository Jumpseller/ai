---
name: jumpseller-liquid
description: "Use when building or editing Jumpseller themes, Liquid templates, partials, or theme components."
---
# Jumpseller Liquid Theming

Use this skill when building or editing Jumpseller themes using the Liquid templating language.

## ⚠️ Security: never put API credentials in theme code

Theme files (Liquid, HTML, CSS, JS, assets) are **served publicly to every visitor**. **Never** embed credentials in them:

- Do NOT hardcode `JUMPSELLER_LOGIN_KEY`, `JUMPSELLER_AUTH_TOKEN`, a Basic-auth string, or any API/MCP token into Liquid, `<script>` tags, asset files, or client-side `fetch`/`XMLHttpRequest` calls. These credentials are **full-access** — a single leaked key lets anyone read every customer and order and modify the entire store.
- Liquid renders **server-side**, so storefront features get store data from Liquid objects (`order`, `product`, `cart`, `customer`, etc.) **without any credentials**. Use those.
- Need data not available in Liquid (e.g. recent orders for a "social proof / FOMO" widget)? Fetch it **server-side or at build time** with the API/CLI, then inject only the **non-sensitive, anonymized** result into the theme as static content (e.g. "Someone in Santiago just bought X") — never the credentials, and never raw customer PII (full name, email, phone, address).

## What is Liquid

Liquid is the templating language used in Jumpseller themes. It has three types of delimiters:
- `{{ variable }}` — outputs a value
- `{% tag %}` — logic (if, for, assign, render, etc.)
- `{% comment %}…{% endcomment %}` — comment, not rendered

## Theme File Structure

```
theme/
├── templates/           # Page-level templates, one subfolder per template type
│   ├── category/           #   e.g. Default.liquid, "All categories.liquid", "Brands.liquid", "Show subcategories.liquid"
│   ├── product/            #   Default.liquid
│   ├── page/                #   Default.liquid, Blog.liquid, Post.liquid
│   ├── page_category/       #   Default.liquid, Blog.liquid
│   ├── customer/            #   account.liquid, login.liquid, details.liquid, address.liquid, reset_password.liquid
│   ├── layout.liquid        #   wraps every page (<head>, header/footer includes, global <style>)
│   ├── home.liquid
│   ├── searchresults.liquid
│   ├── contactpage.liquid
│   └── error.liquid
├── partials/             # Reusable fragments, rendered with {% render 'name' %} (NOT {% include %})
├── components/           # Configurable sections — each has a .liquid + .json pair
├── assets/               # CSS, JS, images, fonts — reference with the `asset` filter
├── config/
│   ├── theme.json                  # Theme metadata — name goes in `Info.name` (root values must be objects)
│   ├── settings.json                # Merchant-configurable theme settings, keyed flat (see below)
│   ├── options.json                  # Setting *definitions* (labels, types, groups) shown in the settings UI
│   ├── pages.json                    # Per-template-type component ordering, keyed by template name (see below)
│   └── installed-components.json     # Every component *instance* on the store, with its saved options
└── locales/              # Translation strings, one folder per language: en/storefront.po, pt/storefront.po, …
```

Note the locales format: **`.po` files under a per-language folder**, not `en.json`. And a theme exported with the CLI contains no `.jumpseller-store` file — the active store lives in `~/.config/jumpseller/store` (see the `jumpseller-cli` skill).

**A "template type" (e.g. `category`, `page`, `page_category`) can have multiple named variants** — Jumpseller assigns one of them per category/page from the admin (e.g. a category can use `Default`, `All categories`, `Brands`, or `Show subcategories`). `config/pages.json` lists the component order for each variant:

```json
{
  "category": {
    "Default": { "components": ["love-header-1", "love-category-hero-1", "..."] },
    "All categories": { "components": ["header", "category-heading-1", "category-accordion-template-1", "footer"] },
    "Brands": { "components": ["header", "category-heading-1", "category-brands-template-1", "footer"] }
  }
}
```

Inside a template's `.liquid` file, `{{ index_for_top_components }}` / `{{ index_for_components }}` / `{{ index_for_bottom_components }}` render the components assigned in `pages.json` for that page's variant, and `template` is a string you can branch on — `template == 'category'`, or `template contains 'category'` to also match variants suffixed onto the name.

### Local theme development (Jumpseller CLI)

Theme files are edited locally and synced to a store with the **Jumpseller CLI** — the official command-line tool for local theme development. It is a separate project: [`Jumpseller/jumpseller-cli`](https://github.com/Jumpseller/jumpseller-cli).

```bash
npm i -g @jumpseller/cli   # install globally

jumpseller access          # set up store credentials
jumpseller theme --help    # local theme development commands
```

For full command reference — `access` (credentials), `theme export`/`import`/`watch`/`apply`, store resolution, and config file locations — see the **`jumpseller-cli` skill**. The CLI uses the same Login key + Auth Token as the REST API and MCP server.

## Global Liquid Objects

These objects are available depending on the current page/template:

| Object | Description |
|---|---|
| `store` | Store configuration: name, currency, country, logo, social links, `store.category[permalink]` lookup |
| `product` | Current product (product template) |
| `category` | Current category (category template) — **not** `collection`, that object does not exist in Jumpseller |
| `cart` | Current cart: items, totals, item count |
| `customer` | Logged-in customer (blank if not logged in) |
| `order` | Current order (order confirmation) |
| `page` | Current custom page |
| `checkout` | Checkout state (checkout pages) |
| `options` | Merchant's theme-wide settings from `config/settings.json` (e.g. `options.theme_corners_style`) |
| `component` | The current component instance — `component.options.*` for its saved settings, `component.id` |
| `block` | Inside a child component, that block's own instance — `block.options.*`, `block.index` (1-based); `component` there refers to the **parent** |
| `template` | String name of the current template (`'home'`, `'category'`, `'product'`, ...) |
| `request` | Request context, e.g. `request.preview_mode`, `request.section_preview_mode` |

### `product` object

```liquid
{{ product.id }}
{{ product.name }}
{{ product.description }}
{{ product.price | price }}          <!-- formatted with currency, e.g. "$14.990 CLP" -->
{{ product.discount }}               <!-- amount to SUBTRACT from price for the sale price — there is no compare_at_price -->
{{ product.sku }}
{{ product.brand }}
{{ product.stock }}
{{ product.stock_unlimited }}
{{ product.status }}                 <!-- 'available' | 'not-available' | 'active' | 'disabled' | 'open' -->
{{ product.permalink }}
{{ product.url }}

{% for image in product.images %}
  <img src="{{ image.url | resize: width: 640 }}" alt="{{ product.name | escape }}">
{% endfor %}

{% for variant in product.variants %}
  {{ variant.sku }} — {{ variant.price | price }}
{% endfor %}

{% for cat in product.categories %}
  {{ cat.name }}
{% endfor %}
```

To render a discounted price correctly:
```liquid
{% assign discount = product.discount | plus: 0 %}
{% if discount > 0 %}
  <span class="sale-price">{{ product.price | minus: product.discount | price }}</span>
  <span class="was-price">{{ product.price | price }}</span>
{% else %}
  <span>{{ product.price | price }}</span>
{% endif %}
```

### `cart` object

```liquid
{{ cart.item_count }}
{{ cart.total_price | price }}

{% for item in cart.items %}
  {{ item.name }}
  {{ item.quantity }}
  {{ item.price | price }}
  {{ item.line_price | price }}
  {% if item.variant_title %}{{ item.variant_title }}{% endif %}
{% endfor %}
```

### `customer` object

```liquid
{% if customer %}
  {{ customer.name }}
  {{ customer.email }}
  {{ customer.phone }}
  {{ customer.shipping_address }}
  {{ customer.billing_addresses }}
  {{ customer.orders }}
  {{ customer.wishlisted_products }}
  {{ customer.logout_url }}
  {{ customer.edit_url }}
{% else %}
  <!-- visitor is not logged in -->
{% endif %}
```

### `category` object

```liquid
{{ category.name }}
{{ category.description }}          <!-- also used as the SEO meta description fallback -->
{{ category.permalink }}

{% for prod in category.products %}
  {{ prod.name }} — {{ prod.price | price }}
{% endfor %}
```

To look up an *arbitrary* category by permalink (e.g. to feature one category's products from a component on another page — the home page, a banner, etc.), use `store.category`, indexed by **permalink**, not by id or name:
```liquid
{% assign featured = store.category['ver-todo-los-stickers'] %}
{% for prod in featured.products limit: 12 %}
  {{ prod.name }}
{% endfor %}
```

## Liquid Filters

### Money formatting

There is no `money` filter. The real filter is `price`:
```liquid
{{ product.price | price }}                       <!-- "$14.990 CLP" -->
{{ product.price | price | remove: ' CLP' }}       <!-- strip the currency suffix if you only want the symbol/number -->
```

### Image resizing

Two filters exist, and `thumb` is the one official theme code reaches for most often (65 uses against 36 for `resize` in a stock Simple theme):

```liquid
{{ image.url | thumb: '400x400' }}       <!-- fixed WIDTHxHEIGHT box — the common case for cards and thumbnails -->
{{ image.url | resize: '1200x630' }}     <!-- fixed WIDTHxHEIGHT crop -->
{{ image.url | resize: width: 640 }}     <!-- scale by width only, keyword form — also used in official code -->
```
Copy whichever the sibling component you are imitating uses. `product_image_url` is not a real Jumpseller filter.

### String filters

```liquid
{{ product.description | strip_html }}
{{ product.description | truncate: 150 }}
{{ coupon | downcase }}
```

### Translation

Jumpseller does **not** use a `| t` filter with dotted translation keys. It translates literal strings inline with the `t` tag, looked up against `locales/<lang>/storefront.po`:
```liquid
{% t "Read more" %}
{% t "Add %{product_name} to cart", product_name: prod.name %}
```
For a value you need to reuse (e.g. as a fallback default), capture it first:
```liquid
{% capture t_out_of_stock_default %}{% t "Out of Stock" %}{% endcapture %}
{% assign t_out_of_stock = options.t_out_of_stock | default: t_out_of_stock_default %}
```

Use this for labels and button text. Do **not** wrap merchant-owned copy — a headline, a promotional line — in `{% t %}`: that string will never be in the platform's translation tables or in the theme's `.po`, so it gains nothing. Give those their default in the component schema and its preset instead.

### Array filters

```liquid
{{ product.images | first }}
{{ product.images.size }}
{{ product.variants | map: 'price' | min }}
```

### Loading a theme asset

```liquid
<link rel="stylesheet" href="{{ 'my-styles.css' | asset }}">
<script src="{{ 'my-script.js' | asset }}" defer></script>
```

## Rendering Partials and Components

Partials are rendered with `{% render 'name' %}` — **not** `{% include %}`, no `partials/` prefix, no `.liquid` extension:

```liquid
{% render 'product_block', prod: prod, display_option: display_option, block_index: forloop.index %}
{% render 'theme_breadcrumbs' %}
{% render 'sidebar_filters' %}
```

Pass variables as `key: value` pairs after the partial name.

## Component System

Components are configurable sections merchants can add/reorder/customize from the theme's Visual Editor without touching code. Each component has two files, and one entry per placed instance in `config/installed-components.json`.

**`components/my-component.json`** — defines the component and its options:
```json
{
  "name": "My Component",
  "icon": "star",
  "help": "One line shown under the component's name in the editor.",
  "max_usage": null,
  "required": false,
  "tag": "div",
  "classes": "",
  "templates_in": ["home", "category"],
  "options": {
    "title":   { "name": "Title", "type": "input", "default": "Welcome" },
    "body":    { "name": "Body text", "type": "text", "default": "" },
    "image":   { "name": "Background image", "type": "image" },
    "layout":  {
      "name": "Layout", "type": "select", "default": "center",
      "options": [{ "value": "left", "label": "Left" }, { "value": "center", "label": "Center" }]
    },
    "show_button": { "name": "Show button", "type": "checkbox", "default": true },
    "link":        { "name": "Button link", "type": "link", "default": null },
    "link_text":   { "name": "Button text", "type": "input", "default": "Learn more",
                     "visible_if": "{{ block.options.link != blank }}" },
    "columns":     { "name": "Columns", "type": "slider", "default": 3, "min": 1, "max": 6, "step": 1, "unit": "" },

    "margin_top":    { "name": "Top margin", "type": "slider", "default": 48, "min": 0, "max": 80, "step": 4, "unit": "px" },
    "margin_bottom": { "name": "Bottom margin", "type": "slider", "default": 48, "min": 0, "max": 80, "step": 4, "unit": "px" },
    "margin_mobile": { "name": "Margin on mobile", "type": "slider", "default": 100, "min": 1, "max": 100, "step": 1, "unit": "%" },
    "bundle_color":  { "name": "Content colors", "type": "bundle", "default": "default", "pack": "color" },
    "animate":       { "name": "Customize options", "type": "checkbox", "default": false,
                       "visible_if": "{{ options.theme_animate }}" }
  },
  "presentation": [
    { "name": "Appearance", "type": "heading", "before": "margin_top" }
  ],
  "mode": "subcomponents",
  "blocks": [],
  "presets": [{ "name": "My Component", "category": "Information" }]
}
```

### Required top-level keys

Ten keys appear in **every** official schema — `name`, `icon`, `max_usage`, `required`, `tag`, `classes`, `templates_in`, `mode`, `blocks`, `options` (all but `icon` in 100% of them, `icon` in 98%). Omit one and `jumpseller theme import` fails with a **bare 400 that names no field**. Copy the shape of an official component rather than writing a minimal schema from scratch.

- `templates_in` controls which template types this component can be placed on (`"home"`, `"category"`, `"product"`, `"page"`, `"page_category"`, `"searchresults"`, `"contactpage"`, `"error"`, `"customer__login"`, ...).
- `max_usage` is `null` for most sections. On a **child** component it is the cap on how many blocks a parent can hold — official examples: `grid-block` 7, `marquee-item` 10, `image-comparison-link` 4, every `product-template` child 1.
- `presets: [{ name, category }]` is what lists the component in the editor's gallery (`Hero`, `Products`, `Content`, `Information`, `Media`, `Navigation`…). Without it the component exists but a merchant has no obvious way to add it. A preset may also carry `options` and `children` to seed a starting configuration.
- `presentation: [{ type: "heading", name, before }]` groups a long option list under headings in the panel.

### Option types

The platform accepts: `input` (single line), `text` (multi-line), `image`, `link`, `checkbox`, `select` (with `options: [{value,label}]`), `slider` (`min`/`max`/`step`/`unit`), `bundle` (`"pack": "color"` — a palette picker), `ph_icon`, `color`, `video`, `html`, `category`, `product`, `product_list`, `page_list`, `page_category`, `category_list`, `menu_links`.

There is **no `number` and no `url` type** — use `slider` and `link`. There is no top-level `"settings"` array either: options are a flat `{key: {...}}` object, read via `component.options.<key>`, **not** `component.settings.<key>`.

`visible_if` shows or hides an option in the panel. Inside it the current component's options are `block.options.*` (theme-wide settings stay `options.*`) — **never `component.options.*`**, which silently never matches:

```json
"link_text": { "name": "Button text", "type": "input", "default": "Read more",
               "visible_if": "{{ block.options.link != blank }}" }
```

### The standard option tail

Every section component ends its options with the same block, copied verbatim from an official sibling: `margin_top`, `margin_bottom`, `margin_mobile`, `bundle_color`, and the animation set (`animate`, `animate_type`, `animate_repeat`, `animate_delay`). The margins are applied through a partial, never hardcoded:

```liquid
<style>
  #{{ section_id }} {
    --theme-max-width: {{ options.theme_width }};
    {% render 'component_margins', margin_top: component.options.margin_top, margin_bottom: component.options.margin_bottom %}
  }
</style>
```

**`components/my-component.liquid`** — accesses options via `component.options.{key}`:
```liquid
{% assign section_id = 'theme-section-' | append: component.id %}

<section
  id="{{ section_id }}"
  class="container-fluid theme-section"
  data-bundle-color="{{ component.options.bundle_color }}"
  {{ component.attributes }}
>
  <div class="container container--adjust theme-section__container">
    <h2 class="my-component__title check-empty" {{ component.attributes.textfield.title }}>{{ component.options.title }}</h2>

    {% if component.options.body != blank %}
      <div class="my-component__body check-empty" {{ component.attributes.textfield.body }}>{{ component.options.body | newline_to_br }}</div>
    {% endif %}

    {% if component.options.image.url != blank %}
      <img src="{{ component.options.image.url | thumb: '800x' }}" alt="{{ component.options.title | escape }}">
    {% endif %}
  </div>
</section>
```

Image-type options resolve to an object with `.url` — always guard with `!= blank`, since an unset image option is simply absent.

### Inline editing

`{{ component.attributes }}` on the root element and `{{ component.attributes.textfield.<option> }}` on each editable text is what lets a merchant type directly on the page in the Visual Editor. Pair every one of them with the class `check-empty`, which collapses the element when the value is empty. Multi-line `text` options render through `| newline_to_br`.

The exception is text the browser assembles at runtime (a template with placeholders filled by JS): inline editing would fight the re-render, so those fields are edited from the panel only.

### Colors: palettes, not hex

Never write hex values or font names in a component. `config/settings.json` → `active.color` defines the store's palettes (`default`, `system-1` "Dark", `system-2` "Light", …), and `templates/layout.liquid` turns each one into a block of CSS custom properties on `[data-bundle-color="<label>"]`:

`--color-main`, `--color-secondary`, `--color-links`, `--color-links-hover`, `--color-background`, `--color-border`, `--color-button-main-bg` / `-text`, `--color-button-secondary-bg` / `-text`, plus opacity variants `--color-main-op05 … -op7` and `--color-secondary-op05 …`. Type and shape come from `--font-main`, `--font-secondary` and `--radius-style`.

So a component carries `data-bundle-color="{{ component.options.bundle_color }}"` and styles itself with `var(--color-*)`. A block that must look inverted gets its **own** `bundle` option defaulting to `system-1` and its own `data-bundle-color`, rather than hardcoded dark colours.

### Repeatable blocks (parent and child)

A component that holds a list of items is a pair: the parent declares the child type and renders `{% content_for 'children' %}`; the child is a normal component with `"templates_in": []`.

```json
// parent: "mode": "subcomponents", "blocks": [{ "type": "my-component-item" }]
// child:  "templates_in": [], "max_usage": 6
```

```liquid
<div class="my-component__grid" data-layout-size="{{ component.blocks.size }}">
  {% content_for 'children' %}
</div>
```

For anything beyond a uniform row, follow `grid-layout` + `grid-block`, which is the reference implementation of an asymmetric grid built from blocks:

- The **parent** publishes the state on the grid container as data attributes — `data-layout-size="{{ component.blocks.size }}"` and one per layout choice (`data-layout-featured="{{ component.options.featured }}"`) — and the CSS routes the layout off those attributes.
- The **child** works out its own role from `block.index` plus the parent's options: `{% if featured == 'first' and block.index == 1 %}`.
- **Never lay out the grid with `:nth-child`.** `{% content_for 'children' %}` emits markup the parent does not control, so positional selectors silently miss and the grid collapses into one column.
- Whatever the grid decides is never also a per-child option: two children could set it at once. If a role must be selectable, put a single option on the parent naming one child, the way `featured: first | middle | last` does.

### Registering an instance

After writing the component files you still need to (1) add an entry to `config/installed-components.json` keyed by a unique instance id (e.g. `my-component-1`) with `type`, `placement`, `identifier`, `visibility` and `options`, and (2) reference that id in the relevant `config/pages.json` template/variant `components` array.

Write **every** option explicitly — do not rely on the platform to backfill the schema defaults — and give each one the value shape its type uses, because a wrong shape is another bare 400:

| Option type | Stored as |
|---|---|
| `product_list`, `page_list`, `category_list`, `menu_links` | `[]` — **never `null`** |
| `category`, `product`, `link`, `image` | `null` when unset |
| `page_category` | a string, e.g. `"Blog"` |
| `image` when set | the filename inside `assets/library/` |

Children go in a `children` array on the parent instance, each with `ref`, `type`, `identifier`, `visibility`, `options`, `children`.

### Component JavaScript

**No official component ships a JS file.** Behaviour lives in custom elements defined in `assets/theme.js` — `<swiper-slider>`, `<theme-tabs>`, `<store-countdown>`, `<recently-viewed>`, `<video-background>`, `<share-component>`, `<quick-view>` and around two dozen more — and the component's Liquid simply emits the tag with its configuration in attributes:

```liquid
<store-countdown timezone="{{ store.timezone }}" date="{{ component.options.date }}"></store-countdown>
```

Per-component **CSS** is the opposite: `assets/component-<name>.css` loaded from the component's own Liquid with `{{ 'component-<name>.css' | asset }}` is exactly what official components do.

Before writing any JavaScript:

1. **Look for an existing element**: `grep -o "customElements.define(['\"][a-z-]*" assets/theme.js`. There are also global helpers such as `copyToClipboard()` and the `ToastNotification` class. Reusing one means writing nothing.
2. If none fits, define a new custom element following the same pattern. Extend **`CustomHTMLElement`**, the theme's base class, and implement `initialize()` rather than `connectedCallback()` — it carries an `initialized` guard, which matters because the Visual Editor re-renders components. 23 of the theme's elements do this.
3. Put the definition in **`assets/custom.js`**. `theme.js` is part of the theme and is overwritten on update; `custom.js` ships as a stub (a few lines, using jQuery) precisely to be extended, and is already loaded on every page.
4. **Append, never rewrite.** Copy the file aside first, add your code at the end between markers, and never touch anything outside them:
   ```js
   /* === my-component — start === */
   class MyComponent extends CustomHTMLElement { initialize() { /* … */ } }
   customElements.define("my-component", MyComponent);
   /* === my-component — end === */
   ```
5. The file loads on every page, so the code must do nothing when the component is absent — defining a custom element satisfies this on its own.

## Common Patterns

### Conditional rendering by login state
```liquid
{% if customer %}
  <a href="/account">My Account</a>
  <a href="{{ customer.logout_url }}">Log Out</a>
{% else %}
  <a href="/login">Log In</a>
  <a href="/register">Register</a>
{% endif %}
```

### Product availability check
```liquid
{% assign minimum_to_buy = product.minimum_quantity | default: 1 | at_least: 1 %}
{% if product.status == 'available' and product.stock_unlimited %}
  <button type="submit">{% t "Add to Cart" %}</button>
{% elsif product.status == 'available' and product.stock >= minimum_to_buy %}
  <button type="submit">{% t "Add to Cart" %}</button>
{% else %}
  <button disabled>{% t "Out of Stock" %}</button>
{% endif %}
```

### Sale badge
```liquid
{% assign discount = product.discount | plus: 0 %}
{% if discount > 0 %}
  <span class="badge-sale">{% t "Sale" %}</span>
{% endif %}
```

### Paginating a category's products

There is no `theme_pagination` partial and no `collection.total_pages`/`current_page`. Pagination is a Liquid block tag, `{% paginate %}` / `{% endpaginate %}`, which exposes a `paged` object and a ready-made `pager` variable inside the block:

```liquid
{% paginate category.products by 24 %}
  {% for prod in paged.products %}
    {{ prod.name }} — {{ prod.price | price }}
  {% endfor %}

  {% if paged.total_pages > 1 %}
    {{ pager }}   {# renders the platform's own <ul class="pager">...</ul>, styled via CSS — see gotcha below #}
  {% endif %}
{% endpaginate %}
```
`paged.total` is the total item count (post-filter), `paged.products` the current page's items, `paged.total_pages` the page count. The same pattern works for `search.results` on the search results template (`paged.results` instead of `paged.products`).

### Assign and capture
```liquid
{% assign sale = false %}
{% assign discount = product.discount | plus: 0 %}
{% if discount > 0 %}
  {% assign sale = true %}
{% endif %}

{% capture product_url %}/{{ product.permalink }}{% endcapture %}
<a href="{{ product_url }}">{{ product.name }}</a>
```

## Gotchas verified against a live production theme

These were each confirmed by exporting and diffing a real installed Jumpseller theme — the first group against Simple 4.13.7 (2026-07-22), the second against Simple 4.13.8 while building components with an agent (2026-09-17) — rather than assumed from generic Liquid conventions.

- **`{{ pager }}`'s default markup is unstyled outside the base theme's own CSS.** It renders as `<ul class="pager"><li class="page-N[ active]"><a href="?page=N">N</a></li>...<li class="next jump">...</li><li class="last jump">...</li></ul>` with classes `.pager`/`.page-N`/`.active`/`.next`/`.jump`/`.last` (not Bootstrap's `.pagination`/`.page-item`/`.page-link`). If your theme excludes the base theme's CSS on a template (e.g. to avoid Bootstrap utility classes colliding with a custom design system), you must supply your own `.pager` CSS or the pagination links will render bare and unstyled.
- **A component's compiled/ported CSS bundle is only as complete as what was actually used on the page(s) it was extracted from.** If you port a compiled stylesheet (e.g. from an external site export) for use across multiple template types (home *and* category), and a utility class is only used on one of those routes, it silently won't exist in the bundle for the other — this can look like "the CSS just isn't applying" for specific classes (e.g. a `sm:grid-cols-3` breakpoint) with no error at all. Verify the bundle actually contains every class your new markup uses, per template type it's loaded on.
- **`{% render 'theme_breadcrumbs' %}` is called unconditionally near the top of every default template file** (`templates/category/Default.liquid`, `templates/product/Default.liquid`, etc.), before `<main>`. If you build a custom breadcrumb inside your own component and also exclude the base theme's CSS on that template, the native one still renders — just unstyled, floating above your header. Remove or gate the call in the template file itself if you don't want it.
- **Global "corner radius" is a first-class theme setting**, not something to hand-roll per component: `config/settings.json`'s `theme_corners_style` (`'rectangular'` | `'rounded'` | `'rounded-large'`) drives a `--radius-style` CSS custom property consumed throughout the base theme's CSS (buttons, images, cards), and `pb_corners` / `article_block_corners` toggle whether product cards / article cards specifically pick it up. Prefer changing this setting over patching the `product_block` partial or its CSS when a merchant asks for "rounded corners everywhere."
- **The platform's CSS minifier mishandles `calc()` expressions that reference a custom property with no space before the operator.** Authored CSS with `calc(var(--radius) + 8px)` gets minified to `calc(var(--radius)+8px)` — which is invalid per the CSS spec (`calc()` requires whitespace around `+`/`-`) — so the browser silently drops the whole declaration and the element falls back to its unset default (e.g. square corners where you expected rounded ones), with no build error anywhere in the pipeline. If you port compiled CSS that defines its own radii via `calc(var(--token) ± Npx)`, resolve those expressions to literal pixel values yourself before uploading, rather than relying on the token indirection surviving minification.
- **`theme import` answers a bare `400` with no body when the content is invalid — but the server did send an explanation and the CLI throws it away.** The two causes seen so far are a schema missing required top-level keys (`… promo-code: name: Expected a string`) and a `product_list` option stored as `null` instead of `[]`. Read the response body instead of bisecting: check `jumpseller theme import --help` for a verbose flag, and if there is none, patch the CLI temporarily to print it and revert immediately — it is a global npm package.
- **`.theme-block[data-bundle-color]` is forced transparent.** `app.css` declares `.theme-block[data-bundle-color] { background: transparent !important; }` and paints `.theme-block[data-bundle-color] .theme-block__wrapper` instead. A child whose root carries both `class="theme-block"` and `data-bundle-color` can therefore never show its palette background: put the attribute on an inner element (as `grid-block` and `banner` do) or give the block a `.theme-block__wrapper` child and let the theme paint it.
- **The `product_block` partial sizes itself, against the page container and not your cell.** It reads the theme-wide setting `options.pb_columns_desktop`, emits `data-columns-desktop`, and `app.css` turns that into `width: var(--theme-columns-by-N)` at six breakpoints — two of them with `!important`. Reuse the partial (you get price, discount badge, stock and wishlist for free) but neutralise the width inside your own component's scope at every breakpoint, and note that beating an `!important` rule needs `!important` plus enough specificity: the theme's selector is `(0,5,0)`.
- **`product-template` is a closed assembly.** Its `blocks` list names the 17 child types the product info column accepts, so putting a new block next to the buy button means editing that official schema — which a theme update reverts, silently removing the block. A section of your own registered in `pages.json` lands on the same page without touching anything; choose deliberately and write the trade-off down somewhere that survives the conversation.
- **There is no variant-change event.** `theme.js` rebuilds the product page through a fixed list of `rebuildXComponent(...)` calls with no extension point. To react to the selected variant from a component of your own, observe the DOM instead: `<product-stock>` rewrites its `data-label` (`available` | `lowstock` | `out-of-stock`) on every variant change, so a `MutationObserver` on that attribute is a clean hook that touches no official file.
