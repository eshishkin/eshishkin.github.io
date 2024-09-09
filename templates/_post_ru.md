<%*
let title = tp.file.title
let day = tp.date.now("DD")
let month = tp.date.now("MM")
let year = tp.date.now("YYYY")

if (title.startsWith("Untitled")) {
  date = await tp.system.prompt("Date");
  title = await tp.system.prompt("Title");
  if (date) {
    year = date.substring(0, 4);
    month = date.substring(4, 6);
    day = date.substring(6, 8);
  }
  const folder = `/content/posts/${year}/${year}-${month}-${day}/`
  await tp.file.move(folder + "index.ru")
}
-%>

---
layout: post
title: <% title %>
date: <% year %>-<% month %>-<% day %>
URL: /ru/<% year %>/<% month %>/<% day %>/
tags: 
categories: 

---

<% tp.file.cursor(0) %>