<ul>
  {% for post in site.posts %}
    <li>
      <h4>{{ post.title }}</h4>
      <p>{{ post.date }}</p>
      {{ post.excerpt }}
      <a href="{{ post.url }}">Read more</a>
    </li>
  {% endfor %}
</ul>
