---
layout: post
title: "Type 1 Font for Matplotlib"
date: 2016-02-17 16:57:27
comments: true
description: "Type 1 Font for Matplotlib"
keywords: ""
categories: Research

tags:
- Python
- Matplotlib
- Type 1
---

When figures are generated in Matplotlib, they use "Type 3" font by default, which is conflict with some publishers' requirements for posting a paper in some conference. This essay mentions the solution for this issue.

### Check fonts used in a PDF with `PdfFonts`

`PdfFonts` can be installed with `Brew` in Mac:

{% highlight bash %}
	brew install xpdf
{% endhighlight %}

or

{% highlight bash %}
	brew install poppler
{% endhighlight %}

Note that both tools are based on "XQuartz", a X Window System running on OS X. So following the instructions if error occurs.

After you are done with the installation, you can run following command in the terminal to check fonts in a PDF file:

{% highlight bash %}
	pdffonts /path/to/file.pdf
{% endhighlight %}

In OS X, there is an alternative solution to get the font information. However, it doesn't work for `pdflatex` PDFs.

{% highlight bash %}
	strings /path/to/document.pdf | grep -i FontName
{% endhighlight %}

In my case, the figures drawn by Matplotlib have "Type 3". This is not allowed in some publishers' rules. So I need to redraw figures with "Type 1".


### Matplotlib configurations

Use the following statements to configure the Matplotlib. The figures will be plotted with "Type 3" font. However, the fonts of letters and numbers may vary. For example, the letters are bold, while the numbers are not.

{% highlight python %}
matplotlib.rcParams['ps.useafm'] = True
matplotlib.rcParams['pdf.use14corefonts'] = True
matplotlib.rcParams['text.usetex'] = True
{% endhighlight %}

Found an alternative solution online:

{% highlight python %}
matplotlib.rcParams['pdf.fonttype'] = 42
matplotlib.rcParams['ps.fonttype'] = 42
{% endhighlight %}
