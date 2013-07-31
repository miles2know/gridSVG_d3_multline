---
title: gridSVG, d3, and slidify
author: Timely Portfolio
github: {user: timelyportfolio, repo: gridSVG_d3_multline, branch: "gh-pages"}
framework: minimal
mode: selfcontained
highlighter : highlight.js
hitheme     : tomorrow
assets:
  jshead:
  - "http://d3js.org/d3.v3.js"
---

<script src = "http://d3js.org/d3.v3.js" meta-charset = "utf-8"></script>

<a href="https://github.com/timelyportfolio/gridSVG_d3_multline"><img style="position: absolute; top: 0; right: 0; border: 0;" src="https://s3.amazonaws.com/github/ribbons/forkme_right_darkblue_121621.png" alt="Fork me on GitHub"></a>
  



## FRED, Give Us Some Treasury Yield Data


```r
require(quantmod)
require(rCharts)
#now get the US bonds from FRED
USbondssymbols <- paste0("DGS",c(1,2,3,5,7,10,20,30))

ust.xts <- xts()
for (i in 1:length( USbondssymbols ) ) {
  ust.xts <- merge( 
    ust.xts,
    getSymbols( 
      USbondssymbols[i], auto.assign = FALSE,src = "FRED"
    )
  )
}
```

```
## Warning: downloaded length 228127 != reported length 200 Warning:
## downloaded length 164750 != reported length 200 Warning: downloaded length
## 228226 != reported length 200 Warning: downloaded length 228331 !=
## reported length 200 Warning: downloaded length 195485 != reported length
## 200 Warning: downloaded length 228504 != reported length 200 Warning:
## downloaded length 87300 != reported length 200 Warning: downloaded length
## 158996 != reported length 200
```

```r
xtsMelt <- function(data) {
  require(reshape2)
  
  #translate xts to time series to json with date and data
  #for this behavior will be more generic than the original
  #data will not be transformed, so template.rmd will be changed to reflect
  
  
  #convert to data frame
  data.df <- data.frame(
    cbind(format(index(data),"%Y-%m-%d"),coredata(data))
  )
  colnames(data.df)[1] = "date"
  data.melt <- melt(data.df,id.vars=1,stringsAsFactors=FALSE)
  colnames(data.melt) <- c("date","indexname","value")
  #remove periods from indexnames to prevent javascript confusion
  #these . usually come from spaces in the colnames when melted
  data.melt[,"indexname"] <- apply(
    matrix(data.melt[,"indexname"]),
    2,gsub,pattern="[.]",replacement=""
  )
  return(data.melt)
  #return(df2json(na.omit(data.melt)))
}

ust.melt <- na.omit( xtsMelt( ust.xts["2012::",] ) )

ust.melt$date <- as.Date(ust.melt$date)
ust.melt$value <- as.numeric(ust.melt$value)
ust.melt$indexname <- factor(
  ust.melt$indexname, levels = colnames(ust.xts)
)
ust.melt$maturity <- as.numeric(
  substr(
    ust.melt$indexname, 4, length( ust.melt$indexname ) - 4
  )
)
ust.melt$country <- rep( "US", nrow( ust.melt ))
```


## Draw A Graph with Lattice


```r
require(latticeExtra)

#set up height and width
#x11( width = 10, height = 6 )

p1 <- xyplot(
  value ~ date | indexname,
  groups = indexname,
  data =  ust.melt,
  type = "l",
  scales = list(
    x = list(
      at = pretty(ust.melt$date,n=3)[c(1,3)],
      format = "%b %Y"
    )),
  layout = c(8,1)
)
p1

require(rjson)
require(gridSVG)

#set up a function to pull the groups
#which lattice calls subscripts
#use with lapply
#l is the 
getgroup <- function(l,p) {
  return(p$panel.args.common$groups[l$subscripts])
}

#set up a function to get the data and add groups
#in one list so we can pass as json
#accepts a trellis object as p
getlist <- function(p) {
  data <- p$panel.args
  
  #use the getgroup function to get groups
  #lattice places the groups in subscripts
  #in the panel.args.common list
  groups <- lapply(
    data,
    FUN=getgroup,
    p=p
  )
  
  for ( i in 1:length(data) ) {
    data[[i]]$groups <- groups[[i]]
  }
  
  names(data) <- unlist(p$condlevels)
  
  return(
    list(
      strips = p$condlevels,
      groups = unique(unlist(groups)),
      data = data
    )
  )
}

#export our lattice chart
exportlist <- grid.export("", addClasses = TRUE )
```

![plot of chunk unnamed-chunk-3](assets/fig/unnamed-chunk-31.png) ![plot of chunk unnamed-chunk-3](assets/fig/unnamed-chunk-32.png) ![plot of chunk unnamed-chunk-3](assets/fig/unnamed-chunk-33.png) 


