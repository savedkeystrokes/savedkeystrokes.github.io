# Welcome!

## Here are some posts

<ul>
  {% for post in site.posts | where_exp: "item" "item.date <= site.date %}
    <li>
      <a href="{{ post.url }}">{{ post.title }} ({{post.date | date: '%Y-%m-%d' }}) - ({{post.content | number_of_words}} words)</a>
    </li>
  {% endfor %}
</ul>