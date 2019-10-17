---
title: "Building my First Shiny App"
comments: yes
date: '2019-08-03'
tags:
- r
- visualization
- shiny
- interactive
category: R
---



<div class="chunk" id="unnamed-chunk-1"><div class="rcode"><div class="source"><pre class="knitr r"><span class="hl kwd">library</span><span class="hl std">(ggplot2)</span>
<span class="hl kwd">library</span><span class="hl std">(ggthemes)</span>
<span class="hl kwd">library</span><span class="hl std">(dplyr)</span>
<span class="hl kwd">library</span><span class="hl std">(ggrepel)</span>
<span class="hl kwd">library</span><span class="hl std">(tools)</span>
<span class="hl kwd">library</span><span class="hl std">(readxl)</span>
<span class="hl kwd">library</span><span class="hl std">(tidyverse)</span>
<span class="hl kwd">library</span><span class="hl std">(knitr)</span>

<span class="hl kwd">options</span><span class="hl std">(</span><span class="hl kwc">scipen</span><span class="hl std">=</span><span class="hl num">999</span><span class="hl std">)</span>
<span class="hl kwd">theme_set</span><span class="hl std">(</span><span class="hl kwd">theme_minimal</span><span class="hl std">())</span>

<span class="hl std">education</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">read_excel</span><span class="hl std">(</span><span class="hl str">&quot;education.xlsx&quot;</span><span class="hl std">,</span> <span class="hl kwc">skip</span><span class="hl std">=</span><span class="hl num">1</span><span class="hl std">)</span>
<span class="hl std">salary</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">read_excel</span><span class="hl std">(</span><span class="hl str">&quot;national_M2017_dl.xlsx&quot;</span><span class="hl std">)</span>
<span class="hl std">automation</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">read_excel</span><span class="hl std">(</span><span class="hl str">&quot;raw_state_automation_data.xlsx&quot;</span><span class="hl std">)</span>

<span class="hl std">salary1</span> <span class="hl kwb">&lt;-</span> <span class="hl std">salary</span> <span class="hl opt">%&gt;%</span>
<span class="hl kwd">group_by</span><span class="hl std">(OCC_TITLE)</span> <span class="hl opt">%&gt;%</span>
<span class="hl kwd">mutate</span><span class="hl std">(</span><span class="hl kwc">natlwage</span> <span class="hl std">= TOT_EMP</span> <span class="hl opt">*</span> <span class="hl kwd">as.numeric</span><span class="hl std">(A_MEAN))</span> <span class="hl opt">%&gt;%</span>
<span class="hl kwd">filter</span><span class="hl std">(</span><span class="hl opt">!</span><span class="hl kwd">is.na</span><span class="hl std">(TOT_EMP))</span> <span class="hl opt">%&gt;%</span>
<span class="hl kwd">filter</span><span class="hl std">(</span><span class="hl opt">!</span><span class="hl kwd">is.na</span><span class="hl std">(A_MEAN))</span> <span class="hl opt">%&gt;%</span>
<span class="hl kwd">filter</span><span class="hl std">(</span><span class="hl opt">!</span><span class="hl kwd">is.na</span><span class="hl std">(A_MEDIAN))</span>

<span class="hl std">salary1</span><span class="hl opt">$</span><span class="hl std">A_MEDIAN</span> <span class="hl kwb">=</span> <span class="hl kwd">as.numeric</span><span class="hl std">(</span><span class="hl kwd">as.character</span><span class="hl std">(salary1</span><span class="hl opt">$</span><span class="hl std">A_MEDIAN))</span>
<span class="hl std">salary2</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">select</span><span class="hl std">(salary1, OCC_TITLE, TOT_EMP, A_MEDIAN, natlwage)</span> <span class="hl opt">%&gt;%</span>
<span class="hl kwd">distinct</span><span class="hl std">()</span>

<span class="hl kwd">library</span><span class="hl std">(plyr)</span>
<span class="hl std">education1</span> <span class="hl kwb">&lt;-</span> <span class="hl std">education</span> <span class="hl opt">%&gt;%</span> <span class="hl kwd">select</span><span class="hl std">(</span><span class="hl opt">-</span><span class="hl std">...2)</span>

