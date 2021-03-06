# rio: A Swiss-army knife for data I/O #

The aim of **rio** is to make data file I/O in R as easy as possible by implementing three simple functions in Swiss-army knife style:

 - `export` and `import` provide a painless data I/O experience by automatically choosing the appropriate data read or write function based on file extension
 - `convert` wraps `import` and `export` to allow the user to easily convert between file formats (thus providing a FOSS replacement for programs like [Stat/Transfer](https://www.stattransfer.com/) or [Sledgehammer](http://www.openmetadata.org/site/?page_id=1089))

The core advantage of **rio** is that it makes assumptions that the user is probably willing to make. Two of these are important. First, **rio** uses the file extension of a file name to determine what kind of file it is. This is the same logic used by Windows OS, for example, in determining what application is associated with a given file type. By taking away the need to manually match a file type (which a beginner may not recognize) to a particular import or export function, **rio** allows almost all common data formats to be read with the same function. Second, when importing tabular data (CSV, TSV, etc.), **rio** does not convert strings to factors.

Another weakness of the common data import functions is that they only support import of web-based data from websites serving HTTP, not HTTPS. For example, data stored on GitHub as publicly visible files cannot be read directly into R without either manually downloading them or reading them in via **RCurl** or **httr**. The `import` function negates the needs for those steps by supporting HTTPS automatically. Similarly, `import` also reads from single-file .zip and .tar archives automatically, without the need to explicitly decompress them.
 
The package also wraps a variety of faster, more stream-lined I/O packages than those provided by base R or the **foreign** package. Namely, the package uses [**haven**](https://github.com/hadley/haven) for reading and writing SAS, Stata, and SPSS files and will eventually use [**fastread**](https://github.com/hadley/fastread) for intuitive import of text-delimited and fixed-width file formats.

## Supported file formats ##

**rio** supports a variety of different file formats for import and export.

| Format | Import | Export |
| ------ | ------ | ------ |
| Tab-separated data (.tsv) | Yes | Yes |
| Comma-separated data (.csv) | Yes | Yes |
| Pipe-separated data (.psv) | Yes | Yes |
| Fixed-width format data (.fwf) | Yes | Experimental |
| Serialized R objects (.rds) | Yes | Yes |
| Saved R objects (.RData) | Yes | Yes |
| JSON (.json) | Yes | Yes |
| Stata (.dta) | Yes | Yes |
| SPSS and SPSS portable (.sav and .por) | Yes | Yes (.sav only) |
| "XBASE" database files (.dbf) | Yes | Yes |
| Excel (.xlsx) | Yes | Yes |
| Weka Attribute-Relation File Format (.arff) | Yes | Yes |
| R syntax (.R) | Yes | Yes |
| SAS and SAS XPORT (.sas7bdat and .xpt) | Yes |  |
| Minitab (.mtp) | Yes |  |
| Epiinfo (.rec) | Yes |  |
| Systat (.syd) | Yes |  |
| Data Interchange Format (.dif) | Yes |  |
| Fortran data (no recognized extension) | Yes |  |
| Clipboard (as tab-separated) |  | Yes (Mac and Windows) |

## Package Installation ##

The package is available on [CRAN](http://cran.r-project.org/web/packages/rio/) and can be installed directly in R using:

```R
install.packages("rio")
```

The latest development version on GitHub can be installed using **devtools**:

```R
if(!require("devtools")){
    install.packages("devtools")
    library("devtools")
}
install_github("hadley/haven")
install_github("leeper/rio")
```

[![Build Status](https://travis-ci.org/leeper/rio.png?branch=master)](https://travis-ci.org/leeper/rio)

## Examples ##

Because **rio** is meant to streamline data I/O, the package is extremely easy to use. Here are some examples of reading, writing, and converting data files.

### Export ###

Exporting data is handled with one function, `export`:


```r
library("rio")

export(mtcars, "mtcars.csv") # comma-separated values
export(mtcars, "mtcars.rds") # R serialized
export(mtcars, "mtcars.sav") # SPSS
```

### Import ###

Importing data is handled with one function, `import`:


```r
x <- import("mtcars.csv")
y <- import("mtcars.rds")
z <- import("mtcars.sav")

# confirm data match
all.equal(x, y, check.attributes = FALSE)
```

```
## [1] TRUE
```

```r
all.equal(x, z, check.attributes = FALSE)
```

```
## [1] TRUE
```

Note: Because of inconsistencies across underlying packages, the data.frame returned by `import` might vary slightly (in variable classes and attributes) depending on file type.

### Convert ###

The `convert` function links `import` and `export` by constructing a dataframe from the imported file and immediately writing it back to disk. `convert` invisibly returns the file name of the exported file, so that it can be used to programmatically access the new file.


```r
convert("mtcars.sav", "mtcars.dta")
```

It is also possible to use **rio** on the command-line by calling `Rscript` with the `-e` (expression) argument. For example, to convert a file from Stata (.dta) to comma-separated values (.csv), simply do the following:

```
Rscript -e "rio::convert('iris.dta', 'iris.csv')"
```



