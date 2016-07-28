# data.table.examples

https://github.com/Rdatatable/data.table/wiki
```
install.packages("data.table", type = "source",
    repos = "http://Rdatatable.github.io/data.table")
```

https://s3.amazonaws.com/assets.datacamp.com/img/blog/data+table+cheat+sheet.pdf

http://www.r-bloggers.com/advanced-tips-and-tricks-with-data-table/

```
set.seed(45L)
DT <- data.table(V1=c(1L,2L),
V2=LETTERS[1:3],
V3=round(rnorm(4),4),
V4=1:12)

DT[, as.list(summary(V3)), V1]


fwrite

%between%
```

```
microbenchmark::microbenchmark({setthreads(32); CJ(1:1000,1:10000)}, {setthreads(1); CJ(1:1000,1:10000)}, times = 500)
```

```
truelength
alloc.col
```

| function | speed | usability |
| ---- | --- | --- |
| fread / fwrite |  |  |
| dcast / melt |  |  |
| rbind | | |
| inrange / foverlaps |  |  |
| duplicated, frank etc. |  |  |

https://github.com/hadley/dtplyr
