# Discussions

## Repurpose `Choice=Question?-answers` in different lists #195

I don't have the know-how to do so myself, so I can't do a pull request for this. Instead, I am opening an Issue. I am sorry.

lintalist can do a lot, and it is awesome in many ways, some of which I haven't had the need to explore yet. But there is one feature that, to my knowledge, doesn't exist - or I don't know how to make it work properly.

Take the following template:

```ahk
[[Choice=!|Email Person one|email Person two|email person three]]
Hello [[Choice=!|Person 1| Person 2|Person3]]    
...
Sincerely,
me
...
```


Right now, I have to choose the same surname twice to first choose the name I want to address, then choose the corresponding email address.

Is there a way to merge these two into one question, perhaps by creating two arrays from those choices, then index into the second array with the chosen index of the first one? In that way, you could have an indirect mapping even though the key to that array would be a different one. I don't know how lintalist works under the hood sadly, so I can't really point to where this would be implemented. I hope I got the idea across well enough.


### lintalist

I think you can try this: use [Split](https://lintalist.github.io/#Split) and Choice, whereby you use a special character (here $) to split the selected *choice* into two parts so you can use one element of it via `[[sp=1]]` - I'm using a "named" split here but if you only have one you can omit the "\_email" bit from it as well.

```ahk
[[Split_Email=[[Choice=!|John$john@domain1|Betty$bwhite@domain2|email person three$mailaddres@domain]]|$]]
Hello [[SP_Email=1]]
...
Sincerely,
me
...

Email address: [[SP_Email=2]]
```


You could do something similar with `[[var]]` or `[[snippet]]` where you define local variables. Say you have a local variable `John` so if your Choice is `John` you can use `[[var=John]]` to use a "related bit of information" so to speak. Might be tedious to maintain of course if you have many, the split is probably relatively easy - if it is something that needs to be edited often, you can store that info in a text file and just read that info via `[[file=]]` as in `[[Split_Email=[[Choice=!|[[file=data.txt]]]]|$]]`

### Poster

thank you, this does work wonders.
However, I have indeed recently come to the point where I suddenly have to assume several edits per month, which makes this tedious to keep up by hand. How would such a file need to be structured (containing names and addresses) in order to be processed correctly by lintalist?
I have most names combined in an excel file so far, so using that excel file would be best.

### lintalist

AutoHotkey can read Excel files directly but as you can edit CSV directly in Excel too that might be easiest.
(If you must work with Excel directly look for excel examples posted by Fanatic Guru on the AutoHotkey forum and you'll figure it out, doesn't look to complicated but never tried it myself)

So this is your CSV file:

```
"John, Jr",john@domain1
Betty,bwhite@domain2
email person three,mailaddres@domain
```


We need to pre-process it to make it suitable for `[[Choice]]`, so add this to `plugings\MyFunctions.ahk` and restart Lintalist[1]

```ahk
; add this to plugings\MyFunctions.ahk and restart
CSVData(file,char="$")
    {
     global ParseEscapedArray
     FileRead, data, %file%
     Loop, parse, data, `n, `r
        {
         line:=A_LoopField
         Loop, parse, line, CSV
          {
           insert:=A_LoopField
           insert:=StrReplace(insert,"[",ParseEscapedArray[1]) ; escape [ <SB
           insert:=StrReplace(insert,"]",ParseEscapedArray[2]) ; escape ] >SB
           insert:=StrReplace(insert,"|",ParseEscapedArray[3]) ; escape | ^SB
           output .= insert char
          }
         output .= "|"
        }
     Return Trim(output,"|")
    }
```

Update the snippet to this (change file name/path to data.csv of course, it assumes it is in the lintalist folder):

```
[[Split_Email=[[Choice=!|[[CSVData("data.csv")]]]]|$]]
Hello [[SP_Email=1]]
...
Sincerely,
me
...

Email address: [[SP_Email=2]]
```

If you use `A_Tab` in function and `\t` in the snippet instead of the `$` the choice might look a bit better.

[1] See https://lintalist.github.io/#Functions

Edit: more robust `csvdata()`



## Choosing and pasting one of ~20 multi-line pastes using Split- and Choice-commands 

I am in need of creating about twenty static rmd templates, depending on output type and some specificities. These snippets are multiline-pastes, and the easier way would be to just create them all separate and just select by search query.

However, my first idea was to use the `split` and `choice` plugins respectively to do this, similar to your suggestion in https://github.com/lintalist/lintalist/issues/194#issuecomment-845510219

In total, this mess isn't working, but I haven't figured out why:

```autohotkey
[[Split_rmdstyle=[[Choice=!html$---
title: \vspace{3.5in}^|
output:
  html_document:
    fig_caption: true
    number_sections: true
    toc: true
    theme: united
---

\newpage <!--linebreak to newpage-->
#\tableofcontents <!--this is not relevant for html output-->
<!--\listoffigures--> <!--adds a list of figures at this position-->
<!--\listoftables--> <!--adds a list of tables at this position-->
\newpage
<!--Body of the pdf document here-->|pdf$---
title: \vspace{3.5in}^|
author: "Name"
date: "`r Sys.Date()`"
output:
   pdf_document:
      fig_caption: true
      number_sections: true
---

\newpage <!--linebreak to newpage-->
\tableofcontents 
<!--\listoffigures--> <!--adds a list of figures at this position-->
<!--\listoftables--> <!--adds a list of tables at this position-->
\newpage
<!--Body of the pdf document here-->]]]]

[[SP_rmdstyle=2]]
```

This being way too messy to debug, I instead went ahead and tried the following simple test:

```
[[Split_rmd=[[Choice=!|pdf|hello`n´tworkd|html|hello`n´tworkd|word|hello`n´tworkd]]]]
[[SP_rmd=1]]
```
...which also just leaves faulty outputs, such as 

```
`nh 
```

