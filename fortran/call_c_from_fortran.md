# Calling C code from Fortran

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
