---
layout: page
---
{% include JB/setup %}

<ul class="posts">
	{% for post in site.posts %}

  		<li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  		
  		{% if post.teaser %}
  		<p><a href="{{ post.url }}"><img src="{{ post.teaser }}" width="100%"></a></p>
  		{% endif %}
  		
  		{% if post.abstract %}
  		<blockquote>
  		{{ post.abstract }}
  		</blockquote>
  		{% endif %}
  		
  		  {% endfor %}
</ul>

