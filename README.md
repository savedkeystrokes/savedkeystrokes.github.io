# Welcome!

{% for post in site.posts | where_exp: "item" "item.date <= site.date' | limit:1 %}
# My latest blog post ({{post.date | date: '%Y-%m-%d' }})

{{ post.content }}
{% endfor %}

## Here are some posts

<ul>
  {% for post in site.posts | where_exp: "item" "item.date <= site.date' %}
    <li>
      <a href="{{ post.url | remove: ".html" }}">{{ post.title }} ({{post.date | date: '%Y-%m-%d' }}) - ({{post.content | number_of_words}} words)</a>
    </li>
  {% endfor %}
</ul>