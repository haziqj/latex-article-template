# Introduction

This serves to be the definitive template for all articles I will be writing. In addition to providing a template, I also resolve a way to work with a large project which might span several .tex or .Rnw files, and the issue of combining and managing these files to produce the final document. 

# TL;DR

I managed to automate a multi-file (both .tex and .Rnw) project, such that the main document will weave and compile the relevant .Rnw files if any changes are detected in the children or main document itself.

I describe a work-flow which you can emulate below. Feel free to use this template as well (either for your own projects or just to learn how multi-file .Rnw projects work).

Don't forget to install all required software.

# Prerequisites

I have only made tests on macOS 10.12.3 (Sierra). I suspect this will work on Unix systems, but for Windows it might require a little extra work. Make sure to install all of the following software:

1. R (I would also recommend Rstudio), and the following packages:
	- `knitr`
	- `patchDVI`
	- `iprior` (only if you intend to compile the files in my repo)
2. A TeX IDE which has the ability to write custom  scripts. Examples include Texpad (Mac), TeXStudio, Texmaker, TeXShop (Mac), TeXWorks and WinEdt. Also, make sure that the viewer has "sync with source" capability.
3. A current TeX distribution such as TeX Live, MikTeX or MacTeX (Mac). This should install the `latexdiff` Perl script (unless you're on Windows, I think).
4. Python, and then this script:
 	- [`flatex`](https://github.com/johnjosephhorton/flatex)

I hope I'm not missing anything.

# My workflow

[TBC]

# Problem statement

For large LaTeX documents, it is advisable to split the document into smaller parts, e.g. one .tex file for each section. This is to avoid the .tex file becoming overly complex and unwieldy due to the sheer volume of text. Therefore, there must be one parent .tex document which collates all of the child .tex documents which would then be typeset into the desired PDF file. A few problems arise:

1. Each child .tex file ideally should be able to be compiled individually. This way, one avoids compiling all of the other child documents while working on a specific section of the document, which could be very slow.
2. When R code is involved, a pre-processing is required to convert the R code (in the .Rnw files) to .tex, and then proceed as usual to typset it into PDF. The source document is actually the .Rnw file, and not the .tex file, which is seen only to be an intermediary towards the final PDF file. Need a way so that all child .Rnw and .tex files play well with each other.
3. The issue with .Rnw conversion to .pdf is that the .synctex file gets messed up, because the .synctex is traditionally supposed to point the PDF to the .tex file. As mentioned, the source is actually the .Rnw file.
4. When collaborators or reviewers are involved, they may not be comfortable with working with .Rnw files, or possibly would like to work on one single .tex file. Either way, would like a tool to visually markup changes or comments made across versions.

# The `standalone` LaTeX package

One convention to splitting the .tex document into smaller parts, is to literally just shift the text of each chapter, say, into separate .tex (child) documents. These .tex files are linked to parent document via `\input`. Normally, one can work on the individual child .tex files, but they are not compilable, in the sense that they are not a complete "standalone" .tex document that is able to be compiled (because they're just texts from where it would otherwise be in the main parent .tex document!). What you would then have to do is actually compile the whole document just to view a change in the child document, or alternatively never compile it and just "eye it" from the source editor. Either way is unsatisfactory.

To overcome this, we can use the LaTeX `standalone` package. In the main parent document, load the package: `\usepackage{standalone}`. The main document is structured as usual, i.e. begins with a `\documentclass{article}` or whatever, followed by the body of text in between `\begin{document}` and `\end{document}`.

The child documents are actually a `standalone` document-class, i.e. it begins with `\documentclass{standalone}`, followed by the body of text in between `\begin{document}` and `\end{document}`. The child document can be compiled individually, so you can work and make edits to this document alone without having to compile everything else in the parent document.

The child documents are linked to the parent document also using `\input`, but because the `standalone` package is used, there is no issue with each child document having their own document-class and also a `\begin{document}` and `\end{document}`.

To make things neater, you should collate all of the LaTeX packages to be used, and any LaTeX macros that are to be defined into a separate .sty file. In this GitHub example, my .sty file is called `haziq_article.sty`. In it, you will see the list of packages and also some mathematical notations that I often use. All .tex files then include `\usepackage{haziq_article}`, instead of repeating all of the macros and packages list.

I prefer to have all files in one flat directory, and use a `01-intro.tex`, `02-body.tex`, etc. naming system for the files. In fact, I would encourage this (for reasons to be explained later in the .Rnw knitting part and `latexdiff` part). However, if you prefer to have them in separate folders that is fine too. To make this easier, use the `import` package, so that you can use the `\import` or `\subimport` functions which are able to take relative directories, instead of absolute directories. What I mean is that, since the .sty file will be in the parent directory, then you would call this file via `../haziq_article.sty` from one of the upper child directories. 

# Weaving .Rnw files using `knitr`

.Rnw files are LaTeX files which contain "code chunks" to be evaluated (by R). The .Rnw files are pre-processed by R to evaluate all the code chunks and return the results (or plot figures, which is useful!) in a process called "weaving". The end-product of this pre-processing is a .tex file. The .tex file is then converted into PDF in the normal way.

There are two engines to weave R code: 1) Sweave and 2) `knitr`. We will be using `knitr`, as it is more flexible (lots of options) and also able to cache the results. Caching means that the R output is saved so when the .Rnw file is compiled again, the R commands need not be re-run, so this is very useful when your R code takes a long time to run. 

The main integrated development environment (IDE) that works out-of-the-box to compile .Rnw files with `knitr` is Rstudio. While this is probably sufficient for small projects or little editing work on .Rnw files, you'd really like an IDE which is more suited towards LaTeX. This is because you can leverage features such as autocompletion, referencing, easy citation, and so on. I should mention now that RStudio has a guide to deal with multiple .Rnw files in your project (but not .tex files) - see [here](https://support.rstudio.com/hc/en-us/articles/200486298-Working-with-Multiple-Rnw-Files).

Luckily, there are a few TeX editors that are able to **knit** .Rnw files. It is likely that your TeX editor supports knitting .Rnw files using `knitr` as well. See [this](https://yihui.name/knitr/demo/editors/) link for information on how to set this up.

# Patching the .synctex file

In most TeX IDEs, the viewer allows you to "go to source" from the PDF file, and the other way around as well - "go to PDF" from the source. Sometimes, this is implemented using `cmd/ctrl + click`. It is a very useful feature to be able to jump from the PDF and source like this and pinpoint where exactly in your source document you need to make the edits. This ability is provided by the .synctex file, which is an auxilliary file created during the typesetting process from .tex to .pdf. 

When .Rnw files are involved, the .synctex becomes useless because it points to the .tex file instead of the .Rnw file which is the actual source. As a result, the whole convenience of PDF/source syncing breaks down.

There is a way around this, and that is to use the R package called `patchDVI`. There are two main functions which we will be using in this package: they are 1) `SweavePDF()` and 2) `patchSynctex()`. 

