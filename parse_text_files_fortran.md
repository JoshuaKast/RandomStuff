# Parsing Text Data Files

This is a sketch of a function used to see how many rows and columns are in a text data file. For now, it only understands `#` as
a comment character, and doesn't check columns for each row (among other limitations).

```Fortran Free Form
program main
    implicit none
    integer, dimension(2) :: fsize

    fsize = data_file_size('test_data.txt')

    write (*, '(A, I0)') 'Number of rows = ', fsize(2)
    write (*, '(A, I0)') 'Number of columns = ', fsize(1)

contains

function data_file_size(filename) result(size)
    implicit none
    character(len=*), intent(in) :: filename
    integer, dimension(2) :: size

    integer, parameter :: infile = 10
    integer, parameter :: maximum_number_of_columns = 1024
    integer, parameter :: maximum_line_length = 1024
    character(len=15) :: format_string
    integer :: number_of_rows, number_of_columns, ierror, i
    logical :: success
    character(len=maximum_line_length) :: input_line
    real, dimension(maximum_number_of_columns) :: test_array
    character :: input_char
    
    write (format_string, '(AI0A)') '(A', maximum_line_length, ')'

    open(unit=infile, file=filename, status='old', action='read', iostat=ierror)

    do
        read(infile, format_string, iostat=ierror) input_line
        if (input_line(1:1) == '#') then
            read(infile, *)
            cycle
        endif
       
        if (ierror/=0) then
            write (*,*) 'End of file encountered before any data was read.'
            stop
        endif
        exit
    end do

    do number_of_columns = 1, maximum_number_of_columns
        read(input_line, *, iostat=ierror) test_array(1:number_of_columns)
        if (ierror == -1) exit
    end do
    number_of_columns = number_of_columns - 1
    write (*, '(A, I0)') 'Number of columns = ', number_of_columns

    number_of_rows = 0
    rewind(infile)

    do
        read(infile, format_string, iostat=ierror) input_line
        if (ierror/=0) then
            exit
        endif

        if (input_line(1:1) == '#') then
            read(infile, *)
            cycle
        endif

        read(input_line, *, iostat=ierror) test_array(1:number_of_columns)
        if (ierror /= 0) then
            write (*,*) 'Inconsistant number of data columns.'
            stop
        endif

        number_of_rows = number_of_rows + 1
    end do

    write (*, '(A, I0)') 'Number of data rows = ', number_of_rows
    
    close(infile)

    size(1) = number_of_columns
    size(2) = number_of_rows
end


end program main

```
