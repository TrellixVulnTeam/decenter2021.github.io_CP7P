---
title: Download
permalink: /download/
layout: single
classes: wide
sidebar:
  title: "Download"
  nav: sidebar-downloads
---
Latest release
=====
{% for release in site.data.latest-release %}[Download latest DECENTER release ({{release.tag}})]({{release.urlZip}}){: .btn .btn--info}{% break %}{% endfor %}
<br>Older releases can be found [here](https://github.com/LeonardoPedroso/DECENTER_toolbox/releases).

<hr>
Installation
=====

An installation tutorial can be found [here](/{{site.baseurl}}/tutorials/installation/). <br>
<hr>
License
=====
DECENTER is distributed under the MIT License: <!--[<span style="color:purple">possibly change license</span>]-->


Copyright (c) 2020 DECENTER

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
{: .text-justify}

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
{: .text-justify}

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
{: .text-justify}