Even though the name implies using Sweave, there is an option for the `knitr` engine to be used instead. The `SweavePDF()` function takes in the .Rnw file as an argument, then runs it through `knitr` (via `knitr::knit2pdf()`), and finally converts it to PDF using the `pdflatex` engine (or some other latex engine). Along the way, the .synctex is also "patched", and thus we are able to go to the .Rnw source from the PDF and vice versa.

# Using a custom build script in a TeX editor

Unless you are using Rstudio, your PDF viewer most likely won't be able to go to source due to the .synctex issue described above. What we need is to write a custom build script to tell the TeX editor exactly how to deal with .Rnw files. 

In your TeX IDE, find a way to write custom build scripts. Then, instead of simply knitting the .Rnw file via 

```
R -e "knitr::knit2pdf('%.Rnw')"
```

where `%` is the name of the .Rnw file to be knitted (this is texstudio specific, different IDEs handle the name variable differently), we run the `SweavePDF()` function instead, as follows:

```
R -e "patchDVI::SweavePDF('%.Rnw', weave = knitr::knit, envir = globalenv())"
```

Basically, this replaces the `knit2pdf()` function provided by `knitr` with `SweavePDF()` from the `patchDVI` package. Additionally, we provide the argument `weave = knitr::knit` to indicate that we would like to use the `knitr` engine. The `envir = globalenv()` tells `SweavePDF()` to look for some specific and important variables in the R global environment when running the function. This will be explained later on. The output of this build is a PDF file with a patched .synctex, unlike if it was run through `knit2pdf()`.

