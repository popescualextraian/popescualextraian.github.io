---
layout: page
title: Pathfinder
permalink: /pathfinder/
---

Level 4 agentic development: the ideas behind the setup, and
[Pathfinder Dev](https://github.com/popescualextraian/pathfinder-dev-base), the open-source tools
that implement them.

## The method

A method for Level 4 agentic development: principles, a maturity model and nine pillars for the
setup around the agents. **[Read part 1: Purpose and scope](/pathfinder/method/)**; the other parts are coming soon.

## Articles

{% for post in site.categories.pathfinder %}
- {{ post.date | date: "%d %b %Y" }} · [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