<span class="hl std">education1</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">rename</span><span class="hl std">(education1,</span> <span class="hl kwd">c</span><span class="hl std">(</span><span class="hl str">&quot;2016 National Employment Matrix title and code&quot;</span> <span class="hl std">=</span> <span class="hl str">&quot;occupation&quot;</span><span class="hl std">,</span>
                                   <span class="hl str">&quot;Less than high school diploma&quot;</span> <span class="hl std">=</span> <span class="hl str">&quot;lessthanhs&quot;</span><span class="hl std">,</span>
                                   <span class="hl str">&quot;High school diploma or equivalent&quot;</span> <span class="hl std">=</span> <span class="hl str">&quot;hsdiploma&quot;</span><span class="hl std">,</span>
                                   <span class="hl str">&quot;Some college, no degree&quot;</span> <span class="hl std">=</span> <span class="hl str">&quot;somecollege&quot;</span><span class="hl std">,</span>
                                   <span class="hl str">&quot;Associate's degree&quot;</span> <span class="hl std">=</span> <span class="hl str">&quot;associates&quot;</span><span class="hl std">,</span>
                                   <span class="hl str">&quot;Bachelor's degree&quot;</span> <span class="hl std">=</span> <span class="hl str">&quot;bachelors&quot;</span><span class="hl std">,</span>
                                   <span class="hl str">&quot;Master's degree&quot;</span> <span class="hl std">=</span> <span class="hl str">&quot;masters&quot;</span><span class="hl std">,</span>
                                   <span class="hl str">&quot;Doctoral or professional degree&quot;</span> <span class="hl std">=</span> <span class="hl str">&quot;professional&quot;</span><span class="hl std">))</span>

<span class="hl std">education2</span> <span class="hl kwb">&lt;-</span> <span class="hl std">education1</span> <span class="hl opt">%&gt;%</span>
  <span class="hl kwd">group_by</span><span class="hl std">(occupation)</span> <span class="hl opt">%&gt;%</span>
  <span class="hl kwd">mutate</span><span class="hl std">(</span><span class="hl kwc">hsorless</span> <span class="hl std">= lessthanhs</span> <span class="hl opt">+</span> <span class="hl std">hsdiploma,</span>
         <span class="hl kwc">somecollegeorassociates</span> <span class="hl std">= somecollege</span> <span class="hl opt">+</span> <span class="hl std">associates,</span>
         <span class="hl kwc">postgrad</span> <span class="hl std">= masters</span> <span class="hl opt">+</span> <span class="hl std">professional)</span>

<span class="hl std">education2</span> <span class="hl kwb">&lt;-</span> <span class="hl std">education2</span> <span class="hl opt">%&gt;%</span> <span class="hl kwd">drop_na</span><span class="hl std">()</span>