One crucial requirement for this to work is to include the following chunk code at the top of the .Rnw file:

```r
<<include = FALSE, cache = FALSE>>=
patchDVI::useknitr()
@
```

As a side note, this is equivalent to using the LaTeX package `Sweave` and calling `\SweaveOpts{Concordance=TRUE}` if the weaving engine is Sweave. This is not required when using `knitr`, but instead, include the above code chunk.

# Bibliographies and references

A seasoned TeX user knows that one needs to run `bibtex` with a couple more `pdflatex` runs to fully get the bibliographies and references linked within the document. Similarly, the PDF output of the `SweavePDF()` won't have linked bibliographies and references if running for the first time. 

`Latexmk` is a popular Perl script which runs LaTeX the correct number of times to resolve cross-references, bibliographies, etc. We should add to our custom build script this `latexmk` function as well, like so

```
latexmk -pdf -interaction=nonstopmode --synctex=-1 "%.tex"
```

The function is essentially `latexmk -pdf` to get a PDF output, with options `-interaction=nonstopmode` just to tell the TeX engine to run with minimal interaction from the user and as far as possible to "go past" any errors. Also important here is the option `--synctex=-1` to produce the uncompressed .synctex file. However, this means that the patched .synctex file from the `SweavePDF()` output will be messed up again! 

In my testing, setting `--synctex=0` does not resolve the issue.

# Patch the .synctex... again

Using another function from the `patchDVI` package, we can correct the .synctex after it was handled by `latexmk`. The function is simply

```
R -e "patchDVI::patchSynctex('%.synctex')"
```

Finally, we have a sync-able PDF file after putting these three lines in our custom build script.

# A multi-file project

Just to reiterate the annoyance of not having a patched .synctex file: If we were to click on "go to source" in the viewer, it would point to the .tex file and not the .Rnw file. Unfortunately, the .tex file looks very similar to the .Rnw file and one would be inclined to make the mistake of editing the .tex file instead. When the file is woven again, changes will be lost because the edits were not made at the source!

With the patched .synctex, all is well again. But multi-file projects pose a challenge, especially when you want to "make" the entire document after making a change in a child .Rnw file. Note that you would only link the .tex files to the parent file, and not the .Rnw files because they are linked via `\input`, which is a LaTeX command and won't be able to process the .Rnw files directly.

Imagine having several .Rnw child files, and perhaps some .tex child files as well (maybe not all chapters have R code). As these are all linked to the parent file via `\input`, we would want to automate the following process to keep everything up-to-date when a change is made on one of the child.Rnw files:

1. Convert `child.Rnw` to `child.tex`.
2. Convert any other changed .Rnw files (e.g. subfiles or grandchildren of `child.Rnw`).
3. Convert the parent file to PDF.

The `patchDVI` package again has a neat project management feature to automate this. The requirement is that the parent file *must* be a .Rnw file, and *not a .tex file*. All of the .Rnw files are linked to each other, such that a change in one triggers the all of the relevant .Rnw files to compile again. This is achieved by adding special chunk codes at the top of each .Rnw file. 

### The `child.Rnw` file

```r
<<include = FALSE, cache = FALSE>>=
.TexRoot <- "main.tex"
@
```

### The `main.Rnw` file

```r
<<include = FALSE, cache = FALSE>>=
.SweaveFiles <- "child.Rnw"
@
```

## What it does

The `patchDVI` has a function called `SweaveAll()` which is able to Sweave/knit multiple .Rnw files. This function can run automatically after `SweavePDF()` is called, but it needs to look for four specific variables in the R global environment. Three of these are desrcibed below.

1. `.TexRoot` : A character vector of length one with the name of the main .tex file.
2. `.SweaveFiles` : A character vector with the names of the .Rnw files to knit.
3. `.SweaveMake` : `= 1` (default) only knit if changes are detected in the .Rnw files defined in `.SweaveFiles`; `=0` then `.SweaveFiles` is ignored and runs only the explicitly called .Rnw file; and finally `=2` forces all files in `.SweaveFiles` to be knit.

