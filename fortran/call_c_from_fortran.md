# Calling C code from Fortran

There is less information on the web about this than there is for [Calling into Fortran from C++](call_fortran_from_c.md). It seems that most would prefer to have their `main()` be written in C/C++, and call into a few specialized Fortran functions as needed. However, there are use cases for both, and no need to restrict to one method.

`fortran_program.f90`:
```Fortran Free Form
program fortran_program
	use iso_c_binding
	implicit none
	interface
		subroutine c_function(a, b, c) bind(C)
			use iso_c_binding
			integer(C_INT) :: a, b, c
		end subroutine
	end interface

	integer :: a, b, c
	a = 1
	b = 2
	
	write (*, *) "Hello from Fortran"
	
	call c_function(a, b, c)
	
	write (*, *) "C Returned value:", c
	
end program
```


`c_function.cc`:
```C++
#include <stdio.h>

extern "C"
void c_function(int *a, int *b, int *c) {
	printf("Hello from C\n");
	(*c) = (*a) + (*b);
	return;
}
```

Compile with:
```Batchfile
gcc -c .\c_function.cc
gfortran .\fortran_program.f90 .\c_function.o -o test.exe
.\test.exe
```


Outputs:
```
 Hello from Fortran
Hello from C
 C Returned value:           3
```

### Compiling with GCC
Note that the compilation order may be reversed, so that GCC does the final step. If compiling a `.o` file from Fortran using `gcc`, we need to add the `-lgfortran` switch, to let `gcc` be aware of some function calls within the Fortran files. For the above code, we can do:

```Batchfile
gfortran .\fortran_program.f90
gcc -c .\c_function.cc .\fortran_program.o -o test.exe
.\test.exe
```

# References
This method is described in _Guide to Fortran 2008 Programming_ by Walter S. Brainerd. However, in this description, the `extern "C"` line is missing (perhaps because the author is using a C compiler here, not C++? In any case, addition of `extern "C"` lets this example compile.

