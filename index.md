# Jawad Amin

**Customer Engineer at Google | AI Infrastructure**

Welcome to my personal site for AI hardware benchmarking, TPU recipes, and engineering notes.

---

## Blogs & Notes
{% for post in site.posts %}
* **[{{ post.date | date: "%B %d, %Y" }}]** — [{{ post.title }}]({{ post.url | relative_url }})
  <br><sub>{{ post.description }}</sub>
{% else %}
*No posts published yet. Stay tuned!*
{% endfor %}

---

## Public Repositories
* **[qwen3.6-27b-tpu-recipe](https://github.com/jawadaminGOOG/qwen3.6-27b-tpu-recipe)**  
  *Benchmarking infrastructure and setup recipes for serving Qwen 3.6 27B on Cloud TPUs.*
* **[jawadaminGOOG.github.io](https://github.com/jawadaminGOOG/jawadaminGOOG.github.io)**  
  *Personal homepage and engineering notes source repository.*

[View all public repositories directly on GitHub](https://github.com/jawadaminGOOG?tab=repositories)
