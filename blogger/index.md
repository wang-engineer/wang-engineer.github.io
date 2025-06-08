---
layout: default
title: Blog Posts
permalink: /blogger/index/
---

<h1>📝 Blog Posts</h1>

<div id="tag-buttons"></div>
<ul id="blog-list">
  {% for post in site.pages %}
    {% if post.path contains 'blogger/' and post.title %}
      <li class="blog-post" data-tags="{{ post.tags | join: ' ' }}">
        <a href="{{ post.url }}">{{ post.title }}</a>
        <span style="color: gray;"> — Tags: {{ post.tags | join: ', ' }}</span>
      </li>
    {% endif %}
  {% endfor %}
</ul>

<script>
  // Collect all tags
  const posts = document.querySelectorAll('.blog-post');
  const tagSet = new Set();

  posts.forEach(post => {
    const tags = post.dataset.tags.split(' ');
    tags.forEach(tag => tagSet.add(tag));
  });

  // Render tag buttons
  const tagContainer = document.getElementById('tag-buttons');
  const allBtn = document.createElement('button');
  allBtn.textContent = 'Show All';
  allBtn.onclick = () => {
    posts.forEach(p => p.style.display = '');
  };
  tagContainer.appendChild(allBtn);

  tagSet.forEach(tag => {
    const btn = document.createElement('button');
    btn.textContent = tag;
    btn.onclick = () => {
      posts.forEach(p => {
        p.style.display = p.dataset.tags.includes(tag) ? '' : 'none';
      });
    };
    tagContainer.appendChild(btn);
  });
</script>