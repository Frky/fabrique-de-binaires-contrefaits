
# Fabrique de binaires contrefaits 

## Introduction
On se propose dans ce sujet de modifier des programmes binaires compilés afin d'y insérer un *payload*, de manière automatisée.

```
[Binaire innocent] --- (Malicious Binaries Factory) ---> [Binaire patché]
```

L'objectif est, en partant d'un binaire classique, d'en fabriquer un autre qui présente en apparences le même comportement que 
l'original, mais qui en plus exécute le payload.

### Programme cible
Notre cible est l'ensemble des programmes binaires compilés au format ELF 64. On ne supposera pas le code source disponible. 
En fait, à l'exception du format (ELF 64), aucun savoir (ou presque) sur la cible n'est nécessaire a priori. 

### Payloads
Les payloads que l'on injectera aux binaires cibles peuvent être de plusieurs natures. Par exemple, on peut vouloir 
ouvrir un *remote-shell* sur un port quelconque. Vous pouvez trouver tout un tas de payloads en assembleur sur le net. 
Vous pouvez également écrire les votres. C'est à vous de trouver dans quel format injecter vos payloads dans le binaire.  

Comme mentionné dans la suite, on pourra aussi utiliser la possiblité d'injecter un payload pour implémenter un packer complet. 

## Travail demandé

La recherche documentaire est laissée à vos soins.  

### Parser un binaire ELF 64

#### L'entête

Dans un premier temps, il faudra être capable de lire l'entête d'un fichier ELF 64. La documentation sur ce format est fournie 
en annexe. Les champs à lire sont les suivants :

```
struct Elf64_Ehdr_s {
    unsigned char   e_ident[EI_NIDENT];
    Elf64_Half  e_type;
    Elf64_Half  e_machine;
    Elf64_Word  e_version;
    Elf64_Addr  e_entry;
    Elf64_Off   e_phoff;
    Elf64_Off   e_shoff;
    Elf64_Word  e_flags;
    Elf64_Half  e_ehsize;
    Elf64_Half  e_phentsize;
    Elf64_Half  e_phnum;
    Elf64_Half  e_shentsize;
    Elf64_Half  e_shnum;
    Elf64_Half  e_shstrndx;
}; 
```

#### La table des sections

Une fois l'entête lue, on connaît l'adresse de début des entêtes de section. Il faut donc reconstruire la table des sections, en lisant chaque entête de section :

```
struct Elf64_Shdr_s {
    Elf64_Word sh_name;         /* Section name */
    Elf64_Word sh_type;         /* Section type */
    Elf64_Xword sh_flags;       /* Section attributes */
    Elf64_Addr sh_addr;         /* Virtual address in memory */
    Elf64_Off sh_offset;        /* Offset in file */
    Elf64_Xword sh_size;        /* Size of section */
    Elf64_Word sh_link;         /* Link to other section */
    Elf64_Word sh_info;         /* Miscellaneous information */
    Elf64_Xword sh_addralign;   /* Address alignment boundary */
    Elf64_Xword sh_entsize;     /* Size of entries, if section has table */
};
```

#### La table des segments

Idem pour la table des segments :

```
struct Elf64_Phdr_s {
    Elf64_Word p_type;          /* Type of segment */
    Elf64_Word p_flags;         /* Segment attributes */
    Elf64_Off p_offset;         /* Offset in file */
    Elf64_Addr p_vaddr;         /* Virtual address in memory */
    Elf64_Addr p_paddr;         /* Reserved */
    Elf64_Xword p_filesz;       /* Size of segment in file */
    Elf64_Xword p_memsz;        /* Size of segment in memory */
    Elf64_Xword p_align;        /* Alignment of segment */
};
```

#### La table des symboles

Idem pour la table des symboles :

```
struct Elf64_Sym_s {
    Elf64_Word st_name;         /* Symbol name */
    unsigned char st_info;      /* Type and Binding attributes */
    unsigned char st_other;     /* Reserved */
    Elf64_Half st_shndx;        /* Section table index */
    Elf64_Addr st_value;        /* Symbol value */
    Elf64_Xword st_size;        /* Size of object (e.g., common) */
};
```



#### L'affichage

Quelques exemples d'affichage (on s'inspirera largement de la sortie de readelf) :

