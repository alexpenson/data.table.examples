# data.table.examples

| Functions  | 
| ---- | 
| fread / fwrite |  
| dt[i, j, by] |  
| merge / rbind | 
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

## fread (fast and friendly)
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
```
system.time(fwrite(dt, "~/tmp.tsv", quote=FALSE, sep="\t"))
#   user  system elapsed 
#  2.643   0.291   1.710
write.table(dt, "tmp2.tsv", 
                quote = F,
                col.names=T,
                row.names=F,
                sep='\t')
#   user  system elapsed 
# 72.020   1.363  78.741 
```

### Reading a maf file
```
maf <- fread("https://tcga-data.nci.nih.gov/docs/publications/prad_2015/PRAD_Capture_All_Pairs_QCPASS_v6_Nikki_Nov_25.aggregated.capture.tcga.uuid.curated.somatic.maf", verbose = T)
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
### keys

```
maf <- fread("example.maf", key = "Tumor_Sample_Barcode")
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

`j` must return a list

```
maf[, as.list(summary(t_depth)), Tumor_Sample_Barcode]
```

### Internal Optimization - Auto-indexing and GForce
`?datatable.optimize`
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

`Unable to optimize call to mean() and could be very slow.`

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


```
microbenchmark::microbenchmark({setthreads(32); CJ(1:1000,1:10000)}, {setthreads(1); CJ(1:1000,1:10000)}, times = 500)

truelength
alloc.col
```

## inrange / foverlaps

`?foverlaps`

Usually, x is a very large data.table with small interval ranges, and y is much smaller keyed data.table with relatively larger interval spans

Very briefly, foverlaps() collapses the two-column interval in y to one-column of unique values to generate a lookup table, and then performs the join depending on the type of overlap, using the already available binary search feature of data.table.

https://github.com/hadley/dtplyr
