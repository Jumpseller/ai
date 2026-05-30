# Cart — Liquid Example

**File:** `templates/cart.liquid`

```liquid
<div class="cart-page">
  <h1>{{ 'cart.title' | t }}</h1>

  {% if cart.item_count > 0 %}

    <div class="cart-items">
      {% for item in cart.items %}
        <div class="cart-item">
          <img
            src="{{ item.image | product_image_url: '100x100' }}"
            alt="{{ item.name }}"
          >
          <div class="cart-item-details">
            <a href="{{ item.url }}">{{ item.name }}</a>
            {% if item.variant_title %}
              <span class="variant">{{ item.variant_title }}</span>
            {% endif %}
            <span class="price">{{ item.price | money }}</span>
          </div>
          <div class="cart-item-quantity">
            <form action="/cart/update" method="post">
              <input type="hidden" name="line" value="{{ forloop.index }}">
              <input type="number" name="quantity" value="{{ item.quantity }}" min="0">
              <button type="submit">{{ 'cart.update' | t }}</button>
            </form>
          </div>
          <div class="cart-item-total">
            {{ item.line_price | money }}
          </div>
        </div>
      {% endfor %}
    </div>

    <div class="cart-summary">
      <div class="cart-total">
        {{ 'cart.total' | t }}: <strong>{{ cart.total_price | money }}</strong>
      </div>
      <a href="/checkout" class="btn-checkout">{{ 'cart.checkout' | t }}</a>
    </div>

  {% else %}
    <p class="cart-empty">{{ 'cart.empty' | t }}</p>
    <a href="/">{{ 'cart.continue_shopping' | t }}</a>
  {% endif %}

</div>
```
