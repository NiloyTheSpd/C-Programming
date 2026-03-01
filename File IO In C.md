# 🚀 File I/O in C

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![C Standard](https://img.shields.io/badge/C-C11-blue.svg)](<https://en.wikipedia.org/wiki/C11_(C_standard_revision)>)
[![Beginner Friendly](https://img.shields.io/badge/Level-Beginner%20to%20Advanced-brightgreen.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/)

> **The complete undergraduate guide** that weaves together File I/O, Pointers, Dynamic Memory Allocation, and Structs — because real programs use all four at once.

---

## 📚 Complete Table of Contents

- [Part 0 — How to Use This Guide + Compile & Run Setup](#part-0--how-to-use-this-guide--compile--run-setup)
- [Part 1 — Why Files? The Memory Hierarchy Explained](#part-1--why-files-the-memory-hierarchy-explained)
- [Part 2 — Pointers Deep Dive (Foundation for File I/O)](#part-2--pointers-deep-dive-foundation-for-file-io)
- [Part 3 — The FILE \* Pointer — What It Really Is](#part-3--the-file--pointer--what-it-really-is)
- [Part 4 — Opening & Closing Files (fopen, fclose, fflush)](#part-4--opening--closing-files-fopen-fclose-fflush)
- [Part 5 — File Modes — Complete Visual Guide](#part-5--file-modes--complete-visual-guide)
- [Part 6 — Writing to Files (fprintf, fputs, fputc, fwrite)](#part-6--writing-to-files)
- [Part 7 — Reading from Files (fgets, fscanf, fgetc, fread)](#part-7--reading-from-files)
- [Part 8 — File Positioning (fseek, ftell, rewind)](#part-8--file-positioning-fseek-ftell-rewind)
- [Part 9 — Binary Files & Structs — The Powerful Pattern](#part-9--binary-files--structs--the-powerful-pattern)
- [Part 10 — Dynamic Memory Allocation + File I/O (THE BIG CONNECTION)](#part-10--dynamic-memory-allocation--file-io)
- [Part 11 — Pointers + File I/O — Advanced Patterns](#part-11--pointers--file-io--advanced-patterns)
- [Part 12 — Error Handling — The Complete System](#part-12--error-handling--the-complete-system)
- [Part 13 — File Utility Functions (rename, remove, tmpfile)](#part-13--file-utility-functions)
- [Part 14 — CSV File Handling with strtok](#part-14--csv-file-handling-with-strtok)
- [Part 15 — Safe String Handling (Buffer Overflow Prevention)](#part-15--safe-string-handling)
- [Part 16 — stdin, stdout, stderr — The 3 Standard Streams](#part-16--stdin-stdout-stderr--the-3-standard-streams)
- [Part 17 — How the OS Handles Files (File Descriptors, Buffering)](#part-17--how-the-os-handles-files)
- [Part 18 — Common Pitfalls — 10 Traps With Full Explanations](#part-18--common-pitfalls--10-traps-with-full-explanations)
- [Part 19 — Real-World Project: Dynamic Student Database](#part-19--real-world-project-dynamic-student-database)
- [Part 20 — Exam-Style Practice Problems (10 Questions)](#part-20--exam-style-practice-problems)
- [Part 21 — Quick Reference Cheat Sheet](#part-21--quick-reference-cheat-sheet)
- [Part 22 — FAQ — 15 Questions Fully Answered](#part-22--faq--15-questions-fully-answered)
- [Part 23 — What to Learn Next (Roadmap)](#part-23--what-to-learn-next)

---

## Part 0 — How to Use This Guide + Compile & Run Setup

### Reading Paths

This guide covers a lot of ground. Use the path that fits your situation:

**⚡ Exam in 2 days** → Read Parts 1, 3–9, 18, 21. Focus on code patterns you can write from memory.

**🔨 Building a project** → Read Parts 9–11, 13–15, 19. These sections give you the complete production patterns.

**🎓 Want full mastery** → Read everything in order. Each part builds on the previous ones, and the cross-references will make every concept click.

---

### Compile & Run Setup

Every example in this guide compiles with:

```bash
# gcc example.c -o example -Wall -Wextra -std=c11
gcc filename.c -o filename -Wall -Wextra -std=c11

./filename          # Linux / macOS
filename.exe        # Windows (MinGW)
```

**What `-Wall -Wextra` catches (and why you must use them):**

| Flag       | What it catches                                                                                       |
| ---------- | ----------------------------------------------------------------------------------------------------- |
| `-Wall`    | Uninitialized variables, wrong `printf`/`scanf` format specifiers, unused variables, missing `return` |
| `-Wextra`  | Signed/unsigned comparison mismatches, missing function parameter names, shadow variables             |
| `-std=c11` | Enables C11 features; bans `gets()` (which is undefined in C11)                                       |

```bash
# Example of a warning that saves you from a segfault:
int *p;
printf("%d\n", *p);   // Warning: 'p' is used uninitialized — would crash at runtime!
```

> 💡 **Tip:** Treat every warning as an error. Add `-Werror` to your compile flags in production code: `gcc file.c -o file -Wall -Wextra -std=c11 -Werror`

---

### Checking for Memory Leaks (Valgrind)

```bash
# Install on Ubuntu/Debian:
sudo apt install valgrind

# Run your program through valgrind:
valgrind --leak-check=full ./filename

# Good output looks like:
# LEAK SUMMARY: definitely lost: 0 bytes in 0 blocks
```

We'll reference Valgrind concepts throughout Parts 10 and 12.

---

## Part 1 — Why Files? The Memory Hierarchy Explained

### 1.1 — The Memory Hierarchy

Every time your C program runs, it uses memory at multiple levels. Understanding WHERE data lives explains WHY files exist.

```
Speed │  Volatile? │  Typical Size  │  Cost/GB  │  What lives here
──────┼────────────┼────────────────┼───────────┼────────────────────────────
 Fast │  YES(lost) │  ~16 KB        │  $$$$$    │  CPU Registers (your loop vars)
      │  YES(lost) │  ~8 MB         │  $$$$     │  CPU Cache L1/L2/L3
      │  YES(lost) │  8–64 GB       │  $$$      │  RAM — your variables live here
      │  NO (saved)│  256 GB–4 TB   │  $        │  SSD / HDD ← FILES LIVE HERE
 Slow │  NO (saved)│  Unlimited     │  ¢        │  Cloud / Network Storage
```

**Key insight:** Your variables (`int x`, arrays, structs) live in **RAM**. When your program exits, the OS reclaims that RAM. The data is gone. Files live on disk and survive program exit, reboots, and power cuts.

```
                  THE LIFECYCLE OF DATA

  Without Files:                    With Files:

  Program starts                    Program starts
       │                                 │
  int x = 42;  ← RAM                int x = 42;  ← RAM
       │                                 │
  Program exits                     fwrite(&x, ...) ← copies to disk
       │                                 │
  x = ??? ← RAM wiped              Program exits
       │                                 │
  Data GONE ❌                      x is in grades.bin ← disk ✅
                                         │
                                    Next program run
                                         │
                                    fread(&x, ...) ← restores from disk
                                         │
                                    x = 42 again ✅
```

### 1.2 — The Journey of Data: Variable → Buffer → OS → Disk

When you call `fprintf(f, "hello")`, here is what actually happens. This matters for understanding `fflush()` later.

```
Your code:    fprintf(f, "hello\n")
                    │
                    ▼
          ┌─────────────────────┐
          │   C Library Buffer  │  ← In your process's RAM (~4KB–8KB)
          │   "hello\n"         │     Data WAITS here for efficiency
          └─────────────────────┘
                    │
         fflush() or fclose() or buffer fills up
                    │
                    ▼
          ┌─────────────────────┐
          │   OS Kernel Buffer  │  ← In OS-managed RAM
          │   (page cache)      │     OS decides WHEN to actually write
          └─────────────────────┘
                    │
           OS schedules disk write
                    │
                    ▼
          ┌─────────────────────┐
          │   Disk / SSD        │  ← Permanent storage
          │   grades.txt        │     Data is NOW safe from crashes
          └─────────────────────┘
```

> ⚠️ **Warning:** If your program crashes after `fprintf()` but before `fflush()`/`fclose()`, the data in the C library buffer is LOST. Always close your files before exiting.

### 1.3 — What is a Process?

When you run your compiled program, the OS creates a **process** — an isolated, running instance of your program.

```
    OS Memory
    ┌────────────────────────────────────────────────────────┐
    │                                                        │
    │   Process: your_program (PID 4821)                     │
    │   ┌──────────────────────────────────────────────┐    │
    │   │  Code segment    (your compiled machine code)│    │
    │   │  Data segment    (global variables)          │    │
    │   │  Stack           (local variables, call frames)   │
    │   │  Heap            (malloc'd memory)           │    │
    │   │  Open File Table (FILE * pointers)           │    │
    │   └──────────────────────────────────────────────┘    │
    │                                                        │
    │   Process: another_program (PID 4822)  [isolated!]    │
    │   └──────────────────────────────────────────────┘    │
    │                                                        │
    └────────────────────────────────────────────────────────┘
```

Files are the **bridge between processes** — and between program runs. Process A writes a file; Process B reads it. Your program writes a file on Monday; you read it on Friday.

---

## Part 2 — Pointers Deep Dive (Foundation for File I/O)

> 🔗 **Connection:** Nearly everything in File I/O is built on pointers. `FILE *` is a pointer. `fread()`/`fwrite()` take `void *`. Dynamic arrays use `malloc()` which returns a pointer. If pointers are shaky, File I/O will be confusing. Let's fix that.

### 2.1 — What a Pointer Actually Is

A pointer is a variable that **stores a memory address** instead of a value.

```c
// gcc pointer_basics.c -o pointer_basics -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    int x = 42;       // x lives at some address in RAM, say 0x7ffd4a2c
    int *p = &x;      // p STORES the address 0x7ffd4a2c  ← KEY LINE

    printf("Value of x:       %d\n", x);
    printf("Address of x:     %p\n", (void *)&x);
    printf("Value of p:       %p\n", (void *)p);     // same as &x
    printf("What p points to: %d\n", *p);            // dereference = get value at address

    *p = 100;   // change x through the pointer
    printf("x is now: %d\n", x);   // prints 100!
    return 0;
}
```

```
Expected output:
Value of x:       42
Address of x:     0x7ffd4a2c
Value of p:       0x7ffd4a2c
What p points to: 42
x is now: 100
```

**Memory layout:**

```
Address:  │ 0x7ffd4a2c │ 0x7ffd4a30 │
Value:    │     42     │ 0x7ffd4a2c │
Name:     │     x      │     p      │
Type:     │    int     │   int *    │
Size:     │  4 bytes   │  8 bytes   │  (pointer = 8 bytes on 64-bit systems)
```

**Three operations on a pointer:**

```c
int *p = &x;    // & = "address of" — get the address of x
*p = 100;       // * = dereference — follow the pointer, access what's there
p + 1;          // pointer arithmetic — move to next int (4 bytes forward)
```

> 🔗 **Connection to File I/O:** `FILE *fptr = fopen(...)` is exactly this — `fptr` is a pointer that stores the address of a `FILE` struct in RAM. When you call `fprintf(fptr, ...)`, you're passing the address of that FILE struct so the function knows which file to write to.

---

### 2.2 — Pointer to Struct (Critical for File I/O)

Structs and pointers are inseparable in C file programming.

```c
// gcc struct_ptr.c -o struct_ptr -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct {
    int   id;
    char  name[50];
    float cgpa;
} Student;

int main() {
    Student s = {1, "Alice", 3.90};
    Student *ptr = &s;    // ptr points to the Student struct  ← KEY LINE

    // TWO WAYS to access fields — identical result:
    printf("Dot notation:   %s (%.2f)\n", s.name,    s.cgpa);
    printf("Arrow notation: %s (%.2f)\n", ptr->name, ptr->cgpa);
    // ptr->name  is identical to  (*ptr).name

    printf("Size of Student struct: %zu bytes\n", sizeof(Student));
    return 0;
}
```

```
Expected output:
Dot notation:   Alice (3.90)
Arrow notation: Alice (3.90)
Size of Student struct: 60 bytes
```

**Why `sizeof(Student)` might be 60 instead of 55 (1 + 4 + 50 + 4 + 1 bytes) — struct padding:**

```
Struct memory layout (64-bit system):
┌──────────────────────────────────────────────────────┐
│  id    │ 4 bytes  │ offset 0                         │
│ (pad)  │ 0 bytes  │ (int is 4-byte aligned, OK)      │
│  name  │ 50 bytes │ offset 4                         │
│ (pad)  │ 2 bytes  │ pad to reach 4-byte boundary     │
│  cgpa  │ 4 bytes  │ offset 56                        │
│        │          │ Total: 60 bytes                  │
└──────────────────────────────────────────────────────┘
```

> ⚠️ **Warning:** Always use `sizeof(Student)` in `fwrite`/`fread` — NEVER hardcode the number of bytes. Padding makes the actual size unpredictable.

> 🔗 **Connection to File I/O:** `fwrite(&s, sizeof(Student), 1, f)` writes the **exact** bytes of the struct to disk, including padding. `fread(&s, sizeof(Student), 1, f)` reads them back. This works perfectly AS LONG AS you compiled with the same compiler on the same architecture.

---

### 2.3 — Pointer Arithmetic with Arrays

Arrays and pointers are intimately connected in C.

```c
// gcc ptr_arith.c -o ptr_arith -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int *p = arr;    // p points to arr[0] — array name IS a pointer  ← KEY LINE

    // These four expressions are identical:
    printf("%d\n", arr[2]);    // index notation
    printf("%d\n", *(arr+2));  // pointer arithmetic on array name
    printf("%d\n", p[2]);      // index notation on pointer
    printf("%d\n", *(p+2));    // pointer arithmetic on pointer

    // Iterate with pointer arithmetic (used for loaded binary data):
    printf("All elements: ");
    for (int *q = arr; q < arr + 5; q++) {    // ← KEY LINE
        printf("%d ", *q);
    }
    printf("\n");
    return 0;
}
```

```
Expected output:
30
30
30
30
All elements: 10 20 30 40 50
```

```
Memory layout of arr:
Address: 0x100  0x104  0x108  0x10C  0x110
Value:    10     20     30     40     50
Index:   [0]    [1]    [2]    [3]    [4]

p = 0x100           (points to arr[0])
p+1 = 0x104         (jumps 4 bytes = sizeof(int))
p+2 = 0x108         (jumps 8 bytes total)
```

> 🔗 **Connection to File I/O:** After `fread(db, sizeof(Student), count, f)` loads records into a `Student *db` array, you can iterate using `for (Student *p = db; p < db + count; p++)`. This is pointer arithmetic on the loaded data — elegant and efficient.

---

### 2.4 — Double Pointers `**` (Used in Dynamic File Loading)

A double pointer is a pointer to a pointer. It's used when a function needs to **change what a pointer points to** — for example, when `malloc`-ing inside a function.

```c
// gcc double_ptr.c -o double_ptr -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>

typedef struct { int id; char name[30]; float grade; } Student;

// WRONG version — caller's pointer is NOT updated
void loadWrong(Student *arr, int *count) {
    arr = malloc(3 * sizeof(Student));  // local copy only!
    *count = 3;
    // arr is freed when function returns — caller still has NULL
}

// CORRECT version — use double pointer to modify caller's pointer
void loadCorrect(Student **arr, int *count) {   // ← KEY LINE
    *arr = malloc(3 * sizeof(Student));
    *count = 3;
    if (*arr) {
        (*arr)[0] = (Student){1, "Alice", 3.9f};
        (*arr)[1] = (Student){2, "Bob",   3.2f};
        (*arr)[2] = (Student){3, "Carol", 3.7f};
    }
}

int main() {
    Student *db = NULL;
    int count = 0;
    loadCorrect(&db, &count);   // pass address of pointer

    for (int i = 0; i < count; i++) {
        printf("[%d] %s %.1f\n", db[i].id, db[i].name, db[i].grade);
    }
    free(db);
    return 0;
}
```

```
Expected output:
[1] Alice 3.9
[2] Bob 3.2
[3] Carol 3.7
```

```
Double pointer diagram:
main's stack:
┌──────────┐
│ db = 0x0 │ ← Student * (starts as NULL)
└────┬─────┘
     │  &db = 0x7fff1000
     │
loadCorrect receives:  Student **arr = 0x7fff1000
                                          │
*arr = malloc(...)     ────────────────►  writes heap address into db

After the call:
┌──────────────┐         ┌──────────────────────┐
│ db = 0x8a200 │ ──────► │ [Alice][Bob][Carol]   │
└──────────────┘         │ (heap, 3×sizeof)      │
                         └──────────────────────┘
```

> 🔗 **Connection to File I/O:** The pattern `Student **out` in `loadFromFile(Student **out, int *count, const char *filename)` lets the function internally call `malloc`, store data from `fread`, and hand the heap-allocated array back to the caller.

---

### 2.5 — NULL Pointer — The Most Important Pointer Value

`NULL` is defined as address `0` — it means "points to nothing."

```c
// gcc null_demo.c -o null_demo -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *p = NULL;   // p is explicitly "pointing nowhere"  ← KEY LINE

    // Checking for NULL — both are correct, first is clearer:
    if (p == NULL) printf("p is NULL — cannot dereference!\n");
    if (!p)        printf("Same check, shorter form\n");

    // NEVER do this — dereferencing NULL crashes immediately (segfault):
    // *p = 42;    // ← segfault: writing to address 0

    // Safe pattern:
    p = malloc(sizeof(int));
    if (p == NULL) {
        fprintf(stderr, "malloc failed\n");
        return 1;
    }
    *p = 42;
    printf("*p = %d\n", *p);
    free(p);
    p = NULL;   // always set to NULL after free!
    return 0;
}
```

**Where you'll see NULL checks in File I/O:**

```c
FILE *f = fopen("data.txt", "r");
if (f == NULL) { /* file doesn't exist or no permission */ }

Student *db = malloc(n * sizeof(Student));
if (db == NULL) { /* out of memory */ }

char *line = fgets(buf, sizeof(buf), f);
if (line == NULL) { /* EOF or read error */ }
```

> ⚠️ **Warning:** Dereferencing a NULL pointer is **undefined behavior** — your program will almost certainly crash with a segfault. Always check for NULL before using a pointer.

---

### 2.6 — `void *` Pointer (Used in fread/fwrite)

`void *` is a **generic pointer** — it can hold the address of anything, but you can't dereference it directly.

```c
// The signatures of fwrite and fread:
size_t fwrite(const void *ptr, size_t size, size_t count, FILE *stream);
//                  ^^^^^^^^
//          accepts ANY pointer type — int*, char*, Student*, etc.

size_t fread(void *ptr, size_t size, size_t count, FILE *stream);
//           ^^^^^^^^
//        writes the bytes to wherever this points
```

```c
// gcc void_ptr.c -o void_ptr -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    int   x = 42;
    float y = 3.14f;
    char  s[] = "hello";

    // void* can store any pointer without casting:
    void *vp;
    vp = &x;   // fine
    vp = &y;   // fine
    vp = s;    // fine

    // But to dereference, you MUST cast back:
    printf("x via void*: %d\n", *(int *)vp);   // cast to int*, then dereference

    // How fwrite uses void*:
    FILE *f = fopen("demo.bin", "wb");
    if (f) {
        fwrite(&x, sizeof(int), 1, f);    // &x (int*) → void* → fwrite writes 4 bytes
        fwrite(&y, sizeof(float), 1, f);  // &y (float*) → void* → fwrite writes 4 bytes
        fclose(f);
    }
    return 0;
}
```

```
How fwrite(&myStudent, sizeof(Student), 1, f) works internally:

  &myStudent → void *ptr   (the address of your struct)
  sizeof(Student) = 60     (how many bytes per element)
  1                        (how many elements)

  fwrite copies 60 bytes starting at &myStudent directly to the file buffer.
  No formatting, no conversion — raw bytes, exactly as they sit in RAM.
```

> 🔗 **Connection to File I/O:** Because `fread`/`fwrite` use `void *`, they work with ANY data type — `int`, `float`, `char`, or your custom struct. You just provide the address and the size in bytes.

---

## Part 3 — The FILE \* Pointer — What It Really Is

### 3.1 — Inside the FILE Struct

When you declare `FILE *f`, you have a pointer to a hidden struct that `<stdio.h>` manages for you. Here is a simplified view of what `FILE` actually contains:

```c
// Inside <stdio.h> — SIMPLIFIED illustration (actual implementation varies by platform):
typedef struct {
    int    _fd;          // OS file descriptor number (0=stdin, 1=stdout, 2=stderr, 3+=yours)
    char  *_buf;         // pointer to internal I/O buffer in RAM
    int    _buf_size;    // size of that buffer (typically 4096 or 8192 bytes)
    char  *_buf_pos;     // current read/write position within the buffer
    char  *_buf_end;     // one past the last valid byte in the buffer
    int    _flags;       // bitmask: readable? writable? error? EOF?
    // ... more internal fields depending on platform
} FILE;
```

```
After fopen("grades.txt", "r"):

Your stack:               Heap:                      Disk:
┌──────────┐             ┌──────────────────────┐   ┌──────────────────┐
│ FILE *f  │────────────►│  FILE struct          │──►│  grades.txt      │
│ (8 bytes)│             │  _fd:       3         │   │  "Alice  92.5\n" │
└──────────┘             │  _buf:      0x8a200   │   │  "Bob    78.0\n" │
                         │  _buf_size: 4096      │   └──────────────────┘
                         │  _buf_pos:  0x8a200   │
                         │  _flags:    READABLE  │
                         └──────────────────────┘
                         (malloc'd by fopen internally)
```

**Key facts:**

- `fopen()` calls `malloc()` internally to allocate the `FILE` struct on the heap
- `fclose()` does TWO things: flushes the buffer to disk AND `free()`s the `FILE` struct
- If you forget `fclose()`, you have a memory leak AND potentially missing data

> 🔗 **Connection:** `FILE *` is just like any other struct pointer in C. The difference is that you never allocate or free it yourself — `fopen` and `fclose` do it. But the mechanism is identical to your own `malloc`/`free` patterns.

---

### 3.2 — The 3 Pre-Opened FILE Pointers (Standard Streams)

Every C program starts with three FILE \* pointers already open. You never call `fopen()` for them.

```c
FILE *stdin;    // File descriptor 0 — keyboard input (or redirected input)
FILE *stdout;   // File descriptor 1 — terminal output (or redirected output)
FILE *stderr;   // File descriptor 2 — terminal error output (unbuffered by default)
```

```
These pairs are completely equivalent:

printf("hi");                   ≡   fprintf(stdout, "hi");
scanf("%d", &x);                ≡   fscanf(stdin, "%d", &x);
puts("hello");                  ≡   fputs("hello\n", stdout);
getchar();                      ≡   fgetc(stdin);
perror("msg");                  →   writes to stderr (not stdout!)
```

```c
// gcc streams_demo.c -o streams_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    // Writing to stdout (same as printf):
    fprintf(stdout, "This goes to the terminal\n");

    // Writing to stderr (errors, always visible even when stdout is redirected):
    fprintf(stderr, "Error: something went wrong\n");   // ← KEY LINE

    // Reading from stdin (same as scanf):
    int x;
    fscanf(stdin, "%d", &x);
    fprintf(stdout, "You entered: %d\n", x);
    return 0;
}
```

> 💡 **Tip:** `stderr` is unbuffered by default, so error messages appear immediately — even if `stdout` is still buffered and hasn't flushed yet. Always use `stderr` for error messages.

---

## Part 4 — Opening & Closing Files (fopen, fclose, fflush)

### 4.1 — `fopen()` — Open a File

```c
FILE *fopen(const char *filename, const char *mode);
// Returns: FILE* on success, NULL on failure
```

**ALWAYS check the return value:**

```c
// gcc fopen_demo.c -o fopen_demo -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>

int main() {
    FILE *f = fopen("grades.txt", "r");    // ← KEY LINE

    if (f == NULL) {
        // ❌ WRONG — never continue with NULL:
        // fprintf(f, "hello");  // CRASH — segfault

        // ✅ CORRECT — handle the error:
        perror("fopen failed");   // prints "fopen failed: No such file or directory"
        return 1;
    }

    // ✅ Safe to use f here
    printf("File opened successfully!\n");
    fclose(f);   // ALWAYS close what you open
    return 0;
}
```

```
Expected output (if grades.txt doesn't exist):
fopen failed: No such file or directory
```

**Filename paths:**

```c
fopen("data.txt", "r")           // same directory as the executable
fopen("data/students.bin", "r")  // subdirectory (relative path)
fopen("/home/user/data.txt", "r") // absolute path (Linux/Mac)
fopen("C:\\Users\\data.txt", "r") // absolute path (Windows — note double backslash)
```

---

### 4.2 — `fclose()` — Close a File

```c
int fclose(FILE *stream);
// Returns: 0 on success, EOF on error
```

```c
// ❌ WRONG — forgetting fclose causes:
//   1. Data loss (buffer not flushed to disk)
//   2. File descriptor leak (OS resource exhausted)
//   3. Memory leak (FILE struct heap memory not freed)

// ✅ CORRECT:
FILE *f = fopen("output.txt", "w");
if (f == NULL) { perror("Error"); return 1; }
fprintf(f, "Important data\n");
fclose(f);   // flushes buffer + frees FILE struct + releases OS descriptor
```

> ⚠️ **Warning:** After `fclose(f)`, never use `f` again. Set it to `NULL` immediately: `fclose(f); f = NULL;`

---

### 4.3 — `fflush()` — Force Write Without Closing

```c
int fflush(FILE *stream);
// Returns: 0 on success, EOF on error
```

Normally, data you write sits in the C library buffer until:

- The buffer is full (~4KB)
- You call `fclose()`
- You explicitly call `fflush()`

`fflush()` forces the buffer to be sent to the OS (and eventually to disk) without closing the file.

```c
// gcc fflush_demo.c -o fflush_demo -Wall -Wextra -std=c11
#include <stdio.h>
#include <unistd.h>  // for sleep() on Linux — use windows.h + Sleep() on Windows

int main() {
    FILE *log = fopen("realtime.log", "w");
    if (!log) { perror("Error"); return 1; }

    for (int i = 1; i <= 5; i++) {
        fprintf(log, "Event %d occurred\n", i);
        fflush(log);          // ← KEY LINE: write immediately, don't wait
        // Without fflush, another program reading realtime.log would see nothing
        // until the buffer fills or fclose is called
    }

    fclose(log);
    return 0;
}
```

**Special cases:**

```c
fflush(stdout);   // fix for prompt not appearing before scanf:
printf("Enter your name: ");
fflush(stdout);   // force the prompt to appear NOW
scanf("%49s", name);

fflush(NULL);     // flushes ALL open output streams at once
```

> ⚠️ **Warning:** `fflush(stdin)` is **undefined behavior** in standard C. It may appear to work on Windows (which defines it as a Microsoft extension), but it's non-portable and wrong. To discard pending input, use this safe alternative:

```c
// ❌ WRONG — undefined behavior:
fflush(stdin);

// ✅ CORRECT — discard characters until newline:
int ch;
while ((ch = getchar()) != '\n' && ch != EOF);
```

---

### 4.4 — `setvbuf()` — Control Buffering Mode

```c
int setvbuf(FILE *stream, char *buf, int mode, size_t size);
// Must be called AFTER fopen and BEFORE any I/O on the stream
// Returns: 0 on success, nonzero on failure
```

| Mode           | Constant | Meaning                                                    |
| -------------- | -------- | ---------------------------------------------------------- |
| Full buffering | `_IOFBF` | Write when buffer is full (default for files)              |
| Line buffering | `_IOLBF` | Write when `\n` is seen (default for `stdout` to terminal) |
| Unbuffered     | `_IONBF` | Write immediately (default for `stderr`)                   |

```c
// gcc setvbuf_demo.c -o setvbuf_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *log = fopen("instant.log", "w");
    if (!log) { perror("Error"); return 1; }

    // Make log file unbuffered — every write goes to disk immediately:
    setvbuf(log, NULL, _IONBF, 0);    // ← KEY LINE

    fprintf(log, "Entry 1\n");   // written to disk immediately
    fprintf(log, "Entry 2\n");   // written to disk immediately
    // Even if program crashes here, both entries are saved

    fclose(log);
    return 0;
}
```

> 💡 **Tip:** Use `_IONBF` for log files in critical applications where you cannot afford to lose the last few log entries to a buffer.

---

## Part 5 — File Modes — Complete Visual Guide

### 5.1 — Text Mode Visual Map

```
YOUR FILE ON DISK
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

"r"  ──► [=========== existing content ===========]
          ^
          Cursor starts here. CAN read. CANNOT write.
          ❌ Returns NULL if file does not exist.

"w"  ──► [                                         ]
          File is WIPED empty (or created fresh). ⚠️ DANGER
          CAN write. CANNOT read.

"a"  ──► [=========== existing content ===========]^
          Existing content preserved.               |
          All writes go HERE, at the end. ──────────┘
          File created if it does not exist.

"r+" ──► [=========== existing content ===========]
          ^
          CAN read AND write. File MUST exist (no create).
          Cursor at beginning.

"w+" ──► [                                         ]
          File WIPED (or created). CAN read AND write. ⚠️

"a+" ──► [=========== existing content ===========]^
          CAN read from anywhere (use fseek to move).  |
          But ALL writes go here (end). ───────────────┘
```

### 5.2 — Mode Reference Table

| Mode            | Read | Write | Creates? | Wipes? | Cursor starts |
| --------------- | :--: | :---: | :------: | :----: | :------------ |
| `"r"`           |  ✅  |  ❌   |    ❌    |   ❌   | Beginning     |
| `"w"`           |  ❌  |  ✅   |    ✅    |   ✅   | Beginning     |
| `"a"`           |  ❌  |  ✅   |    ✅    |   ❌   | End           |
| `"r+"`          |  ✅  |  ✅   |    ❌    |   ❌   | Beginning     |
| `"w+"`          |  ✅  |  ✅   |    ✅    |   ✅   | Beginning     |
| `"a+"`          |  ✅  |  ✅   |    ✅    |   ❌   | End (writes)  |
| `"rb"`          |  ✅  |  ❌   |    ❌    |   ❌   | Beginning     |
| `"wb"`          |  ❌  |  ✅   |    ✅    |   ✅   | Beginning     |
| `"ab"`          |  ❌  |  ✅   |    ✅    |   ❌   | End           |
| `"r+b"`/`"rb+"` |  ✅  |  ✅   |    ❌    |   ❌   | Beginning     |
| `"w+b"`/`"wb+"` |  ✅  |  ✅   |    ✅    |   ✅   | Beginning     |
| `"a+b"`/`"ab+"` |  ✅  |  ✅   |    ✅    |   ❌   | End (writes)  |

### 5.3 — Choosing the Right Mode

```c
// Use "r"  when: reading a config or input file that must exist
FILE *config = fopen("settings.cfg", "r");

// Use "w"  when: creating a fresh output/report file each run
FILE *report = fopen("report.txt", "w");

// Use "a"  when: adding log entries without overwriting old ones
FILE *log = fopen("app.log", "a");

// Use "rb" when: reading binary struct data (images, databases)
FILE *db = fopen("students.bin", "rb");

// Use "wb" when: writing binary struct data
FILE *db = fopen("students.bin", "wb");

// Use "r+b" when: updating records in-place (fseek + fwrite)
FILE *db = fopen("students.bin", "r+b");
```

> 💡 **Memory trick:** `r` = read (must exist) | `w` = write (wipes/creates) | `a` = append (safe add) | `+` = adds the other direction | `b` = binary (no newline translation)

> ⚠️ **Warning:** `"w"` silently destroys ALL existing content in the file with no confirmation. If you meant `"a"`, this is catastrophic. Double-check before using `"w"` on an existing file.

---

## Part 6 — Writing to Files

Four functions for writing — each suited to different tasks:

| Function  | Best for                          | Example                              |
| --------- | --------------------------------- | ------------------------------------ |
| `fprintf` | Formatted text (numbers, strings) | `fprintf(f, "%s %d\n", name, score)` |
| `fputs`   | Writing a string (no format)      | `fputs("hello\n", f)`                |
| `fputc`   | Writing one character             | `fputc('A', f)`                      |
| `fwrite`  | Writing raw bytes / structs       | `fwrite(&s, sizeof(s), 1, f)`        |

### 6.1 — `fprintf()` — Formatted Write

```c
int fprintf(FILE *stream, const char *format, ...);
// Returns: number of characters written, or negative on error
```

```c
// gcc fprintf_demo.c -o fprintf_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *f = fopen("grades.txt", "w");
    if (!f) { perror("Error"); return 1; }

    // Exactly like printf, but writes to f instead of stdout:
    fprintf(f, "Name: %-15s Grade: %.1f\n", "Alice", 92.5f);   // ← KEY LINE
    fprintf(f, "Name: %-15s Grade: %.1f\n", "Bob",   78.0f);
    fprintf(f, "Name: %-15s Grade: %.1f\n", "Carol", 88.5f);

    fclose(f);
    printf("Written to grades.txt\n");
    return 0;
}
```

```
File contents of grades.txt:
Name: Alice           Grade: 92.5
Name: Bob             Grade: 78.0
Name: Carol           Grade: 88.5
```

### 6.2 — `fputs()` — String Write

```c
int fputs(const char *str, FILE *stream);
// Returns: nonneg on success, EOF on error
// NOTE: fputs does NOT add a newline — include \n in your string if needed
```

```c
// gcc fputs_demo.c -o fputs_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *f = fopen("notes.txt", "w");
    if (!f) { perror("Error"); return 1; }

    fputs("Line one\n", f);    // ← KEY LINE — must include \n manually
    fputs("Line two\n", f);
    // Equivalent to: fprintf(f, "Line one\n"); fprintf(f, "Line two\n");
    // fputs is faster for plain strings — no format parsing overhead

    fclose(f);
    return 0;
}
```

### 6.3 — `fputc()` — Single Character Write

```c
int fputc(int c, FILE *stream);
// Returns: the character written, or EOF on error
```

```c
// gcc fputc_demo.c -o fputc_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *f = fopen("abc.txt", "w");
    if (!f) { perror("Error"); return 1; }

    // Write characters one at a time:
    for (char c = 'A'; c <= 'Z'; c++) {
        fputc(c, f);    // ← KEY LINE
    }
    fputc('\n', f);

    fclose(f);

    // Common use: file copy byte by byte
    // (See Part 20, Question 2)
    return 0;
}
```

```
File contents of abc.txt:
ABCDEFGHIJKLMNOPQRSTUVWXYZ
```

### 6.4 — `fwrite()` — Raw Binary Write

```c
size_t fwrite(const void *ptr, size_t size, size_t count, FILE *stream);
// ptr:   pointer to data to write
// size:  size of each element in bytes
// count: number of elements
// Returns: number of elements successfully written (should equal count)
```

```c
// gcc fwrite_demo.c -o fwrite_demo -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int main() {
    Student students[3] = {
        {1, "Alice", 92.5f},
        {2, "Bob",   78.0f},
        {3, "Carol", 88.5f},
    };

    FILE *f = fopen("students.bin", "wb");
    if (!f) { perror("Error"); return 1; }

    // Write all 3 structs in ONE call:
    size_t written = fwrite(students, sizeof(Student), 3, f);   // ← KEY LINE
    if (written != 3) {
        fprintf(stderr, "Write error: only %zu of 3 written\n", written);
    }
    printf("Wrote %zu students (%zu bytes total)\n", written, written * sizeof(Student));

    fclose(f);
    return 0;
}
```

```
Expected output:
Wrote 3 students (180 bytes total)
```

```
How fwrite lays data on disk (binary file — no formatting, raw bytes):

Byte 0       Byte 59      Byte 60      Byte 119     Byte 120     Byte 179
│ Student 0 (60 bytes)  │ Student 1 (60 bytes)  │ Student 2 (60 bytes)  │
│ id | name     | grade │ id | name     | grade │ id | name     | grade │
│  1 | "Alice\0"| 92.5  │  2 | "Bob\0"  | 78.0  │  3 | "Carol\0"| 88.5  │
```

> 🔗 **Connection:** The raw-byte approach of `fwrite` is why binary files support **random access with `fseek`** — every Student is exactly `sizeof(Student)` bytes, so record N starts at byte `N * sizeof(Student)`. We use this in Part 8.

---

## Part 7 — Reading from Files

| Function | Best for                      | Returns                   |
| -------- | ----------------------------- | ------------------------- |
| `fgets`  | Reading a line of text (SAFE) | `char*` or `NULL` at EOF  |
| `fscanf` | Parsing formatted text        | count of matched items    |
| `fgetc`  | Reading one character         | `int` (char value or EOF) |
| `fread`  | Reading raw bytes / structs   | count of items read       |

### 7.1 — `fgets()` — Safe Line Reading

```c
char *fgets(char *str, int n, FILE *stream);
// Reads at most n-1 characters, stops at \n or EOF
// Always null-terminates — the \n IS included in the string
// Returns: str on success, NULL at EOF or error
```

```c
// gcc fgets_demo.c -o fgets_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *f = fopen("grades.txt", "r");
    if (!f) { perror("Error"); return 1; }

    char line[256];
    while (fgets(line, sizeof(line), f) != NULL) {   // ← KEY LINE
        printf("Read: %s", line);   // \n is already in line, so no extra \n needed
    }

    // After the loop: check if we stopped for EOF or an error:
    if (feof(f))   printf("[Reached end of file]\n");
    if (ferror(f)) perror("[Read error]");

    fclose(f);
    return 0;
}
```

```
Expected output:
Read: Name: Alice           Grade: 92.5
Read: Name: Bob             Grade: 78.0
Read: Name: Carol           Grade: 88.5
[Reached end of file]
```

> ⚠️ **Warning:** `fgets` keeps the `\n` in the string. If you don't want it, strip it: `line[strcspn(line, "\n")] = '\0';` (Part 15 covers this in detail.)

### 7.2 — `fscanf()` — Formatted Read

```c
int fscanf(FILE *stream, const char *format, ...);
// Returns: number of items successfully matched and assigned
// Returns EOF at end of file or on read error
```

```c
// gcc fscanf_demo.c -o fscanf_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *f = fopen("scores.txt", "r");
    if (!f) { perror("Error"); return 1; }

    char name[50];
    float score;

    // scores.txt format: "Alice 92.5\n"
    while (fscanf(f, "%49s %f", name, &score) == 2) {    // ← KEY LINE
        printf("%-15s → %.1f\n", name, score);
    }

    fclose(f);
    return 0;
}
```

> 💡 **Tip:** `fscanf` is convenient but brittle — it fails on unexpected whitespace or extra tokens. For robust parsing, prefer `fgets` + `sscanf` (read a line, then parse the string).

```c
// More robust pattern:
char line[256];
while (fgets(line, sizeof(line), f) != NULL) {
    char name[50]; float score;
    if (sscanf(line, "%49s %f", name, &score) == 2) {
        printf("%-15s → %.1f\n", name, score);
    }
    // If sscanf fails, we skip the bad line and continue — more resilient
}
```

### 7.3 — `fgetc()` — Single Character Read

```c
int fgetc(FILE *stream);
// Returns: the character read as unsigned char converted to int
// Returns: EOF (typically -1) at end of file or on error
// NOTE: return type is int, not char — necessary to hold EOF
```

```c
// gcc fgetc_demo.c -o fgetc_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *f = fopen("abc.txt", "r");
    if (!f) { perror("Error"); return 1; }

    int ch;   // MUST be int, not char — to distinguish EOF from valid chars
    int count = 0;

    while ((ch = fgetc(f)) != EOF) {    // ← KEY LINE
        putchar(ch);   // print to stdout
        count++;
    }
    printf("\nTotal characters: %d\n", count);

    fclose(f);
    return 0;
}
```

> ⚠️ **Warning:** Declare `ch` as `int`, not `char`. On some systems `char` is unsigned, so `(char)EOF == 255`, not `-1`. The `int` type correctly distinguishes EOF from valid character values.

### 7.4 — `fread()` — Raw Binary Read

```c
size_t fread(void *ptr, size_t size, size_t count, FILE *stream);
// ptr:   buffer to read into
// size:  size of each element in bytes
// count: max elements to read
// Returns: number of elements actually read (may be less at EOF)
```

```c
// gcc fread_demo.c -o fread_demo -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int main() {
    FILE *f = fopen("students.bin", "rb");
    if (!f) { perror("Error"); return 1; }

    Student s;
    int count = 0;

    // Read one struct at a time:
    while (fread(&s, sizeof(Student), 1, f) == 1) {    // ← KEY LINE
        printf("[%d] %-15s %.1f\n", s.id, s.name, s.grade);
        count++;
    }
    printf("Total records: %d\n", count);

    fclose(f);
    return 0;
}
```

```
Expected output (from the file written in Part 6):
[1] Alice           92.5
[2] Bob             78.0
[3] Carol           88.5
Total records: 3
```

> 🔗 **Connection to DMA:** Instead of reading one record at a time, Part 10 shows how to use `malloc` to allocate space for ALL records and load them all at once with a single `fread` call — dramatically faster for large databases.

---

## Part 8 — File Positioning (fseek, ftell, rewind)

### 8.1 — The File Cursor

Every open file has an invisible **cursor** (position indicator) that tracks where the next read or write will happen.

```
File contents: [Alice][Bob][Carol][Diana]
Cursor after opening "rb":
                ^
                Position 0

After reading Alice (60 bytes):
[Alice][Bob][Carol][Diana]
        ^
        Position 60

After reading Bob (60 bytes):
[Alice][Bob][Carol][Diana]
               ^
               Position 120
```

### 8.2 — `fseek()` — Move the Cursor

```c
int fseek(FILE *stream, long offset, int whence);
// offset: number of bytes to move
// whence: SEEK_SET (from start) | SEEK_CUR (from current) | SEEK_END (from end)
// Returns: 0 on success, nonzero on error
```

```
whence values:

SEEK_SET ──► offset from the BEGINNING of the file
              fseek(f, 0, SEEK_SET) → rewind to start
              fseek(f, 60, SEEK_SET) → jump to byte 60

SEEK_CUR ──► offset from the CURRENT cursor position
              fseek(f, 10, SEEK_CUR) → skip forward 10 bytes
              fseek(f, -10, SEEK_CUR) → go back 10 bytes

SEEK_END ──► offset from the END of the file (offset is usually negative or 0)
              fseek(f, 0, SEEK_END) → move to end (for ftell to get file size)
              fseek(f, -60, SEEK_END) → last record
```

### 8.3 — `ftell()` — Get Current Position

```c
long ftell(FILE *stream);
// Returns: current position in bytes from beginning of file
// Returns: -1L on error
```

### 8.4 — `rewind()` — Go Back to Start

```c
void rewind(FILE *stream);
// Equivalent to: fseek(f, 0, SEEK_SET) AND clears error/EOF flags
```

### 8.5 — Complete Example: Random Access to Binary Records

```c
// gcc fseek_demo.c -o fseek_demo -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int main() {
    FILE *f = fopen("students.bin", "r+b");
    if (!f) { perror("Error"); return 1; }

    // Get total file size:
    fseek(f, 0, SEEK_END);        // move to end  ← KEY LINE
    long fileSize = ftell(f);     // get position = file size
    int count = fileSize / sizeof(Student);
    printf("File has %d records (%ld bytes)\n", count, fileSize);

    // Jump directly to record index 1 (Bob) — O(1) random access:
    int target = 1;
    fseek(f, (long)target * sizeof(Student), SEEK_SET);    // ← KEY LINE

    Student s;
    fread(&s, sizeof(Student), 1, f);
    printf("Record %d: [%d] %s %.1f\n", target, s.id, s.name, s.grade);

    // Update Bob's grade in-place:
    s.grade = 85.0f;
    fseek(f, (long)target * sizeof(Student), SEEK_SET);
    fwrite(&s, sizeof(Student), 1, f);
    printf("Updated record %d grade to %.1f\n", target, s.grade);

    // Verify — rewind and read all:
    rewind(f);
    printf("\nAll records after update:\n");
    while (fread(&s, sizeof(Student), 1, f) == 1) {
        printf("  [%d] %-15s %.1f\n", s.id, s.name, s.grade);
    }

    fclose(f);
    return 0;
}
```

```
Expected output:
File has 3 records (180 bytes)
Record 1: [2] Bob 78.0
Updated record 1 grade to 85.0

All records after update:
  [1] Alice           92.5
  [2] Bob             85.0
  [3] Carol           88.5
```

```
The magic of binary random access:

Record 0 starts at byte: 0 × sizeof(Student) = 0
Record 1 starts at byte: 1 × sizeof(Student) = 60
Record 2 starts at byte: 2 × sizeof(Student) = 120
Record N starts at byte: N × sizeof(Student)

fseek(f, N * sizeof(Student), SEEK_SET)
→ jumps directly to record N in O(1) time
→ no need to read through records 0 to N-1
```

> 🔗 **Connection to DMA:** With text files you must read from the beginning each time. With binary files and `fseek`, you have a database-like O(1) lookup. Part 10 combines this with `malloc` to build a full in-memory database that can be saved and loaded.

---

## Part 9 — Binary Files & Structs — The Powerful Pattern

### 9.1 — Text vs Binary Files

```
TEXT FILE (grades.txt):          BINARY FILE (students.bin):
┌─────────────────────────┐      ┌─────────────────────────┐
│ Alice 92.5              │      │ 01 00 00 00 41 6C 69 63 │
│ Bob 78.0                │  vs  │ 65 00 00 00 00 00 00 00 │
│ Carol 88.5              │      │ 00 00 00 00 00 00 00 00 │
└─────────────────────────┘      │ ... (raw bytes) ...     │
Human readable ✅                └─────────────────────────┘
Bigger on disk (numbers as text) Smaller on disk ✅
No random access (variable len)  Random access O(1) ✅
Portable (any editor)            Not human-readable
```

### 9.2 — The Complete Binary Struct Pattern

```c
// gcc binary_structs.c -o binary_structs -Wall -Wextra -std=c11
#include <stdio.h>
#include <string.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

// Save array of students to binary file
int saveStudents(const char *filename, Student *arr, int count) {
    FILE *f = fopen(filename, "wb");
    if (!f) { perror("saveStudents: fopen"); return -1; }

    size_t written = fwrite(arr, sizeof(Student), count, f);    // ← KEY LINE
    fclose(f);

    if ((int)written != count) {
        fprintf(stderr, "saveStudents: wrote %zu of %d\n", written, count);
        return -1;
    }
    return 0;
}

// Load students from binary file into a STATIC array (fixed max)
int loadStudents(const char *filename, Student *arr, int maxCount) {
    FILE *f = fopen(filename, "rb");
    if (!f) { perror("loadStudents: fopen"); return 0; }

    int count = (int)fread(arr, sizeof(Student), maxCount, f);    // ← KEY LINE
    fclose(f);
    return count;
}

// Update one record by ID
int updateGrade(const char *filename, int id, float newGrade) {
    FILE *f = fopen(filename, "r+b");
    if (!f) { perror("updateGrade: fopen"); return -1; }

    Student s;
    long pos = 0;
    while (fread(&s, sizeof(Student), 1, f) == 1) {
        if (s.id == id) {
            s.grade = newGrade;
            fseek(f, -((long)sizeof(Student)), SEEK_CUR);   // back one record
            fwrite(&s, sizeof(Student), 1, f);
            fclose(f);
            return 0;   // success
        }
    }
    fclose(f);
    return -1;   // not found
}

int main() {
    // Create and save 3 students:
    Student students[3] = {
        {1, "Alice", 92.5f},
        {2, "Bob",   78.0f},
        {3, "Carol", 88.5f},
    };
    saveStudents("class.bin", students, 3);
    printf("Saved 3 students\n");

    // Load them back:
    Student loaded[10];
    int count = loadStudents("class.bin", loaded, 10);
    printf("Loaded %d students:\n", count);
    for (int i = 0; i < count; i++) {
        printf("  [%d] %-15s %.1f\n", loaded[i].id, loaded[i].name, loaded[i].grade);
    }

    // Update Bob's grade:
    updateGrade("class.bin", 2, 85.0f);
    printf("\nAfter updating Bob:\n");
    count = loadStudents("class.bin", loaded, 10);
    for (int i = 0; i < count; i++) {
        printf("  [%d] %-15s %.1f\n", loaded[i].id, loaded[i].name, loaded[i].grade);
    }

    return 0;
}
```

```
Expected output:
Saved 3 students
Loaded 3 students:
  [1] Alice           92.5
  [2] Bob             78.0
  [3] Carol           88.5

After updating Bob:
  [1] Alice           92.5
  [2] Bob             85.0
  [3] Carol           88.5
```

### 9.3 — Limitation of Static Arrays

The above example uses `Student loaded[10]` — a static array. What if the file has 500 students? You'd need `Student loaded[500]` — and you'd have to guess the maximum in advance.

> 🔗 **Connection to DMA:** Part 10 solves this completely. Instead of guessing the size, we use `fseek`/`ftell` to calculate the EXACT number of records, then `malloc` exactly that amount of memory. No waste, no limit.

---

## Part 10 — Dynamic Memory Allocation + File I/O

> **This is the most important section in the guide.** Real programs combine DMA and File I/O constantly. Understanding this combination is what separates beginner C from intermediate C.

### 10.1 — DMA Foundations (Refresher)

**The four DMA functions:**

```c
#include <stdlib.h>   // malloc, calloc, realloc, free

// MALLOC — allocate uninitialized memory
void *malloc(size_t bytes);
// Returns: pointer to allocated memory, or NULL if failed
// Contents: GARBAGE (whatever was in RAM before)

// CALLOC — allocate zeroed memory
void *calloc(size_t count, size_t size);
// Returns: pointer to count×size bytes of ZEROED memory, or NULL
// Slower than malloc (must zero), safer for arrays

// REALLOC — resize an existing allocation
void *realloc(void *ptr, size_t new_size);
// Returns: pointer to resized block (may be DIFFERENT address!), or NULL on failure
// If NULL returned, ORIGINAL ptr is still valid

// FREE — release memory back to the heap
void free(void *ptr);
// After this call, ptr is a dangling pointer — set it to NULL immediately
```

**Memory diagrams:**

```
BEFORE any DMA:
Heap: [free free free free free free free free free free free ...]

malloc(3 * sizeof(Student)):   // 3 × 60 = 180 bytes
Heap: [###180 bytes###][free free free free free free free ...]
       ^
       returned pointer (contents = GARBAGE)

calloc(3, sizeof(Student)):    // 180 bytes, zeroed
Heap: [000180 bytes000][free free free free free free free ...]
       ^
       returned pointer (all bytes = 0x00)

realloc(ptr, 6 * sizeof(Student)):   // try to grow from 3 to 6 records

  Case A: room to grow in place
  Heap: [###########360 bytes###########][free ...]
         ^ptr stays same address

  Case B: must relocate
  Heap: [free 180 bytes][###########360 bytes###########][...]
         (old freed)      ^new address — ptr CHANGES

free(ptr):
Heap: [free free free free free free free free free free free ...]
  ptr is now DANGLING — do: ptr = NULL;
```

**All four with proper error checking:**

```c
// gcc dma_basics.c -o dma_basics -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct { int id; char name[50]; float grade; } Student;

int main() {
    int n = 5;

    // malloc — uninitialized:
    Student *arr = malloc(n * sizeof(Student));    // ← KEY LINE
    if (arr == NULL) { perror("malloc failed"); return 1; }

    // calloc — zeroed (safer for arrays, grade will be 0.0):
    Student *db = calloc(n, sizeof(Student));      // ← KEY LINE
    if (db == NULL) { perror("calloc failed"); free(arr); return 1; }

    // Use the memory:
    arr[0] = (Student){1, "Alice", 3.9f};
    strncpy(db[0].name, "Bob", sizeof(db[0].name) - 1);

    // realloc — grow to 10 records:
    int new_n = 10;
    Student *temp = realloc(arr, new_n * sizeof(Student));    // ← KEY LINE
    if (temp == NULL) {
        perror("realloc failed");
        // arr is still valid — free it before returning:
        free(arr); free(db);
        return 1;
    }
    arr = temp;   // ONLY update arr after confirming success  ← KEY LINE

    printf("All allocations successful\n");
    printf("arr[0]: [%d] %s %.1f\n", arr[0].id, arr[0].name, arr[0].grade);

    free(arr);  arr = NULL;   // ← KEY LINE: always set to NULL after free
    free(db);   db  = NULL;
    return 0;
}
```

---

### 10.2 — Why DMA + File I/O is The Natural Combination

**The problem with static arrays for file loading:**

```c
// ❌ PROBLEM: size fixed at compile time
Student students[100];   // what if the file has 5? (waste 95 × 60 = 5700 bytes)
                         // what if it has 500? (overflow — undefined behavior!)

// ✅ SOLUTION: dynamic array sized at runtime from the file
Student *students = malloc(actualCount * sizeof(Student));
// Exactly the right size — no waste, no overflow
```

**The complete pattern for loading binary files dynamically:**

```
STEP 1: Open file
STEP 2: fseek to end → ftell → divide by sizeof(T) = EXACT record count
STEP 3: malloc(count * sizeof(T)) — exactly the right amount
STEP 4: rewind, then fread(arr, sizeof(T), count, f) — load all at once
STEP 5: fclose(f)
STEP 6: Work with the data
STEP 7: free(arr), arr = NULL
```

---

### 10.3 — Full Working Example: Dynamic File Loader

```c
// gcc dynamic_loader.c -o dynamic_loader -Wall -Wextra -std=c11
// Load ALL students from a binary file into a dynamically sized array.
// The array is EXACTLY the right size — no guessing, no waste.

#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

// Returns a malloc'd array of students, sets *count to number loaded.
// Returns NULL if file is empty or cannot be opened.
// CALLER IS RESPONSIBLE FOR free()-ing the returned pointer.
Student *loadAll(const char *filename, int *count) {
    *count = 0;
    FILE *f = fopen(filename, "rb");
    if (f == NULL) { perror("loadAll: cannot open file"); return NULL; }

    // Step 1: Get file size → compute exact record count
    fseek(f, 0, SEEK_END);           // move cursor to end  ← KEY LINE
    long fileSize = ftell(f);        // current position = file size
    rewind(f);                       // move cursor back to start

    *count = (int)(fileSize / (long)sizeof(Student));
    if (*count == 0) { fclose(f); return NULL; }

    // Step 2: Allocate EXACTLY enough memory
    Student *arr = malloc((size_t)*count * sizeof(Student));    // ← KEY LINE
    if (arr == NULL) { perror("loadAll: malloc failed"); fclose(f); *count = 0; return NULL; }

    // Step 3: Read all records in one call — fast!
    int readCount = (int)fread(arr, sizeof(Student), (size_t)*count, f);    // ← KEY LINE
    fclose(f);

    if (readCount != *count) {
        fprintf(stderr, "loadAll: expected %d records, got %d\n", *count, readCount);
        *count = readCount;
    }

    return arr;   // caller must free() this!
}

int main() {
    // First, create a test file:
    Student src[4] = {
        {1, "Alice",  92.5f},
        {2, "Bob",    78.0f},
        {3, "Carol",  88.5f},
        {4, "David",  95.0f},
    };
    FILE *f = fopen("students.bin", "wb");
    if (!f) { perror("Error creating test file"); return 1; }
    fwrite(src, sizeof(Student), 4, f);
    fclose(f);

    // Now load dynamically — no hardcoded size!
    int count = 0;
    Student *db = loadAll("students.bin", &count);    // ← KEY LINE

    if (db == NULL) { printf("No data loaded.\n"); return 1; }

    printf("Loaded %d students:\n", count);
    for (int i = 0; i < count; i++) {
        printf("  [%d] %-15s %.1f\n", db[i].id, db[i].name, db[i].grade);
    }

    free(db);    // ALWAYS free what you malloc
    db = NULL;   // prevent dangling pointer
    return 0;
}
```

```
Expected output:
Loaded 4 students:
  [1] Alice           92.5
  [2] Bob             78.0
  [3] Carol           88.5
  [4] David           95.0
```

---

### 10.4 — Growing a File Database Dynamically with realloc

The real-world pattern: load file → add new records → realloc → save back.

```c
// gcc dynamic_append.c -o dynamic_append -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

// Dynamically append a record. Uses doubling strategy for O(1) amortized growth.
// Returns new pointer (may change!). Updates count and capacity.
Student *addRecord(Student *db, int *count, int *capacity, Student newEntry) {
    // If full, double the capacity (amortized O(1) growth):
    if (*count >= *capacity) {
        *capacity = (*capacity == 0) ? 4 : *capacity * 2;
        Student *temp = realloc(db, (size_t)*capacity * sizeof(Student));    // ← KEY LINE
        if (temp == NULL) {
            perror("addRecord: realloc failed");
            return db;   // original unchanged — caller still responsible
        }
        db = temp;
        printf("  [realloc] capacity grown to %d\n", *capacity);
    }
    db[(*count)++] = newEntry;    // ← KEY LINE: add and increment count
    return db;   // MUST reassign — address may have changed!
}

// Save entire array to binary file:
int saveAll(const char *filename, Student *db, int count) {
    FILE *f = fopen(filename, "wb");
    if (!f) { perror("saveAll"); return -1; }
    fwrite(db, sizeof(Student), (size_t)count, f);
    fclose(f);
    return 0;
}

int main() {
    Student *db = NULL;
    int count = 0, capacity = 0;

    // Add records — watch realloc happen:
    printf("Adding records:\n");
    db = addRecord(db, &count, &capacity, (Student){1, "Alice", 92.5f});
    db = addRecord(db, &count, &capacity, (Student){2, "Bob",   78.0f});
    db = addRecord(db, &count, &capacity, (Student){3, "Carol", 88.5f});
    db = addRecord(db, &count, &capacity, (Student){4, "David", 95.0f});
    db = addRecord(db, &count, &capacity, (Student){5, "Eve",   91.0f});

    printf("\nFinal database (%d records, %d capacity):\n", count, capacity);
    for (int i = 0; i < count; i++) {
        printf("  [%d] %-15s %.1f\n", db[i].id, db[i].name, db[i].grade);
    }

    saveAll("saved.bin", db, count);
    printf("Saved to saved.bin\n");

    free(db);
    db = NULL;
    return 0;
}
```

```
Expected output:
Adding records:
  [realloc] capacity grown to 4
  [realloc] capacity grown to 8

Final database (5 records, 8 capacity):
  [1] Alice           92.5
  [2] Bob             78.0
  [3] Carol           88.5
  [4] David           95.0
  [5] Eve             91.0
Saved to saved.bin
```

```
Doubling strategy visualization:
count=1, capacity=4  → [Alice][---][---][---]
count=4, capacity=4  → [Alice][Bob][Carol][David]
count=5, capacity=8  → [Alice][Bob][Carol][David][Eve][---][---][---]
                        (realloc doubled capacity before adding Eve)
```

---

### 10.5 — Memory Leak Prevention with Files

The most common leak patterns when combining DMA and File I/O:

```c
// ── LEAK 1: malloc succeeds, fopen fails, forgot to free ──
// ❌ WRONG:
char *buf = malloc(1024);
FILE *f = fopen("data.txt", "r");
if (f == NULL) { perror("error"); return 1; }  // BUG: buf is leaked!

// ✅ CORRECT:
char *buf = malloc(1024);
if (!buf) { perror("malloc"); return 1; }
FILE *f = fopen("data.txt", "r");
if (f == NULL) { perror("fopen"); free(buf); return 1; }  // free before returning


// ── LEAK 2: realloc failure loses original pointer ──
// ❌ WRONG:
db = realloc(db, new_size);   // if realloc fails, db = NULL and original is LOST
if (db == NULL) { perror("realloc"); return -1; }   // original pointer gone forever!

// ✅ CORRECT:
Student *temp = realloc(db, new_size);    // ← KEY LINE
if (temp == NULL) {
    perror("realloc");
    free(db);    // original is still valid — free it cleanly
    db = NULL;
    return -1;
}
db = temp;   // only update AFTER confirming success


// ── LEAK 3: early return without freeing ──
// ❌ WRONG:
Student *db = malloc(n * sizeof(Student));
FILE *f = fopen("data.bin", "rb");
if (fread(db, sizeof(Student), n, f) != (size_t)n) {
    fclose(f);
    return -1;   // BUG: db is leaked!
}

// ✅ CORRECT:
if (fread(db, sizeof(Student), n, f) != (size_t)n) {
    free(db);    // free all malloc'd memory
    fclose(f);
    return -1;
}


// ── LEAK 4: using fclose but forgetting free ──
// (When you wrote a function that returns a malloc'd buffer AND the caller forgets to free)
// ✅ Pattern: document ownership clearly in the function comment:
// "Returns malloc'd array. CALLER MUST CALL free() on the result."
```

---

### 10.6 — Valgrind-Style Mental Checklist

Before submitting any code that uses DMA + File I/O, check each of these:

```
✅ For every malloc/calloc:      Is there a matching free()?
✅ For every fopen:              Is there a matching fclose()?
✅ For every early return:       Are ALL malloc'd pointers freed first?
✅ For every realloc:            Is the return value stored in a TEMP pointer?
                                  Is the temp checked for NULL before reassigning?
✅ After every free:             Is the pointer set to NULL?
✅ For every NULL return check:  Are all previously allocated resources cleaned up?
✅ For fwrite/fread:             Is the return value checked against expected count?
✅ For every fopen("wb"):        Is this intentional? (wipes existing data!)
```

---

## Part 11 — Pointers + File I/O — Advanced Patterns

### 11.1 — Function Pointers with File Operations

A **function pointer** stores the address of a function, allowing you to choose behavior at runtime.

```c
// gcc funcptr_demo.c -o funcptr_demo -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct { int id; char name[50]; float grade; } Student;

// Two different writer functions with the same signature:
void writeText(FILE *f, Student *s) {
    fprintf(f, "%d,%s,%.2f\n", s->id, s->name, s->grade);
}
void writeBinary(FILE *f, Student *s) {
    fwrite(s, sizeof(Student), 1, f);
}

// A function pointer type for writer functions:
typedef void (*WriterFn)(FILE *, Student *);    // ← KEY LINE

void exportStudents(const char *filename, const char *mode,
                    Student *db, int count, WriterFn writer) {
    FILE *f = fopen(filename, mode);
    if (!f) { perror("exportStudents"); return; }
    for (int i = 0; i < count; i++) {
        writer(f, &db[i]);   // call whichever writer was passed
    }
    fclose(f);
}

int main() {
    Student db[3] = {
        {1, "Alice", 92.5f},
        {2, "Bob",   78.0f},
        {3, "Carol", 88.5f},
    };

    // Select handler at runtime:
    int useBinary = 0;
    WriterFn writer = useBinary ? writeBinary : writeText;    // ← KEY LINE

    exportStudents("output.csv", "w", db, 3, writeText);
    exportStudents("output.bin", "wb", db, 3, writeBinary);

    printf("Exported in both formats\n");
    return 0;
}
```

```
output.csv contents:
1,Alice,92.50
2,Bob,78.00
3,Carol,88.50
```

---

### 11.2 — Pointer to FILE \* (Passing File Handles to Functions)

A common beginner mistake is passing `FILE *` by value when the function needs to open the file.

```c
// ❌ WRONG — passes a copy of the pointer, changes don't propagate back:
void openFile(FILE *f, const char *name) {
    f = fopen(name, "r");   // assigns to LOCAL copy only!
    // When function returns, f (the local copy) is discarded.
    // The caller's pointer is still NULL.
}
// After calling openFile(myFile, "data.txt"), myFile is still NULL!

// ✅ CORRECT — pass a pointer TO the pointer:
void openFile(FILE **f, const char *name) {    // ← KEY LINE
    *f = fopen(name, "r");   // dereference: write the FILE* into caller's variable
}
// Call like this:
// FILE *myFile = NULL;
// openFile(&myFile, "data.txt");   // pass address of the pointer
// Now myFile is properly set by the function.
```

```c
// gcc file_ptr_demo.c -o file_ptr_demo -Wall -Wextra -std=c11
#include <stdio.h>

// Open + check in one reusable function:
int openFileChecked(FILE **f, const char *name, const char *mode) {
    *f = fopen(name, mode);
    if (*f == NULL) {
        perror(name);   // prints the filename in the error message
        return -1;
    }
    return 0;
}

int main() {
    FILE *f = NULL;
    if (openFileChecked(&f, "data.txt", "r") != 0) {
        return 1;
    }
    // f is now valid
    printf("File opened successfully\n");
    fclose(f);
    return 0;
}
```

---

### 11.3 — Pointer Arithmetic on Loaded Binary Data

After `fread` loads an array, walk it efficiently with pointer arithmetic:

```c
// gcc ptr_arith_file.c -o ptr_arith_file -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>

typedef struct { int id; char name[50]; float grade; } Student;

int main() {
    int count = 0;
    FILE *f = fopen("students.bin", "rb");
    if (!f) { perror("Error"); return 1; }

    fseek(f, 0, SEEK_END);
    count = (int)(ftell(f) / (long)sizeof(Student));
    rewind(f);

    Student *db = malloc((size_t)count * sizeof(Student));
    if (!db) { perror("malloc"); fclose(f); return 1; }
    fread(db, sizeof(Student), (size_t)count, f);
    fclose(f);

    // Pointer arithmetic iteration — same as index-based but more C-idiomatic:
    Student *end = db + count;    // ← KEY LINE: one-past-end pointer
    for (Student *p = db; p < end; p++) {
        printf("[%d] %-15s %.1f\n", p->id, p->name, p->grade);
    }

    // Find the highest grade using pointer arithmetic:
    Student *best = db;
    for (Student *p = db + 1; p < end; p++) {
        if (p->grade > best->grade) best = p;
    }
    printf("Best student: %s (%.1f)\n", best->name, best->grade);

    free(db); db = NULL;
    return 0;
}
```

---

### 11.4 — `const char *` for Filename Parameters

Always use `const char *` for filename parameters — it signals "this function will not modify the filename" and catches accidental modifications at compile time.

```c
// ❌ Less safe — function could accidentally modify the filename:
int loadFile(char *filename, Student *out, int maxCount);

// ✅ Correct — filename is read-only in this function:
int loadFile(const char *filename, Student *out, int maxCount);    // ← KEY LINE
//            ^^^^^^^^^^^
// Compiler error if you try: filename[0] = 'X'; inside the function.
// Allows passing string literals: loadFile("data.bin", ...) — correct.
```

```c
// String literals are const — passing them to non-const is a warning:
// gcc -Wall will warn: "passing const char* as char*"
loadFile("data.bin", students, 10);   // "data.bin" is a const string literal
// ✅ const char* parameter accepts this cleanly with no warnings
```

---

## Part 12 — Error Handling — The Complete System

### 12.1 — The Three Error-Detection Functions

```c
// Check if we reached end-of-file:
int feof(FILE *stream);   // returns nonzero if EOF flag is set

// Check if a read/write error occurred:
int ferror(FILE *stream); // returns nonzero if error flag is set

// Clear EOF and error flags:
void clearerr(FILE *stream);
```

```c
// gcc error_handling.c -o error_handling -Wall -Wextra -std=c11
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main() {
    FILE *f = fopen("data.txt", "r");
    if (!f) {
        // errno is set by the OS when fopen fails:
        printf("Error code: %d\n", errno);
        printf("Error message: %s\n", strerror(errno));   // ← KEY LINE
        perror("fopen");   // shortcut: "fopen: No such file or directory"
        return 1;
    }

    char buf[256];
    while (fgets(buf, sizeof(buf), f) != NULL) {
        printf("%s", buf);
    }

    // After reading loop — diagnose why we stopped:
    if (feof(f)) {
        printf("[EOF reached normally — all data read]\n");
    } else if (ferror(f)) {
        perror("[Read error]");    // ← KEY LINE
    }

    fclose(f);
    return 0;
}
```

### 12.2 — `errno` — The OS Error Code

`errno` is a global integer set by the OS when a system call fails.

```c
#include <errno.h>    // for errno
#include <string.h>   // for strerror()
```

| Code | Name     | When it occurs                                             |
| ---- | -------- | ---------------------------------------------------------- |
| 2    | `ENOENT` | `fopen("missing.txt", "r")` — file doesn't exist           |
| 13   | `EACCES` | Permission denied (e.g., trying to write a read-only file) |
| 17   | `EEXIST` | File already exists (rare with fopen)                      |
| 24   | `EMFILE` | Too many open files (hit OS limit — forgot to fclose)      |
| 28   | `ENOSPC` | Disk full — fwrite/fclose may fail                         |
| 22   | `EINVAL` | Invalid argument passed to a function                      |

```c
// Comprehensive error check after fopen:
FILE *f = fopen("protected.txt", "w");
if (f == NULL) {
    switch (errno) {
        case ENOENT:  fprintf(stderr, "File or directory not found\n"); break;
        case EACCES:  fprintf(stderr, "Permission denied\n"); break;
        case EMFILE:  fprintf(stderr, "Too many open files — close some!\n"); break;
        case ENOSPC:  fprintf(stderr, "Disk is full\n"); break;
        default:      fprintf(stderr, "Unknown error: %s\n", strerror(errno)); break;
    }
    return 1;
}
```

### 12.3 — `perror()` — The Quick Error Reporter

```c
void perror(const char *prefix);
// Prints: "prefix: <errno message>\n" to stderr
// Automatically uses the current errno value
```

```c
// ❌ Verbose:
fprintf(stderr, "fopen: %s\n", strerror(errno));

// ✅ Concise equivalent:
perror("fopen");    // prints "fopen: No such file or directory"
perror("fopen");    // can prepend filename: perror("grades.txt")
```

### 12.4 — The Complete Error-Safe File Function Pattern

```c
// gcc safe_file.c -o safe_file -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

typedef struct { int id; char name[50]; float grade; } Student;

// Returns 0 on success, -1 on error.
// All errors reported to stderr. Caller gets clean success/fail.
int saveDatabase(const char *filename, Student *db, int count) {
    if (!db || count <= 0) {
        fprintf(stderr, "saveDatabase: invalid arguments\n");
        return -1;
    }

    FILE *f = fopen(filename, "wb");
    if (!f) {
        perror(filename);    // e.g. "students.bin: Permission denied"
        return -1;
    }

    size_t written = fwrite(db, sizeof(Student), (size_t)count, f);
    if ((int)written != count) {
        fprintf(stderr, "saveDatabase: wrote %zu of %d records\n", written, count);
        if (ferror(f)) perror("fwrite");
        fclose(f);
        return -1;
    }

    // Check fclose too — it can fail if final flush fails (e.g., disk full):
    if (fclose(f) != 0) {    // ← KEY LINE: always check fclose on write files
        perror("fclose");
        return -1;
    }
    return 0;
}

int main() {
    Student db[2] = {{1, "Alice", 92.5f}, {2, "Bob", 78.0f}};
    int result = saveDatabase("students.bin", db, 2);
    printf("Save %s\n", result == 0 ? "succeeded" : "failed");
    return result == 0 ? 0 : 1;
}
```

---

## Part 13 — File Utility Functions

### 13.1 — `rename()` — Rename or Move a File

```c
int rename(const char *oldname, const char *newname);
// Returns: 0 on success, nonzero on error
// Also works as a MOVE on the same filesystem
```

```c
// gcc rename_demo.c -o rename_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    // Common pattern: write to temp file, rename to final on success
    // (prevents partial/corrupt files from being seen by other processes)

    FILE *f = fopen("output.tmp", "w");
    if (!f) { perror("Error"); return 1; }
    fprintf(f, "Important data\n");
    fclose(f);

    // Only rename to final name after successful write:
    if (rename("output.tmp", "output.txt") != 0) {    // ← KEY LINE
        perror("rename failed");
        return 1;
    }
    printf("File safely saved as output.txt\n");
    return 0;
}
```

> 💡 **Tip:** The temp-file + rename pattern is a classic technique for atomic file updates. If your program crashes during the write, `output.txt` still has the old valid data — only `output.tmp` is corrupted.

### 13.2 — `remove()` — Delete a File

```c
int remove(const char *filename);
// Returns: 0 on success, nonzero on error
```

```c
// gcc remove_demo.c -o remove_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    // Clean up a temporary file:
    if (remove("output.tmp") != 0) {    // ← KEY LINE
        perror("remove failed");
        // Not necessarily fatal — file might not exist
    } else {
        printf("Temporary file deleted\n");
    }
    return 0;
}
```

> ⚠️ **Warning:** `remove()` is permanent — there is no recycle bin or undo in C. The file is gone immediately.

### 13.3 — `tmpfile()` — Create a Temporary File

```c
FILE *tmpfile(void);
// Creates a temporary binary file opened in "w+b" mode
// Automatically deleted when fclose'd or program exits
// Returns: FILE* on success, NULL on error
```

```c
// gcc tmpfile_demo.c -o tmpfile_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *tmp = tmpfile();    // ← KEY LINE
    if (!tmp) { perror("tmpfile"); return 1; }

    // Use it as scratch space — no filename needed, no cleanup needed:
    fprintf(tmp, "Intermediate result: %d\n", 42);
    rewind(tmp);

    char line[256];
    fgets(line, sizeof(line), tmp);
    printf("Read back from tmp: %s", line);

    fclose(tmp);   // file automatically deleted
    printf("Temp file auto-deleted\n");
    return 0;
}
```

---

## Part 14 — CSV File Handling with strtok

CSV (Comma-Separated Values) is the most common text data format. Parsing it requires `fgets` + `strtok`.

### 14.1 — What is CSV?

```
id,name,grade
1,Alice,92.5
2,Bob,78.0
3,Carol,88.5
```

Each line is a record. Fields are separated by commas. The first line is usually a header.

### 14.2 — `strtok()` — Tokenize a String

```c
char *strtok(char *str, const char *delimiters);
// First call: pass the string to tokenize
// Subsequent calls: pass NULL to continue tokenizing same string
// Returns: pointer to next token, or NULL when done
// WARNING: strtok MODIFIES the original string — work on a copy!
```

```c
// gcc csv_reader.c -o csv_reader -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int parseCSV(const char *filename, Student *arr, int maxCount) {
    FILE *f = fopen(filename, "r");
    if (!f) { perror("parseCSV"); return 0; }

    char line[256];
    int count = 0;

    // Skip header line:
    fgets(line, sizeof(line), f);

    while (count < maxCount && fgets(line, sizeof(line), f) != NULL) {
        // Remove trailing newline:
        line[strcspn(line, "\r\n")] = '\0';    // ← KEY LINE: safe newline removal

        // Parse fields with strtok:
        char *token;

        token = strtok(line, ",");    // ← KEY LINE: first field
        if (!token) continue;
        arr[count].id = atoi(token);

        token = strtok(NULL, ",");    // ← KEY LINE: subsequent fields use NULL
        if (!token) continue;
        strncpy(arr[count].name, token, sizeof(arr[count].name) - 1);
        arr[count].name[sizeof(arr[count].name) - 1] = '\0';

        token = strtok(NULL, ",");
        if (!token) continue;
        arr[count].grade = (float)atof(token);

        count++;
    }

    fclose(f);
    return count;
}

int main() {
    // Create a test CSV file:
    FILE *f = fopen("students.csv", "w");
    if (!f) { perror("Error"); return 1; }
    fprintf(f, "id,name,grade\n");
    fprintf(f, "1,Alice,92.5\n");
    fprintf(f, "2,Bob,78.0\n");
    fprintf(f, "3,Carol,88.5\n");
    fclose(f);

    // Parse it:
    Student students[100];
    int count = parseCSV("students.csv", students, 100);

    printf("Parsed %d students from CSV:\n", count);
    for (int i = 0; i < count; i++) {
        printf("  [%d] %-15s %.1f\n", students[i].id, students[i].name, students[i].grade);
    }
    return 0;
}
```

```
Expected output:
Parsed 3 students from CSV:
  [1] Alice           92.5
  [2] Bob             78.0
  [3] Carol           88.5
```

### 14.3 — Writing CSV

```c
// gcc csv_writer.c -o csv_writer -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct { int id; char name[50]; float grade; } Student;

int exportCSV(const char *filename, Student *db, int count) {
    FILE *f = fopen(filename, "w");
    if (!f) { perror("exportCSV"); return -1; }

    // Header:
    fprintf(f, "id,name,grade\n");    // ← KEY LINE

    // Data rows:
    for (int i = 0; i < count; i++) {
        fprintf(f, "%d,%s,%.2f\n", db[i].id, db[i].name, db[i].grade);
    }

    fclose(f);
    return 0;
}

int main() {
    Student db[3] = {
        {1, "Alice", 92.5f},
        {2, "Bob",   78.0f},
        {3, "Carol", 88.5f},
    };
    exportCSV("output.csv", db, 3);
    printf("Exported to output.csv\n");
    return 0;
}
```

> ⚠️ **Warning:** `strtok` is not re-entrant — calling it on two different strings simultaneously (e.g., in nested loops) will corrupt both. For nested tokenization, use `strtok_r` (POSIX) or manually track positions with `strchr`.

---

## Part 15 — Safe String Handling

### 15.1 — Never Use `gets()` — It Was Removed in C11

```c
// ❌ BANNED — gets() has no buffer size limit and causes buffer overflows:
char buf[100];
gets(buf);   // if user types 200 chars → buffer overflow → security hole!
             // gets() was removed from the C11 standard

// ✅ ALWAYS use fgets with a size limit:
fgets(buf, sizeof(buf), stdin);    // ← KEY LINE: never writes more than sizeof(buf)-1
```

### 15.2 — Safe String Input from File

```c
// gcc safe_strings.c -o safe_strings -Wall -Wextra -std=c11
#include <stdio.h>
#include <string.h>

void processLine(const char *filename) {
    FILE *f = fopen(filename, "r");
    if (!f) { perror("Error"); return; }

    char line[256];
    while (fgets(line, sizeof(line), f) != NULL) {    // ← KEY LINE: safe!
        // Remove trailing \n (and \r for Windows files):
        line[strcspn(line, "\r\n")] = '\0';    // ← KEY LINE: safest newline strip

        // Check if line was too long (fgets truncated it):
        if (strlen(line) == sizeof(line) - 1) {
            fprintf(stderr, "Warning: line truncated (too long)\n");
        }

        // Safe copy to a fixed-size field:
        char name[50];
        strncpy(name, line, sizeof(name) - 1);    // ← KEY LINE: limit copy length
        name[sizeof(name) - 1] = '\0';            // always null-terminate!

        printf("Processed: '%s'\n", name);
    }
    fclose(f);
}
```

### 15.3 — Safe Number Parsing from Strings

```c
// ❌ WRONG — atoi() has no error checking:
int n = atoi("abc");   // returns 0 silently — you can't tell if it failed or was "0"

// ✅ CORRECT — strtol() with error checking:
#include <stdlib.h>
#include <errno.h>

char *endptr;
errno = 0;
long n = strtol("42", &endptr, 10);    // ← KEY LINE: base 10
if (errno != 0) {
    perror("strtol");   // overflow or underflow
} else if (endptr == "42") {
    fprintf(stderr, "No digits found\n");
} else if (*endptr != '\0') {
    fprintf(stderr, "Trailing non-numeric chars: %s\n", endptr);
} else {
    printf("Parsed: %ld\n", n);   // success
}
```

### 15.4 — Safe String Copy

```c
// ❌ WRONG — strcpy has no length limit:
char dest[50];
strcpy(dest, veryLongString);   // if strlen(veryLongString) >= 50 → overflow!

// ✅ CORRECT — strncpy with manual null termination:
strncpy(dest, veryLongString, sizeof(dest) - 1);    // ← KEY LINE
dest[sizeof(dest) - 1] = '\0';                       // ALWAYS do this after strncpy

// Why the manual null terminator? strncpy does NOT add \0 if source is longer than n.
// This is a famous C gotcha.
```

---

## Part 16 — stdin, stdout, stderr — The 3 Standard Streams

### 16.1 — What They Are

Every C program starts with three `FILE *` pointers already open:

```
File Descriptor 0 → stdin  → keyboard input (default)    → can be redirected from a file
File Descriptor 1 → stdout → terminal output (default)   → can be redirected to a file
File Descriptor 2 → stderr → terminal error (default)    → separate channel, for errors only
```

```
Program memory:
┌────────────────────────────────────────────────────────┐
│  stdin  (FILE* fd=0) ◄──── keyboard / input file       │
│  stdout (FILE* fd=1) ────► terminal / output file       │
│  stderr (FILE* fd=2) ────► terminal (separate channel)  │
└────────────────────────────────────────────────────────┘
```

### 16.2 — Equivalent Function Pairs

```c
// gcc streams_equiv.c -o streams_equiv -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    // These pairs are IDENTICAL in behavior:
    printf("hello");                        // stdout
    fprintf(stdout, "hello");               // stdout — explicit

    int x;
    scanf("%d", &x);                        // stdin
    fscanf(stdin, "%d", &x);               // stdin — explicit

    char buf[100];
    fgets(buf, sizeof(buf), stdin);        // stdin — same as scanf for lines

    fputs("error!\n", stderr);             // stderr
    fprintf(stderr, "error!\n");           // stderr — equivalent

    perror("msg");                         // always writes to stderr
    return 0;
}
```

### 16.3 — Shell Redirection

You can redirect the standard streams from the command line:

```bash
# Read stdin from a file (program sees no keyboard — reads file instead):
./program < input.txt

# Redirect stdout to a file (all printf output goes to file, not terminal):
./program > output.txt

# Redirect stderr to a file (error messages captured separately):
./program 2> errors.txt

# Redirect both:
./program < input.txt > output.txt 2> errors.txt

# Append stdout to a file (don't overwrite):
./program >> log.txt

# Redirect stderr to same destination as stdout:
./program > all_output.txt 2>&1
```

```c
// This program behaves differently depending on redirection:
// gcc redirect_demo.c -o redirect_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    printf("This goes to stdout\n");                    // visible in terminal, redirectable
    fprintf(stderr, "This goes to stderr\n");           // always visible even when stdout redirected

    // ./redirect_demo > out.txt
    // → out.txt contains "This goes to stdout"
    // → terminal still shows "This goes to stderr"
    return 0;
}
```

### 16.4 — Why Always Use stderr for Errors

```c
// ❌ WRONG — error messages in stdout pollute pipes and redirects:
printf("Error: file not found!\n");
// If someone runs: ./program > results.txt
// The error message ends up in results.txt mixed with data!

// ✅ CORRECT — errors to stderr, always visible regardless of stdout redirect:
fprintf(stderr, "Error: file not found!\n");    // ← KEY LINE
// Now: ./program > results.txt 2> errors.txt
// → results.txt has clean data
// → errors.txt has the error message
```

> 💡 **Tip:** Libraries and utilities should NEVER print error messages to `stdout`. Always use `stderr`. This is what makes command-line tools composable (piping works correctly).

---

## Part 17 — How the OS Handles Files

### 17.1 — File Descriptors

Every process gets its own **File Descriptor Table** — a list of open files the process is using.

```
Process memory (your program):
┌──────────────────────────────────────────────────────────┐
│  File Descriptor Table                                   │
│  ┌─────┬────────────────────────────────────────────┐   │
│  │  0  │ → stdin  (keyboard)                        │   │
│  │  1  │ → stdout (terminal)                        │   │
│  │  2  │ → stderr (terminal)                        │   │
│  │  3  │ → grades.txt  (opened by first fopen)      │   │
│  │  4  │ → students.bin (opened by second fopen)    │   │
│  │  5  │ → [free slot]                              │   │
│  │ ... │                                            │   │
│  └─────┴────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘

fopen() asks the OS: "give me a new descriptor for this file"
  → OS assigns the next available slot (3, 4, 5, ...)
  → FILE struct stores this fd number in its _fd field

fclose() says to the OS: "release descriptor N"
  → slot becomes free again
```

**The limit:** Each process can have at most ~1024 open files by default on Linux (check with `ulimit -n`).

```c
// This eventually fails if you keep opening without closing:
FILE *files[1025];
for (int i = 0; i < 1025; i++) {
    files[i] = fopen("test.txt", "r");
    if (!files[i]) {
        printf("Failed at i=%d: %s\n", i, strerror(errno));
        // "Too many open files" (EMFILE, code 24) around i=1021
        break;
    }
}
// This is why always closing files matters — it returns descriptors to the OS
```

### 17.2 — The Buffering Pipeline

```
Your fprintf() call's full journey to disk:

fprintf(f, "hello")
      │
      ▼
┌───────────────────────────┐
│  C Library Buffer (4–8KB) │  ← in your process's heap memory (inside FILE struct)
│  "hello"                  │     Data ACCUMULATES here for efficiency
└───────────────────────────┘
      │
      │  fflush(), fclose(), OR buffer is full
      ▼
┌───────────────────────────┐
│  OS Kernel Page Cache     │  ← OS-managed RAM
│  (also buffered)          │     OS decides WHEN to actually write to hardware
└───────────────────────────┘
      │
      │  OS schedules I/O — could be milliseconds or seconds later
      ▼
┌───────────────────────────┐
│  SSD / HDD                │  ← Permanent storage
│  Data is NOW truly safe   │
└───────────────────────────┘
```

**Three buffering modes:**

| Mode           | Constant | Behavior                           | Default for          |
| -------------- | -------- | ---------------------------------- | -------------------- |
| Full buffering | `_IOFBF` | Write when buffer is full (~4–8KB) | Files on disk        |
| Line buffering | `_IOLBF` | Write when `\n` is seen            | `stdout` to terminal |
| Unbuffered     | `_IONBF` | Write immediately                  | `stderr`             |

```c
// Change buffering mode (must call BEFORE any I/O on the stream):
FILE *log = fopen("critical.log", "w");
setvbuf(log, NULL, _IONBF, 0);   // unbuffered — each write goes to OS immediately
```

### 17.3 — Text vs Binary Mode (OS-Level)

```
On Windows:
  Text mode ("r"/"w"):
    Writing \n → \r\n stored on disk  (Windows line ending = CR+LF)
    Reading  \r\n ← \n returned to you  (translation happens automatically)

  Binary mode ("rb"/"wb"):
    What you write = exactly what's on disk — NO translation
    \n stays \n — good for structs, images, any non-text data

On Linux/macOS:
  Text and binary mode are IDENTICAL — \n = \n always.
  But using "rb"/"wb" for struct data is still good practice (portable code).

RULE OF THUMB:
  → Human-readable text (CSV, log, config)  → use "r" / "w"
  → Struct / binary data                    → ALWAYS use "rb" / "wb"
```

### 17.4 — What `errno` Is and How to Use It

`errno` is set by the OS whenever a system call fails. It tells you exactly why it failed.

```c
// gcc errno_demo.c -o errno_demo -Wall -Wextra -std=c11
#include <stdio.h>
#include <errno.h>    // ← for errno constant
#include <string.h>   // ← for strerror()

int main() {
    FILE *f = fopen("nonexistent.txt", "r");
    if (f == NULL) {
        // Method 1: get the integer code
        printf("Error code: %d\n", errno);           // e.g., 2

        // Method 2: get the human-readable string
        printf("Message: %s\n", strerror(errno));    // e.g., "No such file or directory"

        // Method 3: perror() — does both in one call, to stderr
        perror("fopen");                             // ← KEY LINE: "fopen: No such file or directory"
        return 1;
    }
    fclose(f);
    return 0;
}
```

```
Expected output:
Error code: 2
Message: No such file or directory
fopen: No such file or directory
```

> ⚠️ **Warning:** `errno` is overwritten by every subsequent system call. Check it immediately after a failure, before calling any other function. Don't let a successful call overwrite the error code you need.

---

## Part 18 — Common Pitfalls — 10 Traps With Full Explanations

### Pitfall 1 — Using NULL Without Checking

```c
// ❌ WRONG — segfault if file doesn't exist:
FILE *f = fopen("data.txt", "r");
fscanf(f, "%d", &x);    // crashes if f == NULL — reads through a null pointer

// ✅ CORRECT:
FILE *f = fopen("data.txt", "r");
if (f == NULL) { perror("data.txt"); return 1; }
fscanf(f, "%d", &x);
```

### Pitfall 2 — Using "w" When You Meant "a"

```c
// ❌ WRONG — destroys all existing log data:
FILE *log = fopen("app.log", "w");   // WIPES the file!
fprintf(log, "New entry\n");

// ✅ CORRECT — append to existing log:
FILE *log = fopen("app.log", "a");   // safe: adds to end, creates if not exists
fprintf(log, "New entry\n");
```

### Pitfall 3 — Forgetting to Close a File

```c
// ❌ WRONG — file never closed:
void writeReport(const char *filename) {
    FILE *f = fopen(filename, "w");
    if (!f) return;
    fprintf(f, "Report data\n");
    // forgot fclose! Data may not reach disk. File descriptor leaked.
}

// ✅ CORRECT:
void writeReport(const char *filename) {
    FILE *f = fopen(filename, "w");
    if (!f) return;
    fprintf(f, "Report data\n");
    fclose(f);   // flush to disk AND release OS resources
}
```

### Pitfall 4 — `fgetc` Return Type is `int`, Not `char`

```c
// ❌ WRONG — on systems where char is unsigned, EOF never matches:
char ch;
while ((ch = fgetc(f)) != EOF) {   // BUG: char can't hold -1 (EOF) correctly
    putchar(ch);
}

// ✅ CORRECT:
int ch;    // ← KEY LINE: must be int
while ((ch = fgetc(f)) != EOF) {
    putchar(ch);
}
```

### Pitfall 5 — Reading a Binary File in Text Mode (Windows)

```c
// ❌ WRONG on Windows — 0x0D0A bytes read as single \n:
FILE *f = fopen("data.bin", "r");    // text mode!
fread(buf, 1, 100, f);              // byte count is wrong — CR stripped

// ✅ CORRECT:
FILE *f = fopen("data.bin", "rb");   // binary mode — no translation
fread(buf, 1, 100, f);
```

### Pitfall 6 — Not Checking `fread`/`fwrite` Return Values

```c
// ❌ WRONG — silent partial write goes undetected:
fwrite(db, sizeof(Student), count, f);   // what if disk was full? only wrote half!

// ✅ CORRECT:
size_t written = fwrite(db, sizeof(Student), (size_t)count, f);
if ((int)written != count) {
    fprintf(stderr, "Write error: %zu of %d written\n", written, count);
    if (ferror(f)) perror("fwrite");
}
```

### Pitfall 7 — Dangling Pointer After `free()`

```c
// ❌ WRONG — use after free is undefined behavior (UB — anything can happen):
Student *db = malloc(n * sizeof(Student));
fread(db, sizeof(Student), n, f);
free(db);
printf("%s\n", db[0].name);   // UB! db points to freed memory!
                               // may crash, may print garbage, may seem to work (worst case)

// ✅ CORRECT — set to NULL immediately after free:
free(db);
db = NULL;    // ← KEY LINE
// Now db[0].name would cause a clean segfault (NULL dereference)
// rather than silent undefined behavior
```

### Pitfall 8 — Passing `FILE *` By Value When the Function Opens the File

```c
// ❌ WRONG — f inside the function is a local copy, changes don't propagate:
void openIt(FILE *f, const char *name) {
    f = fopen(name, "r");   // only modifies local copy of f!
}
// After call: caller's f is still NULL!

// ✅ CORRECT — pass pointer-to-pointer:
void openIt(FILE **f, const char *name) {    // ← KEY LINE
    *f = fopen(name, "r");
}
// Call: FILE *myFile = NULL; openIt(&myFile, "data.txt");
```

### Pitfall 9 — Struct Padding Breaking Binary File Portability

```c
// ❌ PROBLEM: compiler adds invisible padding bytes for alignment:
typedef struct {
    char  flag;     // 1 byte
                    // 3 bytes padding added by compiler!
    int   value;    // 4 bytes
} Record;
// sizeof(Record) = 8, NOT 5!
// A file written on one machine may be unreadable on a different architecture

// ✅ FIX 1: use __attribute__((packed)) on GCC/Clang — no padding:
typedef struct __attribute__((packed)) {    // ← KEY LINE
    char flag;
    int  value;
} Record;   // now exactly 5 bytes

// ✅ FIX 2: use fixed-width types from <stdint.h>:
#include <stdint.h>
typedef struct {
    uint8_t  flag;    // exactly 1 byte, guaranteed
    int32_t  value;   // exactly 4 bytes, guaranteed
} Record;
// Portable across 32-bit and 64-bit systems
```

### Pitfall 10 — `realloc` Losing Original Pointer on Failure

```c
// ❌ WRONG — if realloc fails, db becomes NULL and the original is leaked:
db = realloc(db, new_size);   // ← BUG: if realloc returns NULL, db is NULL
if (db == NULL) {             // original pointer is GONE — memory leaked!
    perror("realloc");
    return -1;
}

// ✅ CORRECT — always use a temp pointer:
Student *temp = realloc(db, new_size);    // ← KEY LINE
if (temp == NULL) {
    perror("realloc failed");
    free(db);    // original is still valid — free it cleanly
    db = NULL;
    return -1;
}
db = temp;   // only update AFTER confirming success  ← KEY LINE
```

---

## Part 19 — Real-World Project: Dynamic Student Database

This project combines EVERYTHING: dynamic arrays, binary files, CSV export, safe strings, error handling, function pointers, and pointer arithmetic.

### 19.1 — Project Structure

```
Dynamic Student Database
├── Struct: Student (id, name, grade)
├── loadDatabase()   → malloc + fread
├── saveDatabase()   → fwrite with error checking
├── addStudent()     → realloc doubling strategy
├── deleteStudent()  → element shift in array
├── findById()       → returns Student * (pointer to record)
├── exportCSV()      → fprintf text format
├── printTable()     → formatted terminal output
└── freeDatabase()   → free + NULL + count reset
```

### 19.2 — Complete Implementation

```c
// gcc student_db.c -o student_db -Wall -Wextra -std=c11
// Dynamic Student Database — complete production-quality implementation

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

// ─── Data Structure ───────────────────────────────────────────────────────────
typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

// ─── Load from binary file (caller must free the returned pointer) ─────────────
Student *loadDatabase(const char *file, int *count) {
    *count = 0;
    FILE *f = fopen(file, "rb");
    if (!f) {
        if (errno != ENOENT) perror(file);   // ENOENT = file not found = not an error on first run
        return NULL;
    }

    fseek(f, 0, SEEK_END);
    long sz = ftell(f);
    rewind(f);

    *count = (int)(sz / (long)sizeof(Student));
    if (*count == 0) { fclose(f); return NULL; }

    Student *db = malloc((size_t)*count * sizeof(Student));
    if (!db) { perror("loadDatabase: malloc"); fclose(f); *count = 0; return NULL; }

    int got = (int)fread(db, sizeof(Student), (size_t)*count, f);
    fclose(f);
    if (got != *count) *count = got;
    return db;
}

// ─── Save to binary file ───────────────────────────────────────────────────────
int saveDatabase(const char *file, Student *db, int count) {
    FILE *f = fopen(file, "wb");
    if (!f) { perror(file); return -1; }
    size_t w = fwrite(db, sizeof(Student), (size_t)count, f);
    int ok = ((int)w == count);
    if (!ok) perror("saveDatabase: fwrite");
    if (fclose(f) != 0) { perror("saveDatabase: fclose"); return -1; }
    return ok ? 0 : -1;
}

// ─── Add a student (dynamic realloc with doubling) ─────────────────────────────
Student *addStudent(Student *db, int *count, int *cap, Student s) {
    if (*count >= *cap) {
        int newCap = (*cap == 0) ? 4 : *cap * 2;
        Student *tmp = realloc(db, (size_t)newCap * sizeof(Student));
        if (!tmp) { perror("addStudent: realloc"); return db; }
        db = tmp;
        *cap = newCap;
    }
    db[(*count)++] = s;
    return db;
}

// ─── Delete a student by ID (shift elements left) ─────────────────────────────
int deleteStudent(Student *db, int *count, int id) {
    for (int i = 0; i < *count; i++) {
        if (db[i].id == id) {
            // Shift all elements after i one position left:
            memmove(&db[i], &db[i + 1], (size_t)(*count - i - 1) * sizeof(Student));
            (*count)--;
            return 0;   // success
        }
    }
    return -1;   // not found
}

// ─── Find by ID — returns pointer to record, or NULL ──────────────────────────
Student *findById(Student *db, int count, int id) {
    for (Student *p = db; p < db + count; p++) {    // pointer arithmetic iteration
        if (p->id == id) return p;    // return pointer to the found record
    }
    return NULL;
}

// ─── Export to CSV ────────────────────────────────────────────────────────────
int exportCSV(const char *file, Student *db, int count) {
    FILE *f = fopen(file, "w");
    if (!f) { perror(file); return -1; }
    fprintf(f, "id,name,grade\n");
    for (int i = 0; i < count; i++) {
        fprintf(f, "%d,%s,%.2f\n", db[i].id, db[i].name, db[i].grade);
    }
    return fclose(f) == 0 ? 0 : -1;
}

// ─── Print formatted table ────────────────────────────────────────────────────
void printTable(Student *db, int count) {
    if (count == 0) { printf("  (no records)\n"); return; }
    printf("  ┌──────┬─────────────────────┬────────┐\n");
    printf("  │  ID  │ Name                │ Grade  │\n");
    printf("  ├──────┼─────────────────────┼────────┤\n");
    for (int i = 0; i < count; i++) {
        printf("  │ %4d │ %-19s │ %6.2f │\n", db[i].id, db[i].name, db[i].grade);
    }
    printf("  └──────┴─────────────────────┴────────┘\n");
}

// ─── Free database ────────────────────────────────────────────────────────────
void freeDatabase(Student **db, int *count) {
    free(*db);
    *db = NULL;
    *count = 0;
}

// ─── Main demonstration ───────────────────────────────────────────────────────
int main() {
    const char *BIN = "students.bin";
    const char *CSV = "students.csv";

    int count = 0, cap = 0;
    Student *db = loadDatabase(BIN, &count);
    printf("Loaded %d existing records\n\n", count);

    // Add students:
    printf("Adding students...\n");
    db = addStudent(db, &count, &cap, (Student){1, "Alice",   92.5f});
    db = addStudent(db, &count, &cap, (Student){2, "Bob",     78.0f});
    db = addStudent(db, &count, &cap, (Student){3, "Carol",   88.5f});
    db = addStudent(db, &count, &cap, (Student){4, "David",   95.0f});
    db = addStudent(db, &count, &cap, (Student){5, "Eve",     91.0f});

    printf("\nCurrent database:\n");
    printTable(db, count);

    // Find by ID:
    Student *found = findById(db, count, 3);
    if (found) {
        printf("\nFound ID 3: %s (%.1f)\n", found->name, found->grade);
        found->grade = 90.0f;   // update in-place via pointer!
        printf("Updated Carol's grade to 90.0\n");
    }

    // Delete a student:
    printf("\nDeleting student with ID 2 (Bob)...\n");
    deleteStudent(db, &count, 2);

    printf("\nAfter deletion:\n");
    printTable(db, count);

    // Save and export:
    saveDatabase(BIN, db, count);
    exportCSV(CSV, db, count);
    printf("\nSaved to %s and exported to %s\n", BIN, CSV);

    freeDatabase(&db, &count);
    printf("Memory freed. count=%d, db=%s\n", count, db == NULL ? "NULL" : "?");
    return 0;
}
```

```
Expected output:
Loaded 0 existing records

Adding students...

Current database:
  ┌──────┬─────────────────────┬────────┐
  │  ID  │ Name                │ Grade  │
  ├──────┼─────────────────────┼────────┤
  │    1 │ Alice               │  92.50 │
  │    2 │ Bob                 │  78.00 │
  │    3 │ Carol               │  88.50 │
  │    4 │ David               │  95.00 │
  │    5 │ Eve                 │  91.00 │
  └──────┴─────────────────────┴────────┘

Found ID 3: Carol (88.5)
Updated Carol's grade to 90.0

Deleting student with ID 2 (Bob)...

After deletion:
  ┌──────┬─────────────────────┬────────┐
  │  ID  │ Name                │ Grade  │
  ├──────┼─────────────────────┼────────┤
  │    1 │ Alice               │  92.50 │
  │    3 │ Carol               │  90.00 │
  │    4 │ David               │  95.00 │
  │    5 │ Eve                 │  91.00 │
  └──────┴─────────────────────┴────────┘

Saved to students.bin and exported to students.csv
Memory freed. count=0, db=NULL
```

---

## Part 20 — Exam-Style Practice Problems

### Question 1 — Word/Line/Character Counter

**Tags:** File I/O | Text Processing  
**Difficulty:** ⭐

Write a program that reads a text file and counts the number of characters, words, and lines. Print the results.

**Expected output for a file with "Hello world\nGoodbye\n":**

```
Characters: 20
Words: 3
Lines: 2
```

<details>
<summary>💡 Hint</summary>

Read character by character with `fgetc`. Increment line count on `\n`. Track whether you're "inside" a word (non-space) to count words.

</details>

<details>
<summary>✅ Full Solution</summary>

```c
// gcc wc.c -o wc -Wall -Wextra -std=c11
#include <stdio.h>
#include <ctype.h>

int main() {
    FILE *f = fopen("input.txt", "r");
    if (!f) { perror("Error"); return 1; }

    int chars = 0, words = 0, lines = 0;
    int inWord = 0;
    int ch;

    while ((ch = fgetc(f)) != EOF) {    // ← KEY LINE
        chars++;
        if (ch == '\n') lines++;
        if (!isspace(ch)) {
            if (!inWord) { words++; inWord = 1; }
        } else {
            inWord = 0;
        }
    }

    fclose(f);
    printf("Characters: %d\nWords: %d\nLines: %d\n", chars, words, lines);
    return 0;
}
```

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: `fgetc`, EOF handling, `int ch` type (not `char`), character classification with `isspace`.

</details>

---

### Question 2 — File Copy

**Tags:** File I/O | fgetc/fputc  
**Difficulty:** ⭐

Copy the contents of `source.txt` to `dest.txt` byte-by-byte using `fgetc` and `fputc`. Print the number of bytes copied.

<details>
<summary>💡 Hint</summary>

Open source with `"rb"` and dest with `"wb"`. Loop until `fgetc` returns `EOF`.

</details>

<details>
<summary>✅ Full Solution</summary>

```c
// gcc filecopy.c -o filecopy -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *src = fopen("source.txt", "rb");
    if (!src) { perror("source.txt"); return 1; }

    FILE *dst = fopen("dest.txt", "wb");
    if (!dst) { perror("dest.txt"); fclose(src); return 1; }

    int ch, count = 0;
    while ((ch = fgetc(src)) != EOF) {    // ← KEY LINE
        fputc(ch, dst);
        count++;
    }

    fclose(src);
    fclose(dst);
    printf("Copied %d bytes\n", count);
    return 0;
}
```

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: opening two files simultaneously, `fgetc`/`fputc`, proper binary modes, closing both files.

</details>

---

### Question 3 — CSV Reader with Struct

**Tags:** File I/O | Structs | String Parsing  
**Difficulty:** ⭐⭐

Read a CSV file with columns `id,name,grade` into an array of `Student` structs. Print a formatted table.

<details>
<summary>💡 Hint</summary>

Use `fgets` to read lines, `strtok` to split on comma, `atoi`/`atof` to parse numbers. Skip the header line.

</details>

<details>
<summary>✅ Full Solution</summary>

```c
// gcc csv_struct.c -o csv_struct -Wall -Wextra -std=c11
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct { int id; char name[50]; float grade; } Student;

int main() {
    FILE *f = fopen("students.csv", "r");
    if (!f) { perror("Error"); return 1; }

    Student arr[100];
    int count = 0;
    char line[256];
    fgets(line, sizeof(line), f);   // skip header

    while (fgets(line, sizeof(line), f) && count < 100) {
        line[strcspn(line, "\r\n")] = '\0';
        char *t = strtok(line, ",");
        if (!t) continue;
        arr[count].id = atoi(t);
        t = strtok(NULL, ",");
        if (!t) continue;
        strncpy(arr[count].name, t, 49); arr[count].name[49] = '\0';
        t = strtok(NULL, ",");
        if (!t) continue;
        arr[count].grade = (float)atof(t);
        count++;
    }
    fclose(f);

    printf("%-5s %-20s %s\n", "ID", "Name", "Grade");
    printf("─────────────────────────────────\n");
    for (int i = 0; i < count; i++) {
        printf("%-5d %-20s %.1f\n", arr[i].id, arr[i].name, arr[i].grade);
    }
    return 0;
}
```

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: `fgets` + `strtok` CSV parsing, `atoi`/`atof`, `strncpy` safe copy, skipping header.

</details>

---

### Question 4 — Binary Search by ID

**Tags:** File I/O | Binary Files | fseek  
**Difficulty:** ⭐⭐

Given a binary file of `Student` structs sorted by ID, find a student with a given ID using O(1) direct access with `fseek`.

<details>
<summary>💡 Hint</summary>

If IDs are 1-based and contiguous, record with ID `k` is at byte offset `(k-1) * sizeof(Student)`.

</details>

<details>
<summary>✅ Full Solution</summary>

```c
// gcc bin_search.c -o bin_search -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct { int id; char name[50]; float grade; } Student;

Student *findByIdDirect(FILE *f, int id, Student *out) {
    long offset = (long)(id - 1) * (long)sizeof(Student);    // ← KEY LINE
    if (fseek(f, offset, SEEK_SET) != 0) return NULL;
    if (fread(out, sizeof(Student), 1, f) != 1) return NULL;
    if (out->id != id) return NULL;   // verify (guard against gaps)
    return out;
}

int main() {
    FILE *f = fopen("students.bin", "rb");
    if (!f) { perror("Error"); return 1; }

    Student s;
    if (findByIdDirect(f, 3, &s)) {
        printf("Found: [%d] %s %.1f\n", s.id, s.name, s.grade);
    } else {
        printf("Not found\n");
    }
    fclose(f);
    return 0;
}
```

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: `fseek` with `SEEK_SET`, `sizeof` arithmetic for record offsets, O(1) binary file access.

</details>

---

### Question 5 — Find & Replace in File

**Tags:** File I/O | String Processing  
**Difficulty:** ⭐⭐

Read `input.txt`, replace all occurrences of the word "Alice" with "Bob", write result to `output.txt`.

<details>
<summary>💡 Hint</summary>

Read line by line with `fgets`. Use `strstr` to find the word in each line. Use `fputs` or `fprintf` to write, substituting when found.

</details>

<details>
<summary>✅ Full Solution</summary>

```c
// gcc find_replace.c -o find_replace -Wall -Wextra -std=c11
#include <stdio.h>
#include <string.h>

void replaceInLine(const char *line, const char *from, const char *to, FILE *out) {
    size_t flen = strlen(from);
    const char *p = line;
    const char *found;
    while ((found = strstr(p, from)) != NULL) {    // ← KEY LINE
        fwrite(p, 1, (size_t)(found - p), out);   // write before match
        fputs(to, out);                            // write replacement
        p = found + flen;                          // advance past match
    }
    fputs(p, out);   // write remainder
}

int main() {
    FILE *in  = fopen("input.txt",  "r");
    FILE *out = fopen("output.txt", "w");
    if (!in || !out) { perror("Error"); return 1; }

    char line[1024];
    while (fgets(line, sizeof(line), in)) {
        replaceInLine(line, "Alice", "Bob", out);
    }

    fclose(in);
    fclose(out);
    printf("Replacement complete\n");
    return 0;
}
```

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: `strstr` for substring search, `fwrite` for partial line output, two files open simultaneously.

</details>

---

### Question 6 — Dynamic File Loader

**Tags:** DMA | File I/O | fread  
**Difficulty:** ⭐⭐

Load ALL Student records from a binary file into a `malloc`-allocated array. The number of records is unknown at compile time.

<details>
<summary>💡 Hint</summary>

`fseek(f, 0, SEEK_END)` + `ftell(f)` gives file size. Divide by `sizeof(Student)` for record count.

</details>

<details>
<summary>✅ Full Solution</summary>

See Part 10.3 for the complete `loadAll()` implementation — this question tests exactly that pattern.

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: `fseek`/`ftell`/`rewind` for file sizing, `malloc` with computed size, bulk `fread`, proper `free`.

</details>

---

### Question 7 — Merge Two Files

**Tags:** File I/O | Multiple Files  
**Difficulty:** ⭐⭐

Read lines alternately from `file1.txt` and `file2.txt` and write them interleaved to `merged.txt`. When one file is exhausted, continue with the other.

<details>
<summary>💡 Hint</summary>

Use two `FILE *` pointers. In a loop, read one line from each. Stop when both return NULL.

</details>

<details>
<summary>✅ Full Solution</summary>

```c
// gcc merge.c -o merge -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *f1 = fopen("file1.txt", "r");
    FILE *f2 = fopen("file2.txt", "r");
    FILE *out = fopen("merged.txt", "w");
    if (!f1 || !f2 || !out) { perror("Error"); return 1; }

    char l1[256], l2[256];
    char *r1, *r2;
    do {
        r1 = fgets(l1, sizeof(l1), f1);
        r2 = fgets(l2, sizeof(l2), f2);
        if (r1) fputs(l1, out);
        if (r2) fputs(l2, out);
    } while (r1 || r2);    // ← KEY LINE: continue until BOTH exhausted

    fclose(f1); fclose(f2); fclose(out);
    printf("Merged to merged.txt\n");
    return 0;
}
```

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: three files open simultaneously, interleaved reading, correct loop termination with `||`.

</details>

---

### Question 8 — Dynamic CSV Parser

**Tags:** DMA | CSV | realloc  
**Difficulty:** ⭐⭐⭐

Parse a CSV file with an unknown number of rows into a dynamically grown array using `malloc`/`realloc`.

<details>
<summary>💡 Hint</summary>

Start with capacity 4. Double capacity when full using `realloc`. Remember the temp-pointer pattern.

</details>

<details>
<summary>✅ Full Solution</summary>

```c
// gcc dyn_csv.c -o dyn_csv -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct { int id; char name[50]; float grade; } Student;

int main() {
    FILE *f = fopen("students.csv", "r");
    if (!f) { perror("Error"); return 1; }

    Student *arr = NULL;
    int count = 0, cap = 0;
    char line[256];
    fgets(line, sizeof(line), f);   // skip header

    while (fgets(line, sizeof(line), f)) {
        line[strcspn(line, "\r\n")] = '\0';

        if (count >= cap) {
            cap = cap == 0 ? 4 : cap * 2;
            Student *tmp = realloc(arr, (size_t)cap * sizeof(Student));    // ← KEY LINE
            if (!tmp) { perror("realloc"); free(arr); fclose(f); return 1; }
            arr = tmp;
        }

        char *t = strtok(line, ","); if (!t) continue;
        arr[count].id = atoi(t);
        t = strtok(NULL, ","); if (!t) continue;
        strncpy(arr[count].name, t, 49); arr[count].name[49] = '\0';
        t = strtok(NULL, ","); if (!t) continue;
        arr[count].grade = (float)atof(t);
        count++;
    }
    fclose(f);

    printf("Loaded %d rows:\n", count);
    for (int i = 0; i < count; i++) {
        printf("  [%d] %-15s %.1f\n", arr[i].id, arr[i].name, arr[i].grade);
    }

    free(arr);
    return 0;
}
```

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: the realloc doubling strategy, temp-pointer safety, CSV parsing, proper free on all paths.

</details>

---

### Question 9 — Grade Report Generator

**Tags:** File I/O | Statistics | CSV  
**Difficulty:** ⭐⭐⭐

Read `grades.csv` (id, name, grade), compute average, highest, and lowest grade, and write a formatted text report to `report.txt`.

<details>
<summary>✅ Full Solution</summary>

```c
// gcc grade_report.c -o grade_report -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <float.h>

typedef struct { int id; char name[50]; float grade; } Student;

int main() {
    FILE *in = fopen("grades.csv", "r");
    if (!in) { perror("grades.csv"); return 1; }

    Student arr[200];
    int count = 0;
    char line[256];
    fgets(line, sizeof(line), in);   // skip header

    while (fgets(line, sizeof(line), in) && count < 200) {
        line[strcspn(line, "\r\n")] = '\0';
        char *t = strtok(line, ","); if (!t) continue;
        arr[count].id = atoi(t);
        t = strtok(NULL, ","); if (!t) continue;
        strncpy(arr[count].name, t, 49); arr[count].name[49] = '\0';
        t = strtok(NULL, ","); if (!t) continue;
        arr[count].grade = (float)atof(t);
        count++;
    }
    fclose(in);

    if (count == 0) { printf("No data\n"); return 0; }

    float sum = 0, hi = -FLT_MAX, lo = FLT_MAX;
    char hiName[50], loName[50];
    for (int i = 0; i < count; i++) {
        sum += arr[i].grade;
        if (arr[i].grade > hi) { hi = arr[i].grade; strncpy(hiName, arr[i].name, 49); }
        if (arr[i].grade < lo) { lo = arr[i].grade; strncpy(loName, arr[i].name, 49); }
    }

    FILE *out = fopen("report.txt", "w");
    if (!out) { perror("report.txt"); return 1; }

    fprintf(out, "=== Grade Report ===\n");
    fprintf(out, "Total students : %d\n", count);
    fprintf(out, "Average grade  : %.2f\n", sum / count);
    fprintf(out, "Highest grade  : %.2f (%s)\n", hi, hiName);
    fprintf(out, "Lowest grade   : %.2f (%s)\n", lo, loName);
    fprintf(out, "\nFull roster:\n");
    for (int i = 0; i < count; i++) {
        fprintf(out, "  %-20s %.2f\n", arr[i].name, arr[i].grade);
    }
    fclose(out);
    printf("Report written to report.txt\n");
    return 0;
}
```

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: CSV read, statistical computation (min/max/avg), formatted fprintf report writing, `<float.h>` constants.

</details>

---

### Question 10 — File-Backed Contact Manager

**Tags:** DMA | Binary Files | CRUD  
**Difficulty:** ⭐⭐⭐

Implement a contact manager with add, list, search (by name), delete (by ID), and update operations, backed by a binary file.

<details>
<summary>💡 Hint</summary>

Use the dynamic database pattern from Part 19. Load at startup, save at exit. Use `strstr` for name search.

</details>

<details>
<summary>✅ Full Solution</summary>

The complete solution is essentially the Part 19 project adapted for Contact structs. See that section — replace `Student` with:

```c
typedef struct {
    int  id;
    char name[50];
    char phone[20];
    char email[60];
} Contact;
```

And implement the same load/save/add/delete/find functions. Use `strstr(c->name, query)` for name search.

</details>

<details>
<summary>📝 Examiner's Note</summary>

Tests: integration of all concepts — binary file persistence, dynamic array management, CRUD operations, pointer-to-record return from find functions.

</details>

---

## Part 21 — Quick Reference Cheat Sheet

### File I/O Functions

| Function                | Purpose                       | Returns              |
| ----------------------- | ----------------------------- | -------------------- |
| `fopen(name, mode)`     | Open a file                   | `FILE*` or `NULL`    |
| `fclose(f)`             | Close file, flush buffer      | `0` or `EOF`         |
| `fflush(f)`             | Flush buffer without closing  | `0` or `EOF`         |
| `fprintf(f, fmt, ...)`  | Formatted write               | chars written or neg |
| `fputs(str, f)`         | Write string                  | nonneg or `EOF`      |
| `fputc(ch, f)`          | Write one character           | char or `EOF`        |
| `fwrite(ptr, sz, n, f)` | Write binary data             | elements written     |
| `fgets(buf, n, f)`      | Read line (safe)              | `char*` or `NULL`    |
| `fscanf(f, fmt, ...)`   | Parse formatted input         | matched count        |
| `fgetc(f)`              | Read one character            | `int` char or `EOF`  |
| `fread(ptr, sz, n, f)`  | Read binary data              | elements read        |
| `fseek(f, off, whence)` | Move file cursor              | `0` or nonzero       |
| `ftell(f)`              | Get cursor position           | `long` bytes         |
| `rewind(f)`             | Cursor to start + clear flags | `void`               |
| `feof(f)`               | Check EOF flag                | nonzero if at EOF    |
| `ferror(f)`             | Check error flag              | nonzero if error     |
| `perror(msg)`           | Print errno message to stderr | `void`               |
| `rename(old, new)`      | Rename/move file              | `0` or nonzero       |
| `remove(name)`          | Delete file                   | `0` or nonzero       |
| `tmpfile()`             | Create temp file              | `FILE*` or `NULL`    |

### File Mode Quick Reference

| Mode                                                   |  R  |  W  | Create | Wipe | Start |
| ------------------------------------------------------ | :-: | :-: | :----: | :--: | :---: |
| `"r"`                                                  | ✅  | ❌  |   ❌   |  ❌  |  BOF  |
| `"w"`                                                  | ❌  | ✅  |   ✅   |  ✅  |  BOF  |
| `"a"`                                                  | ❌  | ✅  |   ✅   |  ❌  |  EOF  |
| `"r+"`                                                 | ✅  | ✅  |   ❌   |  ❌  |  BOF  |
| `"w+"`                                                 | ✅  | ✅  |   ✅   |  ✅  |  BOF  |
| Add `b` for binary mode: `"rb"`, `"wb"`, `"r+b"`, etc. |

### DMA Quick Reference

| Function  | Syntax                            | Zeroed? | Use when                     |
| --------- | --------------------------------- | :-----: | ---------------------------- |
| `malloc`  | `malloc(n * sizeof(T))`           |  ❌ No  | Speed matters                |
| `calloc`  | `calloc(n, sizeof(T))`            | ✅ Yes  | Need zero-initialized array  |
| `realloc` | `realloc(ptr, new_n * sizeof(T))` |  ❌ No  | Resizing existing allocation |
| `free`    | `free(ptr); ptr = NULL;`          |    —    | After every malloc/calloc    |

**realloc safe pattern (always):**

```c
T *tmp = realloc(arr, new_size);
if (!tmp) { free(arr); arr = NULL; return -1; }
arr = tmp;
```

### Pointer Quick Reference for File I/O

| Pattern                          | Use case                                            |
| -------------------------------- | --------------------------------------------------- |
| `FILE *f`                        | Normal file handle                                  |
| `FILE **f`                       | Function that opens/creates the handle              |
| `void *ptr`                      | Used in `fread`/`fwrite` — accepts any pointer type |
| `Student *db`                    | Dynamically loaded array of records                 |
| `Student **db`                   | Function that `malloc`s and returns the array       |
| `const char *filename`           | Read-only string parameter (safe)                   |
| `Student *p = db; p < db+n; p++` | Pointer arithmetic iteration over array             |

### fseek Reference

```c
fseek(f, 0, SEEK_SET);           // go to start
fseek(f, 0, SEEK_END);           // go to end (for ftell = file size)
fseek(f, N * sizeof(T), SEEK_SET); // jump to record N (binary files)
fseek(f, -sizeof(T), SEEK_CUR);  // back one record
rewind(f);                       // go to start + clear error/EOF flags
```

### errno Quick Reference

| Code | Name     | Meaning             |
| ---- | -------- | ------------------- |
| 2    | `ENOENT` | File not found      |
| 13   | `EACCES` | Permission denied   |
| 17   | `EEXIST` | File already exists |
| 24   | `EMFILE` | Too many open files |
| 28   | `ENOSPC` | Disk full           |
| 22   | `EINVAL` | Invalid argument    |

### Standard Streams

```c
// Equivalent pairs:
printf("x");         ≡  fprintf(stdout, "x");
scanf("%d", &x);     ≡  fscanf(stdin, "%d", &x);
puts("x");           ≡  fputs("x\n", stdout);
getchar();           ≡  fgetc(stdin);
// For errors — ALWAYS use stderr:
fprintf(stderr, "Error: ...\n");
perror("context");   // writes "context: <errno message>" to stderr
```

### The Complete File I/O Template

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main() {
    // 1. Open
    FILE *f = fopen("filename.ext", "mode");
    if (!f) { perror("filename.ext"); return 1; }

    // 2. Read / Write
    // ... your operations here ...

    // 3. Close (check return on write files)
    if (fclose(f) != 0) { perror("fclose"); return 1; }
    return 0;
}
```

### DMA + File I/O Template

```c
// Load unknown-count binary records dynamically:
FILE *f = fopen(filename, "rb");
if (!f) { perror(filename); return NULL; }

fseek(f, 0, SEEK_END);
int count = (int)(ftell(f) / (long)sizeof(T));
rewind(f);

T *arr = malloc((size_t)count * sizeof(T));
if (!arr) { perror("malloc"); fclose(f); return NULL; }

fread(arr, sizeof(T), (size_t)count, f);
fclose(f);
// ... use arr ...
free(arr); arr = NULL;
```

---

## Part 22 — FAQ — 15 Questions Fully Answered

**Q1: Can I have multiple files open at the same time?**  
Yes. Use separate `FILE *` variables for each. The OS allows up to ~1024 open files per process by default. Always `fclose()` when done — descriptors are a limited resource.

**Q2: What's the difference between text and binary mode on Windows?**  
In text mode (`"r"`, `"w"`), Windows translates `\n` in your code to `\r\n` on disk and back. In binary mode (`"rb"`, `"wb"`), no translation happens. On Linux/Mac there is no difference. Always use binary mode for struct data — it's portable and exact.

**Q3: Is `fgets` safe from buffer overflows?**  
Yes — you pass the buffer size, so it never writes more than `size-1` characters plus a null terminator. It's the safe replacement for the banned `gets()`.

**Q4: When should I use text vs binary files?**  
Use **text** (`"r"`, `"w"`) when humans need to read the file (CSV, log, config). Use **binary** (`"rb"`, `"wb"`) for structs and computed data — it's faster, more compact, and supports O(1) random access with `fseek`.

**Q5: What happens if `fwrite` partially writes?**  
It returns the number of complete elements written. If less than requested, check `ferror()`. Common cause: disk full (`ENOSPC`). Check the return value — don't assume success.

**Q6: Why must `fgetc()` return `int`, not `char`?**  
`EOF` is typically `-1`. If `char` is `unsigned` on your platform, it holds values 0–255 and can never equal `-1`, so the loop never terminates. `int` holds both 0–255 AND `-1` correctly.

**Q7: When should I use `fflush()`?**  
Use it for real-time logging (write immediately without closing), when you write then switch to reading the same file, or when your `printf` prompt doesn't appear before `scanf` input.

**Q8: What is `errno` and when is it set?**  
`errno` is a global integer set by the OS to indicate why a system call failed. Check it immediately after a failed `fopen`, `fread`, `fwrite`, etc. Use `perror()` or `strerror(errno)` to print the message.

**Q9: After `free()`, why set the pointer to NULL?**  
Because `free()` doesn't modify the pointer — it just marks the heap memory as available. After free, the pointer is "dangling" — it points to memory the OS may give to someone else. Setting to NULL causes a clean segfault on accidental use instead of silent corruption.

**Q10: Why does `sizeof(MyStruct)` sometimes return more than the sum of field sizes?**  
The compiler adds **padding bytes** between fields to ensure each field is aligned to its natural boundary (e.g., `int` aligned to 4 bytes). Use `sizeof` always — never hardcode struct sizes.

**Q11: Why use `realloc` with a temp pointer?**  
If `realloc` fails, it returns `NULL` but the original allocation is still valid. If you write `arr = realloc(arr, new_size)` and it fails, `arr` becomes `NULL` and you've lost the original pointer — a memory leak. The temp pattern preserves it.

**Q12: What's the difference between `feof()` returning true vs `fgets()` returning NULL?**  
`feof()` only returns true AFTER you've already attempted to read past the end. `fgets()` returning NULL means either EOF or an error occurred — check both with `feof()` and `ferror()`.

**Q13: How do I check how many records are in a binary file without reading them all?**  
`fseek(f, 0, SEEK_END); long size = ftell(f); int count = size / sizeof(MyStruct);`. This is O(1) — instant regardless of file size.

**Q14: Is it safe to use the same `FILE *` for both reading and writing?**  
Yes, with modes `"r+"`, `"w+"`, or `"a+"`. But between switching from write to read (or vice versa), you must call either `fflush()` (if you were writing) or `fseek()`/`rewind()` (always). Failing to do this causes undefined behavior.

**Q15: What's the safest way to strip the `\n` from a `fgets()` result?**  
`line[strcspn(line, "\r\n")] = '\0';` — this handles both `\n` (Linux) and `\r\n` (Windows files) and is safe even if there is no newline (line was truncated).

---

## Part 23 — What to Learn Next (Roadmap)

You've mastered File I/O in C. Here is the recommended progression:

### Level 1 — Immediate Next Steps

**Linked lists with file persistence** — instead of shifting arrays on delete, use a linked list. Save by walking the list and `fwrite`-ing each node; load by `fread`-ing all and rebuilding `next` pointers.

**More advanced error handling** — `setjmp`/`longjmp` for error recovery, structured cleanup patterns, error propagation through return codes.

**CSV edge cases** — quoted fields (`"Smith, Jr."` contains a comma), escaped quotes, multiline fields. Look at RFC 4180 for the full CSV spec.

### Level 2 — Intermediate C

**POSIX file I/O** — `open()`, `read()`, `write()`, `lseek()`, `close()`. These are the lower-level OS calls that `fopen`/`fread` are built on. You'll use them for file locking and advanced I/O.

**File locking** — `flock()` or `lockf()` to safely share files between multiple processes running simultaneously.

**Memory-mapped files** — `mmap()` maps a file directly into your process's address space. Reads/writes to the pointer automatically go to the file. Extremely fast for large files.

### Level 3 — Databases and Systems

**SQLite in C** — an embedded SQL database stored in a single file. The `sqlite3` C API is clean and well-documented. Use it when you need queries, transactions, or indexes.

**Hash tables** — instead of linear search through your array, use a hash of the ID to jump directly to the right record. Combine with binary files for a home-made database.

**Multi-process programming** — `fork()`, pipes, signals. Files become a coordination mechanism between processes.

### Level 4 — Production Systems

**Buffered I/O vs direct I/O** — understanding when to bypass the C library buffer (`O_DIRECT` on Linux) for maximum performance in database systems.

**Write-ahead logging (WAL)** — the technique used by databases like SQLite and PostgreSQL to ensure data consistency across crashes. You write to a log file first, then apply to the main file.

**Network I/O** — `socket()`, `connect()`, `send()`, `recv()`. The same read/write concepts apply, but to network connections instead of files.

---

### Learning Resources

```
C Programming Language (K&R)    — Kernighan & Ritchie, 2nd Ed.
                                   The original, still the best reference.

C Programming: A Modern Approach — K.N. King
                                   Best textbook for learning C systematically.

Linux Man Pages                  — man fopen, man malloc, man errno
                                   The definitive reference, always accurate.

Beej's Guide to C Programming   — beej.us/guide/bgc/
                                   Free, online, excellent C11 coverage.

CS:APP (Computer Systems)       — Bryant & O'Hallaron
                                   Deep dive into how files and I/O work at the OS level.
```

---

## 📜 License

This guide is released under the **MIT License** — free to use, share, and modify with attribution.

```
MIT License
Copyright (c) 2024–2025 — C File I/O Guide Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software.
```

---

## 🌟 Found This Helpful?

If this guide helped you understand C File I/O, Pointers, and Dynamic Memory:

- ⭐ **Star this repo** so your classmates can find it
- 🐛 **Open an issue** if you spot an error or have a question
- 🔀 **Submit a PR** to add exercises, fix explanations, or improve diagrams
- 📢 **Share it** with your study group and friends in CSE

_You now have the complete toolkit to build real, persistent, production-quality C programs. Files are how your code leaves a mark on the world — use them well._ 🎓

---

_Made with ❤️ for CSE students — contributions welcome!_
