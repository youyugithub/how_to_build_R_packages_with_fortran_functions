# how to build R packages with fortran functions
how to build R packages with fortran functions

1. In R studio, create R package project by     
File -> New project -> New directory -> R package with Rcpp

2. Put fortran functions in /scr (e.g. f77.f, f90.f90)

3. In NAMESPACE, make sure that the followings are there

```
useDynLib(packagewithfortran2, .registration=TRUE)
exportPattern("^[[:alpha:]]+")
importFrom(Rcpp, evalCpp)
```

4. Wrap the functions using R functions:

```
f90add_dotfortran <- function(a, b)
{
  ret <- .Fortran("f90add", as.double(a), as.double(b), ret=double(1))
  ret$ret
}
```

5. Build the package. Done.

Reference: https://github.com/wrathematics/CompiledExamples

# However the above method won't work when there are combinations of .f and c++ arma files

Here is a workaround by using a c wrapper:

1. Put [f90.f90] file in src:
```
subroutine add(AA,ndim)
integer::ndim
double precision::AA(ndim,ndim,ndim)
AA(1,1,1)=AA(1,1,1)+1d0
end subroutine
```
2. write c wrapper [wrappers_c.c]
```
#include <R.h>
#include <Rinternals.h>
#include "prototypes.h"

SEXP R_f90add(SEXP AA, SEXP ndim)
{
  //SEXP ret;
  //PROTECT(ret = allocVector(REALSXP, 1));
  
  F77_CALL(add)(REAL(AA), ndim);
  
  //UNPROTECT(1);
  return AA;
}
```
3. write c header [prototypes.h]
```
#ifndef PROTOTYPES_H
#define PROTOTYPES_H

#include <R.h>
#include <Rinternals.h>

#ifdef __cplusplus
extern "C" {
#endif

void F77_NAME(add)(double *AA, int *ndim);

#ifdef __cplusplus
}
#endif

#endif
```
4. Finally, add an R wrapper [fortran_wrappers.R]
```
f90add_cwrap <- function(AA, ndim)
{
  .Call("R_f90add", AA, ndim)
}
```

Reference: https://github.com/wrathematics/CompiledExamples

# When there are arrays data transfer from C to fortran will cause errors.

In this case, put the following in the [src/Makevars]:

```
PKG_CFLAGS = -w
PKG_FFLAGS = $(SAFE_FFLAGS) -w
PKG_LIBS = $(RCPP_LDFLAGS) $(FLIBS) -lstdc++

FT_OBJS = f90.o f90add.o f90_local_const_cov_est_mult.o 

OBJECTS = $(CISH_OBJS) $(FT_OBJS) $(R_OBJS)

all: $(SHLIB)
$(SHLIB): $(OBJECTS)


clean:
	@rm -rf *.o *.mod *.d *.rc *.so *.dylib *.dll *.a *.lib $(SHLIB) $(OBJECTS)
```

f90.o f90add.o f90_local_const_cov_est_mult.o 

is the list of fortran o files
