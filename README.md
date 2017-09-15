# Welcome, here are some posts

<ul>
  {% for post in site.posts | where_exp: "item" "item.date <= site.date"  %}
    <li>
      <a href="{{ post.url }}">{{ post.title }} ({{post.date | date: ‘%Y %m %d’ }}) - ({{page.excerpt | number_of_words}} words)</a>
    </li>
  {% endfor %}
</ul>