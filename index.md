# Hello, bro!

<ul>
  {% for post in site.posts %}
    <li>
      {{ post.excerpt }}
      <a href="{{ post.url }}">Read more</a>
    </li>
  {% endfor %}
</ul>
