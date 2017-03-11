<style type="text/css">

#index {
    font-size: 2em;
    list-style-type: none;
}

</style>

<ul>
  {% for post in site.posts %}
    <li id="index">
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>