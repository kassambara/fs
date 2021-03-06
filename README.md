
<!-- README.md is generated from README.Rmd. Please edit that file -->

# fs

[![lifecycle](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://tidyverse.org/lifecycle/#maturing)
[![Travis build
status](https://travis-ci.org/r-lib/fs.svg?branch=master)](https://travis-ci.org/r-lib/fs)
[![AppVeyor Build
Status](https://ci.appveyor.com/api/projects/status/github/r-lib/fs?branch=master&svg=true)](https://ci.appveyor.com/project/r-lib/fs)
[![Coverage
status](https://codecov.io/gh/r-lib/fs/branch/master/graph/badge.svg)](https://codecov.io/github/r-lib/fs?branch=master)

<p align="center">

<img src="https://i.imgur.com/NAux1Xc.png" width = "75%"/>

</p>

**fs** provides a cross-platform, uniform interface to file system
operations. It shares the same back-end component as
[nodejs](https://nodejs.org), the
[libuv](http://docs.libuv.org/en/v1.x/fs.html) C library, which brings
the benefit of extensive real-world use and rigorous cross-platform
testing. The name, and some of the interface, is partially inspired by
Rust’s [fs module](https://doc.rust-lang.org/std/fs/index.html).

## Installation

You can install the released version of **fs** from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("fs")
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("r-lib/fs")
```

## Comparison vs base equivalents

**fs** functions smooth over some of the idiosyncrasies of file handling
with base R functions:

  - Vectorization. All **fs** functions are vectorized, accepting
    multiple paths as input. Base functions are inconsistently
    vectorized.

  - Predictable return values that always convey a path. All **fs**
    functions return a character vector of paths, a named integer or a
    logical vector, where the names give the paths. Base return values
    are more varied: they are often logical or contain error codes which
    require downstream processing.

  - Explicit failure. If **fs** operations fail, they throw an error.
    Base functions tend to generate a warning and a system dependent
    error code. This makes it easy to miss a failure.

  - UTF-8 all the things. **fs** functions always convert input paths to
    UTF-8 and return results as UTF-8. This gives you path encoding
    consistency across OSes. Base functions rely on the native system
    encoding.

  - Naming convention. **fs** functions use a consistent naming
    convention. Because base R’s functions were gradually added over
    time there are a number of different conventions used (e.g.
    `path.expand()` vs `normalizePath()`; `Sys.chmod()` vs
    `file.access()`).

### Tidy paths

**fs** functions always return ‘tidy’ paths. Tidy paths

  - Always use `/` to delimit directories
  - never have multiple `/` or trailing `/`

Tidy paths are also coloured (if your terminal supports it) based on the
file permissions and file type. This colouring can be customised or
extended by setting the `LS_COLORS` environment variable, in the same
output format as [GNU
dircolors](http://www.bigsoft.co.uk/blog/index.php/2008/04/11/configuring-ls_colors).

## Usage

**fs** functions are divided into four main categories:

  - `path_` for manipulating and constructing paths
  - `file_` for files
  - `dir_` for directories
  - `link_` for links

Directories and links are special types of files, so `file_` functions
will generally also work when applied to a directory or link.

``` r
library(fs)

# Construct a path to a file with `path()`
path("foo", "bar", letters[1:3], ext = "txt")
#> foo/bar/a.txt foo/bar/b.txt foo/bar/c.txt

# list files in the current directory
dir_ls()
#> CRAN-RELEASE         DESCRIPTION          LICENSE.md           
#> NAMESPACE            NEWS.md              R                    
#> README.Rmd           README.md            _pkgdown.yml         
#> appveyor.yml         bar                  check.R              
#> codecov.yml          cran-comments.md     docs                 
#> example              follow.R             fs.Rcheck            
#> fs.Rproj             fs_1.2.2.9000.tar.gz inst                 
#> man                  man-roxygen          script.R             
#> src                  tests

# create a new directory
tmp <- dir_create(file_temp())
tmp
#> /var/folders/dt/r5s12t392tb5sk181j3gs4zw0000gn/T/RtmpM84M38/file16f7126ece07c

# create new files in that directory
file_create(path(tmp, "my-file.txt"))
dir_ls(tmp)
#> /var/folders/dt/r5s12t392tb5sk181j3gs4zw0000gn/T/RtmpM84M38/file16f7126ece07c/my-file.txt

# remove files from the directory
file_delete(path(tmp, "my-file.txt"))
dir_ls(tmp)
#> character(0)

# remove the directory
dir_delete(tmp)
```

**fs** is designed to work well with the pipe, though because it is a
minimal-dependency infrastructure package it doesn’t provide the pipe
itself. You will need to attach
[magrittr](http://magrittr.tidyverse.org) or similar.

``` r
library(magrittr)

paths <- file_temp() %>%
  dir_create() %>%
  path(letters[1:5]) %>%
  file_create()
paths
#> /var/folders/dt/r5s12t392tb5sk181j3gs4zw0000gn/T/RtmpM84M38/file16f713ca22ebf/a
#> /var/folders/dt/r5s12t392tb5sk181j3gs4zw0000gn/T/RtmpM84M38/file16f713ca22ebf/b
#> /var/folders/dt/r5s12t392tb5sk181j3gs4zw0000gn/T/RtmpM84M38/file16f713ca22ebf/c
#> /var/folders/dt/r5s12t392tb5sk181j3gs4zw0000gn/T/RtmpM84M38/file16f713ca22ebf/d
#> /var/folders/dt/r5s12t392tb5sk181j3gs4zw0000gn/T/RtmpM84M38/file16f713ca22ebf/e

paths %>% file_delete()
```

**fs** functions also work well in conjunction with other
[tidyverse](https://www.tidyverse.org/) packages, like
[dplyr](http://dplyr.tidyverse.org) and
[purrr](http://purrr.tidyverse.org).

Some examples…

``` r
suppressMessages(
  library(tidyverse))
```

Filter files by type, permission and size

``` r
dir_info("src", recursive = FALSE) %>%
  filter(type == "file", permissions == "u+r", size > "10KB") %>%
  arrange(desc(size)) %>%
  select(path, permissions, size, modification_time)
#> # A tibble: 10 x 4
#>    path                permissions        size modification_time  
#>    <fs::path>          <fs::perms> <fs::bytes> <dttm>             
#>  1 src/RcppExports.o   rw-r--r--        655.5K 2018-05-20 17:39:19
#>  2 src/dir.o           rw-r--r--        442.7K 2018-05-20 17:39:19
#>  3 src/fs.so           rwxr-xr-x        435.3K 2018-05-20 17:39:29
#>  4 src/id.o            rw-r--r--        383.2K 2018-05-20 17:39:18
#>  5 src/file.o          rw-r--r--        347.5K 2018-05-20 17:39:18
#>  6 src/path.o          rw-r--r--        257.4K 2018-05-20 17:39:18
#>  7 src/link.o          rw-r--r--        224.3K 2018-05-20 17:39:18
#>  8 src/utils.o         rw-r--r--        117.9K 2018-05-20 17:39:18
#>  9 src/error.o         rw-r--r--         17.3K 2018-05-20 17:39:15
#> 10 src/RcppExports.cpp rw-r--r--         11.4K 2018-05-20 17:39:14
```

Tabulate and display folder size.

``` r
dir_info("src", recursive = TRUE) %>%
  group_by(directory = path_dir(path)) %>%
  tally(wt = size, sort = TRUE)
#> # A tibble: 54 x 2
#>    directory                                        n
#>    <fs::path>                             <fs::bytes>
#>  1 src                                          2.86M
#>  2 src/libuv                                    2.44M
#>  3 src/libuv/src/unix                           1.08M
#>  4 src/libuv/autom4te.cache                     1.08M
#>  5 src/libuv/test                             865.36K
#>  6 src/libuv/src/win                          683.14K
#>  7 src/libuv/docs/src/static                  328.32K
#>  8 src/libuv/m4                               319.95K
#>  9 src/libuv/include                          192.33K
#> 10 src/libuv/docs/src/static/diagrams.key     184.04K
#> # ... with 44 more rows
```

Read a collection of files into one data frame.

`dir_ls()` returns a named vector, so it can be used directly with
`purrr::map_df(.id)`.

``` r
# Create separate files for each species
iris %>%
  split(.$Species) %>%
  map(select, -Species) %>%
  iwalk(~ write_tsv(.x, paste0(.y, ".tsv")))

# Show the files
iris_files <- dir_ls(glob = "*.tsv")
iris_files
#> setosa.tsv     versicolor.tsv virginica.tsv

# Read the data into a single table, including the filenames
iris_files %>%
  map_df(read_tsv, .id = "file", col_types = cols(), n_max = 2)
#> # A tibble: 6 x 5
#>   file           Sepal.Length Sepal.Width Petal.Length Petal.Width
#>   <chr>                 <dbl>       <dbl>        <dbl>       <dbl>
#> 1 setosa.tsv              5.1         3.5          1.4         0.2
#> 2 setosa.tsv              4.9         3            1.4         0.2
#> 3 versicolor.tsv          7           3.2          4.7         1.4
#> 4 versicolor.tsv          6.4         3.2          4.5         1.5
#> 5 virginica.tsv           6.3         3.3          6           2.5
#> 6 virginica.tsv           5.8         2.7          5.1         1.9

file_delete(iris_files)
```

## Feedback wanted\!

We hope **fs** is a useful tool for both analysis scripts and packages.
Please open [GitHub issues](https://github.com/r-lib/fs) for any feature
requests or bugs.

In particular, we have found non-ASCII filenames in non-English locales
on Windows to be especially tricky to reproduce and handle correctly.
Feedback from users who use commonly have this situation is greatly
appreciated.
