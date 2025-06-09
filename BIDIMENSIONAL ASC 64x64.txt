.data
    v: .space 4194304                  # Vectorul de memorie (1024 blocuri de 4 bytes fiecare)
    t: .space 4194304
    n: .space 4                     # Număr de operații
    x: .space 4                     # Dimensiune temporară pentru citire
    contor: .space 4
    contor2: .space 4
    descriptor: .space 4
    operatie: .space 4
    numar_fisiere: .space 4
    auxiliar: .space 4
    auxiliar2: .space 4
    auxiliar3: .space 4
    valoare: .space 4
    RIGHT: .space 4
    LEFT: .space 4
    linie: .space 4
    fisier: .space 4
    nr_blocuri: .space 4
    indice_final: .long 0
    dimensiune: .space 4
    formatScanf: .asciz "%d\n"      # Format pentru scanf
    formatPrintf: .asciz "%d\n"
    formatPrintAdd: .asciz "%d: ((%d, %d), (%d, %d))\n"      # Format pentru printf
    formatPrintGet: .asciz "((%d, %d), (%d, %d))\n"
    formatPrintDelete: .asciz "%d -> %d\n"
    formatHello: .asciz "Hello, World!\n"
    formatHELLO: .asciz "HELLO, WORLD!\n"

.text
.global main

main:
    # Citim numărul de operații
    push $n
    push $formatScanf
    call scanf
    add $8, %esp

    mov n, %ebx 

    mov $v, %edi
    mov $0, %ecx

# INITIALIZAM VECTORUL V CU 64 ZEROURI
init_memory:
    mov $0, %eax
    mov $0, %ebx
    
init_loop:
    cmp $1048576, %ebx
    je et_loop

    mov %eax, (%edi, %ebx, 4)
    inc %ebx
    jmp init_loop

# BUCLA PRINCIPALA PENTRU OPERATII
et_loop:
    cmp n, %ecx
    je et_exit

    mov %ecx, contor

    # Citim tipul operației
    push $x
    push $formatScanf
    call scanf
    add $8, %esp

    mov x, %eax                   # Tipul operației

    cmp $1, %eax                  # ADD - 1
    je et_add                     

    cmp $2, %eax                  # GET - 2 
    je et_get

    cmp $3, %eax                  # DELETE - 3
    je et_delete

    cmp $4, %eax                  # DEFRAG - 4
    je et_defrag

et_add_done:
et_get_done:
et_delete_done:
et_defrag_done:

    mov contor, %ecx
    inc %ecx
    jmp et_loop

# OPERATIA ADD
et_add:
    # CITIM NR FISIERE
    push $x
    push $formatScanf
    call scanf
    add $8, %esp

    mov x, %eax

    mov $0, %ecx

et_add_files:
    cmp %eax, %ecx
    je et_add_done

    mov %eax, numar_fisiere
    mov %ecx, contor2

    # CITIM DESCRIPTOR FISIER
    push $x
    push $formatScanf
    call scanf
    add $8, %esp

    mov x, %edx
    mov %edx, descriptor

    # CITIM DIMENSIUNE FISIER
    push $x
    push $formatScanf
    call scanf
    add $8, %esp

    mov x, %edx
    mov %edx, dimensiune

    mov descriptor, %ebx
    mov dimensiune, %edx

    # edx - dimensiune
    # ebx - descriptor

    jmp et_add_vector

et_add_vector_done:

    mov numar_fisiere, %eax
    mov contor2, %ecx
    inc %ecx
    jmp et_add_files

et_add_vector:
    mov %ebx, descriptor 

    add $7, %edx
    shr $3, %edx
    # Parte intraga superioara dimeniune/8

/*
    push %eax
    push %edx
    push %ecx
    push %edx
    push $formatPrintf
    call printf
    add $8, %esp
    pop %ecx
    pop %edx
    pop %eax
*/

    mov $v, %edi
    mov $0, %ebx               # INDEX INCAPUT
    mov $0, %esi               # NR ZEROURI UNUL LANGA ALTUL

