# Product Page — Liquid Example

**File:** `templates/product.liquid`

```liquid
<div class="product-page">

  <div class="product-images">
    {% for image in product.images %}
      <img
        src="{{ image.url | product_image_url: '800x800' }}"
        alt="{{ product.name }}"
        {% if forloop.first %}class="active"{% endif %}
      >
    {% endfor %}
  </div>

  <div class="product-info">
    <h1>{{ product.name }}</h1>

    <div class="product-price">
      {% if product.compare_at_price > product.price %}
        <span class="price-compare">{{ product.compare_at_price | money }}</span>
        <span class="badge-sale">Sale</span>
      {% endif %}
      <span class="price">{{ product.price | money }}</span>
    </div>

    <div class="product-description">
      {{ product.description }}
    </div>

    {% unless product.variants == empty %}
      <div class="product-variants">
        <label for="variant-select">{{ 'product.choose_variant' | t }}</label>
        <select name="variant_id" id="variant-select">
          {% for variant in product.variants %}
            <option
              value="{{ variant.id }}"
              {% if variant.stock <= 0 and variant.stock_unlimited == false %}disabled{% endif %}
            >
              {% for option in variant.options %}
                {{ option.value }}{% unless forloop.last %} / {% endunless %}
              {% endfor %}
              — {{ variant.price | money }}
            </option>
          {% endfor %}
        </select>
      </div>
    {% endunless %}

    <form action="/cart/add" method="post">
      <input type="hidden" name="product_id" value="{{ product.id }}">
      <input type="number" name="quantity" value="1" min="1">

      {% if product.status == 'available' and product.stock > 0 or product.stock_unlimited %}
        <button type="submit" class="btn-add-to-cart">
          {{ 'product.add_to_cart' | t }}
        </button>
      {% else %}
        <button type="button" class="btn-sold-out" disabled>
          {{ 'product.out_of_stock' | t }}
        </button>
      {% endif %}
    </form>

  </div>
</div>
```
