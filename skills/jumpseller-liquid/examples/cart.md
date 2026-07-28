# Cart — Liquid Example

**File:** `templates/cart.liquid`

In Jumpseller the in-progress cart is the `order` object; its line items are iterated as `orderedproduct`. (The legacy `cart` object is deprecated.)

```liquid
<div class="cart-page">
  <h1>{{ 'cart.title' | t }}</h1>

  {% if order.products_count > 0 %}

    <div class="cart-items">
      {% for orderedproduct in order.products %}
        <div class="cart-item">

          <a href="{{ orderedproduct.url }}">
            <img
              src="{{ orderedproduct.image | product_image_url: '100x100' }}"
              alt="{{ orderedproduct.name }}"
            >
          </a>

          <div class="cart-item-details">
            <a href="{{ orderedproduct.url }}">{{ orderedproduct.name }}</a>
            {% if orderedproduct.options.size > 0 %}
              <span class="cart-item-variant">
                {% for option in orderedproduct.options %}{{ option }}{% endfor %}
              </span>
            {% endif %}
            <span class="cart-item-price">{{ orderedproduct.price | money }}</span>
          </div>

          <div class="cart-item-quantity">
            <form action="/cart/update" method="post">
              <input type="hidden" name="line" value="{{ forloop.index }}">
              <input type="number" name="quantity" value="{{ orderedproduct.qty }}" min="0">
              <button type="submit">{{ 'cart.update' | t }}</button>
            </form>
          </div>

          <div class="cart-item-total">
            {{ orderedproduct.subtotal | money }}
          </div>

        </div>
      {% endfor %}
    </div>

    <div class="cart-summary">
      <div class="cart-subtotal">
        {{ 'cart.subtotal' | t }}: {{ order.total | money }}
      </div>
      <p class="cart-taxes-note">{{ 'cart.taxes_note' | t }}</p>
      <a href="/checkout" class="btn-checkout">
        {{ 'cart.checkout' | t }}
      </a>
    </div>

  {% else %}

    <p class="cart-empty">{{ 'cart.empty' | t }}</p>
    <a href="/" class="btn-continue">{{ 'cart.continue_shopping' | t }}</a>

  {% endif %}
</div>
```
