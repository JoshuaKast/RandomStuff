# Calling into Fortran from C++

This is on Windows 10, MinGW64 compiler. 

## Basic operation:
First the C++ part:

`c_caller.cc`:
```C++
#include <stdio.h>

// Tell C++ how to interact with the Fortran subroutine:
// Our subroutine is of type "int", and it accepts two "int" as arguments
extern "C" int fortran_add(int n1, int n2);


// Main function.
// Here we will just call into the Fortran code
int main() {
	int response;
	printf("Hello from C\n");
	response = fortran_add(2, 3);
	printf("Fortran returns %d\n", response);
	return 0;
}
```

Now the Fortran part:
```Fortran Free Form
module fortran_function
use iso_c_binding
implicit none

contains

	! This is the subroutine being called by C++
	function fortran_add(a, b) result(c) bind(C, name='fortran_add') 
		integer(c_int), value :: a, b
		integer(c_int) :: c
		
		write (*, *) "Hello from Fortran"
		c = a + b
		
	end function
end module
```

Compile with:
```Batchfile
gcc -c .\c_caller.cc
gfortran .\c_caller.o .\fortran_function.f90 -o test.exe
.\test.exe
```

Output:
```
Hello from C
 Hello from Fortran
Fortran returns 5
 ```

## With a callback function:
This could be used, for example, for the Fortran subroutine to check back with the C++ periodically, and report progress or see if the user wants to cancel.

First the C++ part. This has two parts: a "main" function, and a "callback":

`c_caller.cc`:
```C++
#include <stdio.h>

// Tell C++ how to interact with the Fortran subroutine:
// Our subroutine is of type "void" (returns nothing), and accepts one argument: a function pointer
extern "C" void fortran_subroutine(int (*function)());

// This function is being called "back" from Fortran
// Can use this to update the C++ portion of the program on progress, or to ask things from the C++ part
int callback_function() {
	printf("Hello from callback!\n");
	return 1;
}

// Main function.
// Here we will just call into the Fortran code
int main() {
   printf("Hello from C\n");
   fortran_subroutine(callback_function);
   return 0;
}
```

Now the Fortran part: just has the one subroutine:

`fortran_function.f90`:
```Fortran Free Form
module fortran_function
use iso_c_binding
implicit none

contains

	! This is the subroutine being called by C++
	subroutine fortran_subroutine(c_callback) bind(C, name='fortran_subroutine')
		
		! Interface for the function "c_callback". This lets Fortran know how to interact with the function
		interface
			integer(C_INT) function c_callback() bind(C)
			use iso_c_binding
			end function
		end interface
		
		write (*, *) "Hello from fortran"
		
		! Call back into C++
		write (*, *) "C Says: ", c_callback()

	end subroutine
end module
```

Compile and run:
```Batchfile
gcc -c .\c_caller.cc
gfortran .\c_caller.o .\fortran_function.f90 -o test.exe
.\test.exe
```

Output:
```
Hello from C
 Hello from fortran
Hello from callback!
 C Says:            1
 ```
 
 ## With additional libraries
 Using libraries is trivial when they're added to the Fortran part. When adding to the C++ part, it's more challenging, since the last compilation step is done with `gfortran`. To compile with additional libraries in the C++ part, do:
 ```Batchfile
gcc -c .\c_caller.cc
gfortran .\c_caller.o .\fortran_function.f90 -o test.exe -llibname
.\test.exe
```
We add to the last compiler calls (it won't hurt it to add to `gcc` call either, but it won't change anything). The `gfortran` doesn't intercept the call directly - it passes things on to the linker `ld`. What this all means - we can do the final compilation with either `gcc` or `gfortran`, with one caveat: `gcc` needs one library which is provided by default to `gfortran`, which is `-lgfortran`. If we do:
 ```Batchfile
gfortran .\fortran_function.f90 -c
gcc .\c_caller.cc .\fortran_function.o -llibname -o test.exe
```
We will see an error:
```
undefined reference to `_gfortran_st_write'
undefined reference to `_gfortran_transfer_character_write'
undefined reference to `_gfortran_st_write_done'
```

But add `-lgfortran` and all is well:
 ```Batchfile
gfortran .\fortran_function.f90 -c
gcc .\c_caller.cc .\fortran_function.o -llibname -lgfortran -o test.exe
```
