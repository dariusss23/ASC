.data
    v: .space 4096                  # Vectorul de memorie (1024 blocuri de 4 bytes fiecare)
    t: .space 4096
    n: .space 4                     # Număr de operații
    x: .space 4                     # Dimensiune temporară pentru citire
    contor: .space 4
    contor2: .space 4
    descriptor: .space 4
    operatie: .space 4
    numar_fisiere: .space 4
    RIGHT: .space 4
    LEFT: .space 4
    dimensiune: .space 4
    formatScanf: .asciz "%d\n"      # Format pentru scanf
    formatPrintf: .asciz "%d\n"
    formatPrintAdd: .asciz "%d: (%d, %d)\n"      # Format pentru printf
    formatPrintGet: .asciz "(%d, %d)\n"
    formatPrintDelete: .asciz "%d: (%d, %d)\n"
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

# INITIALIZAM VECTORUL V CU 1024 ZEROURI
init_memory:
    mov $0, %eax
    mov $0, %ebx
    
init_loop:
    cmp $1024, %ebx
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
    cmp $1024, %ebx
    je et_error

    mov (%edi, %ebx, 4), %eax    

    cmp $0, %eax  
    jne skip_block

    inc %esi                     # CRESTEM NR ZEROURI
    cmp %edx, %esi               # AVEM DESTULE BLOCURI ?
    je et_allocate_blocks           # DACA DA, ALOCAM

    inc %ebx                     # BLOC URMTOR
    jmp et_find_blocks

et_error:
    mov $0, %eax
    mov descriptor, %ebx
    push %eax
    push %eax
    push %ebx
    push $formatPrintAdd
    call printf
    add $12, %esp

    jmp et_add_vector_done
    
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
    mov RIGHT, %edx
    mov descriptor, %eax
    mov LEFT, %ecx

    # AFISEZ DESCRIPTOR : (CAPAT_STANGA, CAPAT_DREAPTA)
    push %edx
    push %ecx
    push %eax
    push $formatPrintAdd
    call printf
    add $8, %esp

    jmp et_add_vector_done

et_get:
    # CITIM DESCRIPTOR FISIER
    push $x
    push $formatScanf
    call scanf
    add $8, %esp

    mov x, %eax                   # DESCRIPTOR

    mov $v, %edi
    mov $0, %ebx                  # EBX - NR BLOCURI OCUPATE

find_file:
    cmp $1024, %ebx
    je get_file_not_found         # FISIERUL NU A FOST GASIT

    mov (%edi, %ebx, 4), %edx     # VERIF ALOCARE BLOC CURENT

    cmp %eax, %edx                # CMP CU DESCRIPTOR FISIER CAUTAT
    je get_file_found             # AM GASIT FISIER -> CAUTAM INTERVAL

    inc %ebx                      # BLOC URMATOR
    jmp find_file                 # CONTINUI CAUTAREA

get_file_found:
    mov %ebx, LEFT

/*
    push %ebx
    push $formatPrintf
    call printf
    add $8, %esp
*/

et_file_loop:
    inc %ebx
    mov (%edi, %ebx, 4), %edx     # VERIFICAM ALOCARE BLOC CURENT

    cmp %eax, %edx                # CMP CU DESCRIPTOR FISIER
    je et_file_loop               # DACA AM GASIT -> AFISEZ INTERVAL

    dec %ebx
    
    # AFISEZ INTERVALUL UNDE IL GASIM (CAPAT_STANGA, CAPAT_DREAPTA)
    mov LEFT, %eax
    push %ebx
    push %eax
    push $formatPrintGet
    call printf
    add $12, %esp

    jmp et_get_done

