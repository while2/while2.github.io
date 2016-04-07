---
layout: page
---

{% include JB/setup %}
<ul class="posts">
    {% for post in site.posts %}
      <div>
        <span>{{ post.date | date: "%Y-%m-%d" }}</span> &#10147; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>

        {% if post.teaser %}
          <p><a href="{{ post.url }}"><img src="{{ post.teaser }}" width="100%"></a></p>
          <hr/>
        {% endif %}

        {% if post.abstract %}
          <p>{{ post.abstract }}</p>
          <hr/>
        {% endif %}

      </div>
    {% endfor %}

</ul>
