---
layout: page
title: Articles
---

<div>
{% for tag in site.tags %}
  {% assign tag_size = site.posts | where_exp: 'post', 'post.tags contains tag.name' | size %}
  <span>
    <a href="/tags/{{ tag.slug }}" class="tag tag-{{ tag.name }}" style="font-size: {{ tag_size | times: 100 | divided_by: site.tags.size | plus: 70  }}% !important;">
      <span>{{ tag.name }}</span>
      <span>
        ({{ tag_size }})
      </span>
    </a>
  </span>
{% endfor %}
</div>