et_find_blocks:
    mov (%edi, %ebx, 4), %eax

    mov %eax, auxiliar2
    mov %ebx, auxiliar
    mov %edx, dimensiune

    cmp $0, %ebx
    jne et_no_divzero

et_no8:
    mov auxiliar, %ebx
    mov dimensiune, %edx
    mov auxiliar2, %eax

    cmp $0, %eax  
    jne skip_block

    inc %esi                     # CRESTEM NR ZEROURI
    cmp %edx, %esi               # AVEM DESTULE BLOCURI ?
    je et_allocate_blocks           # DACA DA, ALOCAM

    inc %ebx                     # BLOC URMTOR
    jmp et_find_blocks

et_no_divzero:
    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx

    cmp $0, %edx
    je et_new_line

    jmp et_no8

et_new_line:
    mov $1, %esi
    mov auxiliar, %ebx
    mov dimensiune, %edx
    inc %ebx
    jmp et_find_blocks

skip_block:
    mov $0, %esi               # RESETAM NR ZEROURI
    inc %ebx                   # BLOC URMATOR
    jmp et_find_blocks

et_allocate_blocks:
    mov %ebx, %ecx              # CAPAT DEREAPTA
    mov %ecx, RIGHT
    sub %edx, %ecx               # CAPAT STANGA
    inc %ecx
    mov %ecx, LEFT

    mov $0, %esi

et_allocation_loop:
    cmp %edx, %esi               # SUNT TOARE BLOCURILE ALOCATE
    je et_element_show           # DACA DA, AFISEZ

    mov descriptor, %eax
    mov %eax, (%edi, %ecx, 4)    # PUN PE BLOC DESCRIPTORUL FISIERULUI
    inc %ecx                     # BLOC URMATOR
    inc %esi                     # INCREMENTEZ NR ALOCARI
    jmp et_allocation_loop

et_element_show:
    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CATUL IMPARTIRII(%EAX) LUI EBX LA 8 ESTE LINIA PE CARE SE AFLA FISIERUL

    mov %eax, linie

    mov LEFT, %ebx

    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CAPATUL DIN STAGA DE PE LINIE ESTE LEFT%8 (%EDX-RESTUL)

    mov %edx, LEFT

    mov RIGHT, %ebx

    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CAPATUL DIN DREAPTA DE PE LINIE ESTE DREAPTA%8 (%EDX-RESTUL)

    mov %edx, RIGHT

    mov descriptor, %eax
    mov linie, %ebx
    mov LEFT, %ecx
    mov RIGHT, %edx

    # AFISEZ DESCRIPTOR : (START (NR_LINIE, NR_COLOANA), FINISH(NR_LINIE, NR_COLOANA))
    push %edx
    push %ebx
    push %ecx
    push %ebx
    push %eax
    push $formatPrintAdd
    call printf
    add $24, %esp

    jmp et_add_vector_done

et_get:
    # CITIM DESCRIPTOR FISIER
    push $x
    push $formatScanf
    call scanf
    add $8, %esp

    mov x, %eax                   # DESCRIPTOR
    mov %eax, descriptor

    mov $v, %edi
    mov $0, %ebx

find_file:
    cmp $1048576, %ebx
    je get_file_not_found         # FISIERUL NU A FOST GASIT

    mov (%edi, %ebx, 4), %edx     # VERIF ALOCARE BLOC CURENT

    cmp %eax, %edx                # CMP CU DESCRIPTOR FISIER CAUTAT
    je get_file_found             # AM GASIT FISIER -> CAUTAM INTERVAL

    inc %ebx                      # BLOC URMATOR
    jmp find_file                 # CONTINUI CAUTAREA

