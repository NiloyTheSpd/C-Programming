# File I/O in C — A Beginner to Pro Guide

> **Who is this for?** Students who know variables, loops, and functions in C, and want to learn how to work with files — starting from zero and building to real-world programs.

**How to compile every example in this guide:**
```bash
gcc filename.c -o filename -Wall -Wextra -std=c11
./filename        # Linux / macOS
filename.exe      # Windows
```
The `-Wall -Wextra` flags make the compiler warn you about common mistakes. Always use them.

---

## Table of Contents

- [Part 0 — How to Use This Guide](#part-0--how-to-use-this-guide)
- [Part 1 — Why Files? (And How Memory Works)](#part-1--why-files)
- [Part 2 — Pointers — The Foundation You Need](#part-2--pointers)
- [Part 3 — The FILE * Pointer — What It Really Is](#part-3--the-file--pointer)
- [Part 4 — Opening & Closing Files](#part-4--opening--closing-files)
- [Part 5 — File Modes — Which One Do I Use?](#part-5--file-modes)
- [Part 6 — Writing to Files](#part-6--writing-to-files)
- [Part 7 — Reading from Files](#part-7--reading-from-files)
- [Part 8 — Moving Around in a File (fseek, ftell, rewind)](#part-8--moving-around-in-a-file)
- [Part 9 — Binary Files & Structs](#part-9--binary-files--structs)
- [Part 10 — Dynamic Memory + File I/O (The Big Combo)](#part-10--dynamic-memory--file-io)
- [Part 11 — Pointers + File I/O — Advanced Patterns](#part-11--advanced-patterns)
- [Part 12 — Error Handling](#part-12--error-handling)
- [Part 13 — File Utility Functions](#part-13--file-utility-functions)
- [Part 14 — CSV Files with strtok](#part-14--csv-files)
- [Part 15 — Safe String Handling](#part-15--safe-string-handling)
- [Part 16 — stdin, stdout, stderr](#part-16--stdin-stdout-stderr)
- [Part 17 — How the OS Handles Files](#part-17--how-the-os-handles-files)
- [Part 18 — 10 Common Mistakes (And How to Fix Them)](#part-18--10-common-mistakes)
- [Part 19 — Real-World Project: Student Database](#part-19--real-world-project)
- [Part 20 — Practice Problems](#part-20--practice-problems)
- [Part 21 — Quick Reference Cheat Sheet](#part-21--cheat-sheet)
- [Part 22 — FAQ](#part-22--faq)
- [Part 23 — What to Learn Next](#part-23--what-to-learn-next)

---

## Part 0 — How to Use This Guide

If you're **learning from scratch**, read every part in order. Each one builds on the previous.

If you have an **exam soon**, focus on Parts 1, 3–9, and 18.

If you're **building a project**, jump to Parts 9, 10, and 19.

Every code example in this guide is complete and runnable. Copy it, compile it, and run it yourself. Reading is not enough — you need to type and run the code to really understand it.

---

## Part 1 — Why Files?

### The Core Problem: Your Program Forgets Everything

Think about a calculator app. You type in a number, it computes a result — but as soon as you close the app, that result is gone. That's because your program stores data in **RAM** (memory), which is wiped clean when the program ends.

Files solve this. A file lives on your hard drive or SSD, which keeps its contents even when the computer is turned off.

```
Without Files:                       With Files:

Program starts                       Program starts
   │                                    │
int score = 100;  ← stored in RAM   int score = 100;
   │                                    │
Program exits                        Write score to "save.txt"
   │                                    │
score = ??? ← RAM wiped             Program exits
   │                                    │
Data is GONE ❌                      "save.txt" still has 100 ✅
                                        │
                                     Next time: read from "save.txt"
                                        │
                                     score = 100 again ✅
```

**Real-world examples of files:**
- A game saving your progress
- A spreadsheet storing your data
- A log file recording what happened in a program

### How Data Gets to Disk

When you write to a file, the data doesn't instantly land on disk. It goes through two buffers first:

```
Your code writes data
        │
        ▼
  C Library Buffer  ← sits in your program's RAM
  (collects data, ~4KB)
        │
  buffer fills up, or you call fflush() or fclose()
        │
        ▼
  OS Kernel Buffer  ← managed by the operating system
        │
  OS decides when to write to disk
        │
        ▼
  Disk (your file)  ← data is now safe ✅
```

**Why does this matter?** If your program crashes after writing but before `fclose()`, data in the buffer is lost. That's why you must always close your files.

---

## Part 2 — Pointers

> **Why are we learning this here?** Almost everything in File I/O uses pointers. `FILE *` is a pointer. `fread()` and `fwrite()` take pointers. Dynamic arrays use pointers. You need this foundation first.

### What Is a Pointer?

A pointer is a variable that stores a **memory address** instead of a value. Think of RAM as a huge grid of boxes, each with an address. A normal variable holds a value in one box. A pointer holds the *address* of another box.

```
Normal variable:       Pointer:
┌─────────────┐        ┌─────────────┐     ┌─────────────┐
│  x = 42     │        │  p = 0x100  │────►│  x = 42     │
│  address:   │        │  address:   │     │  address:   │
│  0x100      │        │  0x200      │     │  0x100      │
└─────────────┘        └─────────────┘     └─────────────┘
                        p stores the address of x
```

```c
// gcc pointer_basics.c -o pointer_basics -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    int x = 42;
    int *p = &x;   // p stores the address of x  ← KEY LINE
                   // & means "the address of"
                   // int* means "pointer to an int"

    printf("Value of x:        %d\n",  x);
    printf("Address of x:      %p\n",  (void *)&x);
    printf("Value of p:        %p\n",  (void *)p);   // same as &x
    printf("Value p points to: %d\n",  *p);          // * means "what p points to"

    *p = 100;   // change x through the pointer
    printf("x is now: %d\n", x);   // prints 100
    return 0;
}
```

**Three things you do with pointers:**
```c
int *p = &x;   // & = "address of" — get x's address, store in p
int v  = *p;   // * = dereference  — follow p, read the value there
p + 1;         // pointer arithmetic — move to the next int in memory
```

### Pointers to Structs

You'll use this pattern constantly in file programs. When a pointer points to a struct, you access fields with `->` instead of `.`:

```c
// gcc struct_ptr.c -o struct_ptr -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int main() {
    Student s = {1, "Alice", 92.5f};
    Student *ptr = &s;   // ptr points to s

    // These two lines do the exact same thing:
    printf("%s\n", s.name);      // dot notation — use when you have the struct directly
    printf("%s\n", ptr->name);   // arrow notation — use when you have a pointer ← KEY

    return 0;
}
```

**Rule:** Use `.` when you have a variable. Use `->` when you have a pointer.

### Pointer Arithmetic with Arrays

In C, an array name is a pointer to its first element. You can move through an array using pointer arithmetic:

```c
int arr[5] = {10, 20, 30, 40, 50};
int *p = arr;   // p points to arr[0]

// These four lines all print 30 (the third element):
printf("%d\n", arr[2]);    // index notation
printf("%d\n", *(arr+2));  // pointer arithmetic on array
printf("%d\n", p[2]);      // index on pointer
printf("%d\n", *(p+2));    // pointer arithmetic on pointer
```

When you do `p + 1`, the pointer moves forward by `sizeof(int)` bytes — it automatically jumps to the next element, whatever type it is.

### Double Pointers `**`

A double pointer is a pointer to a pointer. You need this when a function needs to give you back a new pointer (like after a `malloc` inside a function):

```c
// gcc double_ptr.c -o double_ptr -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>

typedef struct { int id; char name[30]; } Student;

// WRONG — the caller's pointer is never updated
void loadWrong(Student *arr) {
    arr = malloc(3 * sizeof(Student));   // only changes the local copy
}

// CORRECT — use ** to change the caller's pointer
void loadCorrect(Student **arr) {        // ← KEY: double pointer
    *arr = malloc(3 * sizeof(Student));  // changes what the caller's pointer points to
    (*arr)[0].id = 1;
}

int main() {
    Student *db = NULL;
    loadCorrect(&db);   // pass the address of db  ← KEY LINE
    printf("id = %d\n", db[0].id);   // prints 1
    free(db);
    return 0;
}
```

**When you need `**`:** Any time a function needs to `malloc` something and hand you the pointer back.

### ✏️ Practice — Part 2

1. Declare an `int` variable with value `5`. Create a pointer to it. Print the value using the pointer, then change it to `10` through the pointer.
2. Create a struct with two fields. Make a pointer to it. Print both fields using `->`.
3. Write a function `void setToZero(int *p)` that sets the value at the pointer to 0. Test it.

---

## Part 3 — The FILE * Pointer

### What Is FILE *?

When you open a file in C, you get back a `FILE *` — a pointer to a struct that the C library created for you. This struct holds everything the library needs to manage the file: which file on disk, where you are in it, a buffer for data, and so on.

Think of it like a TV remote. The remote (your `FILE *`) is what you hold in your hand and use. The television (the actual file on disk) is what gets controlled. The `FILE *` is your handle to the file.

```
Your code:
    FILE *f = fopen("grades.txt", "r");

What happens in memory:
┌────────────┐           ┌──────────────────┐           ┌──────────────┐
│  f         │ ─────────►│  FILE struct      │ ─────────►│  grades.txt  │
│ (your ptr) │           │  buffer: [...]   │           │  on disk     │
└────────────┘           │  position: 0     │           └──────────────┘
                         │  mode: read      │
                         └──────────────────┘
```

`fopen()` calls `malloc()` internally to create the FILE struct. `fclose()` flushes any buffered data to disk, then frees that struct. This is why forgetting `fclose()` causes both a memory leak and potentially missing data.

### The 3 Pre-Opened Streams

Every C program starts with three files already open. You don't call `fopen()` for them:

```c
stdin   // keyboard input
stdout  // terminal output
stderr  // terminal error output (not buffered — always appears immediately)
```

```c
// These pairs are equivalent:
printf("hi");                 ≡  fprintf(stdout, "hi");
scanf("%d", &x);              ≡  fscanf(stdin,  "%d", &x);
```

---

## Part 4 — Opening & Closing Files

### Opening a File with fopen()

```c
FILE *fopen(const char *filename, const char *mode);
// Returns a FILE* on success, or NULL if it fails
```

**Always check if fopen() returned NULL.** If the file doesn't exist (for reading), or the path is wrong, or permissions are denied, `fopen()` returns `NULL`. Using a NULL pointer crashes your program.

```c
// gcc fopen_demo.c -o fopen_demo -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>

int main() {
    FILE *f = fopen("grades.txt", "r");

    // ❌ WRONG — never skip this check:
    // fprintf(f, "hello");  // if f is NULL, this crashes

    // ✅ CORRECT — always check:
    if (f == NULL) {
        perror("Could not open file");  // prints the reason why it failed
        return 1;
    }

    printf("File opened successfully!\n");
    fclose(f);   // always close what you open
    return 0;
}
```

`perror()` is your best friend here. It automatically prints a human-readable explanation of why the open failed, like "No such file or directory" or "Permission denied".

### Closing a File with fclose()

```c
int fclose(FILE *stream);
// Returns 0 on success, EOF on error
```

`fclose()` does three things at once:
1. Flushes any buffered data to disk
2. Frees the FILE struct from memory
3. Releases the OS file descriptor

```c
// ✅ Always close your files:
FILE *f = fopen("output.txt", "w");
if (!f) { perror("Error"); return 1; }

fprintf(f, "Important data\n");
fclose(f);   // ← data is now guaranteed to be on disk
f = NULL;    // good habit: prevents accidentally using f after close
```

### Flushing Without Closing: fflush()

`fflush()` forces buffered data to be sent to the OS without closing the file. Use it when you need to write a log entry immediately, or when a prompt isn't appearing before `scanf`:

```c
printf("Enter your name: ");
fflush(stdout);    // force the prompt to appear NOW before waiting for input
scanf("%49s", name);
```

> ⚠️ **Common Mistake:** `fflush(stdin)` is undefined behavior in standard C. Never use it to clear input. To discard leftover input, use this instead:
> ```c
> int ch;
> while ((ch = getchar()) != '\n' && ch != EOF);
> ```

### ✏️ Practice — Part 4

1. Open a file called `"test.txt"` for reading. Check if it opened. Print a message and exit gracefully if it didn't.
2. Open a file for writing, write your name to it, then close it. Open it again for reading and print what's in it.

---

## Part 5 — File Modes

Think of modes like permissions: what are you allowed to do with this file, and what happens to existing content?

```
"r"   Read only. File must already exist. Cursor starts at beginning.
"w"   Write only. WIPES the file if it exists, or creates it fresh.
"a"   Append only. Writes go to end. Existing content is preserved.
"r+"  Read and write. File must exist. Cursor at beginning.
"w+"  Read and write. WIPES if exists, or creates fresh.
"a+"  Read (anywhere) and write (always at end).

Adding "b" to any mode (e.g. "rb", "wb") means binary mode.
Use binary mode whenever you're working with structs or raw data.
```

| Mode  | Read | Write | File must exist? | Wipes existing? |
|-------|:----:|:-----:|:----------------:|:---------------:|
| `"r"` | ✅ | ❌ | Yes | No |
| `"w"` | ❌ | ✅ | No (creates) | **Yes** ⚠️ |
| `"a"` | ❌ | ✅ | No (creates) | No |
| `"r+"` | ✅ | ✅ | Yes | No |
| `"rb"` | ✅ | ❌ | Yes | No |
| `"wb"` | ❌ | ✅ | No (creates) | **Yes** ⚠️ |

**Choosing a mode:**
```c
fopen("settings.cfg", "r")    // reading a config — must already exist
fopen("report.txt",   "w")    // fresh output every run
fopen("app.log",      "a")    // add log entries, keep old ones
fopen("records.bin",  "rb")   // read struct data
fopen("records.bin",  "wb")   // write struct data
fopen("records.bin",  "r+b")  // update records in-place
```

> ⚠️ **Warning:** `"w"` silently destroys all existing content with no warning. If you meant `"a"`, this is catastrophic. Double-check before using `"w"` on an important file.

---

## Part 6 — Writing to Files

Four functions for writing. Pick the one that matches your data:

| Function | Best for | Example |
|----------|----------|---------|
| `fprintf` | Formatted text (numbers, strings) | `fprintf(f, "%s %d\n", name, score)` |
| `fputs` | A plain string | `fputs("hello\n", f)` |
| `fputc` | One character at a time | `fputc('A', f)` |
| `fwrite` | Raw bytes / structs (binary) | `fwrite(&s, sizeof(s), 1, f)` |

### fprintf — Formatted Write

`fprintf` is exactly like `printf`, but the first argument is a file:

```c
// gcc write_demo.c -o write_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *f = fopen("grades.txt", "w");
    if (!f) { perror("Error"); return 1; }

    // Write formatted data — just like printf, but to a file
    fprintf(f, "%-15s %5.1f\n", "Alice", 92.5f);
    fprintf(f, "%-15s %5.1f\n", "Bob",   78.0f);
    fprintf(f, "%-15s %5.1f\n", "Carol", 88.5f);

    fclose(f);
    printf("Done. Check grades.txt\n");
    return 0;
}
```

File contents of `grades.txt`:
```
Alice            92.5
Bob              78.0
Carol            88.5
```

### fputs and fputc

```c
// fputs writes a string (no automatic newline — add \n yourself)
fputs("Line one\n", f);

// fputc writes one character at a time
for (char c = 'A'; c <= 'Z'; c++) {
    fputc(c, f);
}
fputc('\n', f);
```

### fwrite — Binary Write

`fwrite` copies raw bytes directly to disk. It's used for structs and binary data:

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
        {3, "Carol", 88.5f}
    };

    FILE *f = fopen("students.bin", "wb");
    if (!f) { perror("Error"); return 1; }

    // Write all 3 structs at once
    // fwrite(pointer to data, size of each item, number of items, file)
    size_t written = fwrite(students, sizeof(Student), 3, f);

    if (written != 3) {
        fprintf(stderr, "Error: only wrote %zu of 3 students\n", written);
    }
    printf("Wrote %zu students\n", written);

    fclose(f);
    return 0;
}
```

The binary file stores the raw bytes of each struct packed together — no commas, no formatting, just bytes. This makes it fast and easy to read back with `fread`.

### ✏️ Practice — Part 6

1. Open a file and write 5 lines, each containing a number from 1 to 5 and the word "hello".
2. Create a struct for a `Book` (title, author, year). Write 3 books to a binary file using `fwrite`.

---

## Part 7 — Reading from Files

The mirror of Part 6. Four functions for reading:

| Function | Best for |
|----------|----------|
| `fgets` | Reading one line of text (safe) |
| `fscanf` | Reading formatted text (numbers, words) |
| `fgetc` | One character at a time |
| `fread` | Raw bytes / structs (binary) |

### fgets — Read a Line of Text

`fgets` is the safest way to read a line. It reads at most `size-1` characters and always adds a null terminator:

```c
// gcc fgets_demo.c -o fgets_demo -Wall -Wextra -std=c11
#include <stdio.h>
#include <string.h>

int main() {
    FILE *f = fopen("grades.txt", "r");
    if (!f) { perror("Error"); return 1; }

    char line[100];   // buffer to hold each line

    // fgets returns the buffer on success, NULL when there's nothing left to read
    while (fgets(line, sizeof(line), f) != NULL) {
        // fgets includes the '\n' at the end — strip it if needed:
        line[strcspn(line, "\n")] = '\0';   // ← safe way to remove newline

        printf("Read: [%s]\n", line);
    }

    fclose(f);
    return 0;
}
```

> ⚠️ **Common Mistake:** `fgets` keeps the `\n` at the end of the string. If you compare or print the line, you'll get unexpected results. Always strip it with `line[strcspn(line, "\n")] = '\0';`

### fscanf — Read Formatted Data

`fscanf` reads data matching a format — like `scanf`, but from a file:

```c
// gcc fscanf_demo.c -o fscanf_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    FILE *f = fopen("grades.txt", "r");
    if (!f) { perror("Error"); return 1; }

    char name[50];
    float grade;

    // Read pairs of (name, grade) until end of file
    while (fscanf(f, "%49s %f", name, &grade) == 2) {  // ← == 2 means both were read
        printf("Name: %-15s Grade: %.1f\n", name, grade);
    }

    fclose(f);
    return 0;
}
```

**When to use fscanf vs fgets?** Use `fgets` for whole lines (especially if they have spaces). Use `fscanf` for structured data with known formats (like "123 Alice 3.9").

### fgetc — Read One Character

```c
int c;
while ((c = fgetc(f)) != EOF) {   // fgetc returns int, not char — important!
    putchar(c);                    // echo to terminal
}
```

> ⚠️ **Common Mistake:** Storing the result of `fgetc` in a `char` instead of `int`. `EOF` is -1, but an `unsigned char` can never be -1, so the loop never ends. Always use `int c = fgetc(f)`.

### fread — Binary Read

The exact reverse of `fwrite`:

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
    // fread(where to put the data, size of each item, number of items, file)
    while (fread(&s, sizeof(Student), 1, f) == 1) {  // read one struct at a time
        printf("[%d] %-15s %.1f\n", s.id, s.name, s.grade);
    }

    fclose(f);
    return 0;
}
```

### How to Detect End of File

```c
// ✅ CORRECT — check the return value of the read function:
while (fgets(line, sizeof(line), f) != NULL) { ... }
while (fread(&s, sizeof(s), 1, f) == 1) { ... }
while ((c = fgetc(f)) != EOF) { ... }

// ❌ WRONG — never do this:
while (!feof(f)) {
    fgets(line, sizeof(line), f);
    // Problem: feof only becomes true AFTER you've already read past the end
    // This processes the last line twice!
}
```

### ✏️ Practice — Part 7

1. Read a file line by line and print the line number before each line.
2. Read the binary file of books you created in Part 6 Practice, and print each book's details.
3. Count how many characters and how many lines are in a text file.

---

## Part 8 — Moving Around in a File

### The File Position Cursor

Think of a file like a tape. There's a cursor (a position marker) that keeps track of where you are. Every read or write moves the cursor forward. `fseek`, `ftell`, and `rewind` let you move the cursor wherever you want.

```c
fseek(f, offset, from_where);   // move the cursor
ftell(f);                       // ask "where is the cursor right now?"
rewind(f);                      // go back to the very beginning
```

For `from_where`, use one of these three constants:

| Constant | Meaning |
|----------|---------|
| `SEEK_SET` | From the beginning of the file |
| `SEEK_CUR` | From the current position |
| `SEEK_END` | From the end of the file |

```c
// gcc fseek_demo.c -o fseek_demo -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int main() {
    FILE *f = fopen("students.bin", "rb");
    if (!f) { perror("Error"); return 1; }

    // How many students are in the file? (without reading them all)
    fseek(f, 0, SEEK_END);                       // jump to end
    long fileSize = ftell(f);                    // how many bytes?
    int count = (int)(fileSize / sizeof(Student)); // how many structs fit?
    printf("File has %d students\n", count);

    // Read the 2nd student directly (index 1), without reading index 0
    fseek(f, 1 * sizeof(Student), SEEK_SET);     // jump to record 1
    Student s;
    fread(&s, sizeof(Student), 1, f);
    printf("2nd student: [%d] %s %.1f\n", s.id, s.name, s.grade);

    rewind(f);   // go back to start
    fclose(f);
    return 0;
}
```

**Why this matters for binary files:** Because every struct is the same size, you can jump to any record instantly. Record N starts at byte `N * sizeof(Student)`. This is called **random access** — you can reach any record in the same amount of time, no matter how large the file is.

### ✏️ Practice — Part 8

1. Open a binary file, seek to the last record, read it, and print it.
2. Find the size of a file in bytes using `fseek` and `ftell`.

---

## Part 9 — Binary Files & Structs

### Why Binary?

You have two options for saving data: text files or binary files.

| | Text File | Binary File |
|---|---|---|
| Human readable | ✅ Yes | ❌ No |
| Slower to read/write | Yes (parse each value) | No (raw copy) |
| Supports random access | Hard | Easy ✅ |
| Use when | Humans need to read it | Performance and direct access matter |

For a student database, a config file, or a log — use text. For a game save, a records database, or any program reading/writing lots of data — use binary.

### Writing and Reading Structs

```c
// gcc binary_demo.c -o binary_demo -Wall -Wextra -std=c11
#include <stdio.h>
#include <string.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int main() {
    // --- WRITE ---
    Student students[3] = {
        {1, "Alice", 92.5f},
        {2, "Bob",   78.0f},
        {3, "Carol", 88.5f}
    };

    FILE *fw = fopen("students.bin", "wb");
    if (!fw) { perror("Write error"); return 1; }
    fwrite(students, sizeof(Student), 3, fw);
    fclose(fw);
    printf("Written 3 students to binary file\n");

    // --- READ BACK ---
    FILE *fr = fopen("students.bin", "rb");
    if (!fr) { perror("Read error"); return 1; }

    Student s;
    printf("\nReading back:\n");
    while (fread(&s, sizeof(Student), 1, fr) == 1) {
        printf("  [%d] %-15s Grade: %.1f\n", s.id, s.name, s.grade);
    }
    fclose(fr);

    return 0;
}
```

### Updating a Single Record In-Place

Use `"r+b"` to update one record without rewriting the whole file:

```c
// gcc update_record.c -o update_record -Wall -Wextra -std=c11
#include <stdio.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int main() {
    FILE *f = fopen("students.bin", "r+b");
    if (!f) { perror("Error"); return 1; }

    // Update the grade of the 2nd student (index 1):
    int index = 1;
    fseek(f, index * sizeof(Student), SEEK_SET);  // jump to record 1

    Student s;
    fread(&s, sizeof(Student), 1, f);    // read it
    printf("Before: %s — %.1f\n", s.name, s.grade);

    s.grade = 95.0f;                     // change the grade

    fseek(f, index * sizeof(Student), SEEK_SET);  // jump back to same spot
    fwrite(&s, sizeof(Student), 1, f);   // write it back

    printf("After:  %s — %.1f\n", s.name, s.grade);
    fclose(f);
    return 0;
}
```

> ⚠️ **Common Mistake:** After reading with `fread`, if you want to write to the same position, you must `fseek` back first. The cursor moved forward when you read.

### ✏️ Practice — Part 9

1. Create a struct for a `Product` (id, name, price). Write 4 products to a binary file, read them all back, and print them.
2. Update the price of the 3rd product in the binary file without rewriting all records.

---

## Part 10 — Dynamic Memory + File I/O

### The Problem with Fixed-Size Arrays

```c
Student students[100];   // What if the file has 5? (waste 95 slots)
                         // What if it has 500? (overflow — crash!)
```

The solution: use `malloc` to create an array that's exactly the right size at runtime.

### Quick malloc/calloc/realloc/free Review

```c
// malloc — allocate memory (not initialized — contains garbage)
int *arr = malloc(5 * sizeof(int));
if (arr == NULL) { perror("malloc failed"); return 1; }

// calloc — allocate memory AND zero it out
int *arr = calloc(5, sizeof(int));
if (arr == NULL) { perror("calloc failed"); return 1; }

// realloc — grow or shrink existing allocation
int *temp = realloc(arr, 10 * sizeof(int));  // always use a temp variable!
if (temp == NULL) {
    // arr is still valid — free it before returning
    free(arr);
    return 1;
}
arr = temp;   // only update arr after confirming success

// free — release memory when done
free(arr);
arr = NULL;   // prevent using a dangling pointer by accident
```

> ⚠️ **Common Mistake:** Writing `arr = realloc(arr, newSize)` directly. If `realloc` fails, it returns NULL and `arr` loses its old value — you've now leaked memory. Always use a temp pointer.

### The Pattern: Load a Whole File Dynamically

```c
// gcc dynamic_loader.c -o dynamic_loader -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

// Load all students from a file. Returns a malloc'd array. Caller must free().
// Sets *count to the number of students loaded.
Student *loadAll(const char *filename, int *count) {
    *count = 0;

    FILE *f = fopen(filename, "rb");
    if (!f) { perror("Cannot open file"); return NULL; }

    // Step 1: Find out how many records are in the file
    fseek(f, 0, SEEK_END);
    long fileSize = ftell(f);
    rewind(f);
    *count = (int)(fileSize / sizeof(Student));

    if (*count == 0) { fclose(f); return NULL; }

    // Step 2: Allocate exactly the right amount of memory
    Student *arr = malloc((size_t)*count * sizeof(Student));
    if (!arr) { perror("malloc failed"); fclose(f); *count = 0; return NULL; }

    // Step 3: Read all records at once
    fread(arr, sizeof(Student), (size_t)*count, f);
    fclose(f);

    return arr;  // caller is responsible for free()-ing this
}

int main() {
    // First create a test file
    Student src[4] = {
        {1, "Alice", 92.5f},
        {2, "Bob",   78.0f},
        {3, "Carol", 88.5f},
        {4, "David", 95.0f}
    };
    FILE *f = fopen("students.bin", "wb");
    if (!f) { perror("Error"); return 1; }
    fwrite(src, sizeof(Student), 4, f);
    fclose(f);

    // Now load dynamically
    int count = 0;
    Student *db = loadAll("students.bin", &count);
    if (!db) { printf("No data loaded.\n"); return 1; }

    printf("Loaded %d students:\n", count);
    for (int i = 0; i < count; i++) {
        printf("  [%d] %-15s %.1f\n", db[i].id, db[i].name, db[i].grade);
    }

    free(db);    // always free what you malloc
    db = NULL;
    return 0;
}
```

### Growing a Database Dynamically with realloc

```c
// gcc dynamic_append.c -o dynamic_append -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

// Add a record to the array. Returns the (possibly new) pointer.
// Uses a doubling strategy: capacity grows as 4 → 8 → 16 → ...
Student *addRecord(Student *db, int *count, int *capacity, Student newEntry) {
    if (*count >= *capacity) {
        *capacity = (*capacity == 0) ? 4 : *capacity * 2;
        Student *temp = realloc(db, (size_t)*capacity * sizeof(Student));
        if (!temp) { perror("realloc failed"); return db; }
        db = temp;
        printf("  [grew capacity to %d]\n", *capacity);
    }
    db[(*count)++] = newEntry;
    return db;   // must reassign — address may have changed after realloc
}

int main() {
    Student *db = NULL;
    int count = 0, capacity = 0;

    db = addRecord(db, &count, &capacity, (Student){1, "Alice", 92.5f});
    db = addRecord(db, &count, &capacity, (Student){2, "Bob",   78.0f});
    db = addRecord(db, &count, &capacity, (Student){3, "Carol", 88.5f});
    db = addRecord(db, &count, &capacity, (Student){4, "David", 95.0f});
    db = addRecord(db, &count, &capacity, (Student){5, "Eve",   91.0f});

    printf("\nDatabase (%d students):\n", count);
    for (int i = 0; i < count; i++) {
        printf("  [%d] %s — %.1f\n", db[i].id, db[i].name, db[i].grade);
    }

    // Save to disk
    FILE *f = fopen("saved.bin", "wb");
    if (f) { fwrite(db, sizeof(Student), count, f); fclose(f); }
    printf("Saved to saved.bin\n");

    free(db);
    db = NULL;
    return 0;
}
```

### ✏️ Practice — Part 10

1. Write a function that loads all integers from a text file into a dynamically allocated array. The function should count them first, malloc the right size, then fill the array.
2. Extend the dynamic student database to support deleting a student by ID (shift remaining records left, decrement count, then save the file).

---

## Part 11 — Advanced Patterns

### Using Function Pointers for Generic File Processing

```c
// Process every record in a file with a user-supplied function
void processAll(const char *filename, void (*handler)(Student *)) {
    FILE *f = fopen(filename, "rb");
    if (!f) { perror("Error"); return; }

    Student s;
    while (fread(&s, sizeof(Student), 1, f) == 1) {
        handler(&s);   // call the provided function on each record
    }
    fclose(f);
}

void printStudent(Student *s) {
    printf("[%d] %-15s %.1f\n", s->id, s->name, s->grade);
}

// Usage:
processAll("students.bin", printStudent);
```

### Passing FILE * to Functions

Pass `FILE *` like any other pointer — functions can read or write through it:

```c
// Write a separator line to any open file
void writeSeparator(FILE *f) {
    fprintf(f, "----------------------------\n");
}

// Usage:
writeSeparator(stdout);         // write to terminal
writeSeparator(reportFile);     // write to a file
```

---

## Part 12 — Error Handling

Good programs don't crash when something goes wrong — they explain what went wrong and clean up properly.

### The Four Error Tools

```c
// 1. perror() — print human-readable error for the last operation that failed
FILE *f = fopen("missing.txt", "r");
if (!f) {
    perror("Could not open file");
    // Prints: "Could not open file: No such file or directory"
    return 1;
}

// 2. errno — the error code number (set by the OS after any failed system call)
#include <errno.h>
if (!f) {
    printf("Error code: %d\n", errno);
    printf("Error text: %s\n", strerror(errno));
}

// 3. ferror() — check if an error occurred on an open file
if (ferror(f)) {
    fprintf(stderr, "A read/write error occurred\n");
    clearerr(f);  // reset the error flag
}

// 4. feof() — check if you've reached the end of the file
if (feof(f)) {
    printf("Reached end of file\n");
}
```

### A Reliable Error-Handling Pattern

```c
// gcc error_handling.c -o error_handling -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int saveRecord(const char *filename, Student *s) {
    FILE *f = fopen(filename, "ab");
    if (!f) {
        perror("saveRecord: fopen failed");
        return -1;   // error code
    }

    size_t written = fwrite(s, sizeof(Student), 1, f);
    if (written != 1) {
        fprintf(stderr, "saveRecord: write failed (disk full?)\n");
        fclose(f);
        return -2;
    }

    if (fclose(f) != 0) {
        perror("saveRecord: fclose failed");
        return -3;
    }

    return 0;  // success
}

int main() {
    Student s = {1, "Alice", 92.5f};

    int result = saveRecord("students.bin", &s);
    if (result != 0) {
        fprintf(stderr, "Failed to save record (error %d)\n", result);
        return 1;
    }

    printf("Saved successfully.\n");
    return 0;
}
```

### ✏️ Practice — Part 12

1. Write a function that opens a file, reads a number from it, and returns -1 with a message if anything fails.
2. Write a function that writes to a file and checks the return value of `fwrite`. Print an error if not all bytes were written.

---

## Part 13 — File Utility Functions

### Rename, Delete, and Temporary Files

```c
#include <stdio.h>

// Rename a file
int result = rename("old_name.txt", "new_name.txt");
if (result != 0) { perror("rename failed"); }

// Delete a file
result = remove("unwanted.txt");
if (result != 0) { perror("remove failed"); }

// Create a temporary file (auto-deleted when closed or program exits)
FILE *tmp = tmpfile();
if (!tmp) { perror("tmpfile failed"); return 1; }
fprintf(tmp, "temporary data");
// No need to clean up — it's deleted automatically
fclose(tmp);
```

### The Safe Update Pattern (Edit Without Data Loss)

Never modify an important file directly — if your program crashes mid-write, you corrupt the file. Instead, write to a temp file and rename:

```c
// gcc safe_update.c -o safe_update -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
    // Write to a temp file first
    FILE *tmp = fopen("grades_tmp.txt", "w");
    if (!tmp) { perror("Error"); return 1; }

    fprintf(tmp, "Alice  95.0\n");
    fprintf(tmp, "Bob    82.0\n");
    fclose(tmp);

    // Only replace the real file after temp write succeeds
    remove("grades.txt");                      // remove old
    rename("grades_tmp.txt", "grades.txt");    // rename temp to real ← KEY LINE

    printf("File updated safely.\n");
    return 0;
}
```

---

## Part 14 — CSV Files

CSV (Comma-Separated Values) is a text format where each line is a record and values are separated by commas. Example: `1,Alice,92.5`

```c
// gcc csv_demo.c -o csv_demo -Wall -Wextra -std=c11
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

int main() {
    // --- WRITE CSV ---
    FILE *fw = fopen("students.csv", "w");
    if (!fw) { perror("Error"); return 1; }

    fprintf(fw, "id,name,grade\n");  // header line
    fprintf(fw, "1,Alice,92.5\n");
    fprintf(fw, "2,Bob,78.0\n");
    fprintf(fw, "3,Carol,88.5\n");
    fclose(fw);

    // --- READ CSV ---
    FILE *fr = fopen("students.csv", "r");
    if (!fr) { perror("Error"); return 1; }

    char line[100];
    fgets(line, sizeof(line), fr);  // skip header

    printf("Reading students:\n");
    while (fgets(line, sizeof(line), fr) != NULL) {
        line[strcspn(line, "\r\n")] = '\0';  // strip newline (handles \r\n too)

        // strtok splits the line by "," — modifies line in place
        char *token = strtok(line, ",");     // first token: id
        int id = atoi(token);

        token = strtok(NULL, ",");           // second token: name
        char name[50];
        strncpy(name, token, sizeof(name) - 1);
        name[sizeof(name) - 1] = '\0';

        token = strtok(NULL, ",");           // third token: grade
        float grade = (float)atof(token);

        printf("  [%d] %-15s %.1f\n", id, name, grade);
    }

    fclose(fr);
    return 0;
}
```

> ⚠️ **Common Mistake:** `strtok` modifies the string it's working on. If you need the original string later, make a copy with `strcpy` before calling `strtok`.

---

## Part 15 — Safe String Handling

### Buffer Overflow — What It Is and Why It Matters

A buffer overflow happens when you write more characters than a buffer can hold. It corrupts memory silently and can crash your program or create security vulnerabilities.

```c
// ❌ WRONG — gets() is banned in C11 (removed from the standard):
char name[50];
gets(name);           // reads with no limit — overflows if input > 49 chars

// ❌ WRONG — scanf with %s has no limit:
scanf("%s", name);    // same problem

// ✅ CORRECT — fgets with a size limit:
fgets(name, sizeof(name), stdin);   // safe: reads at most 49 chars

// ✅ CORRECT — scanf with a width limit:
scanf("%49s", name);   // safe: reads at most 49 chars (one less than buffer)

// ✅ CORRECT — strncpy instead of strcpy:
strncpy(dest, src, sizeof(dest) - 1);   // copies at most N-1 chars
dest[sizeof(dest) - 1] = '\0';          // always null-terminate manually
```

### Removing the Newline from fgets

`fgets` keeps the `\n` at the end. This is the recommended safe way to remove it:

```c
char line[100];
fgets(line, sizeof(line), f);
line[strcspn(line, "\r\n")] = '\0';  // removes \n on Linux and \r\n on Windows
```

---

## Part 16 — stdin, stdout, stderr

### The Three Standard Streams

```c
stdin   // file descriptor 0 — keyboard input (or piped input)
stdout  // file descriptor 1 — terminal output (or piped output)
stderr  // file descriptor 2 — terminal error (always goes to terminal)
```

`stdout` is line-buffered (flushes when `\n` is printed, or when buffer fills). `stderr` is unbuffered (every write goes through immediately). That's why error messages always appear right away.

### Redirecting Streams from the Terminal

```bash
./program < input.txt         # feed input.txt into stdin
./program > output.txt        # send stdout to output.txt
./program 2> errors.txt       # send stderr to errors.txt
./program > out.txt 2>&1      # send both stdout and stderr to out.txt
```

Your program doesn't need to change at all — `printf` and `scanf` automatically use whatever `stdout` and `stdin` are connected to.

```c
// gcc streams_demo.c -o streams_demo -Wall -Wextra -std=c11
#include <stdio.h>

int main() {
    // Normal output — redirectable
    printf("This is regular output\n");

    // Error output — always goes to terminal, even if stdout is redirected
    fprintf(stderr, "Error: something went wrong\n");

    return 0;
}
```

---

## Part 17 — How the OS Handles Files

### File Descriptors

The OS identifies every open file with a small integer called a **file descriptor** (fd). Your C `FILE *` is a wrapper around this. When you call `fopen()`, the OS creates an entry in a table for your process and gives back an fd:

- `0` → stdin
- `1` → stdout
- `2` → stderr
- `3` → your first `fopen()` file
- `4` → your second `fopen()` file
- ...

Each process can have a limited number of open files at once (typically around 1024). If you keep opening files without closing them, you'll eventually hit this limit and `fopen()` will return NULL.

### Why Buffering Exists

Reading and writing disk storage is slow compared to RAM. The C library batches your small `fprintf` calls into a larger chunk, then writes that chunk to disk in one go — much more efficient. This is why data doesn't always appear on disk immediately.

**Three buffering modes:**

| Mode | When data is written |
|------|---------------------|
| Full buffering (`_IOFBF`) | When buffer is full (~4KB). Default for files. |
| Line buffering (`_IOLBF`) | When a `\n` is encountered. Default for stdout to terminal. |
| Unbuffered (`_IONBF`) | Immediately. Default for stderr. |

You can control this with `setvbuf()`, but for most programs the defaults are fine.

---

## Part 18 — 10 Common Mistakes

### Mistake 1: Not Checking if fopen Returned NULL

```c
// ❌ WRONG:
FILE *f = fopen("data.txt", "r");
fgets(line, sizeof(line), f);  // crash if f is NULL

// ✅ CORRECT:
FILE *f = fopen("data.txt", "r");
if (!f) { perror("Cannot open file"); return 1; }
```

### Mistake 2: Forgetting fclose

```c
// ❌ WRONG:
FILE *f = fopen("log.txt", "w");
fprintf(f, "entry\n");
return 0;   // data may never reach disk!

// ✅ CORRECT:
fclose(f);  // always close before returning
return 0;
```

### Mistake 3: Using "w" When You Meant "a"

```c
// ❌ WRONG — destroys all existing log entries:
FILE *log = fopen("app.log", "w");

// ✅ CORRECT — adds to existing entries:
FILE *log = fopen("app.log", "a");
```

### Mistake 4: Storing fgetc Result in char Instead of int

```c
// ❌ WRONG — loop may never end on some platforms:
char c;
while ((c = fgetc(f)) != EOF) { ... }

// ✅ CORRECT:
int c;
while ((c = fgetc(f)) != EOF) { ... }
```

### Mistake 5: Using while(!feof(f)) to Read

```c
// ❌ WRONG — processes the last item twice:
while (!feof(f)) {
    fread(&s, sizeof(s), 1, f);
    print(s);
}

// ✅ CORRECT:
while (fread(&s, sizeof(s), 1, f) == 1) {
    print(s);
}
```

### Mistake 6: Not Stripping \n from fgets

```c
// ❌ WRONG — "Alice\n" != "Alice":
fgets(name, sizeof(name), f);
if (strcmp(name, "Alice") == 0) { ... }  // never matches!

// ✅ CORRECT:
fgets(name, sizeof(name), f);
name[strcspn(name, "\n")] = '\0';
if (strcmp(name, "Alice") == 0) { ... }  // works correctly
```

### Mistake 7: Writing arr = realloc(arr, ...) Directly

```c
// ❌ WRONG — if realloc fails, arr becomes NULL and you leak memory:
arr = realloc(arr, newSize);

// ✅ CORRECT:
int *temp = realloc(arr, newSize);
if (!temp) { free(arr); return 1; }
arr = temp;
```

### Mistake 8: Using fflush(stdin)

```c
// ❌ WRONG — undefined behavior in standard C:
fflush(stdin);

// ✅ CORRECT — discard leftover input characters:
int ch;
while ((ch = getchar()) != '\n' && ch != EOF);
```

### Mistake 9: Using fread/fwrite Without Checking the Return Value

```c
// ❌ WRONG — assumes write always succeeds:
fwrite(&s, sizeof(s), 1, f);

// ✅ CORRECT:
if (fwrite(&s, sizeof(s), 1, f) != 1) {
    fprintf(stderr, "Write failed (disk full?)\n");
}
```

### Mistake 10: Using a FILE* After fclose

```c
// ❌ WRONG — undefined behavior:
fclose(f);
fprintf(f, "oops");

// ✅ CORRECT:
fclose(f);
f = NULL;  // any accidental use will now segfault clearly, not silently corrupt
```

---

## Part 19 — Real-World Project: Student Database

This project puts everything together: binary files, dynamic memory, structs, and error handling. It supports adding, listing, searching, and deleting students.

```c
// gcc student_db.c -o student_db -Wall -Wextra -std=c11
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FILENAME "students.bin"

typedef struct {
    int   id;
    char  name[50];
    float grade;
} Student;

// Load all students from file into a dynamically allocated array.
// Returns pointer (caller must free). Sets *count.
Student *loadAll(int *count) {
    *count = 0;
    FILE *f = fopen(FILENAME, "rb");
    if (!f) return NULL;  // file doesn't exist yet — that's OK

    fseek(f, 0, SEEK_END);
    long size = ftell(f);
    rewind(f);

    *count = (int)(size / sizeof(Student));
    if (*count == 0) { fclose(f); return NULL; }

    Student *arr = malloc((size_t)*count * sizeof(Student));
    if (!arr) { perror("malloc failed"); fclose(f); *count = 0; return NULL; }

    fread(arr, sizeof(Student), (size_t)*count, f);
    fclose(f);
    return arr;
}

// Save the entire array to the file.
void saveAll(Student *arr, int count) {
    FILE *f = fopen(FILENAME, "wb");
    if (!f) { perror("Cannot save"); return; }
    fwrite(arr, sizeof(Student), (size_t)count, f);
    fclose(f);
}

// Add a student.
void addStudent(void) {
    int count = 0;
    Student *arr = loadAll(&count);

    // Resize array by 1
    Student *temp = realloc(arr, (size_t)(count + 1) * sizeof(Student));
    if (!temp) { perror("realloc failed"); free(arr); return; }
    arr = temp;

    Student *s = &arr[count];
    s->id = count + 1;
    printf("Name: "); scanf("%49s", s->name);
    printf("Grade: "); scanf("%f", &s->grade);
    count++;

    saveAll(arr, count);
    printf("Student added (ID %d).\n", s->id);
    free(arr);
}

// List all students.
void listStudents(void) {
    int count = 0;
    Student *arr = loadAll(&count);

    if (!arr || count == 0) {
        printf("No students found.\n");
        free(arr);
        return;
    }

    printf("\n%-5s %-20s %s\n", "ID", "Name", "Grade");
    printf("-------------------------------\n");
    for (int i = 0; i < count; i++) {
        printf("%-5d %-20s %.1f\n", arr[i].id, arr[i].name, arr[i].grade);
    }
    printf("Total: %d students\n\n", count);

    free(arr);
}

// Search by name.
void searchByName(const char *query) {
    int count = 0;
    Student *arr = loadAll(&count);

    int found = 0;
    for (int i = 0; i < count; i++) {
        if (strstr(arr[i].name, query) != NULL) {
            printf("Found: [%d] %s — %.1f\n", arr[i].id, arr[i].name, arr[i].grade);
            found++;
        }
    }
    if (!found) printf("No students matching '%s'\n", query);

    free(arr);
}

// Delete by ID — shifts remaining records left.
void deleteById(int id) {
    int count = 0;
    Student *arr = loadAll(&count);

    int found = -1;
    for (int i = 0; i < count; i++) {
        if (arr[i].id == id) { found = i; break; }
    }

    if (found == -1) {
        printf("Student ID %d not found.\n", id);
        free(arr);
        return;
    }

    // Shift everything after 'found' one position to the left
    for (int i = found; i < count - 1; i++) {
        arr[i] = arr[i + 1];
    }
    count--;

    saveAll(arr, count);
    printf("Deleted student ID %d.\n", id);
    free(arr);
}

int main(void) {
    int choice;
    char name[50];
    int id;

    while (1) {
        printf("\n=== Student Database ===\n");
        printf("1. Add student\n");
        printf("2. List all\n");
        printf("3. Search by name\n");
        printf("4. Delete by ID\n");
        printf("5. Exit\n");
        printf("Choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: addStudent(); break;
            case 2: listStudents(); break;
            case 3:
                printf("Search name: "); scanf("%49s", name);
                searchByName(name);
                break;
            case 4:
                printf("Delete ID: "); scanf("%d", &id);
                deleteById(id);
                break;
            case 5: printf("Goodbye!\n"); return 0;
            default: printf("Invalid choice.\n");
        }
    }
}
```

---

## Part 20 — Practice Problems

### Beginner

1. Write a program that asks the user for 5 numbers, writes them to a file, then reads and prints them back.
2. Write a program that copies a text file. Open source as `"r"`, open destination as `"w"`, and copy character by character with `fgetc`/`fputc`.
3. Count the number of lines in a text file.

### Intermediate

4. Write a program that reads a text file and prints only the lines containing a given word (search from command-line argument: `./search "word" file.txt`).
5. Create a binary file of 10 integers. Then use `fseek` to read and print only the even-indexed ones (index 0, 2, 4, ...).
6. Write a log function `void logEvent(const char *msg)` that appends a message with a timestamp to `"app.log"`. Call it 5 times with different messages.

### Advanced

7. Write a CSV parser that reads a file with columns `id,name,grade`, stores all rows in a dynamically allocated array of structs, and then prints the student with the highest grade.
8. Write a program that reads a binary student file, sorts the array by grade (descending), and writes the sorted array back to a new file.
9. Implement a basic "mark deleted" system: instead of shifting records, add a `deleted` field to the struct. The delete function sets `deleted = 1`. The list function skips records where `deleted == 1`.

---

## Part 21 — Cheat Sheet

### Open / Close
```c
FILE *f = fopen("file.txt", "r");   // open
if (!f) { perror("Error"); return 1; }  // check
fclose(f);                          // close
fflush(f);                          // force write to OS without closing
```

### Write
```c
fprintf(f, "%s %d\n", str, num);    // formatted text
fputs("line\n", f);                 // plain string
fputc('A', f);                      // one character
fwrite(ptr, sizeof(T), count, f);   // raw binary
```

### Read
```c
fgets(buf, sizeof(buf), f);         // one line (safe)
fscanf(f, "%49s %f", str, &num);    // formatted (check return value)
int c = fgetc(f);                   // one character (must be int, not char)
fread(ptr, sizeof(T), count, f);    // raw binary
```

### File Position
```c
fseek(f, offset, SEEK_SET);         // from beginning
fseek(f, offset, SEEK_CUR);         // from current position
fseek(f, offset, SEEK_END);         // from end (use negative offset)
long pos = ftell(f);                // where am I?
rewind(f);                          // go to beginning
```

### Error Checking
```c
perror("message");                  // print error with OS explanation
ferror(f);                          // was there an error on this file?
feof(f);                            // are we at end of file?
```

### Modes Quick Reference
```
"r"   read text       "rb"   read binary
"w"   write text      "wb"   write binary   ← WIPES existing!
"a"   append text     "ab"   append binary
"r+"  read+write      "r+b"  read+write binary
```

### Dynamic Memory
```c
T *arr = malloc(n * sizeof(T));     // allocate (not zeroed)
T *arr = calloc(n, sizeof(T));      // allocate (zeroed)
T *tmp = realloc(arr, newN * sizeof(T));  // resize (use temp!)
if (!tmp) { free(arr); return 1; }
arr = tmp;
free(arr); arr = NULL;              // always set to NULL after free
```

---

## Part 22 — FAQ

**Q: Can I have multiple files open at the same time?**
Yes. Use a separate `FILE *` for each one. Always `fclose()` each when done — there's a limit of about 1024 open files per process.

**Q: What's the difference between text mode and binary mode on Windows?**
In text mode, Windows automatically translates `\n` to `\r\n` when writing, and back when reading. Binary mode skips this. Always use binary mode (`"rb"`, `"wb"`) for structs — it behaves identically on all platforms.

**Q: Is fgets safe from buffer overflows?**
Yes. You pass the buffer size, so it will never write more than `size-1` characters plus a null terminator. It's the safe replacement for the banned `gets()`.

**Q: Why must fgetc return int, not char?**
`EOF` is typically -1. If `char` is unsigned on your platform, it holds 0–255 and can never equal -1, so your loop never ends. `int` holds both 0–255 and -1.

**Q: What happens if fwrite partially writes?**
It returns the number of elements written. If it's less than you requested, check `ferror()`. Common cause: disk full. Always check the return value.

**Q: Why use sizeof(MyStruct) instead of adding up the field sizes?**
The compiler may add "padding" bytes between fields to align them in memory. `sizeof` gives you the true size including padding. Always use `sizeof` — never hardcode byte counts.

**Q: Why set a pointer to NULL after free()?**
`free()` releases the memory but doesn't modify the pointer. The pointer still points to freed memory — called a "dangling pointer". If you accidentally use it, behavior is undefined. Setting to NULL means any accidental use causes an immediate, obvious crash instead of silent memory corruption.

**Q: How do I count records in a binary file without reading them all?**
```c
fseek(f, 0, SEEK_END);
long size = ftell(f);
int count = size / sizeof(MyStruct);
```
This is instant regardless of file size.

**Q: After free(), why set the pointer to NULL?**
`free()` doesn't modify the pointer — it just marks the heap memory as available. Setting to NULL prevents accidentally using a dangling pointer later.

---

## Part 23 — What to Learn Next

Once you're comfortable with File I/O in C, here's a practical order for what to learn next:

**Next steps (good for coursework):**
- Linked lists with file persistence — instead of fixed-size arrays, use linked lists and write each node to disk
- Error propagation — returning error codes through a chain of function calls
- Command-line arguments (`argc`, `argv`) to pass filenames to your program

**Intermediate (useful for projects):**
- POSIX file I/O — `open()`, `read()`, `write()` — the lower-level OS calls that `fopen` is built on
- File locking — safely sharing files between multiple programs running at the same time
- Memory-mapped files — mapping a file directly into memory so you can access it like an array

**Advanced (for systems programming):**
- SQLite in C — an embedded database stored in one file, with a clean C API
- Sockets and network I/O — the same read/write concepts applied to network connections
- Write-ahead logging — how databases like SQLite survive crashes without data loss

**Recommended books:**
- *The C Programming Language* — Kernighan & Ritchie (the original C reference)
- *C Programming: A Modern Approach* — K.N. King (the best textbook for learning C systematically)
- Beej's Guide to C Programming — free at beej.us/guide/bgc/ (excellent and practical)

---

*Made for students learning C for the first time — built on the fundamentals, aimed at real programs.*
