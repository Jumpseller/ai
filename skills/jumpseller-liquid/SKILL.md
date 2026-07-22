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
- `{# comment #}` — comment, not rendered

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
├── locales/              # Translation strings (en.json, es.json, etc.)
└── .jumpseller-store     # Binds a local theme folder to a store domain
```

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

The real filter is `resize`, and it accepts **two different calling conventions** — both are used in official theme code:
```liquid
{{ image.url | resize: '1200x630' }}     <!-- fixed WIDTHxHEIGHT crop -->
{{ image.url | resize: width: 640 }}     <!-- scale by width only, keyword form -->
```
`product_image_url` is not a real Jumpseller filter.

### String filters

```liquid
{{ product.description | strip_html }}
{{ product.description | truncate: 150 }}
{{ coupon | downcase }}
```

### Translation

Jumpseller does **not** use a `| t` filter with dotted translation keys. It translates literal strings inline with the `t` tag, looked up against `locales/*.json`:
```liquid
{% t "Read more" %}
{% t "Add %{product_name} to cart", product_name: prod.name %}
```
For a value you need to reuse (e.g. as a fallback default), capture it first:
```liquid
{% capture t_out_of_stock_default %}{% t "Out of Stock" %}{% endcapture %}
{% assign t_out_of_stock = options.t_out_of_stock | default: t_out_of_stock_default %}
```

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
  "max_usage": 1,
  "required": false,
  "tag": "div",
  "classes": "",
  "templates_in": ["home", "category"],
  "options": {
    "title": { "name": "Title", "type": "input", "default": "Welcome" },
    "body": { "name": "Body text", "type": "text", "default": "" },
    "background_image": { "name": "Background image", "type": "image" },
    "layout": {
      "name": "Layout", "type": "select", "default": "center",
      "options": [{ "value": "left", "label": "Left" }, { "value": "center", "label": "Center" }]
    },
    "show_button": { "name": "Show button", "type": "checkbox", "default": true },
    "columns": { "name": "Columns", "type": "number", "default": 3 },
    "button_url": { "name": "Button URL", "type": "input", "default": "/" }
  }
}
```

- `templates_in` controls which template types this component can be placed on (`"home"`, `"category"`, `"product"`, `"page"`, `"page_category"`, `"searchresults"`, `"contactpage"`, `"error"`, `"customer__login"`, ...).
- Option `type` is one of: `input` (single line), `text` (multi-line), `image`, `select`, `checkbox`, `number`, `category` (a category picker), `url`.
- There is no top-level `"settings"` array — options are a flat `{key: {...}}` object, and they're read via `component.options.<key>`, **not** `component.settings.<key>`.

**`components/my-component.liquid`** — accesses options via `component.options.{key}`:
```liquid
<div class="my-component layout-{{ component.options.layout }}">
  <h2>{{ component.options.title }}</h2>

  {% if component.options.body != blank %}
    <p>{{ component.options.body }}</p>
  {% endif %}

  {% if component.options.background_image.url != blank %}
    <img src="{{ component.options.background_image.url }}" alt="{{ component.options.title }}">
  {% endif %}

  {% if component.options.show_button %}
    <a href="{{ component.options.button_url }}">Learn more</a>
  {% endif %}
</div>
```

Image-type options resolve to an object with `.url` — always guard with `!= blank` and fall back to a default image, since an unset image option is simply absent (not an error, but `.url` on it is blank).

**Registering an instance** — after writing the component files, you still need to (1) add an entry to `config/installed-components.json` keyed by a unique instance id (e.g. `my-component-1`) with `type`, `placement`, `identifier`, `visibility`, and `options` (explicitly duplicating the JSON schema's defaults is the safest approach — don't rely on the platform to backfill them), and (2) reference that instance id in the relevant `config/pages.json` template/variant's `components` array.

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

## Gotchas verified against a live production theme (2026-07-22)

These were each confirmed by exporting and diffing a real installed Jumpseller theme (Simple 4.13.7), rather than assumed from generic Liquid conventions — several of the corrections above exist precisely because Jumpseller's own object model and component system diverge from those generic assumptions.

- **`{{ pager }}`'s default markup is unstyled outside the base theme's own CSS.** It renders as `<ul class="pager"><li class="page-N[ active]"><a href="?page=N">N</a></li>...<li class="next jump">...</li><li class="last jump">...</li></ul>` with classes `.pager`/`.page-N`/`.active`/`.next`/`.jump`/`.last` (not Bootstrap's `.pagination`/`.page-item`/`.page-link`). If your theme excludes the base theme's CSS on a template (e.g. to avoid Bootstrap utility classes colliding with a custom design system), you must supply your own `.pager` CSS or the pagination links will render bare and unstyled.
- **A component's compiled/ported CSS bundle is only as complete as what was actually used on the page(s) it was extracted from.** If you port a compiled stylesheet (e.g. from an external site export) for use across multiple template types (home *and* category), and a utility class is only used on one of those routes, it silently won't exist in the bundle for the other — this can look like "the CSS just isn't applying" for specific classes (e.g. a `sm:grid-cols-3` breakpoint) with no error at all. Verify the bundle actually contains every class your new markup uses, per template type it's loaded on.
- **`{% render 'theme_breadcrumbs' %}` is called unconditionally near the top of every default template file** (`templates/category/Default.liquid`, `templates/product/Default.liquid`, etc.), before `<main>`. If you build a custom breadcrumb inside your own component and also exclude the base theme's CSS on that template, the native one still renders — just unstyled, floating above your header. Remove or gate the call in the template file itself if you don't want it.
- **Global "corner radius" is a first-class theme setting**, not something to hand-roll per component: `config/settings.json`'s `theme_corners_style` (`'rectangular'` | `'rounded'` | `'rounded-large'`) drives a `--radius-style` CSS custom property consumed throughout the base theme's CSS (buttons, images, cards), and `pb_corners` / `article_block_corners` toggle whether product cards / article cards specifically pick it up. Prefer changing this setting over patching the `product_block` partial or its CSS when a merchant asks for "rounded corners everywhere."
- **The platform's CSS minifier mishandles `calc()` expressions that reference a custom property with no space before the operator.** Authored CSS with `calc(var(--radius) + 8px)` gets minified to `calc(var(--radius)+8px)` — which is invalid per the CSS spec (`calc()` requires whitespace around `+`/`-`) — so the browser silently drops the whole declaration and the element falls back to its unset default (e.g. square corners where you expected rounded ones), with no build error anywhere in the pipeline. If you port compiled CSS that defines its own radii via `calc(var(--token) ± Npx)`, resolve those expressions to literal pixel values yourself before uploading, rather than relying on the token indirection surviving minification.