get_file_found:
    mov %ebx, auxiliar2

    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CATUL IMPARTIRII(%EAX) LUI EBX LA 8 ESTE LINIA PE CARE SE AFLA FISIERUL

    mov %eax, linie

    mov auxiliar2, %ebx

    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CAPATUL DIN STAGA DE PE LINIE ESTE LEFT%8 (%EDX-RESTUL)

    mov %edx, LEFT

    mov auxiliar2, %ebx

    mov descriptor, %eax

et_file_loop:
    inc %ebx
    mov (%edi, %ebx, 4), %edx     # VERIFICAM ALOCARE BLOC CURENT

    cmp %eax, %edx                # CMP CU DESCRIPTOR FISIER
    je et_file_loop               # DACA SUNT EGALE, CONTINUAM CAUTAREA

    dec %ebx

    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CAPATUL DIN DREAPTA DE PE LINIE ESTE RIGHT%8 (%EDX-RESTUL)

    mov %edx, RIGHT

    mov linie, %ebx
    mov LEFT, %ecx
    mov RIGHT, %edx
    
    # AFISEZ COORDONATELE FISIERULUI (START : (NR_LINIE, NR_COLOANA), FINISH : (NR_LINIE, NR_COLOANA))
    push %edx
    push %ebx
    push %ecx
    push %ebx
    push $formatPrintGet
    call printf
    add $20, %esp

    # LINIA + STANGA AFISEAZA BINE

    jmp et_get_done

get_file_not_found:
    # FISIER INVALID -> AFISEZ (0,0)
    mov $0, %eax
    push %eax
    push %eax
    push %eax
    push %eax
    push $formatPrintGet
    call printf
    add $20, %esp
    
    jmp et_get_done

et_delete:
    # CITIM DESCRIPTOR FISIER
    push $x
    push $formatScanf
    call scanf
    add $8, %esp

    mov x, %eax                   # DESCRIPTOR
    mov %eax, descriptor

    mov $v, %edi 
    mov $0, %ebx                  # INDEX CAUTARE

delete_file_loop:
    cmp $1048576, %ebx
    je et_delete_print          # IES DACA NU GASESC FISIERUL

    mov (%edi, %ebx, 4), %edx     # BLOC CURENT

    cmp %eax, %edx                # CMP CU DESCRIPTOR FISIER DE STERS
    je delete_file_found          # GASIT, STERGEM FISIERUK

    inc %ebx                      # CONTINUI CAUTAREA
    jmp delete_file_loop

delete_file_found:
    mov $0, %eax
    mov %eax, (%edi, %ebx, 4)
    inc %ebx 
    mov descriptor, %eax
    jmp delete_file_loop

et_delete_print:
    mov $0, %ebx
    mov (%edi, %ebx, 4), %edx     # VOM LUA DESCRIPTORII RAND PE RAND

et_delete_print_loop:
    cmp $1048576, %ebx
    je et_delete_done

    mov (%edi, %ebx, 4), %eax

    cmp %eax, %edx
    je delete_skip_block

    jmp delete_interval

    inc %ebx
    jmp et_delete_print_loop

delete_skip_block:
    inc %ebx
    inc %ecx
    jmp et_delete_print_loop

delete_interval:
    dec %ebx
    dec %ecx

    mov %edx, valoare

    mov %ebx, RIGHT
    sub %ecx, %ebx
    mov %ebx, LEFT

    mov RIGHT, %ebx
    mov LEFT, %ecx

    cmp $0, %edx                 # DACA E 0 NU AFISEZ INTERVALUL, CAUTAM IN CELELALTE BLOCURI
    je et_skip_zero

    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CATUL IMPARTIRII(%EAX) LUI EBX LA 8 ESTE LINIA PE CARE SE AFLA FISIERUL

    mov %eax, linie

    mov LEFT, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CAPATUL DIN STAGA DE PE LINIE ESTE LEFT%8 (%EDX-RESTUL)

    mov %edx, LEFT

    mov RIGHT, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CAPATUL DIN DREAPTA DE PE LINIE ESTE RIGHT%8 (%EDX-RESTUL)

    # LINIE + LEFT E BINE
    # RIGHT E PARTIAL BINE

    # NU E BUN #EAX

    mov valoare, %eax
    mov linie, %ebx
    mov LEFT, %ecx

    # AFISEZ DESCRIPTOR : (START (NR_LINIE, NR_COLOANA), FINISH(NR_LINIE, NR_COLOANA))
    push %edx
    push %ebx
    push %ecx
    push %ebx
    push %eax
    push $formatPrintAdd
    call printf
    add $24, %esp


    mov RIGHT, %ebx

