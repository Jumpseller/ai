# Category Page — Liquid Example

**File:** `templates/category.liquid`

```liquid
<div class="category-page">

  <header class="category-header">
    <h1>{{ category.name }}</h1>
    {% if category.description %}
      <p class="category-description">{{ category.description }}</p>
    {% endif %}
  </header>

  {% if category.products.size > 0 %}

    <div class="product-grid">
      {% for product in category.products %}
        <div class="product-card">
          <a href="{{ product.url }}">

            {% if product.images.first %}
              <div class="product-card-image">
                <img
                  src="{{ product.images.first.url | product_image_url: '400x400' }}"
                  alt="{{ product.name }}"
                >
                {% if product.compare_at_price > product.price %}
                  <span class="badge-sale">Sale</span>
                {% endif %}
                {% if product.status != 'available' or product.stock <= 0 and product.stock_unlimited == false %}
                  <span class="badge-soldout">{{ 'product.out_of_stock' | t }}</span>
                {% endif %}
              </div>
            {% endif %}

            <div class="product-card-info">
              <h2 class="product-card-name">{{ product.name }}</h2>
              <div class="product-card-price">
                {% if product.compare_at_price > product.price %}
                  <span class="price-compare">{{ product.compare_at_price | money }}</span>
                {% endif %}
                <span class="price">{{ product.price | money }}</span>
              </div>
            </div>

          </a>
        </div>
      {% endfor %}
    </div>

    {% if paged.total_pages > 1 %}
      <nav class="pagination" aria-label="Category pagination">
        {% if paged.current_page > 1 %}
          <a href="?page={{ paged.current_page | minus: 1 }}" class="pagination-prev">
            &larr; {{ 'pagination.prev' | t }}
          </a>
        {% endif %}

        <span class="pagination-info">
          {{ paged.current_page }} / {{ paged.total_pages }}
        </span>

        {% if paged.current_page < paged.total_pages %}
          <a href="?page={{ paged.current_page | plus: 1 }}" class="pagination-next">
            {{ 'pagination.next' | t }} &rarr;
          </a>
        {% endif %}
      </nav>
    {% endif %}

  {% else %}
    <p class="category-empty">{{ 'category.no_products' | t }}</p>
  {% endif %}

</div>
```