<div>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="504px" height="504px" viewBox="0 0 504 504" version="1.1">
  <metadata xmlns:gridsvg="http://www.stat.auckland.ac.nz/~paul/R/gridSVG/">
    <gridsvg:generator name="gridSVG" version="1.3-0" time="2013-07-31 13:59:09"/>
    <gridsvg:argument name="name" value=""/>
    <gridsvg:argument name="exportCoords" value="none"/>
    <gridsvg:argument name="exportMappings" value="none"/>
    <gridsvg:argument name="exportJS" value="none"/>
    <gridsvg:argument name="res" value="72"/>
    <gridsvg:argument name="prefix" value=""/>
    <gridsvg:argument name="addClasses" value="TRUE"/>
    <gridsvg:argument name="indent" value="TRUE"/>
    <gridsvg:argument name="htmlWrapper" value="FALSE"/>
    <gridsvg:argument name="usePaths" value="vpPaths"/>
    <gridsvg:argument name="uniqueNames" value="TRUE"/>
    <gridsvg:separator name="id.sep" value="."/>
    <gridsvg:separator name="gPath.sep" value="::"/>
    <gridsvg:separator name="vpPath.sep" value="::"/>
  </metadata>
  <g transform="translate(0, 504) scale(1, -1)">
    <g id="gridSVG" fill="none" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" font-size="12" font-family="Helvetica, Arial, FreeSans, Liberation Sans, Nimbus Sans L, sans-serif" opacity="1" stroke-linecap="round" stroke-linejoin="round" stroke-miterlimit="10" stroke-opacity="1" fill-opacity="0" font-weight="normal" font-style="normal">
      <g id="plot_01.background.1" class="rect grob gDesc">
        <rect id="plot_01.background.1.1" x="0" y="0" width="504" height="504" fill="none" stroke="none" stroke-opacity="0" fill-opacity="0"/>
      </g>
      <g id="plot_01.toplevel.vp.1" font-size="12" class="pushedvp viewport">
        <g id="plot_01.toplevel.vp::plot_01.xlab.vp.1" class="pushedvp viewport">
          <g id="plot_01.xlab.1" class="text grob gDesc">
            <g id="plot_01.xlab.1.1" transform="translate(259.09, 16.31)" stroke-width="0.1">
              <g id="plot_01.xlab.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.xlab.1.1.text" text-anchor="middle" opacity="1" stroke="rgb(0,0,0)" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.xlab.1.1.tspan.1" dy="4.31" x="0">date</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.ylab.vp.1" class="pushedvp viewport">
          <g id="plot_01.ylab.1" class="text grob gDesc">
            <g id="plot_01.ylab.1.1" transform="translate(10.31, 249.11)" stroke-width="0.1">
              <g id="plot_01.ylab.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ylab.1.1.text" transform="rotate(-90)" text-anchor="middle" opacity="1" stroke="rgb(0,0,0)" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ylab.1.1.tspan.1" dy="4.31" x="0">value</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.figure.vp.1" class="pushedvp viewport"/>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.1.1.vp.1.clipPath">
            <rect x="43.18" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.1.1.vp.1" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.1.1.vp.1.clipPath)" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.strip.1.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.top.panel.1.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.top.panel.1.1.1.1" points="61.45,461.81 61.45,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.top.panel.1.1.1.2" points="91.61,461.81 91.61,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.left.1.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.left.panel.1.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.left.panel.1.1.1.1" points="43.18,65.44 37.51,65.44" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.left.panel.1.1.1.2" points="43.18,162.62 37.51,162.62" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.left.panel.1.1.1.3" points="43.18,259.8 37.51,259.8" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.left.panel.1.1.1.4" points="43.18,356.98 37.51,356.98" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticklabels.left.panel.1.1.1" class="text grob gDesc">
            <g id="plot_01.ticklabels.left.panel.1.1.1.1" transform="translate(31.85, 65.44)" stroke-width="0.1">
              <g id="plot_01.ticklabels.left.panel.1.1.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.left.panel.1.1.1.1.text" text-anchor="end" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.left.panel.1.1.1.1.tspan.1" dy="3.59" x="0">0</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.left.panel.1.1.1.2" transform="translate(31.85, 162.62)" stroke-width="0.1">
              <g id="plot_01.ticklabels.left.panel.1.1.1.2.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.left.panel.1.1.1.2.text" text-anchor="end" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.left.panel.1.1.1.2.tspan.1" dy="3.59" x="0">1</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.left.panel.1.1.1.3" transform="translate(31.85, 259.8)" stroke-width="0.1">
              <g id="plot_01.ticklabels.left.panel.1.1.1.3.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.left.panel.1.1.1.3.text" text-anchor="end" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.left.panel.1.1.1.3.tspan.1" dy="3.59" x="0">2</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.left.panel.1.1.1.4" transform="translate(31.85, 356.98)" stroke-width="0.1">
              <g id="plot_01.ticklabels.left.panel.1.1.1.4.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.left.panel.1.1.1.4.text" text-anchor="end" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.left.panel.1.1.1.4.tspan.1" dy="3.59" x="0">3</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.panel.1.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.bottom.panel.1.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.bottom.panel.1.1.1.1" points="61.45,50.8 61.45,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.bottom.panel.1.1.1.2" points="91.61,50.8 91.61,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticklabels.bottom.panel.1.1.1" class="text grob gDesc">
            <g id="plot_01.ticklabels.bottom.panel.1.1.1.1" transform="translate(61.45, 39.47)" stroke-width="0.1">
              <g id="plot_01.ticklabels.bottom.panel.1.1.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.bottom.panel.1.1.1.1.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.bottom.panel.1.1.1.1.tspan.1" dy="7.18" x="0">Jul 2012</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.bottom.panel.1.1.1.2" transform="translate(91.61, 39.47)" stroke-width="0.1">
              <g id="plot_01.ticklabels.bottom.panel.1.1.1.2.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.bottom.panel.1.1.1.2.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.bottom.panel.1.1.1.2.tspan.1" dy="7.18" x="0">Jul 2013</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.1.1.vp.2.clipPath">
            <rect x="43.18" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.1.1.vp.2" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.1.1.vp.2.clipPath)" class="pushedvp viewport">
          <g id="plot_01.xyplot.lines.group.1.panel.1.1.1" class="lines grob gDesc">
            <polyline id="plot_01.xyplot.lines.group.1.panel.1.1.1.1" points="46.5,77.1 46.58,77.1 46.66,76.13 46.75,77.1 46.99,76.13 47.08,76.13 47.16,76.13 47.24,76.13 47.32,75.16 47.66,76.13 47.74,76.13 47.82,76.13 47.9,76.13 48.15,77.1 48.23,77.1 48.32,77.1 48.4,77.1 48.48,77.1 48.73,77.1 48.81,78.07 48.89,78.07 48.98,79.04 49.06,79.04 49.31,79.04 49.39,79.04 49.47,80.02 49.56,80.02 49.64,80.02 49.89,80.02 49.97,82.93 50.05,82.93 50.13,81.96 50.22,82.93 50.55,81.96 50.63,81.96 50.71,81.96 50.79,82.93 51.04,81.96 51.13,82.93 51.21,82.93 51.29,82.93 51.37,81.96 51.62,81.96 51.7,81.96 51.79,82.93 51.87,82.93 51.95,82.93 52.2,82.93 52.28,84.87 52.36,85.85 52.45,85.85 52.53,85.85 52.78,85.85 52.86,86.82 52.94,85.85 53.03,83.9 53.11,83.9 53.36,83.9 53.44,82.93 53.52,82.93 53.6,82.93 53.69,83.9 53.93,82.93 54.02,84.87 54.1,83.9 54.18,83.9 54.27,83.9 54.51,83.9 54.6,83.9 54.68,82.93 54.76,82.93 54.84,81.96 55.09,82.93 55.17,82.93 55.26,82.93 55.34,81.96 55.42,82.93 55.67,81.96 55.75,82.93 55.84,82.93 55.92,82.93 56,83.9 56.25,84.87 56.33,83.9 56.41,82.93 56.5,83.9 56.58,82.93 56.83,82.93 56.91,82.93 56.99,82.93 57.07,82.93 57.16,82.93 57.41,83.9 57.49,83.9 57.57,84.87 57.65,84.87 57.74,84.87 57.98,85.85 58.07,85.85 58.15,84.87 58.23,85.85 58.31,84.87 58.64,84.87 58.73,83.9 58.81,82.93 58.89,81.96 59.14,82.93 59.22,82.93 59.31,82.93 59.39,82.93 59.47,83.9 59.72,82.93 59.8,83.9 59.88,82.93 59.97,82.93 60.05,82.93 60.3,82.93 60.38,82.93 60.46,84.87 60.55,83.9 60.63,83.9 60.88,83.9 60.96,85.85 61.04,85.85 61.12,86.82 61.21,85.85 61.45,85.85 61.54,85.85 61.7,83.9 61.78,84.87 62.03,84.87 62.12,84.87 62.2,84.87 62.28,84.87 62.36,84.87 62.61,82.93 62.69,82.93 62.78,82.93 62.86,81.96 62.94,81.96 63.19,81.96 63.27,82.93 63.35,81.96 63.44,82.93 63.52,81.96 63.77,82.93 63.85,80.99 63.93,81.96 64.02,81.96 64.1,80.99 64.35,80.99 64.43,83.9 64.51,83.9 64.59,84.87 64.68,82.93 64.92,83.9 65.01,83.9 65.09,83.9 65.17,84.87 65.26,84.87 65.5,83.9 65.59,84.87 65.67,83.9 65.75,83.9 65.83,83.9 66.08,82.93 66.16,82.93 66.25,82.93 66.33,81.96 66.41,80.99 66.74,80.99 66.83,81.96 66.91,82.93 66.99,82.93 67.24,82.93 67.32,82.93 67.4,82.93 67.49,81.96 67.57,82.93 67.82,82.93 67.9,82.93 67.98,82.93 68.06,82.93 68.15,82.93 68.4,82.93 68.48,82.93 68.56,81.96 68.64,80.99 68.73,81.96 68.97,81.96 69.06,80.99 69.14,80.99 69.22,82.93 69.3,82.93 69.63,82.93 69.72,82.93 69.8,82.93 69.88,82.93 70.13,83.9 70.21,82.93 70.3,82.93 70.38,82.93 70.46,82.93 70.71,83.9 70.79,82.93 70.87,82.93 70.96,83.9 71.04,83.9 71.29,82.93 71.45,82.93 71.54,82.93 71.62,83.9 71.87,83.9 71.95,83.9 72.03,82.93 72.11,84.87 72.2,82.93 72.53,82.93 72.61,82.93 72.69,81.96 72.77,80.99 73.02,80.99 73.11,80.99 73.19,81.96 73.35,83.9 73.6,81.96 73.68,82.93 73.77,82.93 73.85,82.93 73.93,82.93 74.18,82.93 74.26,82.93 74.34,82.93 74.43,82.93 74.51,82.93 74.76,82.93 74.84,80.99 74.92,79.04 75.01,79.04 75.09,78.07 75.34,78.07 75.42,80.99 75.5,80.02 75.58,80.02 75.67,80.02 75.91,80.99 76.08,80.99 76.16,80.02 76.25,80.02 76.49,80.99 76.66,80.02 76.74,80.02 76.82,80.02 77.07,80.02 77.15,79.04 77.24,78.07 77.32,79.04 77.4,79.04 77.65,79.04 77.73,79.04 77.82,79.04 77.9,79.04 77.98,79.04 78.31,79.04 78.39,80.02 78.48,80.02 78.56,80.02 78.81,80.99 78.89,80.02 78.97,80.02 79.05,80.02 79.14,80.02 79.39,80.02 79.47,80.02 79.55,80.02 79.63,80.02 79.72,79.04 79.96,80.02 80.05,79.04 80.13,80.02 80.21,80.99 80.29,81.96 80.62,81.96 80.71,81.96 80.79,80.99 80.87,80.99 81.12,80.99 81.2,81.96 81.29,81.96 81.37,81.96 81.45,80.99 81.7,80.99 81.78,80.02 81.86,80.02 81.95,80.02 82.03,80.02 82.28,80.02 82.36,80.02 82.44,80.02 82.53,80.02 82.61,79.04 82.86,80.02 82.94,80.02 83.02,80.02 83.1,79.04 83.19,79.04 83.43,79.04 83.52,79.04 83.6,79.04 83.68,79.04 84.01,79.04 84.1,79.04 84.18,78.07 84.26,78.07 84.34,78.07 84.59,78.07 84.67,78.07 84.76,77.1 84.84,77.1 84.92,76.13 85.17,77.1 85.25,78.07 85.33,78.07 85.42,77.1 85.5,77.1 85.75,77.1 85.83,77.1 85.91,78.07 86,77.1 86.08,77.1 86.33,77.1 86.41,76.13 86.49,76.13 86.57,76.13 86.66,76.13 86.9,76.13 86.99,75.16 87.07,76.13 87.15,76.13 87.24,76.13 87.48,78.07 87.57,77.1 87.65,77.1 87.73,77.1 87.81,77.1 88.06,77.1 88.14,77.1 88.23,76.13 88.31,77.1 88.39,77.1 88.72,78.07 88.81,79.04 88.89,78.07 88.97,79.04 89.22,79.04 89.3,79.04 89.38,79.04 89.47,79.04 89.55,79.04 89.8,79.04 89.88,79.04 89.96,79.04 90.04,79.04 90.13,78.07 90.38,78.07 90.46,78.07 90.54,78.07 90.62,79.04 90.71,78.07 90.95,80.99 91.04,81.96 91.12,80.99 91.2,80.02 91.28,80.02 91.53,80.02 91.61,79.04 91.7,79.04 91.86,80.02 92.11,79.04 92.19,79.04 92.28,78.07 92.36,78.07 92.44,77.1 92.69,76.13 92.77,75.16 92.85,76.13 92.94,76.13 93.02,76.13 93.27,75.16 93.35,77.1 93.43,77.1 93.52,77.1 93.6,76.13 93.85,76.13" stroke-dasharray="none" stroke="rgb(0,128,255)" stroke-width="0.75" opacity="1" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.panel.1.1.off.vp.2" class="pushedvp viewport">
          <g id="plot_01.border.panel.1.1.1" class="rect grob gDesc">
            <rect id="plot_01.border.panel.1.1.1.1" x="43.18" y="50.8" width="53.98" height="396.61" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.1.1.vp.1" class="pushedvp viewport">
          <defs>
            <clipPath id="plot_01.toplevel.vp::plot_01.strip.1.1.vp::plot_01.strip.default.1.1.clipPath">
              <rect x="43.18" y="447.41" width="53.98" height="14.4" fill="none" stroke="none"/>
            </clipPath>
          </defs>
          <g id="plot_01.toplevel.vp::plot_01.strip.1.1.vp::plot_01.strip.default.1.1" clip-path="url(#plot_01.toplevel.vp::plot_01.strip.1.1.vp::plot_01.strip.default.1.1.clipPath)" class="pushedvp viewport">
            <g id="plot_01.bg.strip.1.1.1" class="rect grob gDesc">
              <rect id="plot_01.bg.strip.1.1.1.1" x="43.18" y="447.41" width="53.98" height="14.4" fill="rgb(255,229,204)" stroke="rgb(255,229,204)" stroke-opacity="1" fill-opacity="1"/>
            </g>
            <g id="plot_01.textr.strip.1.1.1" class="text grob gDesc">
              <g id="plot_01.textr.strip.1.1.1.1" transform="translate(53.83, 454.61)" stroke-width="0.1">
                <g id="plot_01.textr.strip.1.1.1.1.scale" transform="scale(1, -1)">
                  <text x="0" y="0" id="plot_01.textr.strip.1.1.1.1.text" text-anchor="start" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                    <tspan id="plot_01.textr.strip.1.1.1.1.tspan.1" dy="4.31" x="0">DGS1</tspan>
                  </text>
                </g>
              </g>
            </g>
          </g>
          <g id="plot_01.toplevel.vp::plot_01.strip.1.1.vp::plot_01.strip.default.off.1.1" class="pushedvp viewport">
            <g id="plot_01.border.strip.1.1.1" class="rect grob gDesc">
              <rect id="plot_01.border.strip.1.1.1.1" x="43.18" y="447.41" width="53.98" height="14.4" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.2.1.vp.1.clipPath">
            <rect x="97.16" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.2.1.vp.1" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.2.1.vp.1.clipPath)" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.strip.2.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.top.panel.2.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.top.panel.2.1.1.1" points="115.43,461.81 115.43,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.top.panel.2.1.1.2" points="145.59,461.81 145.59,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticklabels.top.panel.2.1.1" class="text grob gDesc">
            <g id="plot_01.ticklabels.top.panel.2.1.1.1" transform="translate(115.43, 473.15)" stroke-width="0.1">
              <g id="plot_01.ticklabels.top.panel.2.1.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.top.panel.2.1.1.1.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.top.panel.2.1.1.1.tspan.1" dy="0" x="0">Jul 2012</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.top.panel.2.1.1.2" transform="translate(145.59, 473.15)" stroke-width="0.1">
              <g id="plot_01.ticklabels.top.panel.2.1.1.2.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.top.panel.2.1.1.2.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.top.panel.2.1.1.2.tspan.1" dy="0" x="0">Jul 2013</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.left.2.1.off.vp.1" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.panel.2.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.bottom.panel.2.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.bottom.panel.2.1.1.1" points="115.43,50.8 115.43,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.bottom.panel.2.1.1.2" points="145.59,50.8 145.59,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.2.1.vp.2.clipPath">
            <rect x="97.16" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.2.1.vp.2" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.2.1.vp.2.clipPath)" class="pushedvp viewport">
          <g id="plot_01.xyplot.lines.group.2.panel.2.1.1" class="lines grob gDesc">
            <polyline id="plot_01.xyplot.lines.group.2.panel.2.1.1.1" points="100.47,91.68 100.56,89.73 100.64,91.68 100.72,89.73 100.97,90.71 101.05,88.76 101.14,88.76 101.22,86.82 101.3,88.76 101.63,85.85 101.71,88.76 101.8,90.71 101.88,90.71 102.13,90.71 102.21,88.76 102.29,86.82 102.37,86.82 102.46,86.82 102.71,86.82 102.79,86.82 102.87,87.79 102.95,87.79 103.04,87.79 103.28,88.76 103.37,89.73 103.45,91.68 103.53,91.68 103.61,91.68 103.86,93.62 103.94,93.62 104.03,93.62 104.11,93.62 104.19,93.62 104.52,95.56 104.61,93.62 104.69,95.56 104.77,95.56 105.02,94.59 105.1,94.59 105.18,94.59 105.27,94.59 105.35,92.65 105.6,95.56 105.68,94.59 105.76,94.59 105.85,96.54 105.93,97.51 106.18,97.51 106.26,99.45 106.34,104.31 106.42,101.4 106.51,101.4 106.75,103.34 106.84,105.28 106.92,103.34 107,101.4 107.08,101.4 107.33,100.42 107.42,97.51 107.5,98.48 107.58,97.51 107.66,97.51 107.91,97.51 107.99,100.42 108.08,99.45 108.16,99.45 108.24,96.54 108.49,96.54 108.57,92.65 108.65,94.59 108.74,93.62 108.82,91.68 109.07,91.68 109.15,91.68 109.23,91.68 109.32,91.68 109.4,93.62 109.65,91.68 109.73,91.68 109.81,90.71 109.89,90.71 109.98,90.71 110.22,91.68 110.31,91.68 110.39,91.68 110.47,92.65 110.56,91.68 110.8,91.68 110.89,91.68 110.97,91.68 111.05,91.68 111.13,91.68 111.38,93.62 111.46,93.62 111.55,94.59 111.63,96.54 111.71,96.54 111.96,94.59 112.04,94.59 112.13,92.65 112.21,93.62 112.29,94.59 112.62,94.59 112.7,91.68 112.79,91.68 112.87,89.73 113.12,89.73 113.2,89.73 113.28,90.71 113.36,91.68 113.45,92.65 113.7,91.68 113.78,94.59 113.86,94.59 113.94,94.59 114.03,93.62 114.27,93.62 114.36,94.59 114.44,96.54 114.52,96.54 114.6,95.56 114.85,95.56 114.93,95.56 115.02,95.56 115.1,95.56 115.18,97.51 115.43,94.59 115.51,94.59 115.68,92.65 115.76,91.68 116.01,91.68 116.09,91.68 116.17,91.68 116.26,89.73 116.34,89.73 116.59,88.76 116.67,89.73 116.75,86.82 116.84,86.82 116.92,86.82 117.17,86.82 117.25,86.82 117.33,86.82 117.41,87.79 117.5,89.73 117.74,87.79 117.83,87.79 117.91,88.76 117.99,88.76 118.07,88.76 118.32,88.76 118.41,91.68 118.49,93.62 118.57,93.62 118.65,91.68 118.9,91.68 118.98,91.68 119.07,91.68 119.15,93.62 119.23,93.62 119.48,93.62 119.56,95.56 119.64,90.71 119.73,90.71 119.81,92.65 120.06,92.65 120.14,91.68 120.22,91.68 120.31,91.68 120.39,86.82 120.72,87.79 120.8,89.73 120.88,91.68 120.97,89.73 121.21,89.73 121.3,89.73 121.38,89.73 121.46,88.76 121.55,91.68 121.79,89.73 121.88,89.73 121.96,91.68 122.04,91.68 122.12,91.68 122.37,91.68 122.45,91.68 122.54,90.71 122.62,89.73 122.7,87.79 122.95,89.73 123.03,87.79 123.12,87.79 123.2,87.79 123.28,91.68 123.61,89.73 123.69,91.68 123.78,92.65 123.86,91.68 124.11,91.68 124.19,91.68 124.27,94.59 124.35,93.62 124.44,94.59 124.69,96.54 124.77,93.62 124.85,93.62 124.93,95.56 125.02,94.59 125.26,94.59 125.43,94.59 125.51,94.59 125.59,92.65 125.84,92.65 125.92,94.59 126.01,91.68 126.09,91.68 126.17,91.68 126.5,91.68 126.59,89.73 126.67,88.76 126.75,88.76 127,89.73 127.08,91.68 127.16,91.68 127.33,93.62 127.58,91.68 127.66,91.68 127.74,91.68 127.82,89.73 127.91,89.73 128.16,89.73 128.24,89.73 128.32,89.73 128.4,89.73 128.49,89.73 128.73,88.76 128.82,88.76 128.9,89.73 128.98,91.68 129.06,88.76 129.31,89.73 129.39,92.65 129.48,92.65 129.56,92.65 129.64,90.71 129.89,90.71 130.06,90.71 130.14,90.71 130.22,91.68 130.47,89.73 130.63,91.68 130.72,91.68 130.8,91.68 131.05,91.68 131.13,89.73 131.21,88.76 131.3,90.71 131.38,90.71 131.63,90.71 131.71,90.71 131.79,90.71 131.87,92.65 131.96,90.71 132.29,90.71 132.37,90.71 132.45,87.79 132.53,92.65 132.78,93.62 132.87,94.59 132.95,91.68 133.03,91.68 133.11,91.68 133.36,89.73 133.44,91.68 133.53,91.68 133.61,89.73 133.69,89.73 133.94,91.68 134.02,93.62 134.1,93.62 134.19,91.68 134.27,93.62 134.6,93.62 134.68,91.68 134.77,90.71 134.85,91.68 135.1,89.73 135.18,89.73 135.26,91.68 135.34,89.73 135.43,89.73 135.67,88.76 135.76,89.73 135.84,89.73 135.92,89.73 136.01,91.68 136.25,91.68 136.34,91.68 136.42,91.68 136.5,91.68 136.58,89.73 136.83,90.71 136.91,88.76 137,90.71 137.08,91.68 137.16,90.71 137.41,88.76 137.49,89.73 137.58,89.73 137.66,89.73 137.99,87.79 138.07,89.73 138.15,88.76 138.24,86.82 138.32,88.76 138.57,88.76 138.65,88.76 138.73,88.76 138.81,88.76 138.9,86.82 139.15,86.82 139.23,88.76 139.31,88.76 139.39,88.76 139.48,88.76 139.72,88.76 139.81,87.79 139.89,87.79 139.97,87.79 140.05,86.82 140.3,84.87 140.38,86.82 140.47,84.87 140.55,84.87 140.63,86.82 140.88,86.82 140.96,86.82 141.05,86.82 141.13,86.82 141.21,90.71 141.46,88.76 141.54,90.71 141.62,90.71 141.71,87.79 141.79,90.71 142.04,90.71 142.12,90.71 142.2,90.71 142.29,90.71 142.37,90.71 142.7,93.62 142.78,94.59 142.86,95.56 142.95,94.59 143.19,94.59 143.28,96.54 143.36,94.59 143.44,94.59 143.52,96.54 143.77,96.54 143.86,98.48 143.94,98.48 144.02,96.54 144.1,93.62 144.35,91.68 144.43,91.68 144.52,95.56 144.6,97.51 144.68,102.37 144.93,106.25 145.01,107.23 145.09,103.34 145.18,100.42 145.26,100.42 145.51,98.48 145.59,98.48 145.67,100.42 145.84,104.31 146.09,101.4 146.17,101.4 146.25,102.37 146.33,98.48 146.42,101.4 146.66,98.48 146.75,98.48 146.83,96.54 146.91,96.54 147,96.54 147.24,96.54 147.33,97.51 147.41,98.48 147.49,96.54 147.57,95.56 147.82,97.51" stroke-dasharray="none" stroke="rgb(255,0,255)" stroke-width="0.75" opacity="1" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.panel.2.1.off.vp.2" class="pushedvp viewport">
          <g id="plot_01.border.panel.2.1.1" class="rect grob gDesc">
            <rect id="plot_01.border.panel.2.1.1.1" x="97.16" y="50.8" width="53.98" height="396.61" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.2.1.vp.1" class="pushedvp viewport">
          <defs>
            <clipPath id="plot_01.toplevel.vp::plot_01.strip.2.1.vp::plot_01.strip.default.1.1.clipPath">
              <rect x="97.16" y="447.41" width="53.98" height="14.4" fill="none" stroke="none"/>
            </clipPath>
          </defs>
          <g id="plot_01.toplevel.vp::plot_01.strip.2.1.vp::plot_01.strip.default.1.1" clip-path="url(#plot_01.toplevel.vp::plot_01.strip.2.1.vp::plot_01.strip.default.1.1.clipPath)" class="pushedvp viewport">
            <g id="plot_01.bg.strip.2.1.1" class="rect grob gDesc">
              <rect id="plot_01.bg.strip.2.1.1.1" x="97.16" y="447.41" width="53.98" height="14.4" fill="rgb(255,229,204)" stroke="rgb(255,229,204)" stroke-opacity="1" fill-opacity="1"/>
            </g>
            <g id="plot_01.textr.strip.2.1.1" class="text grob gDesc">
              <g id="plot_01.textr.strip.2.1.1.1" transform="translate(107.81, 454.61)" stroke-width="0.1">
                <g id="plot_01.textr.strip.2.1.1.1.scale" transform="scale(1, -1)">
                  <text x="0" y="0" id="plot_01.textr.strip.2.1.1.1.text" text-anchor="start" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                    <tspan id="plot_01.textr.strip.2.1.1.1.tspan.1" dy="4.31" x="0">DGS2</tspan>
                  </text>
                </g>
              </g>
            </g>
          </g>
          <g id="plot_01.toplevel.vp::plot_01.strip.2.1.vp::plot_01.strip.default.off.1.1" class="pushedvp viewport">
            <g id="plot_01.border.strip.2.1.1" class="rect grob gDesc">
              <rect id="plot_01.border.strip.2.1.1.1" x="97.16" y="447.41" width="53.98" height="14.4" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.3.1.vp.1.clipPath">
            <rect x="151.14" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.3.1.vp.1" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.3.1.vp.1.clipPath)" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.strip.3.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.top.panel.3.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.top.panel.3.1.1.1" points="169.41,461.81 169.41,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.top.panel.3.1.1.2" points="199.57,461.81 199.57,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.left.3.1.off.vp.1" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.panel.3.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.bottom.panel.3.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.bottom.panel.3.1.1.1" points="169.41,50.8 169.41,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.bottom.panel.3.1.1.2" points="199.57,50.8 199.57,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticklabels.bottom.panel.3.1.1" class="text grob gDesc">
            <g id="plot_01.ticklabels.bottom.panel.3.1.1.1" transform="translate(169.41, 39.47)" stroke-width="0.1">
              <g id="plot_01.ticklabels.bottom.panel.3.1.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.bottom.panel.3.1.1.1.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.bottom.panel.3.1.1.1.tspan.1" dy="7.18" x="0">Jul 2012</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.bottom.panel.3.1.1.2" transform="translate(199.57, 39.47)" stroke-width="0.1">
              <g id="plot_01.ticklabels.bottom.panel.3.1.1.2.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.bottom.panel.3.1.1.2.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.bottom.panel.3.1.1.2.tspan.1" dy="7.18" x="0">Jul 2013</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.3.1.vp.2.clipPath">
            <rect x="151.14" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.3.1.vp.2" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.3.1.vp.2.clipPath)" class="pushedvp viewport">
          <g id="plot_01.xyplot.lines.group.3.panel.3.1.1" class="lines grob gDesc">
            <polyline id="plot_01.xyplot.lines.group.3.panel.3.1.1.1" points="154.45,104.31 154.53,104.31 154.62,104.31 154.7,104.31 154.95,102.37 155.03,101.4 155.11,98.48 155.19,99.45 155.28,98.48 155.61,97.51 155.69,99.45 155.77,100.42 155.85,102.37 156.1,103.34 156.19,103.34 156.27,98.48 156.35,95.56 156.43,96.54 156.68,95.56 156.76,94.59 156.85,95.56 156.93,95.56 157.01,97.51 157.26,96.54 157.34,99.45 157.42,99.45 157.51,102.37 157.59,100.42 157.84,104.31 157.92,104.31 158,102.37 158.09,106.25 158.17,106.25 158.5,108.2 158.58,106.25 158.66,107.23 158.75,107.23 158.99,104.31 159.08,105.28 159.16,107.23 159.24,107.23 159.33,105.28 159.57,107.23 159.66,104.31 159.74,106.25 159.82,108.2 159.9,110.14 160.15,111.11 160.23,115 160.32,123.75 160.4,119.86 160.48,120.83 160.73,123.75 160.81,125.69 160.9,121.8 160.98,119.86 161.06,118.89 161.31,117.92 161.39,114.03 161.47,115 161.56,114.03 161.64,115 161.89,114.03 161.97,119.86 162.05,116.94 162.13,114.03 162.22,109.17 162.47,110.14 162.55,106.25 162.63,107.23 162.71,107.23 162.8,105.28 163.04,106.25 163.13,106.25 163.21,104.31 163.29,104.31 163.37,104.31 163.62,103.34 163.7,104.31 163.79,103.34 163.87,103.34 163.95,103.34 164.2,102.37 164.28,103.34 164.37,103.34 164.45,104.31 164.53,101.4 164.78,101.4 164.86,100.42 164.94,100.42 165.03,101.4 165.11,100.42 165.36,101.4 165.44,102.37 165.52,104.31 165.61,104.31 165.69,106.25 165.94,105.28 166.02,105.28 166.1,104.31 166.18,106.25 166.27,105.28 166.6,106.25 166.68,102.37 166.76,99.45 166.84,98.48 167.09,99.45 167.18,98.48 167.26,101.4 167.34,101.4 167.42,103.34 167.67,101.4 167.75,105.28 167.84,104.31 167.92,105.28 168,101.4 168.25,102.37 168.33,103.34 168.41,105.28 168.5,105.28 168.58,106.25 168.83,103.34 168.91,106.25 168.99,106.25 169.08,104.31 169.16,105.28 169.41,103.34 169.49,103.34 169.65,103.34 169.74,101.4 169.98,100.42 170.07,101.4 170.15,100.42 170.23,99.45 170.32,98.48 170.56,95.56 170.65,96.54 170.73,94.59 170.81,95.56 170.89,93.62 171.14,92.65 171.22,92.65 171.31,92.65 171.39,95.56 171.47,98.48 171.72,95.56 171.8,94.59 171.89,96.54 171.97,95.56 172.05,97.51 172.3,97.51 172.38,101.4 172.46,102.37 172.55,102.37 172.63,100.42 172.88,100.42 172.96,103.34 173.04,106.25 173.12,106.25 173.21,106.25 173.46,105.28 173.54,106.25 173.62,101.4 173.7,100.42 173.79,101.4 174.03,101.4 174.12,100.42 174.2,100.42 174.28,99.45 174.36,94.59 174.69,95.56 174.78,96.54 174.86,98.48 174.94,97.51 175.19,97.51 175.27,97.51 175.36,97.51 175.44,96.54 175.52,99.45 175.77,100.42 175.85,99.45 175.93,99.45 176.02,100.42 176.1,100.42 176.35,99.45 176.43,99.45 176.51,98.48 176.6,98.48 176.68,95.56 176.93,95.56 177.01,95.56 177.09,95.56 177.17,96.54 177.26,98.48 177.59,99.45 177.67,99.45 177.75,98.48 177.83,98.48 178.08,98.48 178.17,100.42 178.25,105.28 178.33,105.28 178.41,105.28 178.66,106.25 178.74,105.28 178.83,104.31 178.91,107.23 178.99,105.28 179.24,104.31 179.4,102.37 179.49,102.37 179.57,102.37 179.82,102.37 179.9,105.28 179.98,100.42 180.07,99.45 180.15,99.45 180.48,97.51 180.56,97.51 180.64,96.54 180.73,96.54 180.97,97.51 181.06,100.42 181.14,101.4 181.31,101.4 181.55,100.42 181.64,100.42 181.72,99.45 181.8,99.45 181.88,98.48 182.13,98.48 182.21,98.48 182.3,96.54 182.38,96.54 182.46,97.51 182.71,97.51 182.79,96.54 182.88,96.54 182.96,98.48 183.04,98.48 183.29,101.4 183.37,103.34 183.45,103.34 183.54,103.34 183.62,102.37 183.87,102.37 184.03,103.34 184.11,101.4 184.2,100.42 184.45,100.42 184.61,101.4 184.69,104.31 184.78,105.28 185.02,105.28 185.11,102.37 185.19,101.4 185.27,101.4 185.35,101.4 185.6,101.4 185.68,100.42 185.77,100.42 185.85,103.34 185.93,102.37 186.26,102.37 186.35,101.4 186.43,101.4 186.51,106.25 186.76,109.17 186.84,107.23 186.92,106.25 187.01,106.25 187.09,104.31 187.34,102.37 187.42,105.28 187.5,103.34 187.59,103.34 187.67,103.34 187.92,104.31 188,105.28 188.08,108.2 188.16,106.25 188.25,106.25 188.58,108.2 188.66,106.25 188.74,104.31 188.82,104.31 189.07,101.4 189.16,101.4 189.24,100.42 189.32,100.42 189.4,99.45 189.65,99.45 189.73,100.42 189.82,102.37 189.9,104.31 189.98,106.25 190.23,107.23 190.31,105.28 190.39,106.25 190.48,106.25 190.56,104.31 190.81,102.37 190.89,101.4 190.97,102.37 191.06,102.37 191.14,103.34 191.39,102.37 191.47,102.37 191.55,100.42 191.63,100.42 191.96,100.42 192.05,100.42 192.13,98.48 192.21,97.51 192.3,97.51 192.54,98.48 192.63,98.48 192.71,100.42 192.79,99.45 192.87,97.51 193.12,96.54 193.2,97.51 193.29,99.45 193.37,99.45 193.45,99.45 193.7,99.45 193.78,99.45 193.87,98.48 193.95,99.45 194.03,96.54 194.28,96.54 194.36,96.54 194.44,94.59 194.53,94.59 194.61,98.48 194.86,98.48 194.94,99.45 195.02,99.45 195.1,99.45 195.19,102.37 195.44,104.31 195.52,105.28 195.6,104.31 195.68,101.4 195.77,104.31 196.01,104.31 196.1,103.34 196.18,105.28 196.26,106.25 196.34,105.28 196.67,113.06 196.76,113.06 196.84,113.06 196.92,115.97 197.17,114.03 197.25,112.09 197.34,112.09 197.42,112.09 197.5,115.97 197.75,118.89 197.83,120.83 197.91,120.83 198,118.89 198.08,113.06 198.33,113.06 198.41,112.09 198.49,121.8 198.58,125.69 198.66,133.46 198.91,136.38 198.99,137.35 199.07,132.49 199.15,129.58 199.24,129.58 199.48,128.61 199.57,127.63 199.65,130.55 199.81,140.27 200.06,134.44 200.15,134.44 200.23,136.38 200.31,128.61 200.39,129.58 200.64,129.58 200.72,127.63 200.81,123.75 200.89,124.72 200.97,122.77 201.22,122.77 201.3,123.75 201.38,127.63 201.47,125.69 201.55,122.77 201.8,124.72" stroke-dasharray="none" stroke="rgb(0,100,0)" stroke-width="0.75" opacity="1" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.panel.3.1.off.vp.2" class="pushedvp viewport">
          <g id="plot_01.border.panel.3.1.1" class="rect grob gDesc">
            <rect id="plot_01.border.panel.3.1.1.1" x="151.14" y="50.8" width="53.98" height="396.61" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.3.1.vp.1" class="pushedvp viewport">
          <defs>
            <clipPath id="plot_01.toplevel.vp::plot_01.strip.3.1.vp::plot_01.strip.default.1.1.clipPath">
              <rect x="151.14" y="447.41" width="53.98" height="14.4" fill="none" stroke="none"/>
            </clipPath>
          </defs>
          <g id="plot_01.toplevel.vp::plot_01.strip.3.1.vp::plot_01.strip.default.1.1" clip-path="url(#plot_01.toplevel.vp::plot_01.strip.3.1.vp::plot_01.strip.default.1.1.clipPath)" class="pushedvp viewport">
            <g id="plot_01.bg.strip.3.1.1" class="rect grob gDesc">
              <rect id="plot_01.bg.strip.3.1.1.1" x="151.14" y="447.41" width="53.98" height="14.4" fill="rgb(255,229,204)" stroke="rgb(255,229,204)" stroke-opacity="1" fill-opacity="1"/>
            </g>
            <g id="plot_01.textr.strip.3.1.1" class="text grob gDesc">
              <g id="plot_01.textr.strip.3.1.1.1" transform="translate(161.79, 454.61)" stroke-width="0.1">
                <g id="plot_01.textr.strip.3.1.1.1.scale" transform="scale(1, -1)">
                  <text x="0" y="0" id="plot_01.textr.strip.3.1.1.1.text" text-anchor="start" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                    <tspan id="plot_01.textr.strip.3.1.1.1.tspan.1" dy="4.31" x="0">DGS3</tspan>
                  </text>
                </g>
              </g>
            </g>
          </g>
          <g id="plot_01.toplevel.vp::plot_01.strip.3.1.vp::plot_01.strip.default.off.1.1" class="pushedvp viewport">
            <g id="plot_01.border.strip.3.1.1" class="rect grob gDesc">
              <rect id="plot_01.border.strip.3.1.1.1" x="151.14" y="447.41" width="53.98" height="14.4" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.4.1.vp.1.clipPath">
            <rect x="205.11" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.4.1.vp.1" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.4.1.vp.1.clipPath)" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.strip.4.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.top.panel.4.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.top.panel.4.1.1.1" points="223.38,461.81 223.38,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.top.panel.4.1.1.2" points="253.54,461.81 253.54,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticklabels.top.panel.4.1.1" class="text grob gDesc">
            <g id="plot_01.ticklabels.top.panel.4.1.1.1" transform="translate(223.38, 473.15)" stroke-width="0.1">
              <g id="plot_01.ticklabels.top.panel.4.1.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.top.panel.4.1.1.1.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.top.panel.4.1.1.1.tspan.1" dy="0" x="0">Jul 2012</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.top.panel.4.1.1.2" transform="translate(253.54, 473.15)" stroke-width="0.1">
              <g id="plot_01.ticklabels.top.panel.4.1.1.2.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.top.panel.4.1.1.2.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.top.panel.4.1.1.2.tspan.1" dy="0" x="0">Jul 2013</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.left.4.1.off.vp.1" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.panel.4.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.bottom.panel.4.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.bottom.panel.4.1.1.1" points="223.38,50.8 223.38,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.bottom.panel.4.1.1.2" points="253.54,50.8 253.54,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.4.1.vp.2.clipPath">
            <rect x="205.11" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.4.1.vp.2" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.4.1.vp.2.clipPath)" class="pushedvp viewport">
          <g id="plot_01.xyplot.lines.group.4.panel.4.1.1" class="lines grob gDesc">
            <polyline id="plot_01.xyplot.lines.group.4.panel.4.1.1.1" points="208.43,151.93 208.51,151.93 208.59,150.96 208.67,149.01 208.92,148.04 209,149.01 209.09,145.13 209.17,147.07 209.25,143.18 209.58,142.21 209.67,145.13 209.75,149.99 209.83,153.87 210.08,155.82 210.16,154.84 210.24,144.15 210.33,140.27 210.41,138.32 210.66,136.38 210.74,134.44 210.82,135.41 210.91,134.44 210.99,141.24 211.24,139.3 211.32,145.13 211.4,145.13 211.48,149.01 211.57,144.15 211.81,148.04 211.9,144.15 211.98,144.15 212.06,149.99 212.14,150.96 212.48,154.84 212.56,150.96 212.64,150.96 212.72,151.93 212.97,147.07 213.05,147.07 213.14,149.99 213.22,151.93 213.3,147.07 213.55,149.99 213.63,146.1 213.71,148.04 213.8,151.93 213.88,152.9 214.13,154.84 214.21,161.65 214.29,175.25 214.38,173.31 214.46,175.25 214.71,182.05 214.79,184 214.87,177.2 214.95,175.25 215.04,172.34 215.28,171.36 215.37,166.51 215.45,167.48 215.53,163.59 215.62,166.51 215.86,165.53 215.95,172.34 216.03,167.48 216.11,163.59 216.19,151.93 216.44,152.9 216.52,148.04 216.61,151.93 216.69,152.9 216.77,149.01 217.02,148.04 217.1,150.96 217.19,149.01 217.27,147.07 217.35,149.01 217.6,146.1 217.68,149.01 217.76,149.01 217.85,146.1 217.93,145.13 218.18,145.13 218.26,147.07 218.34,145.13 218.42,145.13 218.51,141.24 218.76,142.21 218.84,140.27 218.92,140.27 219,142.21 219.09,138.32 219.33,136.38 219.42,137.35 219.5,138.32 219.58,137.35 219.66,138.32 219.91,138.32 219.99,141.24 220.08,137.35 220.16,140.27 220.24,139.3 220.57,139.3 220.66,132.49 220.74,130.55 220.82,125.69 221.07,131.52 221.15,131.52 221.23,136.38 221.32,135.41 221.4,134.44 221.65,132.49 221.73,138.32 221.81,134.44 221.9,136.38 221.98,131.52 222.23,132.49 222.31,134.44 222.39,137.35 222.47,136.38 222.56,139.3 222.8,135.41 222.89,138.32 222.97,136.38 223.05,132.49 223.13,135.41 223.38,130.55 223.47,132.49 223.63,131.52 223.71,127.63 223.96,126.66 224.04,126.66 224.13,127.63 224.21,126.66 224.29,126.66 224.54,123.75 224.62,125.69 224.7,123.75 224.79,125.69 224.87,122.77 225.12,120.83 225.2,120.83 225.28,119.86 225.37,121.8 225.45,128.61 225.7,124.72 225.78,123.75 225.86,126.66 225.94,124.72 226.03,130.55 226.27,128.61 226.36,134.44 226.44,136.38 226.52,137.35 226.61,134.44 226.85,134.44 226.94,138.32 227.02,143.18 227.1,146.1 227.18,144.15 227.43,143.18 227.51,143.18 227.6,134.44 227.68,134.44 227.76,135.41 228.01,133.46 228.09,132.49 228.18,132.49 228.26,129.58 228.34,122.77 228.67,125.69 228.75,125.69 228.84,131.52 228.92,127.63 229.17,129.58 229.25,130.55 229.33,133.46 229.41,128.61 229.5,135.41 229.75,136.38 229.83,134.44 229.91,133.46 229.99,133.46 230.08,131.52 230.32,131.52 230.41,129.58 230.49,126.66 230.57,127.63 230.65,125.69 230.9,125.69 230.98,124.72 231.07,124.72 231.15,126.66 231.23,130.55 231.56,130.55 231.65,129.58 231.73,130.55 231.81,130.55 232.06,130.55 232.14,133.46 232.22,141.24 232.31,142.21 232.39,140.27 232.64,142.21 232.72,140.27 232.8,139.3 232.88,145.13 232.97,139.3 233.22,137.35 233.38,135.41 233.46,136.38 233.55,136.38 233.79,133.46 233.88,138.32 233.96,130.55 234.04,128.61 234.12,128.61 234.45,126.66 234.54,126.66 234.62,125.69 234.7,125.69 234.95,127.63 235.03,130.55 235.12,132.49 235.28,133.46 235.53,131.52 235.61,129.58 235.69,127.63 235.78,126.66 235.86,124.72 236.11,126.66 236.19,126.66 236.27,124.72 236.36,123.75 236.44,126.66 236.69,125.69 236.77,127.63 236.85,129.58 236.93,133.46 237.02,133.46 237.26,137.35 237.35,141.24 237.43,140.27 237.51,140.27 237.59,138.32 237.84,140.27 238.01,139.3 238.09,135.41 238.17,135.41 238.42,135.41 238.59,139.3 238.67,144.15 238.75,145.13 239,145.13 239.08,142.21 239.16,140.27 239.25,143.18 239.33,141.24 239.58,141.24 239.66,138.32 239.74,138.32 239.83,142.21 239.91,140.27 240.24,139.3 240.32,139.3 240.4,141.24 240.49,149.99 240.73,151.93 240.82,152.9 240.9,150.96 240.98,150.96 241.07,150.96 241.31,148.04 241.4,150.96 241.48,147.07 241.56,146.1 241.64,147.07 241.89,148.04 241.97,150.96 242.06,154.84 242.14,149.01 242.22,149.99 242.55,151.93 242.64,150.96 242.72,149.01 242.8,147.07 243.05,141.24 243.13,141.24 243.21,141.24 243.3,140.27 243.38,138.32 243.63,139.3 243.71,140.27 243.79,144.15 243.87,148.04 243.96,152.9 244.21,152.9 244.29,150.96 244.37,151.93 244.45,150.96 244.54,147.07 244.78,144.15 244.87,142.21 244.95,144.15 245.03,144.15 245.11,143.18 245.36,143.18 245.44,142.21 245.53,139.3 245.61,140.27 245.94,139.3 246.02,141.24 246.11,136.38 246.19,132.49 246.27,131.52 246.52,134.44 246.6,133.46 246.68,137.35 246.77,137.35 246.85,133.46 247.1,132.49 247.18,134.44 247.26,134.44 247.35,134.44 247.43,135.41 247.68,133.46 247.76,134.44 247.84,133.46 247.92,134.44 248.01,131.52 248.25,131.52 248.34,131.52 248.42,128.61 248.5,128.61 248.58,136.38 248.83,137.35 248.92,138.32 249,138.32 249.08,138.32 249.16,145.13 249.41,146.1 249.49,148.04 249.58,147.07 249.66,142.21 249.74,147.07 249.99,148.04 250.07,147.07 250.15,153.87 250.24,153.87 250.32,152.9 250.65,164.56 250.73,164.56 250.82,163.59 250.9,167.48 251.15,165.53 251.23,167.48 251.31,164.56 251.39,163.59 251.48,172.34 251.72,175.25 251.81,174.28 251.89,177.2 251.97,173.31 252.06,166.51 252.3,168.45 252.39,169.42 252.47,185.94 252.55,192.74 252.63,203.43 252.88,209.26 252.96,210.24 253.05,206.35 253.13,199.55 253.21,202.46 253.46,200.52 253.54,199.55 253.63,203.43 253.79,220.93 254.04,212.18 254.12,211.21 254.2,215.1 254.29,201.49 254.37,204.41 254.62,201.49 254.7,199.55 254.78,194.69 254.86,196.63 254.95,192.74 255.2,193.72 255.28,194.69 255.36,201.49 255.44,199.55 255.53,197.6 255.77,198.57" stroke-dasharray="none" stroke="rgb(255,0,0)" stroke-width="0.75" opacity="1" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.panel.4.1.off.vp.2" class="pushedvp viewport">
          <g id="plot_01.border.panel.4.1.1" class="rect grob gDesc">
            <rect id="plot_01.border.panel.4.1.1.1" x="205.11" y="50.8" width="53.98" height="396.61" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.4.1.vp.1" class="pushedvp viewport">
          <defs>
            <clipPath id="plot_01.toplevel.vp::plot_01.strip.4.1.vp::plot_01.strip.default.1.1.clipPath">
              <rect x="205.11" y="447.41" width="53.98" height="14.4" fill="none" stroke="none"/>
            </clipPath>
          </defs>
          <g id="plot_01.toplevel.vp::plot_01.strip.4.1.vp::plot_01.strip.default.1.1" clip-path="url(#plot_01.toplevel.vp::plot_01.strip.4.1.vp::plot_01.strip.default.1.1.clipPath)" class="pushedvp viewport">
            <g id="plot_01.bg.strip.4.1.1" class="rect grob gDesc">
              <rect id="plot_01.bg.strip.4.1.1.1" x="205.11" y="447.41" width="53.98" height="14.4" fill="rgb(255,229,204)" stroke="rgb(255,229,204)" stroke-opacity="1" fill-opacity="1"/>
            </g>
            <g id="plot_01.textr.strip.4.1.1" class="text grob gDesc">
              <g id="plot_01.textr.strip.4.1.1.1" transform="translate(215.76, 454.61)" stroke-width="0.1">
                <g id="plot_01.textr.strip.4.1.1.1.scale" transform="scale(1, -1)">
                  <text x="0" y="0" id="plot_01.textr.strip.4.1.1.1.text" text-anchor="start" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                    <tspan id="plot_01.textr.strip.4.1.1.1.tspan.1" dy="4.31" x="0">DGS5</tspan>
                  </text>
                </g>
              </g>
            </g>
          </g>
          <g id="plot_01.toplevel.vp::plot_01.strip.4.1.vp::plot_01.strip.default.off.1.1" class="pushedvp viewport">
            <g id="plot_01.border.strip.4.1.1" class="rect grob gDesc">
              <rect id="plot_01.border.strip.4.1.1.1" x="205.11" y="447.41" width="53.98" height="14.4" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.5.1.vp.1.clipPath">
            <rect x="259.09" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.5.1.vp.1" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.5.1.vp.1.clipPath)" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.strip.5.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.top.panel.5.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.top.panel.5.1.1.1" points="277.36,461.81 277.36,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.top.panel.5.1.1.2" points="307.52,461.81 307.52,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.left.5.1.off.vp.1" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.panel.5.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.bottom.panel.5.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.bottom.panel.5.1.1.1" points="277.36,50.8 277.36,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.bottom.panel.5.1.1.2" points="307.52,50.8 307.52,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticklabels.bottom.panel.5.1.1" class="text grob gDesc">
            <g id="plot_01.ticklabels.bottom.panel.5.1.1.1" transform="translate(277.36, 39.47)" stroke-width="0.1">
              <g id="plot_01.ticklabels.bottom.panel.5.1.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.bottom.panel.5.1.1.1.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.bottom.panel.5.1.1.1.tspan.1" dy="7.18" x="0">Jul 2012</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.bottom.panel.5.1.1.2" transform="translate(307.52, 39.47)" stroke-width="0.1">
              <g id="plot_01.ticklabels.bottom.panel.5.1.1.2.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.bottom.panel.5.1.1.2.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.bottom.panel.5.1.1.2.tspan.1" dy="7.18" x="0">Jul 2013</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.5.1.vp.2.clipPath">
            <rect x="259.09" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.5.1.vp.2" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.5.1.vp.2.clipPath)" class="pushedvp viewport">
          <g id="plot_01.xyplot.lines.group.5.panel.5.1.1" class="lines grob gDesc">
            <polyline id="plot_01.xyplot.lines.group.5.panel.5.1.1.1" points="262.4,202.46 262.48,204.41 262.57,204.41 262.65,201.49 262.9,200.52 262.98,202.46 263.06,195.66 263.15,198.57 263.23,193.72 263.56,192.74 263.64,195.66 263.72,204.41 263.81,208.29 264.05,212.18 264.14,210.24 264.22,201.49 264.3,195.66 264.39,192.74 264.63,188.86 264.72,185.94 264.8,188.86 264.88,186.91 264.96,196.63 265.21,193.72 265.29,200.52 265.38,200.52 265.46,204.41 265.54,197.6 265.79,201.49 265.87,195.66 265.96,195.66 266.04,202.46 266.12,204.41 266.45,208.29 266.53,202.46 266.62,201.49 266.7,202.46 266.95,196.63 267.03,197.6 267.11,200.52 267.19,205.38 267.28,199.55 267.53,201.49 267.61,196.63 267.69,198.57 267.77,202.46 267.86,204.41 268.1,204.41 268.19,213.15 268.27,229.67 268.35,227.73 268.43,230.64 268.68,237.45 268.76,238.42 268.85,231.62 268.93,229.67 269.01,226.76 269.26,225.78 269.34,219.95 269.43,220.93 269.51,218.01 269.59,221.9 269.84,220.93 269.92,228.7 270,222.87 270.09,217.04 270.17,203.43 270.42,203.43 270.5,198.57 270.58,202.46 270.67,205.38 270.75,200.52 271,198.57 271.08,201.49 271.16,199.55 271.24,198.57 271.33,199.55 271.57,195.66 271.66,198.57 271.74,199.55 271.82,197.6 271.9,195.66 272.15,194.69 272.24,196.63 272.32,194.69 272.4,195.66 272.48,189.83 272.73,190.8 272.81,187.89 272.9,187.89 272.98,189.83 273.06,185.94 273.31,182.05 273.39,181.08 273.47,181.08 273.56,178.17 273.64,178.17 273.89,180.11 273.97,182.05 274.05,177.2 274.14,182.05 274.22,179.14 274.55,179.14 274.63,168.45 274.71,165.53 274.8,155.82 275.04,163.59 275.13,166.51 275.21,173.31 275.29,172.34 275.38,171.36 275.62,167.48 275.71,174.28 275.79,168.45 275.87,172.34 275.95,168.45 276.2,168.45 276.28,171.36 276.37,174.28 276.45,172.34 276.53,177.2 276.78,172.34 276.86,174.28 276.95,172.34 277.03,168.45 277.11,173.31 277.36,166.51 277.44,170.39 277.61,167.48 277.69,163.59 277.94,160.67 278.02,160.67 278.1,161.65 278.18,160.67 278.27,161.65 278.52,159.7 278.6,161.65 278.68,159.7 278.76,161.65 278.85,157.76 279.09,155.82 279.18,153.87 279.26,153.87 279.34,156.79 279.42,166.51 279.67,161.65 279.75,160.67 279.84,165.53 279.92,160.67 280,169.42 280.25,167.48 280.33,175.25 280.42,176.22 280.5,177.2 280.58,173.31 280.83,174.28 280.91,180.11 280.99,186.91 281.08,189.83 281.16,188.86 281.41,187.89 281.49,186.91 281.57,178.17 281.66,175.25 281.74,176.22 281.99,173.31 282.07,172.34 282.15,173.31 282.23,170.39 282.32,163.59 282.65,165.53 282.73,166.51 282.81,174.28 282.89,171.36 283.14,172.34 283.23,174.28 283.31,179.14 283.39,174.28 283.47,184.97 283.72,184 283.8,181.08 283.89,180.11 283.97,180.11 284.05,176.22 284.3,174.28 284.38,170.39 284.46,165.53 284.55,167.48 284.63,166.51 284.88,166.51 284.96,165.53 285.04,164.56 285.13,169.42 285.21,174.28 285.54,173.31 285.62,171.36 285.7,171.36 285.79,171.36 286.03,171.36 286.12,177.2 286.2,185.94 286.28,187.89 286.37,183.03 286.61,186.91 286.7,183.03 286.78,183.03 286.86,189.83 286.94,182.05 287.19,178.17 287.36,176.22 287.44,178.17 287.52,178.17 287.77,175.25 287.85,181.08 287.94,170.39 288.02,166.51 288.1,166.51 288.43,164.56 288.51,165.53 288.6,164.56 288.68,163.59 288.93,166.51 289.01,171.36 289.09,173.31 289.26,174.28 289.51,171.36 289.59,169.42 289.67,167.48 289.75,166.51 289.84,166.51 290.08,167.48 290.17,166.51 290.25,164.56 290.33,162.62 290.41,166.51 290.66,166.51 290.74,168.45 290.83,173.31 290.91,177.2 290.99,177.2 291.24,182.05 291.32,186.91 291.41,185.94 291.49,185.94 291.57,182.05 291.82,184 291.98,182.05 292.07,177.2 292.15,177.2 292.4,180.11 292.56,186.91 292.65,192.74 292.73,193.72 292.98,192.74 293.06,189.83 293.14,188.86 293.22,191.77 293.31,189.83 293.55,188.86 293.64,185.94 293.72,184.97 293.8,190.8 293.88,187.89 294.22,186.91 294.3,185.94 294.38,187.89 294.46,197.6 294.71,199.55 294.79,201.49 294.88,200.52 294.96,199.55 295.04,201.49 295.29,197.6 295.37,200.52 295.45,196.63 295.54,195.66 295.62,195.66 295.87,196.63 295.95,199.55 296.03,204.41 296.12,198.57 296.2,199.55 296.53,202.46 296.61,199.55 296.69,197.6 296.78,195.66 297.02,186.91 297.11,186.91 297.19,189.83 297.27,187.89 297.36,184.97 297.6,186.91 297.69,188.86 297.77,192.74 297.85,197.6 297.93,204.41 298.18,204.41 298.26,201.49 298.35,202.46 298.43,201.49 298.51,196.63 298.76,192.74 298.84,189.83 298.93,193.72 299.01,191.77 299.09,190.8 299.34,189.83 299.42,188.86 299.5,184 299.59,185.94 299.92,184.97 300,187.89 300.08,182.05 300.16,177.2 300.25,174.28 300.5,177.2 300.58,178.17 300.66,183.03 300.74,182.05 300.83,176.22 301.07,174.28 301.16,177.2 301.24,175.25 301.32,175.25 301.4,176.22 301.65,175.25 301.73,176.22 301.82,175.25 301.9,177.2 301.98,172.34 302.23,172.34 302.31,173.31 302.4,169.42 302.48,169.42 302.56,179.14 302.81,181.08 302.89,183.03 302.97,182.05 303.06,182.05 303.14,189.83 303.39,191.77 303.47,194.69 303.55,193.72 303.64,186.91 303.72,193.72 303.97,194.69 304.05,192.74 304.13,201.49 304.21,201.49 304.3,200.52 304.63,214.12 304.71,212.18 304.79,212.18 304.87,216.07 305.12,214.12 305.21,216.07 305.29,213.15 305.37,210.24 305.45,219.95 305.7,222.87 305.78,221.9 305.87,224.81 305.95,220.93 306.03,214.12 306.28,218.01 306.36,218.98 306.44,236.47 306.53,244.25 306.61,254.94 306.86,261.74 306.94,262.71 307.02,257.85 307.11,251.05 307.19,255.91 307.44,253 307.52,252.02 307.6,256.88 307.77,278.26 308.01,270.49 308.1,267.57 308.18,271.46 308.26,258.83 308.34,259.8 308.59,256.88 308.68,254.94 308.76,251.05 308.84,254.94 308.92,250.08 309.17,250.08 309.25,252.02 309.34,259.8 309.42,259.8 309.5,257.85 309.75,259.8" stroke-dasharray="none" stroke="rgb(255,165,0)" stroke-width="0.75" opacity="1" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.panel.5.1.off.vp.2" class="pushedvp viewport">
          <g id="plot_01.border.panel.5.1.1" class="rect grob gDesc">
            <rect id="plot_01.border.panel.5.1.1.1" x="259.09" y="50.8" width="53.98" height="396.61" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.5.1.vp.1" class="pushedvp viewport">
          <defs>
            <clipPath id="plot_01.toplevel.vp::plot_01.strip.5.1.vp::plot_01.strip.default.1.1.clipPath">
              <rect x="259.09" y="447.41" width="53.98" height="14.4" fill="none" stroke="none"/>
            </clipPath>
          </defs>
          <g id="plot_01.toplevel.vp::plot_01.strip.5.1.vp::plot_01.strip.default.1.1" clip-path="url(#plot_01.toplevel.vp::plot_01.strip.5.1.vp::plot_01.strip.default.1.1.clipPath)" class="pushedvp viewport">
            <g id="plot_01.bg.strip.5.1.1" class="rect grob gDesc">
              <rect id="plot_01.bg.strip.5.1.1.1" x="259.09" y="447.41" width="53.98" height="14.4" fill="rgb(255,229,204)" stroke="rgb(255,229,204)" stroke-opacity="1" fill-opacity="1"/>
            </g>
            <g id="plot_01.textr.strip.5.1.1" class="text grob gDesc">
              <g id="plot_01.textr.strip.5.1.1.1" transform="translate(269.74, 454.61)" stroke-width="0.1">
                <g id="plot_01.textr.strip.5.1.1.1.scale" transform="scale(1, -1)">
                  <text x="0" y="0" id="plot_01.textr.strip.5.1.1.1.text" text-anchor="start" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                    <tspan id="plot_01.textr.strip.5.1.1.1.tspan.1" dy="4.31" x="0">DGS7</tspan>
                  </text>
                </g>
              </g>
            </g>
          </g>
          <g id="plot_01.toplevel.vp::plot_01.strip.5.1.vp::plot_01.strip.default.off.1.1" class="pushedvp viewport">
            <g id="plot_01.border.strip.5.1.1" class="rect grob gDesc">
              <rect id="plot_01.border.strip.5.1.1.1" x="259.09" y="447.41" width="53.98" height="14.4" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.6.1.vp.1.clipPath">
            <rect x="313.06" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.6.1.vp.1" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.6.1.vp.1.clipPath)" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.strip.6.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.top.panel.6.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.top.panel.6.1.1.1" points="331.33,461.81 331.33,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.top.panel.6.1.1.2" points="361.49,461.81 361.49,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticklabels.top.panel.6.1.1" class="text grob gDesc">
            <g id="plot_01.ticklabels.top.panel.6.1.1.1" transform="translate(331.33, 473.15)" stroke-width="0.1">
              <g id="plot_01.ticklabels.top.panel.6.1.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.top.panel.6.1.1.1.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.top.panel.6.1.1.1.tspan.1" dy="0" x="0">Jul 2012</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.top.panel.6.1.1.2" transform="translate(361.49, 473.15)" stroke-width="0.1">
              <g id="plot_01.ticklabels.top.panel.6.1.1.2.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.top.panel.6.1.1.2.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.top.panel.6.1.1.2.tspan.1" dy="0" x="0">Jul 2013</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.left.6.1.off.vp.1" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.panel.6.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.bottom.panel.6.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.bottom.panel.6.1.1.1" points="331.33,50.8 331.33,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.bottom.panel.6.1.1.2" points="361.49,50.8 361.49,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.6.1.vp.2.clipPath">
            <rect x="313.06" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.6.1.vp.2" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.6.1.vp.2.clipPath)" class="pushedvp viewport">
          <g id="plot_01.xyplot.lines.group.6.panel.6.1.1" class="lines grob gDesc">
            <polyline id="plot_01.xyplot.lines.group.6.panel.6.1.1.1" points="316.38,256.88 316.46,259.8 316.54,261.74 316.63,257.85 316.87,257.85 316.96,259.8 317.04,253 317.12,253.97 317.2,249.11 317.54,247.16 317.62,252.02 317.7,260.77 317.78,264.66 318.03,268.54 318.11,267.57 318.2,260.77 318.28,255.91 318.36,253 318.61,247.16 318.69,243.28 318.77,247.16 318.86,246.19 318.94,256.88 319.19,253 319.27,259.8 319.35,260.77 319.44,263.68 319.52,255.91 319.77,258.83 319.85,252.02 319.93,253 320.01,258.83 320.1,260.77 320.43,264.66 320.51,260.77 320.59,258.83 320.68,257.85 320.92,252.02 321.01,253.97 321.09,257.85 321.17,262.71 321.25,258.83 321.5,259.8 321.58,255.91 321.67,257.85 321.75,262.71 321.83,263.68 322.08,263.68 322.16,273.4 322.25,287.98 322.33,287.98 322.41,289.92 322.66,297.7 322.74,296.73 322.82,289.92 322.91,287.98 322.99,284.09 323.24,285.06 323.32,279.23 323.4,280.21 323.48,277.29 323.57,282.15 323.82,281.18 323.9,288.95 323.98,284.09 324.06,278.26 324.15,266.6 324.39,265.63 324.48,260.77 324.56,264.66 324.64,267.57 324.72,261.74 324.97,259.8 325.05,262.71 325.14,259.8 325.22,257.85 325.3,258.83 325.55,255.91 325.63,259.8 325.72,260.77 325.8,257.85 325.88,255.91 326.13,254.94 326.21,257.85 326.29,255.91 326.38,255.91 326.46,251.05 326.71,252.02 326.79,248.14 326.87,247.16 326.96,249.11 327.04,244.25 327.29,238.42 327.37,236.47 327.45,236.47 327.53,230.64 327.62,231.62 327.86,235.5 327.95,239.39 328.03,233.56 328.11,237.45 328.19,235.5 328.53,234.53 328.61,223.84 328.69,219.95 328.77,208.29 329.02,214.12 329.1,218.01 329.19,226.76 329.27,226.76 329.35,225.78 329.6,220.93 329.68,227.73 329.76,221.9 329.85,224.81 329.93,220.93 330.18,219.95 330.26,224.81 330.34,225.78 330.43,223.84 330.51,229.67 330.76,223.84 330.84,226.76 330.92,225.78 331,220.93 331.09,227.73 331.33,221.9 331.42,225.78 331.58,222.87 331.67,218.01 331.91,214.12 332,214.12 332.08,215.1 332.16,211.21 332.24,213.15 332.49,211.21 332.57,214.12 332.66,213.15 332.74,215.1 332.82,210.24 333.07,208.29 333.15,205.38 333.24,204.41 333.32,206.35 333.4,218.98 333.65,214.12 333.73,212.18 333.81,217.04 333.9,212.18 333.98,220.93 334.23,219.95 334.31,226.76 334.39,228.7 334.47,229.67 334.56,225.78 334.8,225.78 334.89,233.56 334.97,240.36 335.05,243.28 335.14,241.33 335.38,242.31 335.47,240.36 335.55,231.62 335.63,228.7 335.71,228.7 335.96,225.78 336.04,224.81 336.13,226.76 336.21,223.84 336.29,218.01 336.62,219.95 336.71,220.93 336.79,228.7 336.87,227.73 337.12,228.7 337.2,230.64 337.28,237.45 337.37,235.5 337.45,248.14 337.7,245.22 337.78,242.31 337.86,239.39 337.94,240.36 338.03,237.45 338.28,234.53 338.36,230.64 338.44,224.81 338.52,226.76 338.61,225.78 338.85,224.81 338.94,224.81 339.02,224.81 339.1,230.64 339.18,235.5 339.51,234.53 339.6,232.59 339.68,230.64 339.76,229.67 340.01,230.64 340.09,235.5 340.18,243.28 340.26,246.19 340.34,239.39 340.59,243.28 340.67,239.39 340.75,240.36 340.84,246.19 340.92,238.42 341.17,234.53 341.33,232.59 341.42,235.5 341.5,235.5 341.75,232.59 341.83,238.42 341.91,228.7 341.99,222.87 342.08,221.9 342.41,219.95 342.49,219.95 342.57,218.98 342.65,218.98 342.9,221.9 342.99,226.76 343.07,229.67 343.23,230.64 343.48,226.76 343.56,224.81 343.65,223.84 343.73,222.87 343.81,222.87 344.06,223.84 344.14,222.87 344.22,220.93 344.31,219.95 344.39,224.81 344.64,223.84 344.72,226.76 344.8,232.59 344.89,234.53 344.97,232.59 345.22,238.42 345.3,244.25 345.38,242.31 345.46,241.33 345.55,237.45 345.79,239.39 345.96,237.45 346.04,234.53 346.13,233.56 346.37,238.42 346.54,246.19 346.62,252.02 346.7,253 346.95,252.02 347.03,249.11 347.12,248.14 347.2,251.05 347.28,249.11 347.53,249.11 347.61,246.19 347.7,244.25 347.78,249.11 347.86,247.16 348.19,246.19 348.27,246.19 348.36,248.14 348.44,257.85 348.69,259.8 348.77,262.71 348.85,262.71 348.93,261.74 349.02,263.68 349.27,259.8 349.35,263.68 349.43,259.8 349.51,258.83 349.6,258.83 349.84,258.83 349.93,261.74 350.01,264.66 350.09,259.8 350.17,260.77 350.5,262.71 350.59,261.74 350.67,258.83 350.75,256.88 351,248.14 351.08,248.14 351.17,251.05 351.25,249.11 351.33,246.19 351.58,248.14 351.66,250.08 351.74,254.94 351.83,259.8 351.91,265.63 352.16,266.6 352.24,262.71 352.32,263.68 352.41,263.68 352.49,260.77 352.74,255.91 352.82,252.02 352.9,255.91 352.98,254.94 353.07,253 353.31,253 353.4,252.02 353.48,247.16 353.56,247.16 353.89,246.19 353.98,248.14 354.06,243.28 354.14,238.42 354.22,232.59 354.47,236.47 354.55,238.42 354.64,244.25 354.72,242.31 354.8,235.5 355.05,232.59 355.13,235.5 355.21,233.56 355.3,232.59 355.38,233.56 355.63,232.59 355.71,234.53 355.79,233.56 355.88,234.53 355.96,230.64 356.21,230.64 356.29,230.64 356.37,226.76 356.45,226.76 356.54,238.42 356.78,240.36 356.87,242.31 356.95,241.33 357.03,241.33 357.12,250.08 357.36,252.02 357.45,255.91 357.53,253.97 357.61,247.16 357.69,254.94 357.94,256.88 358.02,253.97 358.11,262.71 358.19,261.74 358.27,260.77 358.6,274.37 358.69,272.43 358.77,272.43 358.85,275.35 359.1,272.43 359.18,273.4 359.26,269.52 359.35,267.57 359.43,276.32 359.68,281.18 359.76,279.23 359.84,284.09 359.92,278.26 360.01,273.4 360.26,278.26 360.34,279.23 360.42,291.87 360.5,299.64 360.59,310.33 360.83,315.19 360.92,318.11 361,313.25 361.08,307.42 361.16,310.33 361.41,308.39 361.49,306.44 361.58,310.33 361.74,330.74 361.99,322.96 362.07,322.96 362.16,327.82 362.24,318.11 362.32,319.08 362.57,315.19 362.65,313.25 362.73,310.33 362.82,314.22 362.9,308.39 363.15,308.39 363.23,311.3 363.31,319.08 363.4,319.08 363.48,316.16 363.73,319.08" stroke-dasharray="none" stroke="rgb(0,255,0)" stroke-width="0.75" opacity="1" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.panel.6.1.off.vp.2" class="pushedvp viewport">
          <g id="plot_01.border.panel.6.1.1" class="rect grob gDesc">
            <rect id="plot_01.border.panel.6.1.1.1" x="313.06" y="50.8" width="53.98" height="396.61" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.6.1.vp.1" class="pushedvp viewport">
          <defs>
            <clipPath id="plot_01.toplevel.vp::plot_01.strip.6.1.vp::plot_01.strip.default.1.1.clipPath">
              <rect x="313.06" y="447.41" width="53.98" height="14.4" fill="none" stroke="none"/>
            </clipPath>
          </defs>
          <g id="plot_01.toplevel.vp::plot_01.strip.6.1.vp::plot_01.strip.default.1.1" clip-path="url(#plot_01.toplevel.vp::plot_01.strip.6.1.vp::plot_01.strip.default.1.1.clipPath)" class="pushedvp viewport">
            <g id="plot_01.bg.strip.6.1.1" class="rect grob gDesc">
              <rect id="plot_01.bg.strip.6.1.1.1" x="313.06" y="447.41" width="53.98" height="14.4" fill="rgb(255,229,204)" stroke="rgb(255,229,204)" stroke-opacity="1" fill-opacity="1"/>
            </g>
            <g id="plot_01.textr.strip.6.1.1" class="text grob gDesc">
              <g id="plot_01.textr.strip.6.1.1.1" transform="translate(320.38, 454.61)" stroke-width="0.1">
                <g id="plot_01.textr.strip.6.1.1.1.scale" transform="scale(1, -1)">
                  <text x="0" y="0" id="plot_01.textr.strip.6.1.1.1.text" text-anchor="start" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                    <tspan id="plot_01.textr.strip.6.1.1.1.tspan.1" dy="4.31" x="0">DGS10</tspan>
                  </text>
                </g>
              </g>
            </g>
          </g>
          <g id="plot_01.toplevel.vp::plot_01.strip.6.1.vp::plot_01.strip.default.off.1.1" class="pushedvp viewport">
            <g id="plot_01.border.strip.6.1.1" class="rect grob gDesc">
              <rect id="plot_01.border.strip.6.1.1.1" x="313.06" y="447.41" width="53.98" height="14.4" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.7.1.vp.1.clipPath">
            <rect x="367.04" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.7.1.vp.1" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.7.1.vp.1.clipPath)" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.strip.7.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.top.panel.7.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.top.panel.7.1.1.1" points="385.31,461.81 385.31,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.top.panel.7.1.1.2" points="415.47,461.81 415.47,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.left.7.1.off.vp.1" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.panel.7.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.bottom.panel.7.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.bottom.panel.7.1.1.1" points="385.31,50.8 385.31,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.bottom.panel.7.1.1.2" points="415.47,50.8 415.47,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticklabels.bottom.panel.7.1.1" class="text grob gDesc">
            <g id="plot_01.ticklabels.bottom.panel.7.1.1.1" transform="translate(385.31, 39.47)" stroke-width="0.1">
              <g id="plot_01.ticklabels.bottom.panel.7.1.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.bottom.panel.7.1.1.1.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.bottom.panel.7.1.1.1.tspan.1" dy="7.18" x="0">Jul 2012</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.bottom.panel.7.1.1.2" transform="translate(415.47, 39.47)" stroke-width="0.1">
              <g id="plot_01.ticklabels.bottom.panel.7.1.1.2.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.bottom.panel.7.1.1.2.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.bottom.panel.7.1.1.2.tspan.1" dy="7.18" x="0">Jul 2013</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.7.1.vp.2.clipPath">
            <rect x="367.04" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.7.1.vp.2" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.7.1.vp.2.clipPath)" class="pushedvp viewport">
          <g id="plot_01.xyplot.lines.group.7.panel.7.1.1" class="lines grob gDesc">
            <polyline id="plot_01.xyplot.lines.group.7.panel.7.1.1.1" points="370.35,324.91 370.44,328.8 370.52,331.71 370.6,327.82 370.85,327.82 370.93,328.8 371.02,321.02 371.1,322.96 371.18,317.13 371.51,315.19 371.59,321.02 371.68,329.77 371.76,335.6 372.01,339.48 372.09,339.48 372.17,335.6 372.25,331.71 372.34,328.8 372.59,321.99 372.67,317.13 372.75,322.96 372.83,321.99 372.92,333.65 373.16,328.8 373.25,335.6 373.33,335.6 373.41,340.46 373.49,332.68 373.74,335.6 373.82,327.82 373.91,329.77 373.99,335.6 374.07,337.54 374.4,341.43 374.49,336.57 374.57,334.63 374.65,332.68 374.9,326.85 374.98,328.8 375.06,330.74 375.15,337.54 375.23,334.63 375.48,335.6 375.56,330.74 375.64,333.65 375.73,339.48 375.81,340.46 376.06,339.48 376.14,349.2 376.22,364.75 376.3,364.75 376.39,364.75 376.63,370.58 376.72,369.61 376.8,362.81 376.88,360.86 376.96,356.01 377.21,356.98 377.3,353.09 377.38,354.06 377.46,350.17 377.54,356.98 377.79,356.98 377.87,363.78 377.96,358.92 378.04,354.06 378.12,342.4 378.37,339.48 378.45,334.63 378.53,339.48 378.62,342.4 378.7,334.63 378.95,332.68 379.03,336.57 379.11,333.65 379.2,331.71 379.28,332.68 379.53,328.8 379.61,332.68 379.69,333.65 379.77,331.71 379.86,330.74 380.1,330.74 380.19,333.65 380.27,329.77 380.35,329.77 380.44,324.91 380.68,324.91 380.77,321.02 380.85,321.02 380.93,321.99 381.01,317.13 381.26,311.3 381.34,308.39 381.43,306.44 381.51,297.7 381.59,298.67 381.84,300.61 381.92,306.44 382.01,299.64 382.09,304.5 382.17,302.56 382.5,302.56 382.58,290.9 382.67,286.04 382.75,272.43 383,276.32 383.08,282.15 383.16,292.84 383.24,293.81 383.33,294.78 383.58,288.95 383.66,295.75 383.74,288.95 383.82,291.87 383.91,288.95 384.15,287.01 384.24,291.87 384.32,292.84 384.4,288.95 384.48,295.75 384.73,289.92 384.81,292.84 384.9,290.9 384.98,287.01 385.06,296.73 385.31,288.95 385.39,294.78 385.56,292.84 385.64,287.01 385.89,283.12 385.97,281.18 386.05,281.18 386.14,277.29 386.22,279.23 386.47,277.29 386.55,281.18 386.63,280.21 386.72,283.12 386.8,276.32 387.05,274.37 387.13,270.49 387.21,270.49 387.29,272.43 387.38,286.04 387.62,281.18 387.71,280.21 387.79,284.09 387.87,279.23 387.95,288.95 388.2,287.98 388.29,295.75 388.37,297.7 388.45,298.67 388.53,295.75 388.78,295.75 388.86,303.53 388.95,311.3 389.03,315.19 389.11,313.25 389.36,313.25 389.44,311.3 389.52,302.56 389.61,299.64 389.69,299.64 389.94,296.73 390.02,294.78 390.1,296.73 390.19,294.78 390.27,287.98 390.6,288.95 390.68,290.9 390.76,299.64 390.85,300.61 391.09,301.58 391.18,302.56 391.26,310.33 391.34,311.3 391.43,325.88 391.67,321.99 391.76,319.08 391.84,316.16 391.92,316.16 392,315.19 392.25,311.3 392.33,305.47 392.42,298.67 392.5,301.58 392.58,300.61 392.83,299.64 392.91,299.64 393,300.61 393.08,306.44 393.16,313.25 393.49,310.33 393.57,306.44 393.66,303.53 393.74,302.56 393.99,303.53 394.07,309.36 394.15,318.11 394.23,321.02 394.32,313.25 394.57,315.19 394.65,311.3 394.73,313.25 394.81,318.11 394.9,311.3 395.14,306.44 395.31,304.5 395.39,308.39 395.47,309.36 395.72,305.47 395.8,310.33 395.89,300.61 395.97,293.81 396.05,292.84 396.38,289.92 396.47,289.92 396.55,288.95 396.63,289.92 396.88,292.84 396.96,298.67 397.04,300.61 397.21,300.61 397.46,297.7 397.54,296.73 397.62,294.78 397.71,295.75 397.79,295.75 398.04,295.75 398.12,294.78 398.2,293.81 398.28,291.87 398.37,297.7 398.61,296.73 398.7,299.64 398.78,306.44 398.86,307.42 398.94,304.5 399.19,311.3 399.28,317.13 399.36,316.16 399.44,315.19 399.52,310.33 399.77,311.3 399.94,310.33 400.02,306.44 400.1,305.47 400.35,312.27 400.51,321.02 400.6,327.82 400.68,327.82 400.93,327.82 401.01,323.94 401.09,322.96 401.18,325.88 401.26,322.96 401.51,322.96 401.59,320.05 401.67,319.08 401.75,323.94 401.84,321.02 402.17,320.05 402.25,320.05 402.33,321.99 402.42,332.68 402.66,333.65 402.75,336.57 402.83,337.54 402.91,336.57 402.99,340.46 403.24,336.57 403.32,340.46 403.41,336.57 403.49,335.6 403.57,336.57 403.82,335.6 403.9,338.51 403.99,343.37 404.07,336.57 404.15,337.54 404.48,340.46 404.56,339.48 404.65,336.57 404.73,334.63 404.98,326.85 405.06,326.85 405.14,329.77 405.22,328.8 405.31,325.88 405.56,327.82 405.64,329.77 405.72,334.63 405.8,339.48 405.89,346.29 406.13,346.29 406.22,342.4 406.3,342.4 406.38,344.34 406.46,342.4 406.71,336.57 406.79,332.68 406.88,337.54 406.96,334.63 407.04,332.68 407.29,333.65 407.37,332.68 407.46,328.8 407.54,328.8 407.87,327.82 407.95,329.77 408.03,323.94 408.12,318.11 408.2,308.39 408.45,312.27 408.53,315.19 408.61,321.02 408.7,320.05 408.78,312.27 409.03,308.39 409.11,311.3 409.19,309.36 409.27,307.42 409.36,308.39 409.6,308.39 409.69,310.33 409.77,308.39 409.85,310.33 409.93,305.47 410.18,307.42 410.27,307.42 410.35,302.56 410.43,302.56 410.51,316.16 410.76,318.11 410.84,320.05 410.93,319.08 411.01,318.11 411.09,327.82 411.34,330.74 411.42,334.63 411.5,333.65 411.59,326.85 411.67,334.63 411.92,336.57 412,332.68 412.08,340.46 412.17,339.48 412.25,337.54 412.58,352.12 412.66,348.23 412.74,349.2 412.83,352.12 413.07,349.2 413.16,352.12 413.24,347.26 413.32,346.29 413.4,355.03 413.65,359.89 413.74,356.98 413.82,360.86 413.9,356.01 413.98,352.12 414.23,357.95 414.31,356.98 414.4,365.72 414.48,374.47 414.56,382.24 414.81,383.22 414.89,387.1 414.97,383.22 415.06,378.36 415.14,378.36 415.39,375.44 415.47,374.47 415.55,378.36 415.72,396.82 415.97,390.99 416.05,391.96 416.13,395.85 416.21,389.05 416.3,390.02 416.54,386.13 416.63,384.19 416.71,383.22 416.79,388.07 416.88,381.27 417.12,381.27 417.21,383.22 417.29,390.02 417.37,390.02 417.45,387.1 417.7,390.99" stroke-dasharray="none" stroke="rgb(165,42,42)" stroke-width="0.75" opacity="1" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.panel.7.1.off.vp.2" class="pushedvp viewport">
          <g id="plot_01.border.panel.7.1.1" class="rect grob gDesc">
            <rect id="plot_01.border.panel.7.1.1.1" x="367.04" y="50.8" width="53.98" height="396.61" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.7.1.vp.1" class="pushedvp viewport">
          <defs>
            <clipPath id="plot_01.toplevel.vp::plot_01.strip.7.1.vp::plot_01.strip.default.1.1.clipPath">
              <rect x="367.04" y="447.41" width="53.98" height="14.4" fill="none" stroke="none"/>
            </clipPath>
          </defs>
          <g id="plot_01.toplevel.vp::plot_01.strip.7.1.vp::plot_01.strip.default.1.1" clip-path="url(#plot_01.toplevel.vp::plot_01.strip.7.1.vp::plot_01.strip.default.1.1.clipPath)" class="pushedvp viewport">
            <g id="plot_01.bg.strip.7.1.1" class="rect grob gDesc">
              <rect id="plot_01.bg.strip.7.1.1.1" x="367.04" y="447.41" width="53.98" height="14.4" fill="rgb(255,229,204)" stroke="rgb(255,229,204)" stroke-opacity="1" fill-opacity="1"/>
            </g>
            <g id="plot_01.textr.strip.7.1.1" class="text grob gDesc">
              <g id="plot_01.textr.strip.7.1.1.1" transform="translate(374.35, 454.61)" stroke-width="0.1">
                <g id="plot_01.textr.strip.7.1.1.1.scale" transform="scale(1, -1)">
                  <text x="0" y="0" id="plot_01.textr.strip.7.1.1.1.text" text-anchor="start" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                    <tspan id="plot_01.textr.strip.7.1.1.1.tspan.1" dy="4.31" x="0">DGS20</tspan>
                  </text>
                </g>
              </g>
            </g>
          </g>
          <g id="plot_01.toplevel.vp::plot_01.strip.7.1.vp::plot_01.strip.default.off.1.1" class="pushedvp viewport">
            <g id="plot_01.border.strip.7.1.1" class="rect grob gDesc">
              <rect id="plot_01.border.strip.7.1.1.1" x="367.04" y="447.41" width="53.98" height="14.4" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
            </g>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.8.1.vp.1.clipPath">
            <rect x="421.02" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.8.1.vp.1" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.8.1.vp.1.clipPath)" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.strip.8.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.top.panel.8.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.top.panel.8.1.1.1" points="439.29,461.81 439.29,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.top.panel.8.1.1.2" points="469.45,461.81 469.45,467.48" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticklabels.top.panel.8.1.1" class="text grob gDesc">
            <g id="plot_01.ticklabels.top.panel.8.1.1.1" transform="translate(439.29, 473.15)" stroke-width="0.1">
              <g id="plot_01.ticklabels.top.panel.8.1.1.1.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.top.panel.8.1.1.1.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.top.panel.8.1.1.1.tspan.1" dy="0" x="0">Jul 2012</tspan>
                </text>
              </g>
            </g>
            <g id="plot_01.ticklabels.top.panel.8.1.1.2" transform="translate(469.45, 473.15)" stroke-width="0.1">
              <g id="plot_01.ticklabels.top.panel.8.1.1.2.scale" transform="scale(1, -1)">
                <text x="0" y="0" id="plot_01.ticklabels.top.panel.8.1.1.2.text" text-anchor="middle" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="9.6" font-weight="normal" font-style="normal">
                  <tspan id="plot_01.ticklabels.top.panel.8.1.1.2.tspan.1" dy="0" x="0">Jul 2013</tspan>
                </text>
              </g>
            </g>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.left.8.1.off.vp.1" class="pushedvp viewport"/>
        <g id="plot_01.toplevel.vp::plot_01.panel.8.1.off.vp.1" class="pushedvp viewport">
          <g id="plot_01.ticks.bottom.panel.8.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.bottom.panel.8.1.1.1" points="439.29,50.8 439.29,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.bottom.panel.8.1.1.2" points="469.45,50.8 469.45,45.13" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
          <g id="plot_01.ticks.right.panel.8.1.1" class="segments grob gDesc">
            <polyline id="plot_01.ticks.right.panel.8.1.1.1" points="474.99,65.44 480.66,65.44" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.right.panel.8.1.1.2" points="474.99,162.62 480.66,162.62" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.right.panel.8.1.1.3" points="474.99,259.8 480.66,259.8" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
            <polyline id="plot_01.ticks.right.panel.8.1.1.4" points="474.99,356.98 480.66,356.98" stroke="rgb(0,0,0)" opacity="1" stroke-dasharray="none" stroke-width="0.75" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <defs>
          <clipPath id="plot_01.toplevel.vp::plot_01.panel.8.1.vp.2.clipPath">
            <rect x="421.02" y="50.8" width="53.98" height="396.61" fill="none" stroke="none"/>
          </clipPath>
        </defs>
        <g id="plot_01.toplevel.vp::plot_01.panel.8.1.vp.2" clip-path="url(#plot_01.toplevel.vp::plot_01.panel.8.1.vp.2.clipPath)" class="pushedvp viewport">
          <g id="plot_01.xyplot.lines.group.8.panel.8.1.1" class="lines grob gDesc">
            <polyline id="plot_01.xyplot.lines.group.8.panel.8.1.1.1" points="424.33,355.03 424.41,359.89 424.5,362.81 424.58,358.92 424.83,358.92 424.91,360.86 424.99,353.09 425.07,354.06 425.16,348.23 425.49,346.29 425.57,353.09 425.65,361.84 425.74,366.69 425.98,371.55 426.07,371.55 426.15,369.61 426.23,366.69 426.31,363.78 426.56,356.01 426.64,351.15 426.73,357.95 426.81,357.95 426.89,369.61 427.14,364.75 427.22,370.58 427.31,370.58 427.39,376.41 427.47,367.67 427.72,370.58 427.8,362.81 427.88,365.72 427.97,370.58 428.05,372.53 428.38,376.41 428.46,371.55 428.54,369.61 428.63,366.69 428.88,360.86 428.96,363.78 429.04,364.75 429.12,371.55 429.21,367.67 429.45,369.61 429.54,364.75 429.62,368.64 429.7,374.47 429.78,375.44 430.03,373.5 430.11,382.24 430.2,398.76 430.28,396.82 430.36,396.82 430.61,403.62 430.69,401.68 430.78,393.91 430.86,392.93 430.94,387.1 431.19,389.05 431.27,385.16 431.35,387.1 431.44,383.22 431.52,390.99 431.77,390.99 431.85,396.82 431.93,392.93 432.02,388.07 432.1,377.38 432.35,374.47 432.43,369.61 432.51,374.47 432.59,378.36 432.68,370.58 432.92,368.64 433.01,371.55 433.09,369.61 433.17,368.64 433.25,368.64 433.5,364.75 433.59,368.64 433.67,371.55 433.75,369.61 433.83,368.64 434.08,368.64 434.16,372.53 434.25,367.67 434.33,368.64 434.41,363.78 434.66,363.78 434.74,359.89 434.82,359.89 434.91,363.78 434.99,358.92 435.24,352.12 435.32,348.23 435.4,347.26 435.49,337.54 435.57,337.54 435.82,337.54 435.9,345.32 435.98,338.51 436.06,343.37 436.15,342.4 436.48,342.4 436.56,329.77 436.64,324.91 436.73,311.3 436.97,314.22 437.06,321.02 437.14,330.74 437.22,332.68 437.3,334.63 437.55,328.8 437.63,334.63 437.72,327.82 437.8,330.74 437.88,327.82 438.13,324.91 438.21,330.74 438.3,329.77 438.38,325.88 438.46,332.68 438.71,326.85 438.79,328.8 438.87,327.82 438.96,324.91 439.04,333.65 439.29,326.85 439.37,331.71 439.53,329.77 439.62,323.94 439.86,320.05 439.95,318.11 440.03,318.11 440.11,315.19 440.2,316.16 440.44,314.22 440.53,317.13 440.61,317.13 440.69,319.08 440.77,313.25 441.02,310.33 441.1,305.47 441.19,304.5 441.27,307.42 441.35,321.02 441.6,316.16 441.68,314.22 441.77,318.11 441.85,313.25 441.93,322.96 442.18,322.96 442.26,329.77 442.34,332.68 442.43,335.6 442.51,331.71 442.76,331.71 442.84,339.48 442.92,347.26 443,353.09 443.09,350.17 443.34,350.17 443.42,347.26 443.5,339.48 443.58,336.57 443.67,336.57 443.91,333.65 444,332.68 444.08,334.63 444.16,332.68 444.24,325.88 444.57,326.85 444.66,327.82 444.74,337.54 444.82,338.51 445.07,340.46 445.15,341.43 445.24,349.2 445.32,352.12 445.4,365.72 445.65,359.89 445.73,356.98 445.81,354.06 445.9,353.09 445.98,352.12 446.23,348.23 446.31,343.37 446.39,336.57 446.48,340.46 446.56,339.48 446.81,338.51 446.89,338.51 446.97,339.48 447.05,346.29 447.14,353.09 447.47,350.17 447.55,346.29 447.63,343.37 447.71,340.46 447.96,342.4 448.05,348.23 448.13,355.03 448.21,358.92 448.29,351.15 448.54,352.12 448.62,348.23 448.71,350.17 448.79,355.03 448.87,349.2 449.12,344.34 449.28,342.4 449.37,346.29 449.45,348.23 449.7,345.32 449.78,349.2 449.86,340.46 449.95,334.63 450.03,332.68 450.36,329.77 450.44,330.74 450.52,329.77 450.61,330.74 450.85,333.65 450.94,339.48 451.02,340.46 451.19,340.46 451.43,337.54 451.52,336.57 451.6,336.57 451.68,336.57 451.76,338.51 452.01,337.54 452.09,335.6 452.18,335.6 452.26,333.65 452.34,338.51 452.59,337.54 452.67,340.46 452.76,347.26 452.84,347.26 452.92,344.34 453.17,351.15 453.25,356.98 453.33,356.01 453.42,355.03 453.5,350.17 453.75,351.15 453.91,351.15 453.99,346.29 454.08,345.32 454.33,352.12 454.49,360.86 454.57,368.64 454.66,366.69 454.9,366.69 454.99,362.81 455.07,362.81 455.15,364.75 455.23,361.84 455.48,361.84 455.56,358.92 455.65,357.95 455.73,362.81 455.81,359.89 456.14,358.92 456.23,358.92 456.31,360.86 456.39,370.58 456.64,371.55 456.72,374.47 456.8,375.44 456.89,373.5 456.97,377.38 457.22,373.5 457.3,377.38 457.38,374.47 457.47,373.5 457.55,373.5 457.8,372.53 457.88,375.44 457.96,379.33 458.04,373.5 458.13,374.47 458.46,377.38 458.54,376.41 458.62,373.5 458.7,371.55 458.95,364.75 459.04,364.75 459.12,367.67 459.2,366.69 459.28,362.81 459.53,364.75 459.61,366.69 459.7,371.55 459.78,376.41 459.86,381.27 460.11,382.24 460.19,378.36 460.27,378.36 460.36,381.27 460.44,378.36 460.69,374.47 460.77,369.61 460.85,375.44 460.94,371.55 461.02,369.61 461.27,370.58 461.35,369.61 461.43,365.72 461.51,366.69 461.84,364.75 461.93,366.69 462.01,361.84 462.09,356.01 462.18,344.34 462.42,348.23 462.51,351.15 462.59,357.95 462.67,357.95 462.75,349.2 463,345.32 463.08,348.23 463.17,346.29 463.25,344.34 463.33,345.32 463.58,345.32 463.66,347.26 463.75,346.29 463.83,348.23 463.91,344.34 464.16,345.32 464.24,345.32 464.32,340.46 464.41,339.48 464.49,353.09 464.74,355.03 464.82,356.98 464.9,356.01 464.98,357.95 465.07,366.69 465.32,369.61 465.4,373.5 465.48,372.53 465.56,365.72 465.65,373.5 465.89,374.47 465.98,370.58 466.06,377.38 466.14,376.41 466.22,374.47 466.55,387.1 466.64,383.22 466.72,384.19 466.8,386.13 467.05,383.22 467.13,386.13 467.22,381.27 467.3,379.33 467.38,389.05 467.63,391.96 467.71,389.05 467.79,392.93 467.88,389.05 467.96,384.19 468.21,390.99 468.29,390.02 468.37,396.82 468.46,404.59 468.54,411.4 468.79,411.4 468.87,415.28 468.95,413.34 469.03,409.45 469.12,407.51 469.36,403.62 469.45,402.65 469.53,404.59 469.69,423.06 469.94,418.2 470.03,419.17 470.11,423.06 470.19,419.17 470.27,419.17 470.52,416.26 470.6,413.34 470.69,412.37 470.77,418.2 470.85,411.4 471.1,410.43 471.18,413.34 471.26,420.14 471.35,420.14 471.43,416.26 471.68,421.12" stroke-dasharray="none" stroke="rgb(0,128,255)" stroke-width="0.75" opacity="1" stroke-opacity="1" fill="none"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.panel.8.1.off.vp.2" class="pushedvp viewport">
          <g id="plot_01.border.panel.8.1.1" class="rect grob gDesc">
            <rect id="plot_01.border.panel.8.1.1.1" x="421.02" y="50.8" width="53.98" height="396.61" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01.strip.8.1.vp.1" class="pushedvp viewport">
          <defs>
            <clipPath id="plot_01.toplevel.vp::plot_01.strip.8.1.vp::plot_01.strip.default.1.1.clipPath">
              <rect x="421.02" y="447.41" width="53.98" height="14.4" fill="none" stroke="none"/>
            </clipPath>
          </defs>
          <g id="plot_01.toplevel.vp::plot_01.strip.8.1.vp::plot_01.strip.default.1.1" clip-path="url(#plot_01.toplevel.vp::plot_01.strip.8.1.vp::plot_01.strip.default.1.1.clipPath)" class="pushedvp viewport">
            <g id="plot_01.bg.strip.8.1.1" class="rect grob gDesc">
              <rect id="plot_01.bg.strip.8.1.1.1" x="421.02" y="447.41" width="53.98" height="14.4" fill="rgb(255,229,204)" stroke="rgb(255,229,204)" stroke-opacity="1" fill-opacity="1"/>
            </g>
            <g id="plot_01.textr.strip.8.1.1" class="text grob gDesc">
              <g id="plot_01.textr.strip.8.1.1.1" transform="translate(428.33, 454.61)" stroke-width="0.1">
                <g id="plot_01.textr.strip.8.1.1.1.scale" transform="scale(1, -1)">
                  <text x="0" y="0" id="plot_01.textr.strip.8.1.1.1.text" text-anchor="start" stroke="rgb(0,0,0)" opacity="1" fill="rgb(0,0,0)" stroke-opacity="1" fill-opacity="1" font-size="12" font-weight="normal" font-style="normal">
                    <tspan id="plot_01.textr.strip.8.1.1.1.tspan.1" dy="4.31" x="0">DGS30</tspan>
                  </text>
                </g>
              </g>
            </g>
          </g>
          <g id="plot_01.toplevel.vp::plot_01.strip.8.1.vp::plot_01.strip.default.off.1.1" class="pushedvp viewport">
            <g id="plot_01.border.strip.8.1.1" class="rect grob gDesc">
              <rect id="plot_01.border.strip.8.1.1.1" x="421.02" y="447.41" width="53.98" height="14.4" stroke="rgb(0,0,0)" stroke-dasharray="none" stroke-width="0.75" opacity="1" fill="none" stroke-opacity="1" fill-opacity="0"/>
            </g>
          </g>
        </g>
        <g id="plot_01.toplevel.vp::plot_01..1" class="pushedvp viewport"/>
      </g>
    </g>
  </g>
