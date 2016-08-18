# data.table: faster and friendlier

| Functions  | 
| ---- | 
| fread / fwrite |  
| dt[i, j, by] |  
| merge / rbindlist | 
| inrange / foverlaps |  
| dcast / melt |  

## Install latest version

```
install.packages("data.table", type = "source",
    repos = "http://Rdatatable.github.io/data.table")
```

## Helpful references

`?data.table`

https://github.com/Rdatatable/data.table/wiki

https://s3.amazonaws.com/assets.datacamp.com/img/blog/data+table+cheat+sheet.pdf

http://www.r-bloggers.com/advanced-tips-and-tricks-with-data-table/

## No side effects?
In the functional programming paradigm, a function's output is the only thing it can affect, no side effects!

In many cases this is very helpful, however: a function modifying a table cannot touch the input must create a copy 

`data.table` uses pass-by-reference and avoids creating a copy.

https://www.reddit.com/r/rstats/comments/2ymnal/datatable_why/

> dtplyr [data.table backend for dplyr] will always be a bit slower than data.table, because it creates copies of objects rather than mutating in place (that's the dplyr philosophy). Currently, dtplyr is quite a lot slower than bare data.table because the methods aren't quite smart enough.

```
 _________________________________________
/ dtplyr will always be a bit slower than \
| data.table, because it creates copies   |
\ of objects                              /
 -----------------------------------------
     /   
    /  
```
<img src="https://avatars1.githubusercontent.com/u/4196?v=3&s=460" width="75">

https://github.com/hadley/dtplyr

## data.table uses multi-thread by default

Must compile with OpenMP for multi-thread:

https://github.com/Rdatatable/data.table/wiki/Installation#openmp-enabled-compiler-for-mac

`setthreads()` and `getthreads()`

## fread
Benchmarks on 50 MB file (from `?fread`)

```
n=1e6
DT = data.table( a=sample(1:1000,n,replace=TRUE),
                 b=sample(1:1000,n,replace=TRUE),
                 c=rnorm(n),
                 d=sample(c("foo","bar","baz","qux","quux"),n,replace=TRUE),
                 e=rnorm(n),
                 f=sample(1:1000,n,replace=TRUE) )
DT[2,b:=NA_integer_]
DT[4,c:=NA_real_]
DT[3,d:=NA_character_]
DT[5,d:=""]
DT[2,e:=+Inf]
DT[3,e:=-Inf]

write.table(DT,"test.csv",sep=",",row.names=FALSE,quote=FALSE)
cat("File size (MB):", round(file.info("test.csv")$size/1024^2),"\n")
# 50 MB (1e6 rows x 6 columns)

system.time(DF1 <-read.csv("test.csv",stringsAsFactors=FALSE))
# 60 sec (first time in fresh R session)

system.time(DF1 <- read.csv("test.csv",stringsAsFactors=FALSE))
# 30 sec (immediate repeat is faster, varies)

system.time(DF2 <- read.table("test.csv",header=TRUE,sep=",",quote="",
    stringsAsFactors=FALSE,comment.char="",nrows=n,
    colClasses=c("integer","integer","numeric",
                 "character","numeric","integer")))
# 10 sec (consistently). All known tricks and known nrows, see references.

require(data.table)
system.time(DT <- fread("test.csv"))
#  3 sec (faster and friendlier)
```

## fwrite
Benchmark (adapted from `?fwrite`)
```
set.seed(45L)
dt = as.data.table(matrix(as.numeric(sample(5e6*10L)), ncol=10L)) # 381MB

write.table(dt, "tmp2.tsv", 
                quote = F,
                col.names=T,
                row.names=F,
                sep='\t')
#   user  system elapsed 
# 72.020   1.363  78.741 

system.time(fwrite(dt, "~/tmp.tsv", quote=FALSE, sep="\t", verbose = TRUE))
# Maximum line length is 212 calculated in 0.000s
# Writing column names ... done in 0.000s
# Writing data rows in 1011 batches of 4946 rows (each buffer size 1.000MB, turbo=1) ... all 4 threads done
#   user  system elapsed 
#  2.643   0.291   1.710
```

### Reading a maf file (with key columns!)
```
maf <- fread("https://tcga-data.nci.nih.gov/docs/publications/prad_2015/PRAD_Capture_All_Pairs_QCPASS_v6_Nikki_Nov_25.aggregated.capture.tcga.uuid.curated.somatic.maf", 
             verbose = T,
             key = c("Chromosome", "Start_position", "End_position"))
```

```
Input contains no \n. Taking this to be a filename to open
File opened, filesize is 0.011734 GB.
Memory mapping ... ok
Detected eol as \n only (no \r afterwards), the UNIX and Mac standard.
Positioned on line 1 after skip or autostart
This line is the autostart and not blank so searching up for the last non-blank ... line 1
Detecting sep ... '\t'
Detected 87 columns. Longest stretch was from line 2 to line 30
Starting data input on line 2 (either column names or first row of data). First 10 characters: Hugo_Symbo
All the fields on line 2 are character fields. Treating as the column names.
Count of eol: 12349 (including 1 at the end)
Count of sep: 1061928
nrow = MIN( nsep [1061928] / (ncol [87] -1), neol [12349] - endblanks [1] ) = 12348
Type codes ( first 5 rows): 414111144444400440000000444440044444411444444444140004440041440404300000000004411444411
Type codes (middle 5 rows): 414111144444400440000000444440044444411444444444140004440041440404300000000004411444411
Type codes (  last 5 rows): 414111144444400440000000444440044444411444444444140004440041444404300000000004411444411
Type codes: 414111144444400440000000444440044444411444444444140004440041444404300000000004411444411 (after applying colClasses and integer64)
Type codes: 414111144444400440000000444440044444411444444444140004440041444404300000000004411444411 (after applying drop or select (if supplied)
Allocating 87 column slots (87 - 0 dropped)
```
...
```
Read 12348 rows. Exactly what was estimated and allocated up front
   0.001s (  1%) Memory map (rerun may be quicker)
   0.001s (  0%) sep and header detection
   0.035s ( 14%) Count rows (wc -l)
   0.007s (  3%) Column type detection (first, middle and last 5 rows)
   0.001s (  0%) Allocation of 12348x87 result (xMB) in RAM
   0.190s ( 75%) Reading data
   0.001s (  0%) Allocation for type bumps (if any), including gc time if triggered
   0.016s (  6%) Coercing data already read in type bumps (if any)
   0.001s (  0%) Changing na.strings to NA
   0.253s        Total
```

## `DT[ i,  j,  by ]`

Take DT, subset rows by `i`, then compute `j` grouped by `by`
```
    DT[ i,  j,  by ]
        |   |   |
        |   |    -------> grouped by what?
        |    -------> what to do?
         ---> on which rows?
```

To make a new data.table, `j` must return a list:
```
maf[, as.list(summary(t_depth)), Tumor_Sample_Barcode]
```

```
DT = data.table(x=rep(c("b","a","c"),each=3), v=c(1,1,1,2,2,1,1,2,2), y=c(1,3,6), a=1:9, b=9:1)

DT[, m:=mean(v), by=x][]              # add new column by reference by group
                                      # NB: postfix [] is shortcut to print()

DT[, sum(v), by=.(y%%2)]              # expressions in by
DT[, sum(v), by=.(bool = y%%2)]       # same, using a named list to change by column name
DT[, .SD[2], by=x]                    # get 2nd row of each group
DT[, tail(.SD,2), by=x]               # last 2 rows of each group
DT[, lapply(.SD, sum), by=x]          # sum of all (other) columns for each group
DT[, .SD[which.min(v)], by=x]         # nested query by group

DT[, list(MySum=sum(v),
          MyMin=min(v),
          MyMax=max(v)),
    by=.(x, y%%2)]                    # by 2 expressions

DT[, .(a = .(a), b = .(b)), by=x]     # list columns
DT[, .(seq = min(a):max(b)), by=x]    # j is not limited to just aggregations
DT[, sum(v), by=x][V1<20]             # compound query
DT[, sum(v), by=x][order(-V1)]        # ordering results
DT[, c(.N, lapply(.SD,sum)), by=x]    # get number of observations and sum per group
DT[, {tmp <- mean(y); 
      .(a = a-tmp, b = b-tmp)
      }, by=x]                        # anonymous lambdain 'j', j accepts any valid 
                                      # expression. TO REMEMBER: every element of 
                                      # the list becomes a column in result.
```
`?data.table`

### Benchmarks
![Benchmarks image](https://github.com/Rdatatable/data.table/wiki/bench/grouping.1E9.png)

### Internal Optimization - Auto-indexing and GForce
```
set.seed(1L)
dt = lapply(1:20, function(x) sample(c(-100:100), 5e6L, TRUE))
setDT(dt)[, id := sample(1e5, 5e6, TRUE)]
print(object.size(dt), units="Mb") # 400MB, not huge, but will do

# optimisation of 'mean'
options(datatable.optimize = 1L) # optimisation 'on'
system.time(ans1 <- dt[, lapply(.SD, mean), by=id])
#   user  system elapsed 
#  4.934   0.041   5.338 
system.time(ans2 <- dt[, lapply(.SD, base::mean), by=id])
#   user  system elapsed 
# 38.041   0.253  41.950 
identical(ans1, ans2)

# auto indexing
options(datatable.auto.index = FALSE)
system.time(ans1 <- dt[id == 100L]) # vector scan
#   user  system elapsed 
#  0.027   0.003   0.032 
system.time(ans2 <- dt[id == 100L]) # vector scan
#   user  system elapsed 
#  0.026   0.003   0.031 
 
options(datatable.auto.index = TRUE)
system.time(ans1 <- dt[id == 100L]) # index + binary search subset
#   user  system elapsed 
#  0.116   0.004   0.131 
system.time(ans2 <- dt[id == 100L]) # only binary search subset
#   user  system elapsed 
#  0.003   0.000   0.003 
```
`?datatable.optimize`

### looping with `set`
```
M = matrix(1,nrow=100000,ncol=100)
# DF = as.data.frame(M)
DT = as.data.table(M)                          #     80MB
# system.time(for (i in 1:1000) DF[i,1L] <- i) # 591.000s
system.time(for (i in 1:1000) DT[i,V1:=i])     #   1.158s
system.time(for (i in 1:1000) M[i,1L] <- i)    #   0.016s
system.time(for (i in 1:1000) set(DT,i,1L,i))  #   0.027s
```
http://brooksandrew.github.io/simpleblog/articles/advanced-data-table/#fast-looping-with-set

## merge

merge by keys (or otherwise include, for example, `by.x = "ID"`)

JOIN type | DT syntax | data.table::merge() syntax
------ | ----- | -----
INNER | X[Y, nomatch=0] | merge(X, Y, all=FALSE)
LEFT OUTER | Y[X] | merge(X, Y, all.x=TRUE)
RIGHT OUTER | X[Y] | merge(X, Y, all.y=TRUE)
FULL OUTER | - | merge(X, Y, all=TRUE)
FULL OUTER WHERE NULL (NOT INNER) | - | merge(X, Y, all=TRUE), subset NA
https://github.com/ronasta/JOINing-Data-with-R-data.table

## inrange / foverlaps

```
Y = data.table(a=c(8,3,10,7,-10), val=runif(5))
range = data.table(start = 1:5, end = 6:10)
Y[a %inrange% range]
```

```
wg <- fread("curl http://hgdownload.cse.ucsc.edu/goldenPath/hg19/encodeDCC/wgEncodeMapability/wgEncodeDacMapabilityConsensusExcludable.bed.gz | gunzip --stdout")
setnames(wg, c("Chromosome", "Start_position", "End_position", "name", "A", "B"))
wg[, Chromosome := gsub("^chr", "", Chromosome)]
setkeyv(wg, c("Chromosome", "Start_position", "End_position"))

maf_excludable <- foverlaps(maf, wg, nomatch = 0)    ### using TCGA prostate maf from earlier
nrow(maf_excludable)
# 30
```

>Usually, x is a very large data.table with small interval ranges, and y is much smaller keyed data.table with relatively larger interval spans
>
>Very briefly, foverlaps() collapses the two-column interval in y to one-column of unique values to generate a lookup table, and then performs the join depending on the type of overlap, using the already available binary search feature of data.table.

`?foverlaps`

## dcast.data.table

```
set.seed(45)
DT <- data.table(aa=sample(1e4, 1e6, TRUE), 
      bb=sample(1e3, 1e6, TRUE), 
      cc = sample(letters, 1e6, TRUE), dd=runif(1e6))
system.time(dcast(DT, aa ~ cc, fun=sum)) # 0.12 seconds
system.time(dcast(DT, bb ~ cc, fun=mean)) # 0.04 seconds
# reshape2::dcast takes 31 seconds
system.time(dcast(DT, aa + bb ~ cc, fun=sum)) # 1.2 seconds

# NEW FEATURE - multiple value.var and multiple fun.aggregate
dt = data.table(x=sample(5,20,TRUE), y=sample(2,20,TRUE), 
                z=sample(letters[1:2], 20,TRUE), d1 = runif(20), d2=1L)
# multiple value.var
dcast(dt, x + y ~ z, fun=sum, value.var=c("d1","d2"))
# multiple fun.aggregate
dcast(dt, x + y ~ z, fun=list(sum, mean), value.var="d1")
# multiple fun.agg and value.var (all combinations)
dcast(dt, x + y ~ z, fun=list(sum, mean), value.var=c("d1", "d2"))
# multiple fun.agg and value.var (one-to-one)
dcast(dt, x + y ~ z, fun=list(sum, mean), value.var=list("d1", "d2"))
```
`?dcast.data.table`


## melt.data.table
```
set.seed(45)
DT <- data.table(
      i_1 = c(1:5, NA), 
      i_2 = c(NA,6,7,8,9,10), 
      f_1 = factor(sample(c(letters[1:3], NA), 6, TRUE)), 
      f_2 = factor(c("z", "a", "x", "c", "x", "x"), ordered=TRUE), 
      c_1 = sample(c(letters[1:3], NA), 6, TRUE), 
      d_1 = as.Date(c(1:3,NA,4:5), origin="2013-09-01"), 
      d_2 = as.Date(6:1, origin="2012-01-01"))
# add a couple of list cols
DT[, l_1 := DT[, list(c=list(rep(i_1, sample(5,1)))), by = i_1]$c]
DT[, l_2 := DT[, list(c=list(rep(c_1, sample(5,1)))), by = i_1]$c]

# measure.vars can be also a list
# melt "f_1,f_2" and "d_1,d_2" simultaneously in a
# convenient way using internal function patterns()
# as well as retaining the 'factor' attribute
melt(DT, id=1:2, measure=patterns("^f_", "^d_"), value.factor=TRUE)

# same as above, but provide list of columns directly by column names or indices
melt(DT, id=1:2, measure=list(3:4, c("d_1", "d_2")), value.factor=TRUE)

# na.rm=TRUE removes rows with NAs in any 'value' columns
melt(DT, id=1:2, measure=patterns("f_", "d_"), value.factor=TRUE, na.rm=TRUE)

# return 'NA' for missing columns, 'na.rm=TRUE' ignored due to list column
melt(DT, id=1:2, measure=patterns("l_", "c_"), na.rm=TRUE)

```
`?melt.data.table`
