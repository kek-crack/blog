# Раскуриваю как работает nm и otool
## Будем препарировать эльфа

```c
#include <stdlib.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/mman.h>
#include <elf.h>
#include <string.h>

void nm(void *start)
{
    Elf64_Ehdr *ehdr = (Elf64_Ehdr *)start;
    Elf64_Shdr *shdr = (Elf64_Shdr *)(start + ehdr->e_shoff);

    for (size_t i = 0; i < ehdr->e_shnum; ++i)
    {
        if (shdr[i].sh_type == SHT_SYMTAB)
        {
            Elf64_Sym *symtab = (Elf64_Sym *)(start + shdr[i].sh_offset);
            size_t len = shdr[i].sh_size / shdr[i].sh_entsize;

            char *symbol_names = (char *)(start + shdr[shdr[i].sh_link].sh_offset);

            for (size_t j = 0; j < len; ++j)
            {
                if (ELF64_ST_TYPE(symtab[j].st_info) != STT_SECTION &&
                    ELF64_ST_TYPE(symtab[j].st_info) != STT_FILE)
                {
                    printf("%016lx\t%-64s\n", symtab[j].st_value, symbol_names + symtab[j].st_name);
                }
            }

            break;
        }
    }
}

void otool(char *start)
{
    Elf64_Ehdr *ehdr = (Elf64_Ehdr *)start;
    Elf64_Shdr *shdr = (Elf64_Shdr *)(start + ehdr->e_shoff);
    char *strtab = (char *)(start + shdr[ehdr->e_shstrndx].sh_offset);

    for (size_t i = 0; i < ehdr->e_shnum; ++i)
    {
        if (strcmp(&strtab[shdr[i].sh_name], ".text") == 0)
        {
            char *data = (char *)((uint64_t)start + shdr[i].sh_offset);

            for (size_t j = 0; j < shdr[i].sh_size; ++j)
            {
                if (!j)
                {
                    printf("%016lx\t", (uint64_t)&data[0]);
                }
                else if (j % 16 == 0)
                {
                    printf("\n%016lx\t", (uint64_t)&data[j + 1]);
                }
                printf("%02hhx ", data[j]);
            }
            printf("\n");

            break;
        }
    }
}

char is_elf(char *start)
{
    return (start[EI_MAG0] == ELFMAG0 &&
            start[EI_MAG1] == ELFMAG1 &&
            start[EI_MAG2] == ELFMAG2 &&
            start[EI_MAG3] == ELFMAG3);
}

void usage()
{
    puts("Usage ./a.out [ELF64 FILE] -[nm, otool]");
}

int main(int ac, char **av)
{
    int fd;
    struct stat buf;
    void *start;
 
    if (ac != 3)
    {
        usage();
        return (EXIT_FAILURE);
    }

    if ((fd = open(av[1], O_RDONLY)) < 0)
    {
        return (EXIT_FAILURE);
    }

    if ((fstat(fd, &buf)) < 0)
    {
        return (EXIT_FAILURE);
    }

    if ((start = mmap(0, buf.st_size, PROT_READ, MAP_PRIVATE, fd, 0)) == MAP_FAILED)
    {
        return (EXIT_FAILURE);
    }

    if (is_elf(start))
    {
        if (strcmp(av[2], "-nm") == 0)
        {
            nm(start);
        }
        else if (strcmp(av[2], "-otool") == 0)
        {
            printf("%s:\nContents of (__TEXT,__text) section\n", av[1]);
            otool(start);
        }
        else
        {
            usage();
        }
    }

    if (munmap(start, buf.st_size) < 0)
    {
        return (EXIT_FAILURE);
    }

    return (EXIT_SUCCESS);
}
```
