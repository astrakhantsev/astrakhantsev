---
title: 'Pragmatic guide to choosing a license for your program'
excerpt: "There are many guides to choosing a software license, but they are too philosophic; here I try to formulate clear criteria for pragmatic choice."
permalink: /choosing-license/
tags:
  - Tools
  - Social science
---

{% capture fig_img %}
![Pragramtic diagram for license choice](https://astrakhantsev.com/assets/images/license_diagram.png)
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Diagram for pragmatic choice of the most appropriate license</figcaption>
</figure>

**TL;DR:** if you developed a library, especially for ML/NLP, and it doesn't improve state-of-the-art by dozens of percents, then choose Apache 2.0; if it is an end-user application, choose AGPL, unless you are sure that it can be sold -- in this case, ok, keep it proprietary.


There are many [guides](https://opensource.com/law/13/1/which-open-source-software-license-should-i-use) and even [sites](http://choosealicense.com/) to help in choosing a license for your code without understanding of these licenses.
Obviuosly, these guide have to abstract from details in order to be comprehensible and one or two abstractions can't be well-suited for everyone.

Here I'd like to provide yet another abstraction by focusing on practical, even pragmatic, concerns; in other words -- if you want to extract maximum profit from the code without regard to particular types of freedom.

First of all, if you just started and have nothing working yet, then simply postpone this decision. ~~Most probably, you won't need to decide.~~
Feel free to store the code at bitbucket (paranoids like me like private repos) or github -- btw, not everybody knows that no explicit info in github repository means ["exclusive copyright"](http://choosealicense.com/no-license/).

If you have managed to create an end-user application like text editor or game and believe (and want) it to be sellable, then good luck with startup, keep the code proprietary.

If ~~enough time passed~~ you don't belive that the app can be sold -- e.g. because it is yet another text editor -- then go with GPL.
The reason is simple: users would have source code, which gives some pluses for reputation and possible bug fixes, but you are protected from [Embrace, extend and extinguish](https://en.wikipedia.org/wiki/Embrace,_extend_and_extinguish).

From all kinds of GPL, prefer AGPL, because it prevents from Application Service Provider loophole, or SaaS loophole. which is especially relevant, ~~if you are paranoid like me~~ if your application can be used as a service. 

If you are still hesitate if the app can be sold and think about dual licensing, then I have to disappoint you: it is extremely unlikely that dual-licensing will work for you. 
Read this good [meta-analysis](https://wiki.oulu.fi/download/attachments/58197330/ossd_2015_lauri_leimurautio_vuollet.pdf?version=1&modificationDate=1448956483000&api=v2); in short: it can work only if your app is supposed to be used as a part of something and, much harder condition, if it can take a huge market share. In addition to working dual-licensed examples from the paper above, I'd add Stanford NLP, which is still the best Java NLP library.

If you created a library, then basically you have a choice from 2 alternatives: MIT/BSD vs Apache 2.0.
Motivation is simple again: you want to have as many users as possible, because their number is directly proportional to gained reputation (ok, cover it by goodwill), and only permissive licenses can provide that.

LGPL and joke licenses like [WTFPL](http://www.wtfpl.net/) (and other not OSI-approved licenses) [scare away users](https://www.reddit.com/r/programming/comments/4m18kb/stop_putting_your_project_out_under_public_domain/d3rvktv/), especially influential ones, almost as much as copyleft. Public domain [seems](https://www.reddit.com/r/programming/comments/4m18kb/stop_putting_your_project_out_under_public_domain) to have similar effects.

Returning to the final choice, Apache vs MIT/BSD, I should warn that some guys [report](https://www.reddit.com/r/programming/comments/4m18kb/stop_putting_your_project_out_under_public_domain/d3rx4gz/) that paranoid lawyers in their companies prohibit usage of Apache libs (see [this explanation](https://www.reddit.com/r/programming/comments/4m18kb/stop_putting_your_project_out_under_public_domain/d3sbmhg/)). 

Very roughly: if you belive that (a) your lib can be useful for big corporations, (b) these corporation would fear to be patent-trolled, and (c) you don't fear to be patent-trolled (kind of opposite thoughts, from my point of view), then don't use Apache and go with MIT/BSD.
From these 2 choose the one with letters you like more; or the most popular one, which is MIT ([stats](https://github.com/blog/1964-open-source-license-usage-on-github-com) is based on github repos with explicit license; however, 80% contain no license and I still don't understand what 'no license' row of the table means).

One more argument for Apache 2.0 relates to easier combination of your code with existing Apache libs: Apache 2.0 is [compatible](http://www.dwheeler.com/essays/floss-license-slide.html) with MIT/BSD, i.e. you can declare all your code to be Apache, even if you are using MIT libs, but not otherwise. 
Sure, you can license your files as MIT and keep Apache license for other files, since these licenses are permissive after all, but it would require a bit more efforts: prepend license to each file, choose what to write in github and maven, etc.

And if you do not use Apache libs in your code, then I'd suppose that it is either from the case described in _First of all_-paragraph, or it contains a lot of self-invented wheels. (_BTW, why the invention is so simple in English idiom? For instance, Russians tend to reinvent the bicycle in similar cases._)

**Disclaimer:** of course, I am not a layer. (_BTW, why the corresponding acronym is so popular, while it is so terrible after all?_)