</svg>

</div>

<script>
var data = 
{"strips":{"indexname":["DGS1","DGS2","DGS3","DGS5","DGS7","DGS10","DGS20","DGS30"]},"groups":["DGS1","DGS2","DGS3","DGS5","DGS7","DGS10","DGS20","DGS30"],"data":{"DGS1":{"x":[15341,15342,15343,15344,15347,15348,15349,15350,15351,15355,15356,15357,15358,15361,15362,15363,15364,15365,15368,15369,15370,15371,15372,15375,15376,15377,15378,15379,15382,15383,15384,15385,15386,15390,15391,15392,15393,15396,15397,15398,15399,15400,15403,15404,15405,15406,15407,15410,15411,15412,15413,15414,15417,15418,15419,15420,15421,15424,15425,15426,15427,15428,15431,15432,15433,15434,15435,15438,15439,15440,15441,15442,15445,15446,15447,15448,15449,15452,15453,15454,15455,15456,15459,15460,15461,15462,15463,15466,15467,15468,15469,15470,15473,15474,15475,15476,15477,15480,15481,15482,15483,15484,15488,15489,15490,15491,15494,15495,15496,15497,15498,15501,15502,15503,15504,15505,15508,15509,15510,15511,15512,15515,15516,15517,15518,15519,15522,15523,15525,15526,15529,15530,15531,15532,15533,15536,15537,15538,15539,15540,15543,15544,15545,15546,15547,15550,15551,15552,15553,15554,15557,15558,15559,15560,15561,15564,15565,15566,15567,15568,15571,15572,15573,15574,15575,15578,15579,15580,15581,15582,15586,15587,15588,15589,15592,15593,15594,15595,15596,15599,15600,15601,15602,15603,15606,15607,15608,15609,15610,15613,15614,15615,15616,15617,15621,15622,15623,15624,15627,15628,15629,15630,15631,15634,15635,15636,15637,15638,15641,15643,15644,15645,15648,15649,15650,15651,15652,15656,15657,15658,15659,15662,15663,15664,15666,15669,15670,15671,15672,15673,15676,15677,15678,15679,15680,15683,15684,15685,15686,15687,15690,15691,15692,15693,15694,15697,15699,15700,15701,15704,15706,15707,15708,15711,15712,15713,15714,15715,15718,15719,15720,15721,15722,15726,15727,15728,15729,15732,15733,15734,15735,15736,15739,15740,15741,15742,15743,15746,15747,15748,15749,15750,15754,15755,15756,15757,15760,15761,15762,15763,15764,15767,15768,15769,15770,15771,15774,15775,15776,15777,15778,15781,15782,15783,15784,15785,15788,15789,15790,15791,15795,15796,15797,15798,15799,15802,15803,15804,15805,15806,15809,15810,15811,15812,15813,15816,15817,15818,15819,15820,15823,15824,15825,15826,15827,15830,15831,15832,15833,15834,15837,15838,15839,15840,15841,15844,15845,15846,15847,15848,15852,15853,15854,15855,15858,15859,15860,15861,15862,15865,15866,15867,15868,15869,15872,15873,15874,15875,15876,15879,15880,15881,15882,15883,15886,15887,15888,15890,15893,15894,15895,15896,15897,15900,15901,15902,15903,15904,15907,15908,15909,15910,15911,15914],"y":[0.12,0.12,0.11,0.12,0.11,0.11,0.11,0.11,0.1,0.11,0.11,0.11,0.11,0.12,0.12,0.12,0.12,0.12,0.12,0.13,0.13,0.14,0.14,0.14,0.14,0.15,0.15,0.15,0.15,0.18,0.18,0.17,0.18,0.17,0.17,0.17,0.18,0.17,0.18,0.18,0.18,0.17,0.17,0.17,0.18,0.18,0.18,0.18,0.2,0.21,0.21,0.21,0.21,0.22,0.21,0.19,0.19,0.19,0.18,0.18,0.18,0.19,0.18,0.2,0.19,0.19,0.19,0.19,0.19,0.18,0.18,0.17,0.18,0.18,0.18,0.17,0.18,0.17,0.18,0.18,0.18,0.19,0.2,0.19,0.18,0.19,0.18,0.18,0.18,0.18,0.18,0.18,0.19,0.19,0.2,0.2,0.2,0.21,0.21,0.2,0.21,0.2,0.2,0.19,0.18,0.17,0.18,0.18,0.18,0.18,0.19,0.18,0.19,0.18,0.18,0.18,0.18,0.18,0.2,0.19,0.19,0.19,0.21,0.21,0.22,0.21,0.21,0.21,0.19,0.2,0.2,0.2,0.2,0.2,0.2,0.18,0.18,0.18,0.17,0.17,0.17,0.18,0.17,0.18,0.17,0.18,0.16,0.17,0.17,0.16,0.16,0.19,0.19,0.2,0.18,0.19,0.19,0.19,0.2,0.2,0.19,0.2,0.19,0.19,0.19,0.18,0.18,0.18,0.17,0.16,0.16,0.17,0.18,0.18,0.18,0.18,0.18,0.17,0.18,0.18,0.18,0.18,0.18,0.18,0.18,0.18,0.17,0.16,0.17,0.17,0.16,0.16,0.18,0.18,0.18,0.18,0.18,0.18,0.19,0.18,0.18,0.18,0.18,0.19,0.18,0.18,0.19,0.19,0.18,0.18,0.18,0.19,0.19,0.19,0.18,0.2,0.18,0.18,0.18,0.17,0.16,0.16,0.16,0.17,0.19,0.17,0.18,0.18,0.18,0.18,0.18,0.18,0.18,0.18,0.18,0.18,0.16,0.14,0.14,0.13,0.13,0.16,0.15,0.15,0.15,0.16,0.16,0.15,0.15,0.16,0.15,0.15,0.15,0.15,0.14,0.13,0.14,0.14,0.14,0.14,0.14,0.14,0.14,0.14,0.15,0.15,0.15,0.16,0.15,0.15,0.15,0.15,0.15,0.15,0.15,0.15,0.14,0.15,0.14,0.15,0.16,0.17,0.17,0.17,0.16,0.16,0.16,0.17,0.17,0.17,0.16,0.16,0.15,0.15,0.15,0.15,0.15,0.15,0.15,0.15,0.14,0.15,0.15,0.15,0.14,0.14,0.14,0.14,0.14,0.14,0.14,0.14,0.13,0.13,0.13,0.13,0.13,0.12,0.12,0.11,0.12,0.13,0.13,0.12,0.12,0.12,0.12,0.13,0.12,0.12,0.12,0.11,0.11,0.11,0.11,0.11,0.1,0.11,0.11,0.11,0.13,0.12,0.12,0.12,0.12,0.12,0.12,0.11,0.12,0.12,0.13,0.14,0.13,0.14,0.14,0.14,0.14,0.14,0.14,0.14,0.14,0.14,0.14,0.13,0.13,0.13,0.13,0.14,0.13,0.16,0.17,0.16,0.15,0.15,0.15,0.14,0.14,0.15,0.14,0.14,0.13,0.13,0.12,0.11,0.1,0.11,0.11,0.11,0.1,0.12,0.12,0.12,0.11,0.11],"subscripts":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271,272,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,288,289,290,291,292,293,294,295,296,297,298,299,300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394],"groups":["DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1","DGS1"]},"DGS2":{"x":[15341,15342,15343,15344,15347,15348,15349,15350,15351,15355,15356,15357,15358,15361,15362,15363,15364,15365,15368,15369,15370,15371,15372,15375,15376,15377,15378,15379,15382,15383,15384,15385,15386,15390,15391,15392,15393,15396,15397,15398,15399,15400,15403,15404,15405,15406,15407,15410,15411,15412,15413,15414,15417,15418,15419,15420,15421,15424,15425,15426,15427,15428,15431,15432,15433,15434,15435,15438,15439,15440,15441,15442,15445,15446,15447,15448,15449,15452,15453,15454,15455,15456,15459,15460,15461,15462,15463,15466,15467,15468,15469,15470,15473,15474,15475,15476,15477,15480,15481,15482,15483,15484,15488,15489,15490,15491,15494,15495,15496,15497,15498,15501,15502,15503,15504,15505,15508,15509,15510,15511,15512,15515,15516,15517,15518,15519,15522,15523,15525,15526,15529,15530,15531,15532,15533,15536,15537,15538,15539,15540,15543,15544,15545,15546,15547,15550,15551,15552,15553,15554,15557,15558,15559,15560,15561,15564,15565,15566,15567,15568,15571,15572,15573,15574,15575,15578,15579,15580,15581,15582,15586,15587,15588,15589,15592,15593,15594,15595,15596,15599,15600,15601,15602,15603,15606,15607,15608,15609,15610,15613,15614,15615,15616,15617,15621,15622,15623,15624,15627,15628,15629,15630,15631,15634,15635,15636,15637,15638,15641,15643,15644,15645,15648,15649,15650,15651,15652,15656,15657,15658,15659,15662,15663,15664,15666,15669,15670,15671,15672,15673,15676,15677,15678,15679,15680,15683,15684,15685,15686,15687,15690,15691,15692,15693,15694,15697,15699,15700,15701,15704,15706,15707,15708,15711,15712,15713,15714,15715,15718,15719,15720,15721,15722,15726,15727,15728,15729,15732,15733,15734,15735,15736,15739,15740,15741,15742,15743,15746,15747,15748,15749,15750,15754,15755,15756,15757,15760,15761,15762,15763,15764,15767,15768,15769,15770,15771,15774,15775,15776,15777,15778,15781,15782,15783,15784,15785,15788,15789,15790,15791,15795,15796,15797,15798,15799,15802,15803,15804,15805,15806,15809,15810,15811,15812,15813,15816,15817,15818,15819,15820,15823,15824,15825,15826,15827,15830,15831,15832,15833,15834,15837,15838,15839,15840,15841,15844,15845,15846,15847,15848,15852,15853,15854,15855,15858,15859,15860,15861,15862,15865,15866,15867,15868,15869,15872,15873,15874,15875,15876,15879,15880,15881,15882,15883,15886,15887,15888,15890,15893,15894,15895,15896,15897,15900,15901,15902,15903,15904,15907,15908,15909,15910,15911,15914],"y":[0.27,0.25,0.27,0.25,0.26,0.24,0.24,0.22,0.24,0.21,0.24,0.26,0.26,0.26,0.24,0.22,0.22,0.22,0.22,0.22,0.23,0.23,0.23,0.24,0.25,0.27,0.27,0.27,0.29,0.29,0.29,0.29,0.29,0.31,0.29,0.31,0.31,0.3,0.3,0.3,0.3,0.28,0.31,0.3,0.3,0.32,0.33,0.33,0.35,0.4,0.37,0.37,0.39,0.41,0.39,0.37,0.37,0.36,0.33,0.34,0.33,0.33,0.33,0.36,0.35,0.35,0.32,0.32,0.28,0.3,0.29,0.27,0.27,0.27,0.27,0.27,0.29,0.27,0.27,0.26,0.26,0.26,0.27,0.27,0.27,0.28,0.27,0.27,0.27,0.27,0.27,0.27,0.29,0.29,0.3,0.32,0.32,0.3,0.3,0.28,0.29,0.3,0.3,0.27,0.27,0.25,0.25,0.25,0.26,0.27,0.28,0.27,0.3,0.3,0.3,0.29,0.29,0.3,0.32,0.32,0.31,0.31,0.31,0.31,0.31,0.33,0.3,0.3,0.28,0.27,0.27,0.27,0.27,0.25,0.25,0.24,0.25,0.22,0.22,0.22,0.22,0.22,0.22,0.23,0.25,0.23,0.23,0.24,0.24,0.24,0.24,0.27,0.29,0.29,0.27,0.27,0.27,0.27,0.29,0.29,0.29,0.31,0.26,0.26,0.28,0.28,0.27,0.27,0.27,0.22,0.23,0.25,0.27,0.25,0.25,0.25,0.25,0.24,0.27,0.25,0.25,0.27,0.27,0.27,0.27,0.27,0.26,0.25,0.23,0.25,0.23,0.23,0.23,0.27,0.25,0.27,0.28,0.27,0.27,0.27,0.3,0.29,0.3,0.32,0.29,0.29,0.31,0.3,0.3,0.3,0.3,0.28,0.28,0.3,0.27,0.27,0.27,0.27,0.25,0.24,0.24,0.25,0.27,0.27,0.29,0.27,0.27,0.27,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.24,0.24,0.25,0.27,0.24,0.25,0.28,0.28,0.28,0.26,0.26,0.26,0.26,0.27,0.25,0.27,0.27,0.27,0.27,0.25,0.24,0.26,0.26,0.26,0.26,0.26,0.28,0.26,0.26,0.26,0.23,0.28,0.29,0.3,0.27,0.27,0.27,0.25,0.27,0.27,0.25,0.25,0.27,0.29,0.29,0.27,0.29,0.29,0.27,0.26,0.27,0.25,0.25,0.27,0.25,0.25,0.24,0.25,0.25,0.25,0.27,0.27,0.27,0.27,0.27,0.25,0.26,0.24,0.26,0.27,0.26,0.24,0.25,0.25,0.25,0.23,0.25,0.24,0.22,0.24,0.24,0.24,0.24,0.24,0.22,0.22,0.24,0.24,0.24,0.24,0.24,0.23,0.23,0.23,0.22,0.2,0.22,0.2,0.2,0.22,0.22,0.22,0.22,0.22,0.26,0.24,0.26,0.26,0.23,0.26,0.26,0.26,0.26,0.26,0.26,0.29,0.3,0.31,0.3,0.3,0.32,0.3,0.3,0.32,0.32,0.34,0.34,0.32,0.29,0.27,0.27,0.31,0.33,0.38,0.42,0.43,0.39,0.36,0.36,0.34,0.34,0.36,0.4,0.37,0.37,0.38,0.34,0.37,0.34,0.34,0.32,0.32,0.32,0.32,0.33,0.34,0.32,0.31,0.33],"subscripts":[395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788],"groups":["DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2","DGS2"]},"DGS3":{"x":[15341,15342,15343,15344,15347,15348,15349,15350,15351,15355,15356,15357,15358,15361,15362,15363,15364,15365,15368,15369,15370,15371,15372,15375,15376,15377,15378,15379,15382,15383,15384,15385,15386,15390,15391,15392,15393,15396,15397,15398,15399,15400,15403,15404,15405,15406,15407,15410,15411,15412,15413,15414,15417,15418,15419,15420,15421,15424,15425,15426,15427,15428,15431,15432,15433,15434,15435,15438,15439,15440,15441,15442,15445,15446,15447,15448,15449,15452,15453,15454,15455,15456,15459,15460,15461,15462,15463,15466,15467,15468,15469,15470,15473,15474,15475,15476,15477,15480,15481,15482,15483,15484,15488,15489,15490,15491,15494,15495,15496,15497,15498,15501,15502,15503,15504,15505,15508,15509,15510,15511,15512,15515,15516,15517,15518,15519,15522,15523,15525,15526,15529,15530,15531,15532,15533,15536,15537,15538,15539,15540,15543,15544,15545,15546,15547,15550,15551,15552,15553,15554,15557,15558,15559,15560,15561,15564,15565,15566,15567,15568,15571,15572,15573,15574,15575,15578,15579,15580,15581,15582,15586,15587,15588,15589,15592,15593,15594,15595,15596,15599,15600,15601,15602,15603,15606,15607,15608,15609,15610,15613,15614,15615,15616,15617,15621,15622,15623,15624,15627,15628,15629,15630,15631,15634,15635,15636,15637,15638,15641,15643,15644,15645,15648,15649,15650,15651,15652,15656,15657,15658,15659,15662,15663,15664,15666,15669,15670,15671,15672,15673,15676,15677,15678,15679,15680,15683,15684,15685,15686,15687,15690,15691,15692,15693,15694,15697,15699,15700,15701,15704,15706,15707,15708,15711,15712,15713,15714,15715,15718,15719,15720,15721,15722,15726,15727,15728,15729,15732,15733,15734,15735,15736,15739,15740,15741,15742,15743,15746,15747,15748,15749,15750,15754,15755,15756,15757,15760,15761,15762,15763,15764,15767,15768,15769,15770,15771,15774,15775,15776,15777,15778,15781,15782,15783,15784,15785,15788,15789,15790,15791,15795,15796,15797,15798,15799,15802,15803,15804,15805,15806,15809,15810,15811,15812,15813,15816,15817,15818,15819,15820,15823,15824,15825,15826,15827,15830,15831,15832,15833,15834,15837,15838,15839,15840,15841,15844,15845,15846,15847,15848,15852,15853,15854,15855,15858,15859,15860,15861,15862,15865,15866,15867,15868,15869,15872,15873,15874,15875,15876,15879,15880,15881,15882,15883,15886,15887,15888,15890,15893,15894,15895,15896,15897,15900,15901,15902,15903,15904,15907,15908,15909,15910,15911,15914],"y":[0.4,0.4,0.4,0.4,0.38,0.37,0.34,0.35,0.34,0.33,0.35,0.36,0.38,0.39,0.39,0.34,0.31,0.32,0.31,0.3,0.31,0.31,0.33,0.32,0.35,0.35,0.38,0.36,0.4,0.4,0.38,0.42,0.42,0.44,0.42,0.43,0.43,0.4,0.41,0.43,0.43,0.41,0.43,0.4,0.42,0.44,0.46,0.47,0.51,0.6,0.56,0.57,0.6,0.62,0.58,0.56,0.55,0.54,0.5,0.51,0.5,0.51,0.5,0.56,0.53,0.5,0.45,0.46,0.42,0.43,0.43,0.41,0.42,0.42,0.4,0.4,0.4,0.39,0.4,0.39,0.39,0.39,0.38,0.39,0.39,0.4,0.37,0.37,0.36,0.36,0.37,0.36,0.37,0.38,0.4,0.4,0.42,0.41,0.41,0.4,0.42,0.41,0.42,0.38,0.35,0.34,0.35,0.34,0.37,0.37,0.39,0.37,0.41,0.4,0.41,0.37,0.38,0.39,0.41,0.41,0.42,0.39,0.42,0.42,0.4,0.41,0.39,0.39,0.39,0.37,0.36,0.37,0.36,0.35,0.34,0.31,0.32,0.3,0.31,0.29,0.28,0.28,0.28,0.31,0.34,0.31,0.3,0.32,0.31,0.33,0.33,0.37,0.38,0.38,0.36,0.36,0.39,0.42,0.42,0.42,0.41,0.42,0.37,0.36,0.37,0.37,0.36,0.36,0.35,0.3,0.31,0.32,0.34,0.33,0.33,0.33,0.33,0.32,0.35,0.36,0.35,0.35,0.36,0.36,0.35,0.35,0.34,0.34,0.31,0.31,0.31,0.31,0.32,0.34,0.35,0.35,0.34,0.34,0.34,0.36,0.41,0.41,0.41,0.42,0.41,0.4,0.43,0.41,0.4,0.38,0.38,0.38,0.38,0.41,0.36,0.35,0.35,0.33,0.33,0.32,0.32,0.33,0.36,0.37,0.37,0.36,0.36,0.35,0.35,0.34,0.34,0.34,0.32,0.32,0.33,0.33,0.32,0.32,0.34,0.34,0.37,0.39,0.39,0.39,0.38,0.38,0.39,0.37,0.36,0.36,0.37,0.4,0.41,0.41,0.38,0.37,0.37,0.37,0.37,0.36,0.36,0.39,0.38,0.38,0.37,0.37,0.42,0.45,0.43,0.42,0.42,0.4,0.38,0.41,0.39,0.39,0.39,0.4,0.41,0.44,0.42,0.42,0.44,0.42,0.4,0.4,0.37,0.37,0.36,0.36,0.35,0.35,0.36,0.38,0.4,0.42,0.43,0.41,0.42,0.42,0.4,0.38,0.37,0.38,0.38,0.39,0.38,0.38,0.36,0.36,0.36,0.36,0.34,0.33,0.33,0.34,0.34,0.36,0.35,0.33,0.32,0.33,0.35,0.35,0.35,0.35,0.35,0.34,0.35,0.32,0.32,0.32,0.3,0.3,0.34,0.34,0.35,0.35,0.35,0.38,0.4,0.41,0.4,0.37,0.4,0.4,0.39,0.41,0.42,0.41,0.49,0.49,0.49,0.52,0.5,0.48,0.48,0.48,0.52,0.55,0.57,0.57,0.55,0.49,0.49,0.48,0.58,0.62,0.7,0.73,0.74,0.69,0.66,0.66,0.65,0.64,0.67,0.77,0.71,0.71,0.73,0.65,0.66,0.66,0.64,0.6,0.61,0.59,0.59,0.6,0.64,0.62,0.59,0.61],"subscripts":[789,790,791,792,793,794,795,796,797,798,799,800,801,802,803,804,805,806,807,808,809,810,811,812,813,814,815,816,817,818,819,820,821,822,823,824,825,826,827,828,829,830,831,832,833,834,835,836,837,838,839,840,841,842,843,844,845,846,847,848,849,850,851,852,853,854,855,856,857,858,859,860,861,862,863,864,865,866,867,868,869,870,871,872,873,874,875,876,877,878,879,880,881,882,883,884,885,886,887,888,889,890,891,892,893,894,895,896,897,898,899,900,901,902,903,904,905,906,907,908,909,910,911,912,913,914,915,916,917,918,919,920,921,922,923,924,925,926,927,928,929,930,931,932,933,934,935,936,937,938,939,940,941,942,943,944,945,946,947,948,949,950,951,952,953,954,955,956,957,958,959,960,961,962,963,964,965,966,967,968,969,970,971,972,973,974,975,976,977,978,979,980,981,982,983,984,985,986,987,988,989,990,991,992,993,994,995,996,997,998,999,1000,1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1011,1012,1013,1014,1015,1016,1017,1018,1019,1020,1021,1022,1023,1024,1025,1026,1027,1028,1029,1030,1031,1032,1033,1034,1035,1036,1037,1038,1039,1040,1041,1042,1043,1044,1045,1046,1047,1048,1049,1050,1051,1052,1053,1054,1055,1056,1057,1058,1059,1060,1061,1062,1063,1064,1065,1066,1067,1068,1069,1070,1071,1072,1073,1074,1075,1076,1077,1078,1079,1080,1081,1082,1083,1084,1085,1086,1087,1088,1089,1090,1091,1092,1093,1094,1095,1096,1097,1098,1099,1100,1101,1102,1103,1104,1105,1106,1107,1108,1109,1110,1111,1112,1113,1114,1115,1116,1117,1118,1119,1120,1121,1122,1123,1124,1125,1126,1127,1128,1129,1130,1131,1132,1133,1134,1135,1136,1137,1138,1139,1140,1141,1142,1143,1144,1145,1146,1147,1148,1149,1150,1151,1152,1153,1154,1155,1156,1157,1158,1159,1160,1161,1162,1163,1164,1165,1166,1167,1168,1169,1170,1171,1172,1173,1174,1175,1176,1177,1178,1179,1180,1181,1182],"groups":["DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3","DGS3"]},"DGS5":{"x":[15341,15342,15343,15344,15347,15348,15349,15350,15351,15355,15356,15357,15358,15361,15362,15363,15364,15365,15368,15369,15370,15371,15372,15375,15376,15377,15378,15379,15382,15383,15384,15385,15386,15390,15391,15392,15393,15396,15397,15398,15399,15400,15403,15404,15405,15406,15407,15410,15411,15412,15413,15414,15417,15418,15419,15420,15421,15424,15425,15426,15427,15428,15431,15432,15433,15434,15435,15438,15439,15440,15441,15442,15445,15446,15447,15448,15449,15452,15453,15454,15455,15456,15459,15460,15461,15462,15463,15466,15467,15468,15469,15470,15473,15474,15475,15476,15477,15480,15481,15482,15483,15484,15488,15489,15490,15491,15494,15495,15496,15497,15498,15501,15502,15503,15504,15505,15508,15509,15510,15511,15512,15515,15516,15517,15518,15519,15522,15523,15525,15526,15529,15530,15531,15532,15533,15536,15537,15538,15539,15540,15543,15544,15545,15546,15547,15550,15551,15552,15553,15554,15557,15558,15559,15560,15561,15564,15565,15566,15567,15568,15571,15572,15573,15574,15575,15578,15579,15580,15581,15582,15586,15587,15588,15589,15592,15593,15594,15595,15596,15599,15600,15601,15602,15603,15606,15607,15608,15609,15610,15613,15614,15615,15616,15617,15621,15622,15623,15624,15627,15628,15629,15630,15631,15634,15635,15636,15637,15638,15641,15643,15644,15645,15648,15649,15650,15651,15652,15656,15657,15658,15659,15662,15663,15664,15666,15669,15670,15671,15672,15673,15676,15677,15678,15679,15680,15683,15684,15685,15686,15687,15690,15691,15692,15693,15694,15697,15699,15700,15701,15704,15706,15707,15708,15711,15712,15713,15714,15715,15718,15719,15720,15721,15722,15726,15727,15728,15729,15732,15733,15734,15735,15736,15739,15740,15741,15742,15743,15746,15747,15748,15749,15750,15754,15755,15756,15757,15760,15761,15762,15763,15764,15767,15768,15769,15770,15771,15774,15775,15776,15777,15778,15781,15782,15783,15784,15785,15788,15789,15790,15791,15795,15796,15797,15798,15799,15802,15803,15804,15805,15806,15809,15810,15811,15812,15813,15816,15817,15818,15819,15820,15823,15824,15825,15826,15827,15830,15831,15832,15833,15834,15837,15838,15839,15840,15841,15844,15845,15846,15847,15848,15852,15853,15854,15855,15858,15859,15860,15861,15862,15865,15866,15867,15868,15869,15872,15873,15874,15875,15876,15879,15880,15881,15882,15883,15886,15887,15888,15890,15893,15894,15895,15896,15897,15900,15901,15902,15903,15904,15907,15908,15909,15910,15911,15914],"y":[0.89,0.89,0.88,0.86,0.85,0.86,0.82,0.84,0.8,0.79,0.82,0.87,0.91,0.93,0.92,0.81,0.77,0.75,0.73,0.71,0.72,0.71,0.78,0.76,0.82,0.82,0.86,0.81,0.85,0.81,0.81,0.87,0.88,0.92,0.88,0.88,0.89,0.84,0.84,0.87,0.89,0.84,0.87,0.83,0.85,0.89,0.9,0.92,0.99,1.13,1.11,1.13,1.2,1.22,1.15,1.13,1.1,1.09,1.04,1.05,1.01,1.04,1.03,1.1,1.05,1.01,0.89,0.9,0.85,0.89,0.9,0.86,0.85,0.88,0.86,0.84,0.86,0.83,0.86,0.86,0.83,0.82,0.82,0.84,0.82,0.82,0.78,0.79,0.77,0.77,0.79,0.75,0.73,0.74,0.75,0.74,0.75,0.75,0.78,0.74,0.77,0.76,0.76,0.69,0.67,0.62,0.68,0.68,0.73,0.72,0.71,0.69,0.75,0.71,0.73,0.68,0.69,0.71,0.74,0.73,0.76,0.72,0.75,0.73,0.69,0.72,0.67,0.69,0.68,0.64,0.63,0.63,0.64,0.63,0.63,0.6,0.62,0.6,0.62,0.59,0.57,0.57,0.56,0.58,0.65,0.61,0.6,0.63,0.61,0.67,0.65,0.71,0.73,0.74,0.71,0.71,0.75,0.8,0.83,0.81,0.8,0.8,0.71,0.71,0.72,0.7,0.69,0.69,0.66,0.59,0.62,0.62,0.68,0.64,0.66,0.67,0.7,0.65,0.72,0.73,0.71,0.7,0.7,0.68,0.68,0.66,0.63,0.64,0.62,0.62,0.61,0.61,0.63,0.67,0.67,0.66,0.67,0.67,0.67,0.7,0.78,0.79,0.77,0.79,0.77,0.76,0.82,0.76,0.74,0.72,0.73,0.73,0.7,0.75,0.67,0.65,0.65,0.63,0.63,0.62,0.62,0.64,0.67,0.69,0.7,0.68,0.66,0.64,0.63,0.61,0.63,0.63,0.61,0.6,0.63,0.62,0.64,0.66,0.7,0.7,0.74,0.78,0.77,0.77,0.75,0.77,0.76,0.72,0.72,0.72,0.76,0.81,0.82,0.82,0.79,0.77,0.8,0.78,0.78,0.75,0.75,0.79,0.77,0.76,0.76,0.78,0.87,0.89,0.9,0.88,0.88,0.88,0.85,0.88,0.84,0.83,0.84,0.85,0.88,0.92,0.86,0.87,0.89,0.88,0.86,0.84,0.78,0.78,0.78,0.77,0.75,0.76,0.77,0.81,0.85,0.9,0.9,0.88,0.89,0.88,0.84,0.81,0.79,0.81,0.81,0.8,0.8,0.79,0.76,0.77,0.76,0.78,0.73,0.69,0.68,0.71,0.7,0.74,0.74,0.7,0.69,0.71,0.71,0.71,0.72,0.7,0.71,0.7,0.71,0.68,0.68,0.68,0.65,0.65,0.73,0.74,0.75,0.75,0.75,0.82,0.83,0.85,0.84,0.79,0.84,0.85,0.84,0.91,0.91,0.9,1.02,1.02,1.01,1.05,1.03,1.05,1.02,1.01,1.1,1.13,1.12,1.15,1.11,1.04,1.06,1.07,1.24,1.31,1.42,1.48,1.49,1.45,1.38,1.41,1.39,1.38,1.42,1.6,1.51,1.5,1.54,1.4,1.43,1.4,1.38,1.33,1.35,1.31,1.32,1.33,1.4,1.38,1.36,1.37],"subscripts":[1183,1184,1185,1186,1187,1188,1189,1190,1191,1192,1193,1194,1195,1196,1197,1198,1199,1200,1201,1202,1203,1204,1205,1206,1207,1208,1209,1210,1211,1212,1213,1214,1215,1216,1217,1218,1219,1220,1221,1222,1223,1224,1225,1226,1227,1228,1229,1230,1231,1232,1233,1234,1235,1236,1237,1238,1239,1240,1241,1242,1243,1244,1245,1246,1247,1248,1249,1250,1251,1252,1253,1254,1255,1256,1257,1258,1259,1260,1261,1262,1263,1264,1265,1266,1267,1268,1269,1270,1271,1272,1273,1274,1275,1276,1277,1278,1279,1280,1281,1282,1283,1284,1285,1286,1287,1288,1289,1290,1291,1292,1293,1294,1295,1296,1297,1298,1299,1300,1301,1302,1303,1304,1305,1306,1307,1308,1309,1310,1311,1312,1313,1314,1315,1316,1317,1318,1319,1320,1321,1322,1323,1324,1325,1326,1327,1328,1329,1330,1331,1332,1333,1334,1335,1336,1337,1338,1339,1340,1341,1342,1343,1344,1345,1346,1347,1348,1349,1350,1351,1352,1353,1354,1355,1356,1357,1358,1359,1360,1361,1362,1363,1364,1365,1366,1367,1368,1369,1370,1371,1372,1373,1374,1375,1376,1377,1378,1379,1380,1381,1382,1383,1384,1385,1386,1387,1388,1389,1390,1391,1392,1393,1394,1395,1396,1397,1398,1399,1400,1401,1402,1403,1404,1405,1406,1407,1408,1409,1410,1411,1412,1413,1414,1415,1416,1417,1418,1419,1420,1421,1422,1423,1424,1425,1426,1427,1428,1429,1430,1431,1432,1433,1434,1435,1436,1437,1438,1439,1440,1441,1442,1443,1444,1445,1446,1447,1448,1449,1450,1451,1452,1453,1454,1455,1456,1457,1458,1459,1460,1461,1462,1463,1464,1465,1466,1467,1468,1469,1470,1471,1472,1473,1474,1475,1476,1477,1478,1479,1480,1481,1482,1483,1484,1485,1486,1487,1488,1489,1490,1491,1492,1493,1494,1495,1496,1497,1498,1499,1500,1501,1502,1503,1504,1505,1506,1507,1508,1509,1510,1511,1512,1513,1514,1515,1516,1517,1518,1519,1520,1521,1522,1523,1524,1525,1526,1527,1528,1529,1530,1531,1532,1533,1534,1535,1536,1537,1538,1539,1540,1541,1542,1543,1544,1545,1546,1547,1548,1549,1550,1551,1552,1553,1554,1555,1556,1557,1558,1559,1560,1561,1562,1563,1564,1565,1566,1567,1568,1569,1570,1571,1572,1573,1574,1575,1576],"groups":["DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5","DGS5"]},"DGS7":{"x":[15341,15342,15343,15344,15347,15348,15349,15350,15351,15355,15356,15357,15358,15361,15362,15363,15364,15365,15368,15369,15370,15371,15372,15375,15376,15377,15378,15379,15382,15383,15384,15385,15386,15390,15391,15392,15393,15396,15397,15398,15399,15400,15403,15404,15405,15406,15407,15410,15411,15412,15413,15414,15417,15418,15419,15420,15421,15424,15425,15426,15427,15428,15431,15432,15433,15434,15435,15438,15439,15440,15441,15442,15445,15446,15447,15448,15449,15452,15453,15454,15455,15456,15459,15460,15461,15462,15463,15466,15467,15468,15469,15470,15473,15474,15475,15476,15477,15480,15481,15482,15483,15484,15488,15489,15490,15491,15494,15495,15496,15497,15498,15501,15502,15503,15504,15505,15508,15509,15510,15511,15512,15515,15516,15517,15518,15519,15522,15523,15525,15526,15529,15530,15531,15532,15533,15536,15537,15538,15539,15540,15543,15544,15545,15546,15547,15550,15551,15552,15553,15554,15557,15558,15559,15560,15561,15564,15565,15566,15567,15568,15571,15572,15573,15574,15575,15578,15579,15580,15581,15582,15586,15587,15588,15589,15592,15593,15594,15595,15596,15599,15600,15601,15602,15603,15606,15607,15608,15609,15610,15613,15614,15615,15616,15617,15621,15622,15623,15624,15627,15628,15629,15630,15631,15634,15635,15636,15637,15638,15641,15643,15644,15645,15648,15649,15650,15651,15652,15656,15657,15658,15659,15662,15663,15664,15666,15669,15670,15671,15672,15673,15676,15677,15678,15679,15680,15683,15684,15685,15686,15687,15690,15691,15692,15693,15694,15697,15699,15700,15701,15704,15706,15707,15708,15711,15712,15713,15714,15715,15718,15719,15720,15721,15722,15726,15727,15728,15729,15732,15733,15734,15735,15736,15739,15740,15741,15742,15743,15746,15747,15748,15749,15750,15754,15755,15756,15757,15760,15761,15762,15763,15764,15767,15768,15769,15770,15771,15774,15775,15776,15777,15778,15781,15782,15783,15784,15785,15788,15789,15790,15791,15795,15796,15797,15798,15799,15802,15803,15804,15805,15806,15809,15810,15811,15812,15813,15816,15817,15818,15819,15820,15823,15824,15825,15826,15827,15830,15831,15832,15833,15834,15837,15838,15839,15840,15841,15844,15845,15846,15847,15848,15852,15853,15854,15855,15858,15859,15860,15861,15862,15865,15866,15867,15868,15869,15872,15873,15874,15875,15876,15879,15880,15881,15882,15883,15886,15887,15888,15890,15893,15894,15895,15896,15897,15900,15901,15902,15903,15904,15907,15908,15909,15910,15911,15914],"y":[1.41,1.43,1.43,1.4,1.39,1.41,1.34,1.37,1.32,1.31,1.34,1.43,1.47,1.51,1.49,1.4,1.34,1.31,1.27,1.24,1.27,1.25,1.35,1.32,1.39,1.39,1.43,1.36,1.4,1.34,1.34,1.41,1.43,1.47,1.41,1.4,1.41,1.35,1.36,1.39,1.44,1.38,1.4,1.35,1.37,1.41,1.43,1.43,1.52,1.69,1.67,1.7,1.77,1.78,1.71,1.69,1.66,1.65,1.59,1.6,1.57,1.61,1.6,1.68,1.62,1.56,1.42,1.42,1.37,1.41,1.44,1.39,1.37,1.4,1.38,1.37,1.38,1.34,1.37,1.38,1.36,1.34,1.33,1.35,1.33,1.34,1.28,1.29,1.26,1.26,1.28,1.24,1.2,1.19,1.19,1.16,1.16,1.18,1.2,1.15,1.2,1.17,1.17,1.06,1.03,0.93,1.01,1.04,1.11,1.1,1.09,1.05,1.12,1.06,1.1,1.06,1.06,1.09,1.12,1.1,1.15,1.1,1.12,1.1,1.06,1.11,1.04,1.08,1.05,1.01,0.98,0.98,0.99,0.98,0.99,0.97,0.99,0.97,0.99,0.95,0.93,0.91,0.91,0.94,1.04,0.99,0.98,1.03,0.98,1.07,1.05,1.13,1.14,1.15,1.11,1.12,1.18,1.25,1.28,1.27,1.26,1.25,1.16,1.13,1.14,1.11,1.1,1.11,1.08,1.01,1.03,1.04,1.12,1.09,1.1,1.12,1.17,1.12,1.23,1.22,1.19,1.18,1.18,1.14,1.12,1.08,1.03,1.05,1.04,1.04,1.03,1.02,1.07,1.12,1.11,1.09,1.09,1.09,1.09,1.15,1.24,1.26,1.21,1.25,1.21,1.21,1.28,1.2,1.16,1.14,1.16,1.16,1.13,1.19,1.08,1.04,1.04,1.02,1.03,1.02,1.01,1.04,1.09,1.11,1.12,1.09,1.07,1.05,1.04,1.04,1.05,1.04,1.02,1,1.04,1.04,1.06,1.11,1.15,1.15,1.2,1.25,1.24,1.24,1.2,1.22,1.2,1.15,1.15,1.18,1.25,1.31,1.32,1.31,1.28,1.27,1.3,1.28,1.27,1.24,1.23,1.29,1.26,1.25,1.24,1.26,1.36,1.38,1.4,1.39,1.38,1.4,1.36,1.39,1.35,1.34,1.34,1.35,1.38,1.43,1.37,1.38,1.41,1.38,1.36,1.34,1.25,1.25,1.28,1.26,1.23,1.25,1.27,1.31,1.36,1.43,1.43,1.4,1.41,1.4,1.35,1.31,1.28,1.32,1.3,1.29,1.28,1.27,1.22,1.24,1.23,1.26,1.2,1.15,1.12,1.15,1.16,1.21,1.2,1.14,1.12,1.15,1.13,1.13,1.14,1.13,1.14,1.13,1.15,1.1,1.1,1.11,1.07,1.07,1.17,1.19,1.21,1.2,1.2,1.28,1.3,1.33,1.32,1.25,1.32,1.33,1.31,1.4,1.4,1.39,1.53,1.51,1.51,1.55,1.53,1.55,1.52,1.49,1.59,1.62,1.61,1.64,1.6,1.53,1.57,1.58,1.76,1.84,1.95,2.02,2.03,1.98,1.91,1.96,1.93,1.92,1.97,2.19,2.11,2.08,2.12,1.99,2,1.97,1.95,1.91,1.95,1.9,1.9,1.92,2,2,1.98,2],"subscripts":[1577,1578,1579,1580,1581,1582,1583,1584,1585,1586,1587,1588,1589,1590,1591,1592,1593,1594,1595,1596,1597,1598,1599,1600,1601,1602,1603,1604,1605,1606,1607,1608,1609,1610,1611,1612,1613,1614,1615,1616,1617,1618,1619,1620,1621,1622,1623,1624,1625,1626,1627,1628,1629,1630,1631,1632,1633,1634,1635,1636,1637,1638,1639,1640,1641,1642,1643,1644,1645,1646,1647,1648,1649,1650,1651,1652,1653,1654,1655,1656,1657,1658,1659,1660,1661,1662,1663,1664,1665,1666,1667,1668,1669,1670,1671,1672,1673,1674,1675,1676,1677,1678,1679,1680,1681,1682,1683,1684,1685,1686,1687,1688,1689,1690,1691,1692,1693,1694,1695,1696,1697,1698,1699,1700,1701,1702,1703,1704,1705,1706,1707,1708,1709,1710,1711,1712,1713,1714,1715,1716,1717,1718,1719,1720,1721,1722,1723,1724,1725,1726,1727,1728,1729,1730,1731,1732,1733,1734,1735,1736,1737,1738,1739,1740,1741,1742,1743,1744,1745,1746,1747,1748,1749,1750,1751,1752,1753,1754,1755,1756,1757,1758,1759,1760,1761,1762,1763,1764,1765,1766,1767,1768,1769,1770,1771,1772,1773,1774,1775,1776,1777,1778,1779,1780,1781,1782,1783,1784,1785,1786,1787,1788,1789,1790,1791,1792,1793,1794,1795,1796,1797,1798,1799,1800,1801,1802,1803,1804,1805,1806,1807,1808,1809,1810,1811,1812,1813,1814,1815,1816,1817,1818,1819,1820,1821,1822,1823,1824,1825,1826,1827,1828,1829,1830,1831,1832,1833,1834,1835,1836,1837,1838,1839,1840,1841,1842,1843,1844,1845,1846,1847,1848,1849,1850,1851,1852,1853,1854,1855,1856,1857,1858,1859,1860,1861,1862,1863,1864,1865,1866,1867,1868,1869,1870,1871,1872,1873,1874,1875,1876,1877,1878,1879,1880,1881,1882,1883,1884,1885,1886,1887,1888,1889,1890,1891,1892,1893,1894,1895,1896,1897,1898,1899,1900,1901,1902,1903,1904,1905,1906,1907,1908,1909,1910,1911,1912,1913,1914,1915,1916,1917,1918,1919,1920,1921,1922,1923,1924,1925,1926,1927,1928,1929,1930,1931,1932,1933,1934,1935,1936,1937,1938,1939,1940,1941,1942,1943,1944,1945,1946,1947,1948,1949,1950,1951,1952,1953,1954,1955,1956,1957,1958,1959,1960,1961,1962,1963,1964,1965,1966,1967,1968,1969,1970],"groups":["DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7","DGS7"]},"DGS10":{"x":[15341,15342,15343,15344,15347,15348,15349,15350,15351,15355,15356,15357,15358,15361,15362,15363,15364,15365,15368,15369,15370,15371,15372,15375,15376,15377,15378,15379,15382,15383,15384,15385,15386,15390,15391,15392,15393,15396,15397,15398,15399,15400,15403,15404,15405,15406,15407,15410,15411,15412,15413,15414,15417,15418,15419,15420,15421,15424,15425,15426,15427,15428,15431,15432,15433,15434,15435,15438,15439,15440,15441,15442,15445,15446,15447,15448,15449,15452,15453,15454,15455,15456,15459,15460,15461,15462,15463,15466,15467,15468,15469,15470,15473,15474,15475,15476,15477,15480,15481,15482,15483,15484,15488,15489,15490,15491,15494,15495,15496,15497,15498,15501,15502,15503,15504,15505,15508,15509,15510,15511,15512,15515,15516,15517,15518,15519,15522,15523,15525,15526,15529,15530,15531,15532,15533,15536,15537,15538,15539,15540,15543,15544,15545,15546,15547,15550,15551,15552,15553,15554,15557,15558,15559,15560,15561,15564,15565,15566,15567,15568,15571,15572,15573,15574,15575,15578,15579,15580,15581,15582,15586,15587,15588,15589,15592,15593,15594,15595,15596,15599,15600,15601,15602,15603,15606,15607,15608,15609,15610,15613,15614,15615,15616,15617,15621,15622,15623,15624,15627,15628,15629,15630,15631,15634,15635,15636,15637,15638,15641,15643,15644,15645,15648,15649,15650,15651,15652,15656,15657,15658,15659,15662,15663,15664,15666,15669,15670,15671,15672,15673,15676,15677,15678,15679,15680,15683,15684,15685,15686,15687,15690,15691,15692,15693,15694,15697,15699,15700,15701,15704,15706,15707,15708,15711,15712,15713,15714,15715,15718,15719,15720,15721,15722,15726,15727,15728,15729,15732,15733,15734,15735,15736,15739,15740,15741,15742,15743,15746,15747,15748,15749,15750,15754,15755,15756,15757,15760,15761,15762,15763,15764,15767,15768,15769,15770,15771,15774,15775,15776,15777,15778,15781,15782,15783,15784,15785,15788,15789,15790,15791,15795,15796,15797,15798,15799,15802,15803,15804,15805,15806,15809,15810,15811,15812,15813,15816,15817,15818,15819,15820,15823,15824,15825,15826,15827,15830,15831,15832,15833,15834,15837,15838,15839,15840,15841,15844,15845,15846,15847,15848,15852,15853,15854,15855,15858,15859,15860,15861,15862,15865,15866,15867,15868,15869,15872,15873,15874,15875,15876,15879,15880,15881,15882,15883,15886,15887,15888,15890,15893,15894,15895,15896,15897,15900,15901,15902,15903,15904,15907,15908,15909,15910,15911,15914],"y":[1.97,2,2.02,1.98,1.98,2,1.93,1.94,1.89,1.87,1.92,2.01,2.05,2.09,2.08,2.01,1.96,1.93,1.87,1.83,1.87,1.86,1.97,1.93,2,2.01,2.04,1.96,1.99,1.92,1.93,1.99,2.01,2.05,2.01,1.99,1.98,1.92,1.94,1.98,2.03,1.99,2,1.96,1.98,2.03,2.04,2.04,2.14,2.29,2.29,2.31,2.39,2.38,2.31,2.29,2.25,2.26,2.2,2.21,2.18,2.23,2.22,2.3,2.25,2.19,2.07,2.06,2.01,2.05,2.08,2.02,2,2.03,2,1.98,1.99,1.96,2,2.01,1.98,1.96,1.95,1.98,1.96,1.96,1.91,1.92,1.88,1.87,1.89,1.84,1.78,1.76,1.76,1.7,1.71,1.75,1.79,1.73,1.77,1.75,1.74,1.63,1.59,1.47,1.53,1.57,1.66,1.66,1.65,1.6,1.67,1.61,1.64,1.6,1.59,1.64,1.65,1.63,1.69,1.63,1.66,1.65,1.6,1.67,1.61,1.65,1.62,1.57,1.53,1.53,1.54,1.5,1.52,1.5,1.53,1.52,1.54,1.49,1.47,1.44,1.43,1.45,1.58,1.53,1.51,1.56,1.51,1.6,1.59,1.66,1.68,1.69,1.65,1.65,1.73,1.8,1.83,1.81,1.82,1.8,1.71,1.68,1.68,1.65,1.64,1.66,1.63,1.57,1.59,1.6,1.68,1.67,1.68,1.7,1.77,1.75,1.88,1.85,1.82,1.79,1.8,1.77,1.74,1.7,1.64,1.66,1.65,1.64,1.64,1.64,1.7,1.75,1.74,1.72,1.7,1.69,1.7,1.75,1.83,1.86,1.79,1.83,1.79,1.8,1.86,1.78,1.74,1.72,1.75,1.75,1.72,1.78,1.68,1.62,1.61,1.59,1.59,1.58,1.58,1.61,1.66,1.69,1.7,1.66,1.64,1.63,1.62,1.62,1.63,1.62,1.6,1.59,1.64,1.63,1.66,1.72,1.74,1.72,1.78,1.84,1.82,1.81,1.77,1.79,1.77,1.74,1.73,1.78,1.86,1.92,1.93,1.92,1.89,1.88,1.91,1.89,1.89,1.86,1.84,1.89,1.87,1.86,1.86,1.88,1.98,2,2.03,2.03,2.02,2.04,2,2.04,2,1.99,1.99,1.99,2.02,2.05,2,2.01,2.03,2.02,1.99,1.97,1.88,1.88,1.91,1.89,1.86,1.88,1.9,1.95,2,2.06,2.07,2.03,2.04,2.04,2.01,1.96,1.92,1.96,1.95,1.93,1.93,1.92,1.87,1.87,1.86,1.88,1.83,1.78,1.72,1.76,1.78,1.84,1.82,1.75,1.72,1.75,1.73,1.72,1.73,1.72,1.74,1.73,1.74,1.7,1.7,1.7,1.66,1.66,1.78,1.8,1.82,1.81,1.81,1.9,1.92,1.96,1.94,1.87,1.95,1.97,1.94,2.03,2.02,2.01,2.15,2.13,2.13,2.16,2.13,2.14,2.1,2.08,2.17,2.22,2.2,2.25,2.19,2.14,2.19,2.2,2.33,2.41,2.52,2.57,2.6,2.55,2.49,2.52,2.5,2.48,2.52,2.73,2.65,2.65,2.7,2.6,2.61,2.57,2.55,2.52,2.56,2.5,2.5,2.53,2.61,2.61,2.58,2.61],"subscripts":[1971,1972,1973,1974,1975,1976,1977,1978,1979,1980,1981,1982,1983,1984,1985,1986,1987,1988,1989,1990,1991,1992,1993,1994,1995,1996,1997,1998,1999,2000,2001,2002,2003,2004,2005,2006,2007,2008,2009,2010,2011,2012,2013,2014,2015,2016,2017,2018,2019,2020,2021,2022,2023,2024,2025,2026,2027,2028,2029,2030,2031,2032,2033,2034,2035,2036,2037,2038,2039,2040,2041,2042,2043,2044,2045,2046,2047,2048,2049,2050,2051,2052,2053,2054,2055,2056,2057,2058,2059,2060,2061,2062,2063,2064,2065,2066,2067,2068,2069,2070,2071,2072,2073,2074,2075,2076,2077,2078,2079,2080,2081,2082,2083,2084,2085,2086,2087,2088,2089,2090,2091,2092,2093,2094,2095,2096,2097,2098,2099,2100,2101,2102,2103,2104,2105,2106,2107,2108,2109,2110,2111,2112,2113,2114,2115,2116,2117,2118,2119,2120,2121,2122,2123,2124,2125,2126,2127,2128,2129,2130,2131,2132,2133,2134,2135,2136,2137,2138,2139,2140,2141,2142,2143,2144,2145,2146,2147,2148,2149,2150,2151,2152,2153,2154,2155,2156,2157,2158,2159,2160,2161,2162,2163,2164,2165,2166,2167,2168,2169,2170,2171,2172,2173,2174,2175,2176,2177,2178,2179,2180,2181,2182,2183,2184,2185,2186,2187,2188,2189,2190,2191,2192,2193,2194,2195,2196,2197,2198,2199,2200,2201,2202,2203,2204,2205,2206,2207,2208,2209,2210,2211,2212,2213,2214,2215,2216,2217,2218,2219,2220,2221,2222,2223,2224,2225,2226,2227,2228,2229,2230,2231,2232,2233,2234,2235,2236,2237,2238,2239,2240,2241,2242,2243,2244,2245,2246,2247,2248,2249,2250,2251,2252,2253,2254,2255,2256,2257,2258,2259,2260,2261,2262,2263,2264,2265,2266,2267,2268,2269,2270,2271,2272,2273,2274,2275,2276,2277,2278,2279,2280,2281,2282,2283,2284,2285,2286,2287,2288,2289,2290,2291,2292,2293,2294,2295,2296,2297,2298,2299,2300,2301,2302,2303,2304,2305,2306,2307,2308,2309,2310,2311,2312,2313,2314,2315,2316,2317,2318,2319,2320,2321,2322,2323,2324,2325,2326,2327,2328,2329,2330,2331,2332,2333,2334,2335,2336,2337,2338,2339,2340,2341,2342,2343,2344,2345,2346,2347,2348,2349,2350,2351,2352,2353,2354,2355,2356,2357,2358,2359,2360,2361,2362,2363,2364],"groups":["DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10","DGS10"]},"DGS20":{"x":[15341,15342,15343,15344,15347,15348,15349,15350,15351,15355,15356,15357,15358,15361,15362,15363,15364,15365,15368,15369,15370,15371,15372,15375,15376,15377,15378,15379,15382,15383,15384,15385,15386,15390,15391,15392,15393,15396,15397,15398,15399,15400,15403,15404,15405,15406,15407,15410,15411,15412,15413,15414,15417,15418,15419,15420,15421,15424,15425,15426,15427,15428,15431,15432,15433,15434,15435,15438,15439,15440,15441,15442,15445,15446,15447,15448,15449,15452,15453,15454,15455,15456,15459,15460,15461,15462,15463,15466,15467,15468,15469,15470,15473,15474,15475,15476,15477,15480,15481,15482,15483,15484,15488,15489,15490,15491,15494,15495,15496,15497,15498,15501,15502,15503,15504,15505,15508,15509,15510,15511,15512,15515,15516,15517,15518,15519,15522,15523,15525,15526,15529,15530,15531,15532,15533,15536,15537,15538,15539,15540,15543,15544,15545,15546,15547,15550,15551,15552,15553,15554,15557,15558,15559,15560,15561,15564,15565,15566,15567,15568,15571,15572,15573,15574,15575,15578,15579,15580,15581,15582,15586,15587,15588,15589,15592,15593,15594,15595,15596,15599,15600,15601,15602,15603,15606,15607,15608,15609,15610,15613,15614,15615,15616,15617,15621,15622,15623,15624,15627,15628,15629,15630,15631,15634,15635,15636,15637,15638,15641,15643,15644,15645,15648,15649,15650,15651,15652,15656,15657,15658,15659,15662,15663,15664,15666,15669,15670,15671,15672,15673,15676,15677,15678,15679,15680,15683,15684,15685,15686,15687,15690,15691,15692,15693,15694,15697,15699,15700,15701,15704,15706,15707,15708,15711,15712,15713,15714,15715,15718,15719,15720,15721,15722,15726,15727,15728,15729,15732,15733,15734,15735,15736,15739,15740,15741,15742,15743,15746,15747,15748,15749,15750,15754,15755,15756,15757,15760,15761,15762,15763,15764,15767,15768,15769,15770,15771,15774,15775,15776,15777,15778,15781,15782,15783,15784,15785,15788,15789,15790,15791,15795,15796,15797,15798,15799,15802,15803,15804,15805,15806,15809,15810,15811,15812,15813,15816,15817,15818,15819,15820,15823,15824,15825,15826,15827,15830,15831,15832,15833,15834,15837,15838,15839,15840,15841,15844,15845,15846,15847,15848,15852,15853,15854,15855,15858,15859,15860,15861,15862,15865,15866,15867,15868,15869,15872,15873,15874,15875,15876,15879,15880,15881,15882,15883,15886,15887,15888,15890,15893,15894,15895,15896,15897,15900,15901,15902,15903,15904,15907,15908,15909,15910,15911,15914],"y":[2.67,2.71,2.74,2.7,2.7,2.71,2.63,2.65,2.59,2.57,2.63,2.72,2.78,2.82,2.82,2.78,2.74,2.71,2.64,2.59,2.65,2.64,2.76,2.71,2.78,2.78,2.83,2.75,2.78,2.7,2.72,2.78,2.8,2.84,2.79,2.77,2.75,2.69,2.71,2.73,2.8,2.77,2.78,2.73,2.76,2.82,2.83,2.82,2.92,3.08,3.08,3.08,3.14,3.13,3.06,3.04,2.99,3,2.96,2.97,2.93,3,3,3.07,3.02,2.97,2.85,2.82,2.77,2.82,2.85,2.77,2.75,2.79,2.76,2.74,2.75,2.71,2.75,2.76,2.74,2.73,2.73,2.76,2.72,2.72,2.67,2.67,2.63,2.63,2.64,2.59,2.53,2.5,2.48,2.39,2.4,2.42,2.48,2.41,2.46,2.44,2.44,2.32,2.27,2.13,2.17,2.23,2.34,2.35,2.36,2.3,2.37,2.3,2.33,2.3,2.28,2.33,2.34,2.3,2.37,2.31,2.34,2.32,2.28,2.38,2.3,2.36,2.34,2.28,2.24,2.22,2.22,2.18,2.2,2.18,2.22,2.21,2.24,2.17,2.15,2.11,2.11,2.13,2.27,2.22,2.21,2.25,2.2,2.3,2.29,2.37,2.39,2.4,2.37,2.37,2.45,2.53,2.57,2.55,2.55,2.53,2.44,2.41,2.41,2.38,2.36,2.38,2.36,2.29,2.3,2.32,2.41,2.42,2.43,2.44,2.52,2.53,2.68,2.64,2.61,2.58,2.58,2.57,2.53,2.47,2.4,2.43,2.42,2.41,2.41,2.42,2.48,2.55,2.52,2.48,2.45,2.44,2.45,2.51,2.6,2.63,2.55,2.57,2.53,2.55,2.6,2.53,2.48,2.46,2.5,2.51,2.47,2.52,2.42,2.35,2.34,2.31,2.31,2.3,2.31,2.34,2.4,2.42,2.42,2.39,2.38,2.36,2.37,2.37,2.37,2.36,2.35,2.33,2.39,2.38,2.41,2.48,2.49,2.46,2.53,2.59,2.58,2.57,2.52,2.53,2.52,2.48,2.47,2.54,2.63,2.7,2.7,2.7,2.66,2.65,2.68,2.65,2.65,2.62,2.61,2.66,2.63,2.62,2.62,2.64,2.75,2.76,2.79,2.8,2.79,2.83,2.79,2.83,2.79,2.78,2.79,2.78,2.81,2.86,2.79,2.8,2.83,2.82,2.79,2.77,2.69,2.69,2.72,2.71,2.68,2.7,2.72,2.77,2.82,2.89,2.89,2.85,2.85,2.87,2.85,2.79,2.75,2.8,2.77,2.75,2.76,2.75,2.71,2.71,2.7,2.72,2.66,2.6,2.5,2.54,2.57,2.63,2.62,2.54,2.5,2.53,2.51,2.49,2.5,2.5,2.52,2.5,2.52,2.47,2.49,2.49,2.44,2.44,2.58,2.6,2.62,2.61,2.6,2.7,2.73,2.77,2.76,2.69,2.77,2.79,2.75,2.83,2.82,2.8,2.95,2.91,2.92,2.95,2.92,2.95,2.9,2.89,2.98,3.03,3,3.04,2.99,2.95,3.01,3,3.09,3.18,3.26,3.27,3.31,3.27,3.22,3.22,3.19,3.18,3.22,3.41,3.35,3.36,3.4,3.33,3.34,3.3,3.28,3.27,3.32,3.25,3.25,3.27,3.34,3.34,3.31,3.35],"subscripts":[2365,2366,2367,2368,2369,2370,2371,2372,2373,2374,2375,2376,2377,2378,2379,2380,2381,2382,2383,2384,2385,2386,2387,2388,2389,2390,2391,2392,2393,2394,2395,2396,2397,2398,2399,2400,2401,2402,2403,2404,2405,2406,2407,2408,2409,2410,2411,2412,2413,2414,2415,2416,2417,2418,2419,2420,2421,2422,2423,2424,2425,2426,2427,2428,2429,2430,2431,2432,2433,2434,2435,2436,2437,2438,2439,2440,2441,2442,2443,2444,2445,2446,2447,2448,2449,2450,2451,2452,2453,2454,2455,2456,2457,2458,2459,2460,2461,2462,2463,2464,2465,2466,2467,2468,2469,2470,2471,2472,2473,2474,2475,2476,2477,2478,2479,2480,2481,2482,2483,2484,2485,2486,2487,2488,2489,2490,2491,2492,2493,2494,2495,2496,2497,2498,2499,2500,2501,2502,2503,2504,2505,2506,2507,2508,2509,2510,2511,2512,2513,2514,2515,2516,2517,2518,2519,2520,2521,2522,2523,2524,2525,2526,2527,2528,2529,2530,2531,2532,2533,2534,2535,2536,2537,2538,2539,2540,2541,2542,2543,2544,2545,2546,2547,2548,2549,2550,2551,2552,2553,2554,2555,2556,2557,2558,2559,2560,2561,2562,2563,2564,2565,2566,2567,2568,2569,2570,2571,2572,2573,2574,2575,2576,2577,2578,2579,2580,2581,2582,2583,2584,2585,2586,2587,2588,2589,2590,2591,2592,2593,2594,2595,2596,2597,2598,2599,2600,2601,2602,2603,2604,2605,2606,2607,2608,2609,2610,2611,2612,2613,2614,2615,2616,2617,2618,2619,2620,2621,2622,2623,2624,2625,2626,2627,2628,2629,2630,2631,2632,2633,2634,2635,2636,2637,2638,2639,2640,2641,2642,2643,2644,2645,2646,2647,2648,2649,2650,2651,2652,2653,2654,2655,2656,2657,2658,2659,2660,2661,2662,2663,2664,2665,2666,2667,2668,2669,2670,2671,2672,2673,2674,2675,2676,2677,2678,2679,2680,2681,2682,2683,2684,2685,2686,2687,2688,2689,2690,2691,2692,2693,2694,2695,2696,2697,2698,2699,2700,2701,2702,2703,2704,2705,2706,2707,2708,2709,2710,2711,2712,2713,2714,2715,2716,2717,2718,2719,2720,2721,2722,2723,2724,2725,2726,2727,2728,2729,2730,2731,2732,2733,2734,2735,2736,2737,2738,2739,2740,2741,2742,2743,2744,2745,2746,2747,2748,2749,2750,2751,2752,2753,2754,2755,2756,2757,2758],"groups":["DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20","DGS20"]},"DGS30":{"x":[15341,15342,15343,15344,15347,15348,15349,15350,15351,15355,15356,15357,15358,15361,15362,15363,15364,15365,15368,15369,15370,15371,15372,15375,15376,15377,15378,15379,15382,15383,15384,15385,15386,15390,15391,15392,15393,15396,15397,15398,15399,15400,15403,15404,15405,15406,15407,15410,15411,15412,15413,15414,15417,15418,15419,15420,15421,15424,15425,15426,15427,15428,15431,15432,15433,15434,15435,15438,15439,15440,15441,15442,15445,15446,15447,15448,15449,15452,15453,15454,15455,15456,15459,15460,15461,15462,15463,15466,15467,15468,15469,15470,15473,15474,15475,15476,15477,15480,15481,15482,15483,15484,15488,15489,15490,15491,15494,15495,15496,15497,15498,15501,15502,15503,15504,15505,15508,15509,15510,15511,15512,15515,15516,15517,15518,15519,15522,15523,15525,15526,15529,15530,15531,15532,15533,15536,15537,15538,15539,15540,15543,15544,15545,15546,15547,15550,15551,15552,15553,15554,15557,15558,15559,15560,15561,15564,15565,15566,15567,15568,15571,15572,15573,15574,15575,15578,15579,15580,15581,15582,15586,15587,15588,15589,15592,15593,15594,15595,15596,15599,15600,15601,15602,15603,15606,15607,15608,15609,15610,15613,15614,15615,15616,15617,15621,15622,15623,15624,15627,15628,15629,15630,15631,15634,15635,15636,15637,15638,15641,15643,15644,15645,15648,15649,15650,15651,15652,15656,15657,15658,15659,15662,15663,15664,15666,15669,15670,15671,15672,15673,15676,15677,15678,15679,15680,15683,15684,15685,15686,15687,15690,15691,15692,15693,15694,15697,15699,15700,15701,15704,15706,15707,15708,15711,15712,15713,15714,15715,15718,15719,15720,15721,15722,15726,15727,15728,15729,15732,15733,15734,15735,15736,15739,15740,15741,15742,15743,15746,15747,15748,15749,15750,15754,15755,15756,15757,15760,15761,15762,15763,15764,15767,15768,15769,15770,15771,15774,15775,15776,15777,15778,15781,15782,15783,15784,15785,15788,15789,15790,15791,15795,15796,15797,15798,15799,15802,15803,15804,15805,15806,15809,15810,15811,15812,15813,15816,15817,15818,15819,15820,15823,15824,15825,15826,15827,15830,15831,15832,15833,15834,15837,15838,15839,15840,15841,15844,15845,15846,15847,15848,15852,15853,15854,15855,15858,15859,15860,15861,15862,15865,15866,15867,15868,15869,15872,15873,15874,15875,15876,15879,15880,15881,15882,15883,15886,15887,15888,15890,15893,15894,15895,15896,15897,15900,15901,15902,15903,15904,15907,15908,15909,15910,15911,15914],"y":[2.98,3.03,3.06,3.02,3.02,3.04,2.96,2.97,2.91,2.89,2.96,3.05,3.1,3.15,3.15,3.13,3.1,3.07,2.99,2.94,3.01,3.01,3.13,3.08,3.14,3.14,3.2,3.11,3.14,3.06,3.09,3.14,3.16,3.2,3.15,3.13,3.1,3.04,3.07,3.08,3.15,3.11,3.13,3.08,3.12,3.18,3.19,3.17,3.26,3.43,3.41,3.41,3.48,3.46,3.38,3.37,3.31,3.33,3.29,3.31,3.27,3.35,3.35,3.41,3.37,3.32,3.21,3.18,3.13,3.18,3.22,3.14,3.12,3.15,3.13,3.12,3.12,3.08,3.12,3.15,3.13,3.12,3.12,3.16,3.11,3.12,3.07,3.07,3.03,3.03,3.07,3.02,2.95,2.91,2.9,2.8,2.8,2.8,2.88,2.81,2.86,2.85,2.85,2.72,2.67,2.53,2.56,2.63,2.73,2.75,2.77,2.71,2.77,2.7,2.73,2.7,2.67,2.73,2.72,2.68,2.75,2.69,2.71,2.7,2.67,2.76,2.69,2.74,2.72,2.66,2.62,2.6,2.6,2.57,2.58,2.56,2.59,2.59,2.61,2.55,2.52,2.47,2.46,2.49,2.63,2.58,2.56,2.6,2.55,2.65,2.65,2.72,2.75,2.78,2.74,2.74,2.82,2.9,2.96,2.93,2.93,2.9,2.82,2.79,2.79,2.76,2.75,2.77,2.75,2.68,2.69,2.7,2.8,2.81,2.83,2.84,2.92,2.95,3.09,3.03,3,2.97,2.96,2.95,2.91,2.86,2.79,2.83,2.82,2.81,2.81,2.82,2.89,2.96,2.93,2.89,2.86,2.83,2.85,2.91,2.98,3.02,2.94,2.95,2.91,2.93,2.98,2.92,2.87,2.85,2.89,2.91,2.88,2.92,2.83,2.77,2.75,2.72,2.73,2.72,2.73,2.76,2.82,2.83,2.83,2.8,2.79,2.79,2.79,2.81,2.8,2.78,2.78,2.76,2.81,2.8,2.83,2.9,2.9,2.87,2.94,3,2.99,2.98,2.93,2.94,2.94,2.89,2.88,2.95,3.04,3.12,3.1,3.1,3.06,3.06,3.08,3.05,3.05,3.02,3.01,3.06,3.03,3.02,3.02,3.04,3.14,3.15,3.18,3.19,3.17,3.21,3.17,3.21,3.18,3.17,3.17,3.16,3.19,3.23,3.17,3.18,3.21,3.2,3.17,3.15,3.08,3.08,3.11,3.1,3.06,3.08,3.1,3.15,3.2,3.25,3.26,3.22,3.22,3.25,3.22,3.18,3.13,3.19,3.15,3.13,3.14,3.13,3.09,3.1,3.08,3.1,3.05,2.99,2.87,2.91,2.94,3.01,3.01,2.92,2.88,2.91,2.89,2.87,2.88,2.88,2.9,2.89,2.91,2.87,2.88,2.88,2.83,2.82,2.96,2.98,3,2.99,3.01,3.1,3.13,3.17,3.16,3.09,3.17,3.18,3.14,3.21,3.2,3.18,3.31,3.27,3.28,3.3,3.27,3.3,3.25,3.23,3.33,3.36,3.33,3.37,3.33,3.28,3.35,3.34,3.41,3.49,3.56,3.56,3.6,3.58,3.54,3.52,3.48,3.47,3.49,3.68,3.63,3.64,3.68,3.64,3.64,3.61,3.58,3.57,3.63,3.56,3.55,3.58,3.65,3.65,3.61,3.66],"subscripts":[2759,2760,2761,2762,2763,2764,2765,2766,2767,2768,2769,2770,2771,2772,2773,2774,2775,2776,2777,2778,2779,2780,2781,2782,2783,2784,2785,2786,2787,2788,2789,2790,2791,2792,2793,2794,2795,2796,2797,2798,2799,2800,2801,2802,2803,2804,2805,2806,2807,2808,2809,2810,2811,2812,2813,2814,2815,2816,2817,2818,2819,2820,2821,2822,2823,2824,2825,2826,2827,2828,2829,2830,2831,2832,2833,2834,2835,2836,2837,2838,2839,2840,2841,2842,2843,2844,2845,2846,2847,2848,2849,2850,2851,2852,2853,2854,2855,2856,2857,2858,2859,2860,2861,2862,2863,2864,2865,2866,2867,2868,2869,2870,2871,2872,2873,2874,2875,2876,2877,2878,2879,2880,2881,2882,2883,2884,2885,2886,2887,2888,2889,2890,2891,2892,2893,2894,2895,2896,2897,2898,2899,2900,2901,2902,2903,2904,2905,2906,2907,2908,2909,2910,2911,2912,2913,2914,2915,2916,2917,2918,2919,2920,2921,2922,2923,2924,2925,2926,2927,2928,2929,2930,2931,2932,2933,2934,2935,2936,2937,2938,2939,2940,2941,2942,2943,2944,2945,2946,2947,2948,2949,2950,2951,2952,2953,2954,2955,2956,2957,2958,2959,2960,2961,2962,2963,2964,2965,2966,2967,2968,2969,2970,2971,2972,2973,2974,2975,2976,2977,2978,2979,2980,2981,2982,2983,2984,2985,2986,2987,2988,2989,2990,2991,2992,2993,2994,2995,2996,2997,2998,2999,3000,3001,3002,3003,3004,3005,3006,3007,3008,3009,3010,3011,3012,3013,3014,3015,3016,3017,3018,3019,3020,3021,3022,3023,3024,3025,3026,3027,3028,3029,3030,3031,3032,3033,3034,3035,3036,3037,3038,3039,3040,3041,3042,3043,3044,3045,3046,3047,3048,3049,3050,3051,3052,3053,3054,3055,3056,3057,3058,3059,3060,3061,3062,3063,3064,3065,3066,3067,3068,3069,3070,3071,3072,3073,3074,3075,3076,3077,3078,3079,3080,3081,3082,3083,3084,3085,3086,3087,3088,3089,3090,3091,3092,3093,3094,3095,3096,3097,3098,3099,3100,3101,3102,3103,3104,3105,3106,3107,3108,3109,3110,3111,3112,3113,3114,3115,3116,3117,3118,3119,3120,3121,3122,3123,3124,3125,3126,3127,3128,3129,3130,3131,3132,3133,3134,3135,3136,3137,3138,3139,3140,3141,3142,3143,3144,3145,3146,3147,3148,3149,3150,3151,3152],"groups":["DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30","DGS30"]}}}

