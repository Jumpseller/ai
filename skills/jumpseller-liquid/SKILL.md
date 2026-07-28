---
name: jumpseller-liquid
description: "Use when building or editing Jumpseller themes, Liquid templates, partials, or theme components."
---
# Jumpseller Liquid Theming

Use this skill when building or editing Jumpseller themes using the Liquid templating language.

## ⚠️ Security: never put API credentials in theme code

Theme files (Liquid, HTML, CSS, JS, assets) are **served publicly to every visitor**. **Never** embed credentials in them:

- Do NOT hardcode `JUMPSELLER_LOGIN_KEY`, `JUMPSELLER_AUTH_TOKEN`, a Basic-auth string, or any API/MCP token into Liquid, `<script>` tags, asset files, or client-side `fetch`/`XMLHttpRequest` calls. These credentials are **full-access** — a single leaked key lets anyone read every customer and order and modify the entire store.
- Liquid renders **server-side**, so storefront features get store data from Liquid objects (`order`, `product`, `products`, `category`, `customer`, etc.) **without any credentials**. Use those.
- Need data not available in Liquid (e.g. recent orders for a "social proof / FOMO" widget)? Fetch it **server-side or at build time** with the API/CLI, then inject only the **non-sensitive, anonymized** result into the theme as static content (e.g. "Someone in Santiago just bought X") — never the credentials, and never raw customer PII (full name, email, phone, address).

## What is Liquid

Liquid is the templating language used in Jumpseller themes. It has three types of delimiters:
- `{{ variable }}` — outputs a value
- `{% tag %}` — logic (if, for, assign, include, etc.)
- `{# comment #}` — comment, not rendered

## Theme File Structure

```
theme/
├── templates/          # Page-level templates (product.liquid, cart.liquid, etc.)
├── partials/           # Reusable fragments included with {% include %}
├── components/         # Configurable sections — each has a .liquid + .json pair
├── assets/             # CSS, JS, images, fonts
├── config/             # Theme settings (settings.json, settings_data.json)
├── locales/            # Translation strings (en.json, es.json, etc.)
└── .jumpseller-store   # Binds the theme to a store (contains the store domain)
```

The `.jumpseller-store` file contains the store's domain (e.g. `your-store.jumpseller.com`). It is used by the Jumpseller CLI to know which store to sync with. Do not commit API credentials here.

### Local theme development (Jumpseller CLI)

