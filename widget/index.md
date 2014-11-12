---
layout: default
title: Mulu Widget
---
# Mulu Widget

The Mulu widget enables visitors to your web site or blog to shop for products
mentioned on a web page.  It renders a box showing product images which link to
online merchants that sell the product.


## Code Version 1

Configure the parameters in this code, then insert the code into your web site.

{% highlight html %}
<div
    id="mbox_loader"
    data-account="ACCOUNT_IDENTIFIER"
    data-headline="HEADLINE"
    data-footer="FOOTER">
</div>
<script src="//widget.mulu.me/1/mbox_loader.js"></script>
{% endhighlight %}

The parameters are

| Name | Value |
|:-----|:------|
| `data-account` | Account identifier provided by Mulu |
| `data-headline` | Headline text to render |
| `data-footer` | Footer text to render |
{:.table .table-bordered}
