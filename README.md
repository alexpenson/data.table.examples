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

setthreads()
truelength
alloc.col
```

| Example | speed | usability |
| ---- | --- | --- |
| fread |  |  |
