---
permalink: /cv/
title: CV
nav: true
nav_order: 5
redirect_pdf: /assets/pdf/CV.pdf
---

{% assign cv_url = page.redirect_pdf | relative_url %}
<!doctype html>
<html lang="{{ site.lang | default: 'en' }}">
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="refresh" content="0; url={{ cv_url }}" />
    <link rel="canonical" href="{{ cv_url }}" />
    <title>{{ page.title }}</title>
  </head>
  <body>
    <script>
      window.location.replace("{{ cv_url }}");
    </script>
    Redirecting to <a href="{{ cv_url }}">CV</a>…
  </body>
</html>