<span class="hl std">salary2</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">rename</span><span class="hl std">(salary2,</span> <span class="hl kwd">c</span><span class="hl std">(</span><span class="hl str">&quot;OCC_TITLE&quot;</span> <span class="hl std">=</span> <span class="hl str">&quot;occupation&quot;</span><span class="hl std">))</span>
<span class="hl std">salary2</span><span class="hl opt">$</span><span class="hl std">occupation</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">tolower</span><span class="hl std">(salary2</span><span class="hl opt">$</span><span class="hl std">occupation)</span>
<span class="hl std">education2</span><span class="hl opt">$</span><span class="hl std">occupation</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">tolower</span><span class="hl std">(education2</span><span class="hl opt">$</span><span class="hl std">occupation)</span>
<span class="hl std">edsal</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">merge</span><span class="hl std">(</span><span class="hl kwd">as.data.frame</span><span class="hl std">(education2),</span> <span class="hl kwd">as.data.frame</span><span class="hl std">(salary2),</span> <span class="hl kwc">by</span><span class="hl std">=</span><span class="hl str">&quot;occupation&quot;</span><span class="hl std">)</span> <span class="hl opt">%&gt;%</span> <span class="hl kwd">drop_na</span><span class="hl std">()</span>

  <span class="hl std">typicaleducation</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">read_excel</span><span class="hl std">(</span><span class="hl str">&quot;typicaleducation.xlsx&quot;</span><span class="hl std">)</span>
  <span class="hl std">typicaleducation2</span> <span class="hl kwb">&lt;-</span> <span class="hl std">typicaleducation</span> <span class="hl opt">%&gt;%</span> <span class="hl kwd">select</span><span class="hl std">(occupation,typicaled,workexp)</span>
  <span class="hl std">typicaleducation2</span> <span class="hl kwb">&lt;-</span> <span class="hl std">typicaleducation2</span> <span class="hl opt">%&gt;%</span> <span class="hl kwd">drop_na</span><span class="hl std">()</span>
  <span class="hl std">typicaleducation2</span><span class="hl opt">$</span><span class="hl std">occupation</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">tolower</span><span class="hl std">(typicaleducation2</span><span class="hl opt">$</span><span class="hl std">occupation)</span>
  <span class="hl std">edsal2</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">merge</span><span class="hl std">(</span><span class="hl kwd">as.data.frame</span><span class="hl std">(edsal),</span> <span class="hl kwd">as.data.frame</span><span class="hl std">(typicaleducation2),</span> <span class="hl kwc">by</span><span class="hl std">=</span><span class="hl str">&quot;occupation&quot;</span><span class="hl std">)</span>

  <span class="hl kwd">detach</span><span class="hl std">(package</span><span class="hl opt">:</span><span class="hl std">plyr)</span>
  <span class="hl std">edsal3</span> <span class="hl kwb">&lt;-</span> <span class="hl std">edsal2</span> <span class="hl opt">%&gt;%</span>
  <span class="hl kwd">group_by</span><span class="hl std">(typicaled)</span> <span class="hl opt">%&gt;%</span>
  <span class="hl kwd">summarise</span><span class="hl std">(</span><span class="hl kwc">medianwage</span> <span class="hl std">=</span> <span class="hl kwd">mean</span><span class="hl std">(A_MEDIAN))</span>

  <span class="hl std">automationwstates</span> <span class="hl kwb">&lt;-</span> <span class="hl std">automation</span> <span class="hl opt">%&gt;%</span> <span class="hl kwd">select</span><span class="hl std">(</span><span class="hl opt">-</span><span class="hl std">soc)</span>
  <span class="hl std">automation1</span> <span class="hl kwb">&lt;-</span> <span class="hl std">automationwstates</span> <span class="hl opt">%&gt;%</span> <span class="hl kwd">select</span><span class="hl std">(occupation,probability,total)</span>

  <span class="hl std">automation1</span><span class="hl opt">$</span><span class="hl std">occupation</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">str_replace_all</span><span class="hl std">(automation1</span><span class="hl opt">$</span><span class="hl std">occupation,</span> <span class="hl str">&quot;;&quot;</span><span class="hl std">,</span> <span class="hl str">&quot;,&quot;</span><span class="hl std">)</span>
  <span class="hl std">automation1</span><span class="hl opt">$</span><span class="hl std">occupation</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">tolower</span><span class="hl std">(automation</span><span class="hl opt">$</span><span class="hl std">occupation)</span>
  <span class="hl std">data</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">merge</span><span class="hl std">(</span><span class="hl kwd">as.data.frame</span><span class="hl std">(edsal2),</span> <span class="hl kwd">as.data.frame</span><span class="hl std">(automation1),</span> <span class="hl kwc">by</span><span class="hl std">=</span><span class="hl str">&quot;occupation&quot;</span><span class="hl std">)</span>

  <span class="hl std">data</span><span class="hl opt">$</span><span class="hl std">occupation</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">toTitleCase</span><span class="hl std">(data</span><span class="hl opt">$</span><span class="hl std">occupation)</span>
</pre></div>
</div></div>




<div class="chunk" id="unnamed-chunk-3"><div class="rcode"><div class="source"><pre class="knitr r"><span class="hl com"># shinyApp(ui = ui, server = server)</span>
</pre></div>
</div></div>

<div class="chunk" id="unnamed-chunk-4"><div class="rcode"><div class="source"><pre class="knitr r"><span class="hl std">knitr</span><span class="hl opt">::</span><span class="hl kwd">include_app</span><span class="hl std">(</span><span class="hl str">&quot;https://connorrothschild.shinyapps.io/ggvis/&quot;</span><span class="hl std">,</span>
  <span class="hl kwc">height</span> <span class="hl std">=</span> <span class="hl str">&quot;600px&quot;</span><span class="hl std">)</span>
</pre></div>
<div class="error"><pre class="knitr r">## Error in file(con, &quot;rb&quot;): cannot open the connection
</pre></div>
</div></div>

