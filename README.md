# Welcome!

## My latest blog post

{% for post in site.posts limit:1 %}
  post.content
{% endfor %}

## Here are some posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }} ({{post.date | date: '%Y-%m-%d' }}) - ({{post.content | number_of_words}} words)</a>
    </li>
  {% endfor %}
</ul>