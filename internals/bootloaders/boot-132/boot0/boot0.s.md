# boot0.s

This is a work in progress article. See incorrect information or want to add on? Submit a pull request and contribute!

## The Start of the Bootloader

```Assembly
;--------------------------------------------------------------------------
; Boot code is loaded at 0:7C00h.
;
start
    ;
    ; Set up the stack to grow down from kBoot0Segment:kBoot0Stack.
    ; Interrupts should be off while the stack is being manipulated.
    ;
    cli                             ; interrupts off
    xor     ax, ax                  ; zero ax
    mov     ss, ax                  ; ss <- 0
    mov     sp, kBoot0Stack         ; sp <- top of stack
    sti                             ; reenable interrupts
```

1. We disable interrupts. Interrupts can mess with the stack, which we don't want since we're trying to move it.
2. Next, we clear the contents (make 0) of the register `AX`. 
3. Then, we move the contents of the register `AX` (0) into the register `SS`. The register `SS` is our Stack Segment, which should point to the bottom of the stack. 
4. Next we move the contents of `kBoot0Stack` (`0xFFF0`) into the register `SP`, or the Stack Pointer register. This points to the top of the stack. 
5. Then we re-enable interrupts.

---

```Assembly
    mov     es, ax                  ; es <- 0
    mov     ds, ax                  ; ds <- 0
```

This section is pretty simple. You could probably figure it out from the comments in the code, but for completetion:

1. We move the contents of register `AX`, which is 0, into the register `ES`.
2. We do the same for register `DS`.

These are registers related to copying strings. We're probably emptying them since we aren't copying a string, for cleanup.

---

```Assembly
    ;
    ; Relocate boot0 code.
    ;
    mov     si, kBoot0LoadAddr      ; si <- source
    mov     di, kBoot0RelocAddr     ; di <- destination
    ;
    cld                             ; auto-increment SI and/or DI registers
    mov     cx, kSectorBytes/2      ; copy 256 words
    repnz   movsw                   ; repeat string move (word) operation
```

1. We move the pointer for the beginning of the boot0 code (RAM address `0x7C00`) into register `SI`, or Source Index. 
2. Then we move the destination address `0xE000` into register `DI`, or Destination Index. 
3. Then we execute the instruction `cld` for CLear Direction flag. This tells the CPU that we want to copy data going from the source address forward. 
4. Then, we `mov` 256 "words" (16 bits, kSectorBytes is equal to 512. It is divided in half here.) from the source to the destination. Each time this is done, the `SI` and `DI` registers are automatically incremented. 
5. We execute the `repnz` instruction after each time to continue copying until we've copied everything. `repnz` is for REPeat Not Zero, and will repeat the called function `movsw` until the Zero Flag is not zero.

---

```Assembly
    ; Code relocated, jump to start_reloc in relocated location.
    ;
    jmp     0:start_reloc
```

1. The code has been successfully relocated, this tells the CPU to start executing in the new place we stored the bootloader. In this case, the beginning of `start_reloc`, which is defined below.