(the ``` `n ``` is there cuz newlines are stripped by markdown at beginning/end of code blocks. A newline and one letter is pasted respectively.)

In the end, I would need a filtered choice command that allows me to paste one of X multi-line snippets, using one shorthand to find them and then select. I hope this makes any kind of sense. Prepare the snippets once and let then never touch them again. There are more snippets than just the ones above, but those are the only ones I can share as examples, without having to cut sensitive parts.



### Lintalist
1. Create a snippet for each template. Give each snippet a unique shorthand which is also descriptive e.g. rmdtemp_tableofcontents (doesn't matter, you wont have to type these shorthands, just use them to search/select)
2. if snippets share common parts, save that common part into a snippet as well and include them where appropriate in the template (snippet) via `[[snippet=rmd_header]]` 
3. Now your search snippet is a simple snippet+choice `[[Snippet=[[choice=!|rmdtemplate1|rmdtemplate2|rmdtemplate3]]]]`
      timeline-comment-group
      discussion-comment
      js-discussion-comment
      js-minimizable-comment-group
      js-targetable-element
      TimelineItem-body
      discussion-nested-comment-group position-relative
      mt-0 pl-2
One small question, because I am sure there is a simpler way than doing a janky split jank. 
In the resulting choice-dialogue, I would like to have a sensible entry, such as `Header - Word` and `Header - PDF`, instead of the shorthand-entries `rmd_Header_HTML`, `rmd_Header_Word` and so on, while still referring to the `rmd_Header_HTML`-snippet and so on. 

As I use the shorthand often when searching for bundles instead of using it as an actual _shorthand_\*1, these sub-snippets should get an arbitrarily complicated shorthand to restrict them from showing up on searches - which is limited by the fact that the shorthand is printed in the `choice`-plugin. Also, it doesn't look neat, and I _like_ tidy implementations. I can get by without, but I'd like to know if this is possible.


***

\*1: (too many similar ones by system, but visually different, so going over the gui is just faster, and shorthands simply don't work well with the way-too-useful `selected`-plugin)

   
### Lintalist

You can use the same method as [shown here](https://github.com/lintalist/lintalist/discussions/195#discussioncomment-767945) - note there is a TAB character between the description and shorthand. (In the editor you can press <kbd>ctrl</kbd>+<kbd>tab</kbd> to insert a TAB character - or edit the snippet in your regular text editor)

```ahk
[[split=[[choice=!Template|Header - Word	rmd_Header_Word|Header - PDF	rmd_Header_PDF|Header - Excel	rmd_Header_Excel]]|\t]]
[[snippet=[[sp=2]]]]
```
This is how it should look in the `choice`  plugin gui, neatly lined up (depending on how long the descriptions are)[1]

```ahk
Header - Word	rmd_Header_Word
Header - PDF	rmd_Header_PDF
Header - Excel	rmd_Header_Excel
```

*[1] This did gave me the idea to introduce a `last` item from a `split` [#207](https://github.com/lintalist/lintalist/issues/207) - so you could even "hide" the second part entirely by adding 10 tab characters for example*
