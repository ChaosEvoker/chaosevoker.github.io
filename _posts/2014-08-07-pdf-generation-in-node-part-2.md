---
layout: post
title:  "PDF Generation in Node - Part 2"
date:   2014-07-16 15:10:49
comments: true
categories: programming
image:
  feature: abstract-10.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

PDF Generation in Node - Part 2: Rendering the PDF with a Headless Browser

If you missed part 1 of this series, you can find it [here](http://chaosevoker.github.io/programming/2014/07/16/pdf-generation-in-node-part-1.html).

As a reminder, here is how I will ultimately rate the solutions (on a scale of 1 to 5):
<ol>
    <li><strong>Power</strong> - The level of control you have over what the resulting PDF looks like</li>
    <li><strong>Efficiency</strong> - How many resources the strategy consumes</li>
    <li><strong>Ease of Use</strong> - How easy the solution is to implement</li>
    <li><strong>Flexibility</strong> - How easy it is to alter the appearance of the PDF (for example due to a marketing redesign)</li>
</ol>

Alright, no one like recaps right? Enough of that, let's get to...

## Using a Headless Browser

For the examples in this discussion I will be using [NodePDF](https://github.com/TJkrusinski/NodePDF). This node module uses [PhantomJS](http://phantomjs.org/) in the background. It is very possible to just use Phantom directly. However, the syntax for doing so is a lot harder to grok at a first look. So, I'm going to use NodePDF to keep things simple and clear.

For those who may not be familiar with the term, headless browsers are pieces of software that behave like a web browser, but have no GUI (this is why they are "headless"). They have whole suites of functions that allow you to interact with a webpage programmatically. NodePDF will abstract most of the handling of our headless browser away from us, so that we can focus on the PDF, but it'll be important to understand the concept of headless browsing when discussing the strengths and weaknesses of this approach.

So, clearly drawing the PDF directly is a powerful, but low-level solution to PDF generation. What if we wanted something that had less maintenance and code overhead? We'd also prefer to have a solution which utilizes web technologies that we are familiar with. Fortunately, headless browsers provide that solution. We can point a headless browser at a web page and then render it as a PDF file. This allows us to use HTML and CSS to stylize a document and then simply convert it to a PDF all automatically. Since we are using HTML, this also lets us use whatever templating engine we'd like (e.g. Jade) to make our lives easier as well.

Here's how to generate a PDF using NodePDF and PhantomJS:

{% highlight js %}
var NodePDF = require('nodepdf');

var document = new NodePDF('/path/to/HTML/file.html', '/path/to/output/PDF/file.pdf', {
    viewportSize: {
        width: 1440,
        height: 900
    },
    paperSize: {
        pageFormat: 'Letter',
        margin: {
            top: '1.5in',
            left: '1in',
            right: '1in',
            bottom: '1in'
        }
    },
    zoomFactor: 1.0
});

document.on('error', function (error) {
    console.log(error);
});

document.on('done', function (path_to_pdf) {
    console.log(path_to_pdf);
});
{% endhighlight %}

You may notice that this is a similar syntax to Node's HTTP and child process modules. There are also events for the stdout and stderr streams. The options available are the options that PhantomJS provided for its render function, and there are many more than are shown here. The options are fairly extensive and cover a wide range of things. For this example, I've just used a relatively simple baseline.

Fundamentally, we are opening an HTML file in a browser, taking a picture of it, and saving that picture as a PDF. Simple, clean, and easy. However, this solution comes with a significant drawback that may not be clear at first.

When we put a line of text into the PDF document using the direct drawing technique, raw text was placed into the document. But, if we did the same thing with the headless browser technique, then we are putting a *picture of text* into the document. This is a massive jump in space demands. For many applications, this won't matter. For a single page PDF the differences will likely not be as important. However, for the application I was working on, the sizes of the documents could range from a couple of pages to hundreds of pages. Clients wanted their reports delivered by email, but some of the larger reports exceeded the attachment limit! I attempted a variety of hack-like workarounds, but I couldn't bring the size down enough, and this solution proved unworkable as a result.

So, this solution had a fatal weakness for me, but it could very easily be just what you are looking for. Here's how this solution rates:

<ol>
    <li><strong>Power</strong> - 4</li>
    <li><strong>Efficiency</strong> - 1</li>
    <li><strong>Ease of Use</strong> - 4</li>
    <li><strong>Flexibility</strong> - 5</li>
</ol>

You don't lose too much power with this solution, as you can do a lot with what is possible in modern browsers. You gain a ton in how easy this solution is to implement. It doesn't get much easier than this, though there are still a couple of gotchas with using Phantom that can provide a few frustrating moments. However, the real loss here is in the size of the document. These PDFs will be relatively enormous, which can make it unuseable for certain applications (like mine).

I was not ready to give up on using simple HTML templates, however. Designing and tweaking those templates had felt like heaven compared to tweaking the direct drawing technique. Undaunted, I continued searching for an alternative. I delved into the depths of Google until I found a new path. A third solution...

That I will discuss next time!