et_skip_zero:
    mov $0, %ecx
    inc %ebx
    mov (%edi, %ebx, 4), %edx

    jmp et_delete_print_loop

/*
et_delete_print:
    mov $0, %ebx

et_delete_print_loop:
    cmp $100, %ebx
    je et_delete_done

    mov (%edi, %ebx, 4), %ecx

    push %ecx
    push $formatPrintf
    call printf
    add $8, %esp

    inc %ebx
    jmp et_delete_print_loop
*/

# INITIALIZEZ MEMORIE VECTOR AUXILIAR
init_memory_t:
    mov $0, %eax 
    mov $0, %ebx
    mov $t, %esi
    
init_loop_t:
    cmp $1048576, %ebx
    je et_defrag_bloc_dimension

    mov %eax, (%esi, %ebx, 4)     # SETAM BLOCURILE CU VALOAREA 0
    inc %ebx
    jmp init_loop_t

et_defrag:
    mov $0, %ecx
    mov %ecx, indice_final
    jmp init_memory_t

et_defrag_bloc_dimension:
    mov $0, %ebx
    mov $v, %edi
    mov (%edi, %ebx, 4), %edx     # VOM LUA DESCRIPTORII RAND PE RAND

et_defrag_find_elements:
    cmp $1048576, %ebx
    je et_defrag_switch

    mov (%edi, %ebx, 4), %eax

    cmp %eax, %edx
    je defrag_skip_blocks

    jmp defrag_bloc_dimensiune

defrag_skip_blocks:
    inc %ebx
    inc %ecx
    jmp et_defrag_find_elements

defrag_bloc_dimensiune:
    dec %ebx
    dec %ecx

    mov %ebx, RIGHT
    mov %ebx, auxiliar3
    sub %ecx, %ebx
    mov %ebx, LEFT

    mov RIGHT, %ebx
    mov LEFT, %ecx

    mov %ebx, %eax
    sub %ecx, %eax
    inc %eax

    cmp $0, %edx                 # DACA E 0 NU AFISEZ INTERVALUL, CAUTAM IN CELELALTE BLOCURI
    je et_defrag_skip_zero

    # EAX - NR BLOCURI
    # EDX - DESCRIPTOR

    mov %eax, nr_blocuri
    mov %edx, fisier

    mov nr_blocuri, %edx
    mov fisier, %eax

    jmp add_matrix_t

/*
    push %eax
    push %edx
    push $formatPrintDelete
    call printf
    add $12, %esp
*/

et_matrix_t_add_vector_done:
    mov auxiliar3, %ebx
    inc %ebx
    mov $v, %edi

    jmp et_defrag_find_elements 


et_defrag_skip_zero:
    mov $0, %ecx
    inc %ebx
    mov (%edi, %ebx, 4), %edx

    jmp et_defrag_find_elements

add_matrix_t:
    mov $t, %edi
    mov indice_final, %ebx               # INDEX PASTRARE ORDINE FISIERE
    mov $0, %esi               # NR ZEROURI UNUL LANGA ALTUL

et_matrix_t_find_blocks:
    mov (%edi, %ebx, 4), %eax

    mov %eax, auxiliar2
    mov %ebx, auxiliar
    mov %edx, nr_blocuri

    cmp $0, %ebx
    jne et_matrix_t_no_divzero

