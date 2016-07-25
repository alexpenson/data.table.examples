# data.table.examples


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
| inrange / foverlaps |  |  |
| duplicated, frank etc. |  |  |
