I'm a embedded software engineer.

Below are some of my most popular blog posts, which you may be interested in
reading:

{% for post in site.posts %}{% if post.featured %}
- [{{ post.title }}]({{ post.url }}) <small>{{ post.date | date_to_long_string }}</small>{% endif %} {% endfor %}
