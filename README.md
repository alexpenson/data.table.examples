# data.table.examples

https://github.com/Rdatatable/data.table/wiki
```
install.packages("data.table", type = "source",
    repos = "http://Rdatatable.github.io/data.table")
```

https://s3.amazonaws.com/assets.datacamp.com/img/blog/data+table+cheat+sheet.pdf

http://www.r-bloggers.com/advanced-tips-and-tricks-with-data-table/

## fread

Reading a large maf file
```
> fillout <- fread("Proj_05927_C___FILLOUT.maf", verbose = T)
Input contains no \n. Taking this to be a filename to open
File opened, filesize is 1.605653 GB.
Memory mapping ... ok
Detected eol as \n only (no \r afterwards), the UNIX and Mac standard.
Positioned on line 1 after skip or autostart
This line is the autostart and not blank so searching up for the last non-blank ... line 1
Detecting sep ... '\t'
Detected 109 columns. Longest stretch was from line 2 to line 30
Starting data input on line 2 (either column names or first row of data). First 10 characters: Hugo_Symbo
All the fields on line 2 are character fields. Treating as the column names.
Count of eol: 1981300 (including 1 at the end)
Count of sep: 213980292
nrow = MIN( nsep [213980292] / (ncol [109] -1), neol [1981300] - endblanks [1] ) = 1981299
Type codes ( first 5 rows): 4144111444444404444000000000000000444441110004444444444441014414444444444444444044444040000041400413333333304
Type codes (middle 5 rows): 4144411444444404444000000000000000444441110004444444444441014414444444444444444044444040000041400413333333304
Type codes (  last 5 rows): 4144411444444404444000000000000000444441110004444444444441014414444444444444444044444040000041400413333333304
Type codes: 4144411444444404444000000000000000444441110004444444444441014414444444444444444044444040000041400413333333304 (after applying colClasses and integer64)
Type codes: 4144411444444404444000000000000000444441110004444444444441014414444444444444444044444040000041400413333333304 (after applying drop or select (if supplied)
Allocating 109 column slots (109 - 0 dropped)
Read 0.0% of 1981299 rows
Bumping column 108 from LGL to INT on data row 6, field contains '1'
Read 1981299 rows and 109 (of 109) columns from 1.606 GB file in 00:00:36
Read 1981299 rows. Exactly what was estimated and allocated up front
   0.000s (  0%) Memory map (rerun may be quicker)
   0.001s (  0%) sep and header detection
   9.654s ( 27%) Count rows (wc -l)
   0.004s (  0%) Column type detection (first, middle and last 5 rows)
   1.800s (  5%) Allocation of 1981299x109 result (xMB) in RAM
  24.113s ( 67%) Reading data
   0.215s (  1%) Allocation for type bumps (if any), including gc time if triggered
   0.003s (  0%) Coercing data already read in type bumps (if any)
   0.185s (  1%) Changing na.strings to NA
  35.976s        Total
```


## dt[i, j, by]

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

```
set.seed(45L)
DT <- data.table(V1=c(1L,2L),
V2=LETTERS[1:3],
V3=round(rnorm(4),4),
V4=1:12)

DT[, as.list(summary(V3)), V1]
```

```
microbenchmark::microbenchmark({setthreads(32); CJ(1:1000,1:10000)}, {setthreads(1); CJ(1:1000,1:10000)}, times = 500)

truelength
alloc.col
```

# inrange / foverlaps

`?foverlaps`

Usually, x is a very large data.table with small interval ranges, and y is much smaller keyed data.table with relatively larger interval spans

Very briefly, foverlaps() collapses the two-column interval in y to one-column of unique values to generate a lookup table, and then performs the join depending on the type of overlap, using the already available binary search feature of data.table.


| function | speed | usability |
| ---- | --- | --- |
| fread / fwrite |  |  |
| dt[i, j, by] |  |  |
| merge / rbind | | |
| inrange / foverlaps |  |  |
| dcast / melt |  |  |
| duplicated, frank etc. |  |  |

https://github.com/hadley/dtplyr
