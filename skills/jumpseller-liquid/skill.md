# Jumpseller Liquid Theming

Use this skill when building or editing Jumpseller themes using the Liquid templating language.

## What is Liquid

Liquid is the templating language used in Jumpseller themes. It has three types of delimiters:
- `{{ variable }}` — outputs a value
- `{% tag %}` — logic (if, for, assign, include, etc.)
- `{# comment #}` — comment, not rendered

## Theme File Structure

```
theme/
├── templates/       # Page-level templates (product.liquid, cart.liquid, etc.)
├── partials/        # Reusable fragments included in templates
├── components/      # Configurable sections (each has .liquid + .json)
├── assets/          # CSS, JS, images, fonts
├── config/          # Theme settings (settings.json, settings_data.json)
├── locales/         # Translation strings (en.json, es.json, etc.)
└── .jumpseller-store  # Binds the theme to a store (contains store login domain)
```

The `.jumpseller-store` file contains the store's domain (e.g., `your-store.jumpseller.com`). Do not commit credentials here — it is used by the Jumpseller CLI to know which store to sync with.

## Global Liquid Objects

These objects are available in all templates:

| Object | Description |
|---|---|
| `store` | Store configuration: name, currency, country, logo, social links |
| `product` | Current product (on product pages) |
| `collection` | Current category/collection (on category pages) |
| `cart` | Current cart: items, totals, item count |
| `customer` | Logged-in customer (nil if not logged in) |
| `order` | Current order (on order confirmation pages) |
| `page` | Current custom page |
| `checkout` | Checkout state (on checkout pages) |
| `theme` | Theme metadata |

### `store` object

```liquid
{{ store.name }}          <!-- Store name -->
{{ store.currency }}      <!-- e.g. "USD" -->
{{ store.country }}       <!-- e.g. "US" -->
{{ store.email }}         <!-- Contact email -->
{{ store.logo }}          <!-- Logo URL -->
```

### `product` object

```liquid
{{ product.id }}
{{ product.name }}
{{ product.description }}
{{ product.price | money }}
{{ product.compare_at_price | money }}
{{ product.sku }}
{{ product.stock }}
{{ product.status }}      <!-- "available" or "unavailable" -->

{% for image in product.images %}
  <img src="{{ image.url }}" alt="{{ image.alt }}">
{% endfor %}

{% for variant in product.variants %}
  {{ variant.sku }} — {{ variant.price | money }}
{% endfor %}

{% for category in product.categories %}
  {{ category.name }}
{% endfor %}
```

### `cart` object

```liquid
{{ cart.item_count }}
{{ cart.total_price | money }}

{% for item in cart.items %}
  {{ item.name }} × {{ item.quantity }} = {{ item.line_price | money }}
{% endfor %}
```

### `customer` object

```liquid
{% if customer %}
  {{ customer.name }}
  {{ customer.email }}
{% else %}
  <!-- Not logged in -->
{% endif %}
```

### `collection` object

```liquid
{{ collection.name }}
{{ collection.description }}

{% for product in collection.products %}
  {{ product.name }}
{% endfor %}

<!-- Pagination -->
{{ collection.current_page }} / {{ collection.total_pages }}
```

## Liquid Filters

### Money formatting
```liquid
{{ product.price | money }}           <!-- formats with store currency -->
{{ product.price | money_without_currency }}
```

### Image resizing
```liquid
{{ image.url | product_image_url: '300x300' }}
{{ image.url | product_image_url: '600x' }}    <!-- width only -->
```

### String filters
```liquid
{{ product.name | upcase }}
{{ product.name | downcase }}
{{ product.description | strip_html }}
{{ product.description | truncate: 150 }}
```

### Translation
```liquid
{{ 'cart.checkout' | t }}             <!-- looks up key in locales/ -->
{{ 'product.add_to_cart' | t }}
```

## Including Partials

```liquid
{% include 'partials/product-card', product: product %}
{% include 'partials/header' %}
{% include 'partials/footer' %}
```

Pass variables to a partial by adding them after the path as named arguments.

## Component System

Components are configurable sections that merchants can customize from the theme editor. Each component has two files:

**`components/my-component.json`** — defines the settings schema:
```json
{
  "name": "My Component",
  "settings": [
    {
      "type": "text",
      "id": "title",
      "label": "Title",
      "default": "Welcome"
    },
    {
      "type": "image",
      "id": "background_image",
      "label": "Background Image"
    },
    {
      "type": "color",
      "id": "text_color",
      "label": "Text Color",
      "default": "#000000"
    },
    {
      "type": "select",
      "id": "layout",
      "label": "Layout",
      "options": [
        { "value": "left", "label": "Left" },
        { "value": "center", "label": "Center" }
      ],
      "default": "center"
    }
  ]
}
```

Setting types: `text`, `textarea`, `richtext`, `image`, `color`, `select`, `checkbox`, `number`, `url`.

**`components/my-component.liquid`** — uses the settings:
```liquid
<div class="my-component" style="color: {{ component.settings.text_color }}">
  <h2>{{ component.settings.title }}</h2>
  {% if component.settings.background_image %}
    <img src="{{ component.settings.background_image | product_image_url: '1200x' }}"
         alt="{{ component.settings.title }}">
  {% endif %}
</div>
```

Access settings in a component via `component.settings.{setting_id}`.

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
  <button type="submit">Add to Cart</button>
{% else %}
  <button disabled>Out of Stock</button>
{% endif %}
```

### Paginating a collection
```liquid
{% if collection.total_pages > 1 %}
  {% if collection.current_page > 1 %}
    <a href="?page={{ collection.current_page | minus: 1 }}">Previous</a>
  {% endif %}
  {% if collection.current_page < collection.total_pages %}
    <a href="?page={{ collection.current_page | plus: 1 }}">Next</a>
  {% endif %}
{% endif %}
```