For more information, visit the CRAN website for the [package vignette](https://cran.r-project.org/web/packages/patchDVI/vignettes/patchDVI.pdf). 

# Track changes with Git and `latexdiff`

Git is an excellent tool for version control of software, code and even documents including LaTeX files. In addition to being able to track changes, Git can also allow you to see what was changed with each commit. But Git only allows you to see code diff.

To view the diff of the formatted output, you can use the `latexdiff` utility, a Perl script which is available from CTAN (and included in Unix TeX distributions such as MikTeX and TeXLive). The command to do this (in shell)

```
latexdiff old.tex new.tex > diff.tex
```

`latexdiff` then compares the old version with the new version to track what has changed in the source code, and then creates a new `diff.tex` with the markup of these changes. This `diff.tex` is then compiled as usual to produce `diff.pdf`. Look at this thread at [SO](http://stackoverflow.com/questions/6188780/git-latex-workflow).

I've basically written an executable shell script to look for `main.tex` and `main_old.tex` in the parent directory and produce a `main_diff.tex` output. This was intended to be run on the main parent .tex file only. However, because of the `\input` calls to the child .tex files, any changes in the children files will not be tracked. Thus, it is necessary to flatten the document first.

# Flatten the document

The Python script `flatex.py` over at [this](https://github.com/johnjosephhorton/flatex) GitHub repo does exactly what we need. It searches the `main.tex` file for any `\input` and copies the text from the children document in the parent document (replacing the `\input` call), resulting in a one continous, long, flattened document. 

The `latexdiff` actually has a `--flatten` command, but it is more advantageous to do these steps separately. Besides the fact that it might be slow and you might encounter problems with a one swoop `latexdiff --flatten`, you also get the flattened .tex file this way, which your collaborators are fully comfortable working with (instead of the individual .Rnw files). They can make changes or give comments in this document and you can mark it up using `latexdiff`. 

Again, I've written an executable shell script to call `flatex.py` on the `main.tex` file in the directory. 

Note that most flatten scripts I've looked for are not compatible with `\import` (they only search and replace `\input`). This is a reason to use `\input` instead of `\import` in this work-flow. The results are virtually the same, regardless.

# Final thoughts

1. Put all children files and parent files in the same directory, i.e. do not use subdirectories. The reason is that the `SweaveAll()` function is mainly run relative to the directory from which you call the .Rnw file. For instance, running `SweaveAll()` on `main.Rnw` in the root directory, and then needing to knit the children files in their respective subdirectories will cause the processed `child.tex` file to appear in the root directory instead (because it was called by `main.Rnw`).

2. I have had a few hit and miss moments with the ability to sync between PDF and source in the multi-file case. Sometimes, undesirably, the viewer points to the .tex file instead of the .Rnw source. This can happen if the .synctex did not get patched properly. I posit that this can occur if the parent document gets knitted, but the children do not - this results in the .synctex not patching correctly for the children. To overcome this, again, sometimes it works, somtimes it doesn't - is to set `.SweaveMake <- 2` at the top of each .Rnw file, and knit the documents at least once. This should settle things. 

You might think this is a waste of time because all .Rnw files need to be knitted this way, but not necessarily, for two reasons. One, `knitr` allows to cache the results, so knitting should be quite fast. Second, you would actually want to knit everything when are near the end of your project, i.e. making minor adjustments here and there and see how it affects the whole document. Individually, knitting and .synctex is absolutely fine, and we discussed that we can do this using the `standalone` package.

3. A couple of notes on the `standalone` package if using the `flatex` and `latexdiff` scripts. Since each of the children documents are standalones, and have their own `\documentclass` and `\begin` and `\end`, this obviously will result in compilation error when the file is flattened. One way around this is to manually comment out the front and end matters before flattening. This is a small price to pay, I think.

4. The other thing about the `standalone` class is you might have a few citations problem. If you include `\bibliography{file.bib}` at the end in each standalone .tex files, then in the parent document you might see a lot of References section croppint up. To resolve this, either 1) manually comment out the bib command; 2) use `\nobibliography{file.bib}` instead - this skips printing the references section but citations are still valid; or 3) use the '\ifstandalone ... \fi' macro to turn on bibliography only when in standalone mode.


Copyright (C) 2016  Haziq Jamil