get_file_not_found:
    # FISIER INVALID -> AFISEZ (0,0)
    mov $0, %eax
    push %eax
    push %eax
    push $formatPrintGet
    call printf
    add $8, %esp
    
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
    cmp $1024, %ebx
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
    cmp $1024, %ebx
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

    mov %ebx, RIGHT
    sub %ecx, %ebx
    mov %ebx, LEFT

    mov RIGHT, %ebx
    mov LEFT, %ecx

    cmp $0, %edx                 # DACA E 0 NU AFISEZ INTERVALUL, CAUTAM IN CELELALTE BLOCURI
    je et_skip_zero
    
    push %ebx
    push %ecx
    push %edx
    push $formatPrintDelete
    call printf
    add $16, %esp

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
    cmp $1024, %ebx
    je et_defrag_loop2

    mov %eax, (%esi, %ebx, 4)     # SETAM BLOCURILE CU VALOAREA 0
    inc %ebx
    jmp init_loop_t

et_defrag:
    jmp init_memory_t

et_defrag_loop2:
    mov $0, %ebx
    mov $t, %esi

et_defrag_loop3:
    mov $0, %ebx
    mov $0, %ecx
    mov $v, %edi
    mov $t, %esi

et_defrag_loop:
    cmp $1024, %ebx
    je et_defrag_add_zeros

    mov (%edi, %ebx, 4), %eax     # BLOC CURENT DIN V[]
    
    cmp $0, %eax                          # DACA NU E 0, IL PUNEM IN T[]
    jne et_defrag_add_elem

    inc %ebx                             # DACA E 0, TRECEM MAI DEPARTE
    jmp et_defrag_loop

et_defrag_add_elem:
    mov %eax, (%esi, %ecx, 4)
    inc %ecx
    mov %ecx, contor2                    # POZITIE ULTIMUL ELEMENT NENUL
    inc %ebx

    jmp et_defrag_loop                 # CONTINUI CAUTAREA

et_defrag_add_zeros:
    mov contor2, %ebx                    # DE LA ULTIMUL ELEMENT NENUL COMPLETEZ CU 0
    mov $0, %ecx
    mov $t, %esi

et_defrag_add_values_loop:
    cmp %ebx, %ecx
    je et_defrag_add_zeros_loop

    mov (%esi, %ecx, 4), %eax

    inc %ecx
    jmp et_defrag_add_values_loop

et_defrag_add_zeros_loop:
    cmp $1024, %ecx 
    je et_defrag_switch 

    mov $0, %eax
    mov %eax, (%esi, %ecx, 4)     # SETEZ BLOC CU 0
    inc %ecx                      # TREC MAI DEPARTE
    jmp et_defrag_add_zeros_loop


# v[i] <- t[i] ORICARE i = 1 -> 1024
et_defrag_switch:
    mov $t, %esi
    mov $v, %edi
    mov $0, %ebx

et_defrag_switch_arrays:
    cmp $1024, %ebx
    je et_defrag_print

    mov (%esi, %ebx, 4), %eax              # SWITCH (t[i], v[i])
    mov %eax, (%edi, %ebx, 4)
    inc %ebx
    jmp et_defrag_switch_arrays

# CE ESTE MAI JOS ESTE LA FEL CA PRINT DELETE

et_defrag_print:
    mov $0, %ebx
    mov $0, %ecx
    mov (%edi, %ebx, 4), %edx

et_defrag_print_loop:
    cmp $1024, %ebx
    je et_defrag_done 

    mov (%edi, %ebx, 4), %eax

    cmp %eax, %edx
    je defrag_skip_block

    jmp defrag_interval

    inc %ebx
    jmp et_defrag_print_loop

defrag_skip_block:
    inc %ebx
    inc %ecx
    jmp et_defrag_print_loop

defrag_interval:
    dec %ebx
    dec %ecx

    mov %ebx, RIGHT
    sub %ecx, %ebx
    mov %ebx, LEFT

    mov RIGHT, %ebx
    mov LEFT, %ecx

    cmp $0, %edx
    je et_defrag_skip_zero
    
    push %ebx
    push %ecx
    push %edx
    push $formatPrintDelete
    call printf
    add $16, %esp

et_defrag_skip_zero:
    mov $0, %ecx
    inc %ebx
    mov (%edi, %ebx, 4), %edx

    jmp et_defrag_print_loop

    # Ieșim din program
et_exit:
    mov $1, %eax
    mov $0, %ebx
    int $0x80