Theme files are edited locally and synced to a store with the **Jumpseller CLI** — the official command-line tool for local theme development. It is a separate project: [`Jumpseller/jumpseller-cli`](https://github.com/Jumpseller/jumpseller-cli).

```bash
npm i -g @jumpseller/cli   # install globally

jumpseller access          # set up store credentials
jumpseller theme --help    # local theme development commands
```

For full command reference — `access` (credentials), `theme export`/`import`/`watch`/`apply`, store resolution, and config file locations — see the **`jumpseller-cli` skill**. The CLI uses the same Login key + Auth Token as the REST API and MCP server.

## Global Liquid Objects

These objects are available in all templates:

| Object | Description |
|---|---|
| `store` | Store configuration: name, currency, country, logo, social links |
| `products` | Global product access: `.all`, `.featured`, `.latest`, and `.product["permalink"]` lookup — available on **any** page |
| `product` | Current product (available on product pages) |
| `category` | Current category (available on category pages) — `.products` holds its products |
| `customer` | Logged-in customer (`nil` if not logged in) |
| `order` | The in-progress cart **and** the placed order (cart, checkout, and order-confirmation pages) |
| `paged` | Pagination state for the current listing: `.current_page`, `.total_pages`, `.per_page` |
| `page` | Current custom page |
| `checkout` | Checkout state (available on checkout pages) |
| `theme` | Theme metadata |

### `store` object

```liquid
{{ store.name }}        <!-- Store name -->
{{ store.currency }}    <!-- e.g. "USD" -->
{{ store.country }}     <!-- e.g. "US" -->
{{ store.email }}       <!-- Contact email -->
{{ store.logo }}        <!-- Logo URL -->
```

### `products` object (global product access)

Unlike `product` (only on product pages) and `category.products` (only on category pages), the global `products` object lets you fetch products from **any** template — including the homepage, custom pages, and partials.

```liquid
{% for product in products.all %}      <!-- all products -->
{% for product in products.featured %} <!-- featured products -->
{% for product in products.latest %}   <!-- latest products -->
  {{ product.name }} — {{ product.price | money }}
{% endfor %}

<!-- Look up a single product by its permalink -->
{% assign featured = products.product["my-product-permalink"] %}
{{ featured.name }} — {{ featured.price | money }}
```

This is the mechanism for looking up an arbitrary product by permalink. See <https://jumpseller.com/support/liquid/> for the full object reference.

### `product` object

```liquid
{{ product.id }}
{{ product.name }}
{{ product.description }}
{{ product.price | money }}
{{ product.compare_at_price | money }}
{{ product.sku }}
{{ product.stock }}
{{ product.status }}        <!-- "available" or "unavailable" -->
{{ product.permalink }}

{% for image in product.images %}
  <img src="{{ image.url }}" alt="{{ product.name }}">
{% endfor %}

{% for variant in product.variants %}
  {{ variant.sku }} — {{ variant.price | money }}
{% endfor %}

{% for category in product.categories %}
  {{ category.name }}
{% endfor %}
```

### `order` object (cart & order)

In Jumpseller the in-progress cart is the same `order` object used on the checkout and confirmation pages. The loop variable for its line items is `orderedproduct`.

```liquid
{{ order.products_count }}        <!-- total number of items -->
{{ order.total | money }}         <!-- grand total incl. taxes/shipping/discounts -->

{% for orderedproduct in order.products %}
  {{ orderedproduct.name }}
  {{ orderedproduct.name_with_options }}   <!-- name + selected options -->
  {{ orderedproduct.qty }}
  {{ orderedproduct.price | money }}
  {{ orderedproduct.subtotal | money }}    <!-- qty × price -->
  {{ orderedproduct.url }}
  {{ orderedproduct.image | product_image_url: '100x100' }}
  {% for option in orderedproduct.options %}{{ option }}{% endfor %}
{% endfor %}
```

> The legacy `cart` object (`cart.total`, `cart.items`, …) is **deprecated** in favour of `order`. Use `order` in new themes.

### `customer` object

```liquid
{% if customer %}
  {{ customer.name }}
  {{ customer.email }}
  {{ customer.orders_count }}
{% else %}
  <!-- visitor is not logged in -->
{% endif %}
```

### `category` object

```liquid
{{ category.name }}
{{ category.description }}
{{ category.url }}

{% for product in category.products %}
  {{ product.name }} — {{ product.price | money }}
{% endfor %}
```

Pagination for the listing comes from the separate `paged` object, not from `category`:

```liquid
{{ paged.current_page }}   <!-- e.g. 3 -->
{{ paged.total_pages }}    <!-- e.g. 4 -->
{{ paged.per_page }}       <!-- e.g. 12 -->
```

## Liquid Filters

### Money formatting
```liquid
{{ product.price | money }}                     <!-- with currency symbol -->
{{ product.price | money_without_currency }}    <!-- number only -->
```

### Image resizing
```liquid
{{ image.url | product_image_url: '400x400' }}  <!-- crop to square -->
{{ image.url | product_image_url: '800x' }}     <!-- scale by width only -->
{{ image.url | product_image_url: 'x600' }}     <!-- scale by height only -->
```

### String filters
```liquid
{{ product.name | upcase }}
{{ product.name | downcase }}
{{ product.description | strip_html }}
{{ product.description | truncate: 150 }}
{{ product.name | slugify }}
```

### Translation
```liquid
{{ 'cart.checkout' | t }}             <!-- looks up key in locales/ -->
{{ 'product.add_to_cart' | t }}
{{ 'product.out_of_stock' | t }}
{{ 'pagination.prev' | t }}
{{ 'pagination.next' | t }}
```

### Array filters
```liquid
{{ product.images | first }}
{{ product.images.size }}
{{ product.variants | map: 'price' | min }}
```

## Including Partials

```liquid
{% include 'partials/product-card' %}
{% include 'partials/product-card', product: product %}
{% include 'partials/header' %}
{% include 'partials/footer' %}
```

Pass variables to a partial by adding them as named arguments after the path.

## Component System

Components are configurable sections that merchants can customize from the theme editor without touching code. Each component has two files:

**`components/my-component.json`** — defines the settings schema:
```json
{
  "name": "My Component",
  "settings": [
    { "type": "text",     "id": "title",            "label": "Title",            "default": "Welcome" },
    { "type": "textarea", "id": "subtitle",          "label": "Subtitle",         "default": "" },
    { "type": "richtext", "id": "body",              "label": "Body Text",        "default": "" },
    { "type": "image",    "id": "background_image",  "label": "Background Image" },
    { "type": "color",    "id": "text_color",        "label": "Text Color",       "default": "#000000" },
    { "type": "select",   "id": "layout",            "label": "Layout",
      "options": [
        { "value": "left",   "label": "Left" },
        { "value": "center", "label": "Center" }
      ],
      "default": "center"
    },
    { "type": "checkbox", "id": "show_button",       "label": "Show Button",      "default": true },
    { "type": "number",   "id": "columns",           "label": "Columns",          "default": 3 },
    { "type": "url",      "id": "button_url",        "label": "Button URL",       "default": "/" }
  ]
}
```

Setting types: `text`, `textarea`, `richtext`, `image`, `color`, `select`, `checkbox`, `number`, `url`.

**`components/my-component.liquid`** — accesses settings via `component.settings.{id}`:
```liquid
<div class="my-component layout-{{ component.settings.layout }}"
     style="color: {{ component.settings.text_color }}">

  <h2>{{ component.settings.title }}</h2>

  {% if component.settings.subtitle != blank %}
    <p>{{ component.settings.subtitle }}</p>
  {% endif %}

  {% if component.settings.background_image %}
    <img src="{{ component.settings.background_image | product_image_url: '1200x' }}"
         alt="{{ component.settings.title }}">
  {% endif %}

  {% if component.settings.show_button %}
    <a href="{{ component.settings.button_url }}">Learn more</a>
  {% endif %}

</div>
```

## Common Patterns

### Conditional rendering by login state
```liquid
{% if customer %}
  <a href="/account">My Account</a>
  <a href="/logout">Log Out</a>
{% else %}
  <a href="/login">Log In</a>
  <a href="/register">Register</a>
{% endif %}
```

### Product availability check
```liquid
{% if product.status == 'available' and product.stock > 0 %}
  <button type="submit">{{ 'product.add_to_cart' | t }}</button>
{% elsif product.status == 'available' and product.stock_unlimited %}
  <button type="submit">{{ 'product.add_to_cart' | t }}</button>
{% else %}
  <button disabled>{{ 'product.out_of_stock' | t }}</button>
{% endif %}
```

### Sale badge
```liquid
{% if product.compare_at_price > product.price %}
  <span class="badge-sale">Sale</span>
{% endif %}
```

### Paginating a category listing
```liquid
{% if paged.total_pages > 1 %}
  <nav class="pagination">
    {% if paged.current_page > 1 %}
      <a href="?page={{ paged.current_page | minus: 1 }}">&larr; {{ 'pagination.prev' | t }}</a>
    {% endif %}
    <span>{{ paged.current_page }} / {{ paged.total_pages }}</span>
    {% if paged.current_page < paged.total_pages %}
      <a href="?page={{ paged.current_page | plus: 1 }}">{{ 'pagination.next' | t }} &rarr;</a>
    {% endif %}
  </nav>
{% endif %}
```

### Assign and capture
```liquid
{% assign sale = false %}
{% if product.compare_at_price > product.price %}
  {% assign sale = true %}
{% endif %}

{% capture product_url %}/products/{{ product.permalink }}{% endcapture %}
<a href="{{ product_url }}">{{ product.name }}</a>
```