var svg = d3.select("svg");
var line = d3.svg.line()
                  .x(function (d) {
                      return +d.x;
                   })
                  .y(function (d) { return +d.y; })
                  .interpolate("basis");

var parseDate = d3.time.format("%Y-%m-%d").parse,
                    bisectDate = d3.bisector(function (d) { return +d.x; }).left;

function pointsToArray(points) {
  var pointsArray = new Array();

  pointsArray = points.match(/[^ ]+/g);

  pointsArray = pointsArray.map(
                      function (d, i) {
                        return {
                          x: d.split(",")[0],
                          y: d.split(",")[1],
                        }
                      }
                  );

  return pointsArray;
}

var g = svg.selectAll('.lines');

var pointsdata = [];
var pointsline = [];
d3.entries(data.data).forEach(
  function (d,i) {
    pointsline = pointsToArray(d3.select(g[0][i]).select('polyline').attr("points"));
    d.value.groups.forEach(
      function (dd, ii) {
        pointsdata.push(
           {
              x:pointsline[ii].x, y:pointsline[ii].y,
              data: {"x": d.value.x[ii], "y": d.value.y[ii], "group": d.value.groups[ii], "strip": d.key } 
           }
        )
      }
    )
  }
)


//assign data to the line
//g.data(d3.entries(data.data));
g.data(d3.nest().key(function (d) { return d.data.group }).entries(pointsdata));

