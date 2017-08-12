+++
date = "2016-03-04"
title = "Getting Started With hugo-geo"
tags = ["tutorial", "series", "hugo"]
description = "A tutorial on how to use and customize the Hugo-Geo theme"
index = true
highlight = true
+++

{{< highlight html >}}
<section id="main">
  <div>
   <h1 id="title">{{ .Title }}</h1>
    {{ range .Data.Pages }}
        {{ .Render "summary"}}
    {{ end }}
  </div>
</section>
{{< /highlight >}}
