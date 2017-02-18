# Introduction

This serves to be the definitive template for all articles I will be writing. In addition to providing a template, I also resolve a way to work with a large project which might span several .tex or .Rnw files, and the issue of combining and managing these files to produce the final document. 

# Problem statement

For large LaTeX documents, it is advisable to split the document into smaller parts, e.g. one .tex file for each section. This is to avoid the .tex file becoming overly complex and unwieldy due to the sheer volume of text. Therefore, there must be one parent .tex document which collates all of the child .tex documents which would then be typeset into the desired PDF file. A few problems arise:

1. Each child .tex file ideally should be able to be compiled individually. This way, one avoids compiling all of the other child documents while working on a specific section of the document, which could be very slow.
2. When R code is involved, a pre-processing is required to convert the R code (in the .Rnw files) to .tex, and then proceed as usual to typset it into PDF. The source document is actually the .Rnw file, and not the .tex file, which is seen only to be an intermediary towards the final PDF file. Need a way so that all child .Rnw and .tex files play well with each other.
3. The issue with .Rnw conversion to .pdf is that the .synctex file gets messed up, because the .synctex is traditionally supposed to point the PDF to the .tex file. As mentioned, the source is actually the .Rnw file.
4. When collaborators or reviewers are involved, they may not be comfortable with working with .Rnw files, or possibly would like to work on one single .tex file. Either way, would like a tool to visually markup changes or comments made across versions.

# Weaving .Rnw files using `knitr`

.Rnw files are LaTeX files which contain "code chunks" to be evaluated (by R). The .Rnw files are pre-processed by R to evaluate all the code chunks and return the results (or plot figures, which is useful!) in a process called "weaving". The end-product of this pre-processing is a .tex file. The .tex file is then converted into PDF in the normal way.

There are two engines to weave R code: 1) Sweave and 2) `knitr`. We will be using `knitr`, as it is more flexible (lots of options) and also able to cache the results. Caching means that the R output is saved so when the .Rnw file is compiled again, the R commands need not be re-run, so this is very useful when your R code takes a long time to run. 

The main integrated development environment (IDE) that works out-of-the-box to compile .Rnw files with `knitr` is Rstudio. While this is probably sufficient for small projects or little editing work on .Rnw files, you'd really like an IDE which is more suited towards LaTeX. This is because you can leverage features such as autocompletion, referencing, easy citation, and so on. I should mention now that RStudio has a guide to deal with multiple .Rnw files in your project (but not .tex files) - see [here](https://support.rstudio.com/hc/en-us/articles/200486298-Working-with-Multiple-Rnw-Files).
