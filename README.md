# Welcome!

## Here are some posts

<ul>
  {% for post in site.posts  %}
    <li>
      <a href="{{ post.url }}">{{ post.title }} ({{post.date | date: '%Y %m %d' }}) - ({{post.excerpt | number_of_words}} words)</a>
    </li>
  {% endfor %}
</ul>