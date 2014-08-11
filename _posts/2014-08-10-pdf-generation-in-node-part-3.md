---
layout: post
title:  "PDF Generation in Node - Part 3"
date:   2014-08-10
comments: true
categories: programming
image:
  feature: abstract-10.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

PDF Generation in Node - Part 3: Rendering the PDF with an HTML to PDF Converter

Here are the previous posts on the topic:

[Part 1 - Drawing the PDF Directly](http://chaosevoker.github.io/programming/2014/07/16/pdf-generation-in-node-part-1.html).

[Part 2 - Using a Headless Browser](http://chaosevoker.github.io/programming/2014/07/16/pdf-generation-in-node-part-2.html).

As a reminder, here is how I will ultimately rate the solutions (on a scale of 1 to 5):
<ol>
    <li><strong>Power</strong> - The level of control you have over what the resulting PDF looks like</li>
    <li><strong>Efficiency</strong> - How many resources the strategy consumes</li>
    <li><strong>Ease of Use</strong> - How easy the solution is to implement</li>
    <li><strong>Flexibility</strong> - How easy it is to alter the appearance of the PDF (for example due to a marketing redesign)</li>
</ol>

Everyone ready for some shameless self promotion? Great! Let's get started...

## Using a HTML to PDF Converter

For the javascript examples in this discussion I'll be using [html-to-pdf](https://github.com/ChaosEvoker/html-to-pdf). html-to-pdf is a Node module which uses a Java HTML to PDF conversion library called [Flyingsaucer](https://github.com/flyingsaucerproject/flyingsaucer) in the background. Full disclosure: I wrote html-to-pdf (the Node module parts anyway - Flyingsaucer was written by other developers).
There are also a couple of pieces of Java code showing the use of [Flyingsaucer](https://github.com/flyingsaucerproject/flyingsaucer) and [JTidy](http://jtidy.sourceforge.net/).

So, I had struggled with the past couple of solutions for generating PDFs in Node. Drawing directly worked fine, but had a huge development and maintenance overhead. Headless browsers were much easier, but generated files that were far too large for my purposes. Eventually, through a great deal of Google patience, I stumbled on Flyingsaucer. Flyingsaucer is a Java library that reads XML and CSS transforms it into a PDF. Flyingsaucer has a few weaknesses though:
<ul>
    <li>It is designed as an XML to PDF converter, which means it needs to be fed strict XHTML.</li>
    <li>The CSS renderer is custom built and somewhat dated, meaning CSS has to conform to CSS2 standards.</li>
    <li>It's in Java so we have to do some shenanigans to get it working in Node.</li>
</ul>

These were some hurdles, but I had some sweet Jade templates ready to go and wasn't going to give up. So here's a simple Java program using Flyingsaucer:

{% highlight java %}
//Import a bunch of relevant Core Java things here (snipped for conciseness)
import org.xhtmlrenderer.pdf.ITextRenderer;

public class PDFRenderer {
    public static void main(String[] args) throws Exception {

        //Setup the inputs and outputs for the PDF rendering
    	String url = new File("path/to/html/file.html").toURI().toURL().toString();
        OutputStream outputPDF = new FileOutputStream(pdfFilePath);

        //Create the renderer and point it to the HTML document
        ITextRenderer renderer = new ITextRenderer();
        renderer.setDocument(url);

        //Render the PDF document
        renderer.layout();
        renderer.createPDF(outputPDF);

        //Close the stream
        os.close();
        outputPDF.close();
    }
}
{% endhighlight %}

Now this works fine if your HTML is already strictly XHTML. But, since the files I was generating were based on dynamic data, I was using Jade templates and rendering them into HTML. This meant I couldn't be sure I would get XHTML, so I needed to clean up the HTML before handing it off to Flyingsaucer. Fortuantely, there is an HTML syntax library called [JTidy](http://jtidy.sourceforge.net/) which has a Tidy class that can clean up HTML into XHTML. Adding that took a little more code:

{% highlight java %}
//Import a bunch of relevant Core Java things here (snipped for conciseness)
import org.xhtmlrenderer.pdf.ITextRenderer;
import org.w3c.tidy.Tidy;

public class PDFRenderer {
    public static void main(String[] args) throws Exception {

        //Set up input file and output file for cleaning up the HTML
        InputStream inputFileStream = new FileInputStream("path/to/html/file.html");
        String cleanHTMLFile = "temp.html";
        OutputStream cleanedFileStream = new FileOutputStream(cleanHTMLFile);

        //Clean the HTML
    	Tidy htmlCleaner = new Tidy();
    	htmlCleaner.setXHTML(true);
    	htmlCleaner.parse(inputFileStream, cleanedFileStream);

        //Setup the inputs and outputs for the PDF rendering
    	String url = new File(cleanHTMLFile).toURI().toURL().toString();
        OutputStream outputPDF = new FileOutputStream(pdfFilePath);

        //Create the renderer and point it to the XHTML document
        ITextRenderer renderer = new ITextRenderer();
        renderer.setDocument(url);

        //Render the PDF document
        renderer.layout();
        renderer.createPDF(outputPDF);

        //Close the streams (and don't cross them!)
        cleanedFileStream.close();
        outputPDF.close();

		//Clean up the temp file
		File tempFile = new File("temp.html");
		tempFile.delete();
    }
}
{% endhighlight %}

This did the trick quite well. The only thing left was too get it into my Node server. After a small bit of tweaking on the Java code to get it to accept command line arguments instead of using hard coded paths, all I had to do was use a child process in Node to launch the executable JAR of the Java program:

{% highlight js %}
var child_process = require('child_process');

//Again for conciseness, let's save the tedium of constructing the arguments
var renderer = child_process.spawn('java', args);

renderer.on('error', function (error) {
    console.log(error);
});

renderer.on('exit', function (code) {
    console.log(code);
});
{% endhighlight %}

And that did the trick! However, I was unsatisfied with the overhead of dealing with all of the child process handling, so I wrapped it into a module. This ended with the following syntax:

{% highlight js %}
var htmlToPdf = require('html-to-pdf');

htmlToPdf.convertHTMLFile('path/to/source.html', 'path/to/destination.pdf',
    function (error, success) {
        if (error) {
            console.log('Oh noes! Errorz!');
            console.log(error);
        } else {
            console.log('Woot! Success!');
            console.log(success);
        }
    }
);
{% endhighlight %}

The Flyingsaucer converter puts actual text into the resulting document, so the sizes were similar to sizes obtained from direct drawing, but with the convenience of using HTML templates. However this approach has it's own set of drawbacks. There was nothing that could be done about the old version of CSS. As a result, looking at your HTML in the browser won't always give you an accurate representation of how the PDF will look. It also requires you to have Java installed, which usually isn't a big deal, but it can be relevant. I think the rating for html-to-pdf breaks down like this:

<ol>
    <li><strong>Power</strong> - 3</li>
    <li><strong>Efficiency</strong> - 4</li>
    <li><strong>Ease of Use</strong> - 5</li>
    <li><strong>Flexibility</strong> - 5</li>
</ol>

My own solution comes with a tradeoff as well. With the restriction to CSS2, you lose a bit of power in what you can accomplish, requiring you to jump through hoops at times to get the effect you are looking for. Still, html-to-pdf has a very simple and easy syntax and it renders the document very efficiently space-wise. I think each solution in this series has its own advantages and disadvantages. No one solution will be right for every application. Hopefully, this series will have helped you find whichever solution is right for you!

Let me know what you think of the solutions and my ratings of them in the comments, and thanks for reading!
