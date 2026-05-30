# Product Page — Liquid Example

**File:** `templates/product.liquid`

```liquid
<div class="product-page">

  <!-- Image gallery -->
  <div class="product-images">
    {% for image in product.images %}
      <img
        src="{{ image.url | product_image_url: '800x800' }}"
        alt="{{ product.name }}"
        {% if forloop.first %}class="active"{% endif %}
      >
    {% endfor %}
  </div>

  <!-- Product info -->
  <div class="product-info">
    <h1>{{ product.name }}</h1>

    <!-- Price -->
    <div class="product-price">
      {% if product.compare_at_price > product.price %}
        <span class="price-compare">{{ product.compare_at_price | money }}</span>
      {% endif %}
      <span class="price">{{ product.price | money }}</span>
    </div>

    <!-- Description -->
    <div class="product-description">
      {{ product.description }}
    </div>

    <!-- Variants (if any) -->
    {% if product.variants.size > 0 %}
      <div class="product-variants">
        <select name="variant_id" id="variant-select">
          {% for variant in product.variants %}
            <option
              value="{{ variant.id }}"
              {% if variant.stock <= 0 %}disabled{% endif %}
            >
              {% for option in variant.options %}{{ option.value }}{% unless forloop.last %} / {% endunless %}{% endfor %}
              — {{ variant.price | money }}
            </option>
          {% endfor %}
        </select>
      </div>
    {% endif %}

    <!-- Add to cart -->
    <form action="/cart/add" method="post">
      <input type="hidden" name="product_id" value="{{ product.id }}">
      <input type="number" name="quantity" value="1" min="1">
      {% if product.status == 'available' and product.stock > 0 %}
        <button type="submit">{{ 'product.add_to_cart' | t }}</button>
      {% else %}
        <button type="button" disabled>{{ 'product.out_of_stock' | t }}</button>
      {% endif %}
    </form>

  </div>
</div>
```
