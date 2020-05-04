# how to build R packages with fortran functions
how to build R packages with fortran functions

1. In R studio, create R package project by     
File -> New project -> New directory -> R package with Rcpp

2. Put fortran functions in /scr (e.g. f77.f, f90.f90)

3. In NAMESPACE, make sure that the followings are there

useDynLib(packagewithfortran2, .registration=TRUE)
exportPattern("^[[:alpha:]]+")
importFrom(Rcpp, evalCpp)

4. Wrap the functions using R functions:

f90add_dotfortran <- function(a, b)
{
  ret <- .Fortran("f90add", as.double(a), as.double(b), ret=double(1))
  ret$ret
}

5. Build the package. Done.

Reference: https://github.com/wrathematics/CompiledExamples