```
$ ./elph -h ./elph 
[*] Open binary file
[*] Read ELF header
[*] Read section header table
[*] Read program header table
[*] Look for symbol table
[*] Read symbol table
[*] Look for dynamic symbol table
[*] Read dynamic symbol table
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x4009e0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          60736 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         8
  Size of section headers:           64 (bytes)
  Number of section headers:         35
  Section header string table index: 32

$ ./elph -l ./elph 
[*] Open binary file
[*] Read ELF header
[*] Read section header table
[*] Read program header table
[*] Look for symbol table
[*] Read symbol table
[*] Look for dynamic symbol table
[*] Read dynamic symbol table
There are 8 program headers, starting at offset 0x40:

Program Headers:
  Type            Off    VirtAddr PhysAddr FSize  MSize  Flg Al
  PHDR            000040 00400040 00400040 0001c0 0001c0 R E  8
  INTERP          000200 00400200 00400200 00001c 00001c R    1
  LOAD            000000 00400000 00400000 006114 006114 R E  0
  LOAD            006118 00606118 00606118 000408 000478 RW   0
  DYNAMIC         006130 00606130 00606130 0001d0 0001d0 RW   8
  NOTE            00021c 0040021c 0040021c 000044 000044 R    4
                  0053c4 004053c4 004053c4 00029c 00029c R    4
                  000000 00000000 00000000 000000 000000 RW  16
```

Un extrait des options attendues (d'autres seront les bienvenues) :

* `-h`: Informations sur l'entête du binaire (*eg* format du binaire, architecture, nombre de sections, etc.) 
* `-S`: Informations sur les entêtes de section  (section headers)
* `-s`: Afficher la table des symboles
* `-l`: Informations sur les entêtes de segments (program headers)


### Patcher un binaire ELF 64

Pour des raisons de simplicité, on fera en sorte que le payload injecté dans le binaire soit exécuté au tout début du lancement
du programme. Pour ce faire, on modifiera le point d'entrée du programme pour qu'il pointe sur notre payload. 
Afin que l'exécution se déroule correctement une fois le payload exécuté, il faudra ajouter un saut vers
le point d'entrée initial du programme à la fin du payload. 

#### Dans la section `NOTE`

Dans un premier temps, on remplacera la section `NOTE` (facultative) par une nouvelle section de code. En résumé, il faudra :

* Changer le type et les *flags" de la sectin `NOTE`
* Injecter le payload dans la section `NOTE`
* Ajouter un saut au point d'entrée initial du programme à la fin du payload
* Modifier le point d'entrée du programme vers le payload

#### Dans la section `CODE`
Cette partie est laissée à la charge de l'équipe. De nombreux problèmes se posent. Ils devront être détaillés au même
titre que les solutions proposées pour les contourner. 

### Analyser la furtivité du patch
Dans l'ensemble des cas étudiés, on demandera :

* S'il est possible de détecter qu'un binaire a été patché.
* Si oui:
    * Détailler la méthode de détection.
    * Proposer une contremesure.


## Vers un packer
Une fois que l'on sait injecter un payload dans un binaire, on peut proposer une première version de packer de binaire. 


### Packing
Le packing est l'étape qui consiste à chiffrer/compresser tout ou partie du binaire initial (statiquement). 
Cette partie peut être écrite en C, prend en entrée un programme binaire et fabrique un programme binaire chiffré/compressé
(qui ne peut donc plus s'exécuter normalement). 

### Unpacking
L'étape d'unpacking consiste à refabriquer le binaire original à partir de la version compressée/chiffrée. 
C'est fait au début de l'exécution dudit binaire compressé/chiffré. Une routine de déchiffrement est chargée de modifier dynamiquement son contenu afin
de reconstruire les instructions initiales. Cette routine sera injectée dans le binaire compressé/chiffré (mais elle-même ne sera
pas compressée/chiffrée), et exécutée au lancement du programme ... En fait, cette routine représentera un payload que l'on
injectera dans le binaire comme présenté dans la section précédente. La seule différence étant que ce payload influencera 
la suite de l'exécution, puisqu'il sera chargé de réécrire la section `CODE` du binaire.


### Résumé
```
[Binaire] --- (Packing) ---> [Binaire chiffré/compressé] --- (Malicious Binary Factory) ---> [Binaire chiffré/compressé + routine de déchiffrement] 
```


## Autres remarques

* En cas de bon projet, avec des prises d'initiative et des idées originales, le rendu (article de 8 pages) pourra être soumis à 
**GreHack 2016** ([http://grehack.fr/](http://grehack.fr/)).

## Références et documentation
* [ELF_Format.pdf](http://www.skyfree.org/linux/references/ELF_Format.pdf)
* [elf-64-gen.pdf](https://uclibc.org/docs/elf-64-gen.pdf)
* [Additional Code Segment](http://virus.enemy.org/virus-writing-HOWTO/_html/additional.cs.html)
* [Segment Padding](http://virus.enemy.org/virus-writing-HOWTO/_html/segment.padding.html)
* [Exemples de shellcodes](http://shell-storm.org/shellcode/)