et_matrix_t_no8:
    mov auxiliar, %ebx
    mov nr_blocuri, %edx
    mov auxiliar2, %eax

    cmp $0, %eax  
    jne matrix_t_skip_block

    inc %esi                     # CRESTEM NR ZEROURI
    cmp %edx, %esi               # AVEM DESTULE BLOCURI ?
    je et_matrix_t_allocate_blocks           # DACA DA, ALOCAM

    inc %ebx                     # BLOC URMTOR
    jmp et_matrix_t_find_blocks

et_matrix_t_no_divzero:
    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx

    cmp $0, %edx
    je et_matrix_t_new_line

    jmp et_matrix_t_no8

et_matrix_t_new_line:
    mov $1, %esi
    mov auxiliar, %ebx
    mov nr_blocuri, %edx
    inc %ebx
    jmp et_matrix_t_find_blocks

matrix_t_skip_block:
    mov $0, %esi               # RESETAM NR ZEROURI
    inc %ebx                   # BLOC URMATOR
    jmp et_matrix_t_find_blocks

et_matrix_t_allocate_blocks:
    mov %ebx, %ecx              # CAPAT DEREAPTA
    mov %ecx, RIGHT
    sub %edx, %ecx               # CAPAT STANGA
    inc %ecx
    mov %ecx, LEFT

    mov $0, %esi

et_matrix_t_allocation_loop:
    cmp %edx, %esi               # SUNT TOARE BLOCURILE ALOCATE
    je et_matrix_t_element_show           # DACA DA, AFISEZ

    mov fisier, %eax
    mov %eax, (%edi, %ecx, 4)    # PUN PE BLOC DESCRIPTORUL FISIERULUI
    inc %ecx                     # BLOC URMATOR
    inc %esi                     # INCREMENTEZ NR ALOCARI
    jmp et_matrix_t_allocation_loop

et_matrix_t_element_show:
    mov %ebx, indice_final

    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CATUL IMPARTIRII(%EAX) LUI EBX LA 8 ESTE LINIA PE CARE SE AFLA FISIERUL

    mov %eax, linie

    mov LEFT, %ebx

    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CAPATUL DIN STAGA DE PE LINIE ESTE LEFT%8 (%EDX-RESTUL)

    mov %edx, LEFT

    mov RIGHT, %ebx

    mov %ebx, %eax
    mov $0, %edx
    mov $1024, %ebx
    div %ebx
    # CAPATUL DIN DREAPTA DE PE LINIE ESTE DREAPTA%8 (%EDX-RESTUL)

    mov %edx, RIGHT

    mov fisier, %eax
    mov linie, %ebx
    mov LEFT, %ecx
    mov RIGHT, %edx

    # AFISEZ DESCRIPTOR : (START (NR_LINIE, NR_COLOANA), FINISH(NR_LINIE, NR_COLOANA))
    push %edx
    push %ebx
    push %ecx
    push %ebx
    push %eax
    push $formatPrintAdd
    call printf
    add $24, %esp

/*
    mov indice_final, %ebx
    push %ebx
    push $formatPrintf
    call printf
    add $8, %esp
*/

    mov indice_final, %ebx
    inc %ebx
    mov %ebx, indice_final

    jmp et_matrix_t_add_vector_done

et_defrag_switch:
    mov $t, %esi
    mov $v, %edi
    mov $0, %ebx

et_defrag_switch_matrix:
    cmp $1048576, %ebx
    je et_defrag_done

    mov (%esi, %ebx, 4), %eax              # SWITCH (t[i], v[i])
    mov %eax, (%edi, %ebx, 4)
    inc %ebx
    jmp et_defrag_switch_matrix

    # Ieșim din program
et_exit:
    mov $1, %eax
    mov $0, %ebx
    int $0x80
