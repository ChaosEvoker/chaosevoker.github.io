---
layout: post
title:  "PDF Generation in Node - Part 1"
date:   2014-07-16 15:10:49
categories: node js programming technology
image:
  feature: abstract-10.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

I've recently been neck-deep in refactoring a project. I recently got to the automated report generation code in the project, and it made me all nostalgic. That's the feeling where you need to suddenly run to the bathroom right?

I've tried quite a variety of strategies in creating PDF reports on a Node server automatically. Each has a set of strengths and weaknesses, and there isn't a single perfect solution. I've broken down the types of solutions into 3 distinct categories. I will be using a particular technology for each of the examples, but you should know that each category has a variety of technologies/libraries/etc. within it. There are subtleties between each of them, but for now I am just going to go over the larger categories and the major differences between them. Because everyone likes it when stuff is broken down into numbers, I'll also be rating the categories on a scale of 1 to 5 on the following fields:
1. **Power** - The level of control you have over what the resulting PDF looks like
2. **Efficiency** - How many resources the strategy consumes
3. **Ease of Use** - How easy the solution is to implement
4. **Flexibility** - How easy it is to alter the appearance of the PDF (for example due to a marketing redesign)

With that, let's get to the categories!

## Draw the PDF Directly

For the examples in this section I'll be using [PDFKit](http://pdfkit.org/).

Drawing the PDF is by far the most powerful solution. However, with great power comes great amounts of code:
```js
var PDFDocument = require('pdfkit');
var document = new PDFDocument;
document.font('Times-Roman')
    .fontSize(20)
    .text('Hello World!', 100, 100);
```

Now obviously, this is not a "great amount" of code. Heck, I sort of made it seem larger with all those extra new lines and indentations!

If you program with a library like this for any significant amount of time, you'll likely end up writing in a similar way. PDFKit's API is very chainable, which significantly cuts down on lines of code, but does result in some very long lines. Styling your code like this will really help future readability.

Let's add a bit of complexity:
```js
document.rect(100, 20, 100, 100)
    .fill('gray');
document.font('Times-Roman')
    .fontSize(12)
    .fillColor('white')
    .text('White text on a gray background!', 110, 60);
```
This added complexity is making another potential problem with direct drawing apparent. There are a lot of magic numbers. It can be very hard to keep all of your hardcoded numbers organized. It is possible to keep everything together without using magic numbers, but it is a time consuming development effort.

The real drawback of this solution, however, is the amount of effort it takes to change the document's appearance. In my case, marketing came back with a design once that they wanted to try for a report we were generating. It was so different, we were forced to tell them that we would be (more or less) starting over! Given that the visual design of things often changes at a rapid pace, this is obviously a large drawback.

Now, this overview has been pretty negative so far. I won't lie and say that this is my prefered solution. It isn't. But, this solution is extremely powerful. If you want to draw some crazy mathematically calculated fractal and put it in a PDF, you can do that without jumping through hoops. This solution trades conviencne for power, and it has power in spades.

The other benefit to this solution that will become more apparent when we loook at the next solution, is resource consumption. This is fast and disk efficient, and you are unlikely to run into any size barriers using this method.

**Final Rating**
* *Power* - 5
* *Efficiency* - 5
* *Ease of Use* - 2
* *Flexibility* - 1

Ultimately, drawing the PDF directly is a solution of extremes. If you need large amounts of precise control over the document's appearance and you need it to be efficiently sized, then directly drawing it will serve you well. However, it will take a much greater effort to develop and change the PDF (relative to other solutions).

So that took up way more text than I expected when I started writing this post. So, I'm going to break it up into several posts. Next time we will cover the world of PDF generation through headless browsers!
