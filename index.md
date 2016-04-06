---
layout: page
---

{% include JB/setup %}
<ul class="posts">
    {% for post in site.posts %}
      <div>
        <span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>

        {% if post.teaser %}
          <a href="{{ post.url }}"><img src="{{ post.teaser }}" width="100%"></a>
        {% endif %}

        {% if post.abstract %}
          {{ post.abstract }}
        {% endif %}

      </div>
    {% endfor %}

</ul>