//loop through each polyline and add a path to contain the data for tootips/hover
g[0].forEach(function(d) {   
  var mypath = d3.select(d).append("path")
  mypath
    .datum(mypath.datum().values)
    .attr("d",line)
})

var focus = svg.selectAll(".focus")
                    .data(d3.entries(data.groups)).enter().append("g")
                          .attr("class", "focus")
                          .attr("id", function (d, i) { return "focus-" + i; })
                          .style("display", "none");

focus.append("circle")
                  .attr("r", 4.5)
                  .attr("stroke", "black")
                  .attr("fill-opacity", "0");

focus.append("text")
  .attr("x", 9)
  .attr("dy", ".35em");

svg
  .on("mouseover", function () { focus.style("display", null); })
  .on("mouseout", function () { focus.style("display", "none"); })
  .on("mousemove", mousemove);

function mousemove() {
  var x0 = d3.mouse(this)[0];
  var i;

  for(i1 = 0; i1<data.groups.length; i1++) {
    groupdata = d3.select(g.select("path")[0][i1]).datum().values
    if(d3.max(groupdata, function(d){return d.x}) > x0) {
      i = bisectDate(groupdata, x0, 1);
      break;
    }
  }

  d3.entries(data.groups).forEach(function (group, i1) {
      groupdata = d3.select(g.select("path")[0][i1]).datum().values

      var d;
      d = groupdata[i];
      //if (Boolean(d1)) { d = x0 - d0.x > d1.x - x0 ? d1 : d0 } else { d = d0 };
      d3.select("#focus-" + i1)
              .attr("transform", "translate(" + d.x + "," + ((+d3.select("rect").attr("height")) - d.y) + ")")
              .attr("fill", "black"); //color(0));
      d3.select("#focus-" + i1).select("text")
              .text(d.data.group + ": " + d.data.y)
             // .attr("fill", "black"); //color(0));
  });
}
</script>