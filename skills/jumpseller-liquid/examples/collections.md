# Collections (Category) Page — Liquid Example

**File:** `templates/category.liquid`

```liquid
<div class="collection-page">

  <header class="collection-header">
    <h1>{{ collection.name }}</h1>
    {% if collection.description %}
      <p>{{ collection.description }}</p>
    {% endif %}
  </header>

  {% if collection.products.size > 0 %}

    <div class="product-grid">
      {% for product in collection.products %}
        <div class="product-card">
          <a href="{{ product.url }}">
            {% if product.images.first %}
              <img
                src="{{ product.images.first.url | product_image_url: '400x400' }}"
                alt="{{ product.name }}"
              >
            {% endif %}
            <h2>{{ product.name }}</h2>
            <div class="product-price">
              {% if product.compare_at_price > product.price %}
                <span class="price-compare">{{ product.compare_at_price | money }}</span>
              {% endif %}
              <span class="price">{{ product.price | money }}</span>
            </div>
            {% if product.status != 'available' or product.stock <= 0 %}
              <span class="badge-soldout">{{ 'product.out_of_stock' | t }}</span>
            {% endif %}
          </a>
        </div>
      {% endfor %}
    </div>

    <!-- Pagination -->
    {% if collection.total_pages > 1 %}
      <nav class="pagination">
        {% if collection.current_page > 1 %}
          <a href="?page={{ collection.current_page | minus: 1 }}">&larr; {{ 'pagination.prev' | t }}</a>
        {% endif %}

        <span>{{ collection.current_page }} / {{ collection.total_pages }}</span>

        {% if collection.current_page < collection.total_pages %}
          <a href="?page={{ collection.current_page | plus: 1 }}">{{ 'pagination.next' | t }} &rarr;</a>
        {% endif %}
      </nav>
    {% endif %}

  {% else %}
    <p>{{ 'collection.no_products' | t }}</p>
  {% endif %}

</div>
```
