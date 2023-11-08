---
author: kevinkoo001@gmail.com
comments: false
date: 2015-02-16 06:46:51+00:00
layout: post
link: http://dandylife.net/blog/archives/388
slug: pe-file-format-2
title: PE File Format
wordpress_id: 388
categories:
- Attack &amp; Defense, Cyber Warfare
---

[![pe_format](http://dandylife.net/blog/wp-content/uploads/2015/02/pe_format.png)](http://dandylife.net/blog/wp-content/uploads/2015/02/pe_format.png)





**(1) IMAGE_DOS_HEADER**







Note that the highlighted lines are the fields we need to focus on.



    
    typedef struct _IMAGE_DOS_HEADER {
       WORD  e_magic;      /* 00: MZ Header signature */
       WORD  e_cblp;       /* 02: Bytes on last page of file */
       WORD  e_cp;         /* 04: Pages in file */
       WORD  e_crlc;       /* 06: Relocations */
       WORD  e_cparhdr;    /* 08: Size of header in paragraphs */
       WORD  e_minalloc;   /* 0a: Minimum extra paragraphs needed */
       WORD  e_maxalloc;   /* 0c: Maximum extra paragraphs needed */
       WORD  e_ss;         /* 0e: Initial (relative) SS value */
       WORD  e_sp;         /* 10: Initial SP value */
       WORD  e_csum;       /* 12: Checksum */
       WORD  e_ip;         /* 14: Initial IP value */
       WORD  e_cs;         /* 16: Initial (relative) CS value */
       WORD  e_lfarlc;     /* 18: File address of relocation table */
       WORD  e_ovno;       /* 1a: Overlay number */
       WORD  e_res[4];     /* 1c: Reserved words */
       WORD  e_oemid;      /* 24: OEM identifier (for e_oeminfo) */
       WORD  e_oeminfo;    /* 26: OEM information; e_oemid specific */
       WORD  e_res2[10];   /* 28: Reserved words */
       DWORD e_lfanew;     /* 3c: Offset (pointer) to PE header */
    } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;




The following image illustrates DOS HEADER.




![1](http://dandylife.net/blog/wp-content/uploads/2015/02/1.png)








**(2) DOS Stub**




This part is no longer using after 32 bit mode. The following image shows an example of DOS Stub.


[![2](http://dandylife.net/blog/wp-content/uploads/2015/02/2.png)](http://dandylife.net/blog/wp-content/uploads/2015/02/2.png)





**(3) IMAGE_NT_HEADERS (Size:0xF8)**






    
    typedef struct _IMAGE_NT_HEADERS {
     DWORD Signature; /* "PE"\0\0 */       /* 0x00 */
     IMAGE_FILE_HEADER FileHeader;         /* 0x04 */
     IMAGE_OPTIONAL_HEADER32 OptionalHeader;       /* 0x18 */
    } IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
    




An example of NT Headers is as following. Note that the starting address matches the value "0x000000E0" with the one from _e_lfanew field_ in DOS Header. NT Header contains both Image File Header and Image Optional Header.




[![3](http://dandylife.net/blog/wp-content/uploads/2015/02/3.png)](http://dandylife.net/blog/wp-content/uploads/2015/02/3.png)










**(4) IMAGE_FILE_HEADER**





	





	
  * _Machine_ field specifies the architecture; 0x14c means x86 and 0x8664 means x86-64.

	
  * _TimeDateStamp_ field has Unix timestamp whose epoc is 00:00:00 UTC on Jan. 1st, 1970 at link time.

	
  * SizeOfOptionalHeader field indicates the number of section headers.

	
  * _Characteristics_ field shows the property of an executable file. See the below.


	



    
    typedef struct _IMAGE_FILE_HEADER {
     WORD  Machine;
     WORD  NumberOfSections;
     DWORD TimeDateStamp;
     DWORD PointerToSymbolTable;
     DWORD NumberOfSymbols;
     WORD  SizeOfOptionalHeader;
     WORD  Characteristics;
    } IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
    



    
    /* These are the settings of the Machine field. */
    #define IMAGE_FILE_MACHINE_UNKNOWN      0
    #define IMAGE_FILE_MACHINE_I860         0x014d
    #define IMAGE_FILE_MACHINE_I386         0x014c
    #define IMAGE_FILE_MACHINE_R3000        0x0162
    #define IMAGE_FILE_MACHINE_R4000        0x0166
    #define IMAGE_FILE_MACHINE_R10000       0x0168
    #define IMAGE_FILE_MACHINE_WCEMIPSV2    0x0169
    #define IMAGE_FILE_MACHINE_ALPHA        0x0184
    #define IMAGE_FILE_MACHINE_SH3          0x01a2
    #define IMAGE_FILE_MACHINE_SH3DSP       0x01a3
    #define IMAGE_FILE_MACHINE_SH3E         0x01a4
    #define IMAGE_FILE_MACHINE_SH4          0x01a6
    #define IMAGE_FILE_MACHINE_SH5          0x01a8
    #define IMAGE_FILE_MACHINE_ARM          0x01c0
    #define IMAGE_FILE_MACHINE_THUMB        0x01c2
    #define IMAGE_FILE_MACHINE_ARMNT        0x01c4
    #define IMAGE_FILE_MACHINE_ARM64        0xaa64
    #define IMAGE_FILE_MACHINE_AM33         0x01d3
    #define IMAGE_FILE_MACHINE_POWERPC      0x01f0
    #define IMAGE_FILE_MACHINE_POWERPCFP    0x01f1
    #define IMAGE_FILE_MACHINE_IA64         0x0200
    #define IMAGE_FILE_MACHINE_MIPS16       0x0266
    #define IMAGE_FILE_MACHINE_ALPHA64      0x0284
    #define IMAGE_FILE_MACHINE_MIPSFPU      0x0366
    #define IMAGE_FILE_MACHINE_MIPSFPU16    0x0466
    #define IMAGE_FILE_MACHINE_AXP64        IMAGE_FILE_MACHINE_ALPHA64
    #define IMAGE_FILE_MACHINE_TRICORE      0x0520
    #define IMAGE_FILE_MACHINE_CEF          0x0cef
    #define IMAGE_FILE_MACHINE_EBC          0x0ebc
    #define IMAGE_FILE_MACHINE_AMD64        0x8664
    #define IMAGE_FILE_MACHINE_M32R         0x9041
    #define IMAGE_FILE_MACHINE_CEE          0xc0ee



    
    /* These defines the meanings of the bits in the Characteristics field */
    #define IMAGE_FILE_RELOCS_STRIPPED      0x0001 /* No relocation info */
    #define IMAGE_FILE_EXECUTABLE_IMAGE     0x0002
    #define IMAGE_FILE_LINE_NUMS_STRIPPED   0x0004
    #define IMAGE_FILE_LOCAL_SYMS_STRIPPED  0x0008
    #define IMAGE_FILE_AGGRESIVE_WS_TRIM    0x0010
    #define IMAGE_FILE_LARGE_ADDRESS_AWARE  0x0020
    #define IMAGE_FILE_16BIT_MACHINE        0x0040
    #define IMAGE_FILE_BYTES_REVERSED_LO    0x0080
    #define IMAGE_FILE_32BIT_MACHINE        0x0100
    #define IMAGE_FILE_DEBUG_STRIPPED       0x0200
    #define IMAGE_FILE_REMOVABLE_RUN_FROM_SWAP      0x0400
    #define IMAGE_FILE_NET_RUN_FROM_SWAP    0x0800
    #define IMAGE_FILE_SYSTEM               0x1000
    #define IMAGE_FILE_DLL                  0x2000
    #define IMAGE_FILE_UP_SYSTEM_ONLY       0x4000
    #define IMAGE_FILE_BYTES_REVERSED_HI    0x8000







**(5) IMAGE_OPTIONAL_HEADER**








Note that Optional header is not all optional!!








	
  * _AddressOfEntryPoint _specifies the RVA which the loader starts code execution

	
  * _SizeOfImage_ tells the amount of contiguous memory reserved to load the binary into memory.

	
  * _SectionAlignment_ specifies that sections should be aligned on boundaries of multiples of this value.

	
  * _FileAlignment _field tells that data has to be written to a file in chucks no smaller than this value.
i.e 0x200 or 512 in HDD sector size

	
  * _ImageBase_ field specifies the preferred virtual memory address for the beginning of the binary.

	
  * _DLLCharateristics_ field provides the loader with security options like ASLR and DEP NX memory regions.
-> Not limited to DLLs, IDE compiler with the /DYNAMICBASE option

	
  * DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES] has two fields: _VirtualAddress _and _Size_



    
    typedef struct _IMAGE_OPTIONAL_HEADER {
    /* Standard fields */
    WORD  Magic; /* 0x10b or 0x107 */     /* 0x00 */
    BYTE  MajorLinkerVersion;
    BYTE  MinorLinkerVersion;
    DWORD SizeOfCode;
    DWORD SizeOfInitializedData;
    DWORD SizeOfUninitializedData;
    DWORD AddressOfEntryPoint;            /* 0x10 */
    DWORD BaseOfCode;
    DWORD BaseOfData;
    /* NT additional fields */
    DWORD ImageBase;
    DWORD SectionAlignment;               /* 0x20 */
    DWORD FileAlignment;
    WORD  MajorOperatingSystemVersion;
    WORD  MinorOperatingSystemVersion;
    WORD  MajorImageVersion;
    WORD  MinorImageVersion;
    WORD  MajorSubsystemVersion;          /* 0x30 */
    WORD  MinorSubsystemVersion;
    DWORD Win32VersionValue;
    DWORD SizeOfImage;
    DWORD SizeOfHeaders;
    DWORD CheckSum;                       /* 0x40 */
    WORD  Subsystem;
    WORD  DllCharacteristics;
    DWORD SizeOfStackReserve;
    DWORD SizeOfStackCommit;
    DWORD SizeOfHeapReserve;              /* 0x50 */
    DWORD SizeOfHeapCommit;
    DWORD LoaderFlags;
    DWORD NumberOfRvaAndSizes;
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES]; /* 0x60 */
    /* 0xE0 */
    } IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;



    
    // Directory Entries (16 entries are pre-defined)
    #define IMAGE_DIRECTORY_ENTRY_EXPORT         0     /*Export Directory */
    #define IMAGE_DIRECTORY_ENTRY_IMPORT         1     /*Import Directory */
    #define IMAGE_DIRECTORY_ENTRY_RESOURCE       2     /*Resource Directory */
    #define IMAGE_DIRECTORY_ENTRY_EXCEPTION      3     /*Exception Directory */
    #define IMAGE_DIRECTORY_ENTRY_SECURITY       4     /*Security Directory */
    #define IMAGE_DIRECTORY_ENTRY_BASERELOC      5     /*Base Relocation Table */
    #define IMAGE_DIRECTORY_ENTRY_DEBUG          6     /*Debug Directory */
    #define IMAGE_DIRECTORY_ENTRY_COPYRIGHT      7     /* (x86 usage) */
    #define IMAGE_DIRECTORY_ENTRY_ARCHITECTURE   7     /* Architecture Specific Data */
    #define IMAGE_DIRECTORY_ENTRY_GLOBALPTR 	 8     /* RVA of GP */
    #define IMAGE_DIRECTORY_ENTRY_TLS            9	   /* TLS Directory */
    #define IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG    10    /* Load Configuration Directory */
    #define IMAGE_DIRECTORY_ENTRY_LOAD_BOUND_IMPORT 	11     /* Bound Import Directory in headers */
    #define IMAGE_DIRECTORY_ENTRY_LOAD_IAT 				12     /* Import Address Table */
    #define IMAGE_DIRECTORY_ENTRY_LOAD_DELAY_IMPORT 	13     /* Delay Load Import Descriptors */
    #define IMAGE_DIRECTORY_ENTRY_LOAD_COM_DESCRIPTOR 	14     /* COM Runtime descriptor */




Each directory has similar structure as following:






    
    typedef struct _IMAGE_DATA_DIRECTORY {
     DWORD VirtualAddress;
     DWORD Size;
    } IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
    




**(6) IMAGE_SECTION_HEADER (.text, .data, .rsrc, .reloc, ...)**











	
  * _VirtualAddress_ specifies the RVA (Relative Virtual Address) of the section relative to _ImageBase._

	
  * PointerToRawData specifies a relative offset to store the actual section data from the file .

	
  * _SizeOfRawData_ indicates the size of memory allocation for the section. The value is Mics.VirtualSize which is rounded up to the multiple of alignment.

	
  * _PointerToRawData_ field indicates the actual file offset from the section.

	
  * See the below for a _characteristics_ field.



    
    # define IMAGE_SIZEOF_SHORT_NAME 8
    typedef struct _IMAGE_SECTION_HEADER {
     BYTE  Name[IMAGE_SIZEOF_SHORT_NAME];
     union {
       DWORD PhysicalAddress;
       DWORD VirtualSize;
     } Misc;
     DWORD VirtualAddress;
     DWORD SizeOfRawData;
     DWORD PointerToRawData;
     DWORD PointerToRelocations;
     DWORD PointerToLinenumbers;
     WORD  NumberOfRelocations;
     WORD  NumberOfLinenumbers;
     DWORD Characteristics;
    } IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
    


[![4](http://dandylife.net/blog/wp-content/uploads/2015/02/4.png)](http://dandylife.net/blog/wp-content/uploads/2015/02/4.png)

    
    /* These defines are for the Characteristics bitfield. */
    /* #define IMAGE_SCN_TYPE_REG                   0x00000000 - Reserved */
    /* #define IMAGE_SCN_TYPE_DSECT                 0x00000001 - Reserved */
    /* #define IMAGE_SCN_TYPE_NOLOAD                0x00000002 - Reserved */
    /* #define IMAGE_SCN_TYPE_GROUP                 0x00000004 - Reserved */
    #define IMAGE_SCN_TYPE_NO_PAD                0x00000008 /* Reserved */
    /* #define IMAGE_SCN_TYPE_COPY                  0x00000010 - Reserved */
    
    #define IMAGE_SCN_CNT_CODE                      0x00000020
    #define IMAGE_SCN_CNT_INITIALIZED_DATA          0x00000040
    #define IMAGE_SCN_CNT_UNINITIALIZED_DATA        0x00000080
    
    #define IMAGE_SCN_LNK_OTHER                     0x00000100
    #define IMAGE_SCN_LNK_INFO                      0x00000200
    /* #define      IMAGE_SCN_TYPE_OVER             0x00000400 - Reserved */
    #define IMAGE_SCN_LNK_REMOVE                    0x00000800
    #define IMAGE_SCN_LNK_COMDAT                    0x00001000
    
    /*                                              0x00002000 - Reserved */
    /* #define IMAGE_SCN_MEM_PROTECTED              0x00004000 - Obsolete */
    #define IMAGE_SCN_MEM_FARDATA                   0x00008000
    
    /* #define IMAGE_SCN_MEM_SYSHEAP                0x00010000 - Obsolete */
    #define IMAGE_SCN_MEM_PURGEABLE                 0x00020000
    #define IMAGE_SCN_MEM_16BIT                     0x00020000
    #define IMAGE_SCN_MEM_LOCKED                    0x00040000
    #define IMAGE_SCN_MEM_PRELOAD                   0x00080000
    
    #define IMAGE_SCN_ALIGN_1BYTES                  0x00100000
    #define IMAGE_SCN_ALIGN_2BYTES                  0x00200000
    #define IMAGE_SCN_ALIGN_4BYTES                  0x00300000
    #define IMAGE_SCN_ALIGN_8BYTES                  0x00400000
    #define IMAGE_SCN_ALIGN_16BYTES                 0x00500000  /* Default */
    #define IMAGE_SCN_ALIGN_32BYTES                 0x00600000
    #define IMAGE_SCN_ALIGN_64BYTES                 0x00700000
    #define IMAGE_SCN_ALIGN_128BYTES                0x00800000
    #define IMAGE_SCN_ALIGN_256BYTES                0x00900000
    #define IMAGE_SCN_ALIGN_512BYTES                0x00A00000
    #define IMAGE_SCN_ALIGN_1024BYTES               0x00B00000
    #define IMAGE_SCN_ALIGN_2048BYTES               0x00C00000
    #define IMAGE_SCN_ALIGN_4096BYTES               0x00D00000
    #define IMAGE_SCN_ALIGN_8192BYTES               0x00E00000
    /*                                              0x00F00000 - Unused */
    #define IMAGE_SCN_ALIGN_MASK                    0x00F00000
    #define IMAGE_SCN_LNK_NRELOC_OVFL               0x01000000
    #define IMAGE_SCN_MEM_DISCARDABLE               0x02000000
    #define IMAGE_SCN_MEM_NOT_CACHED                0x04000000
    #define IMAGE_SCN_MEM_NOT_PAGED                 0x08000000
    #define IMAGE_SCN_MEM_SHARED                    0x10000000
    #define IMAGE_SCN_MEM_EXECUTE                   0x20000000
    #define IMAGE_SCN_MEM_READ                      0x40000000
    #define IMAGE_SCN_MEM_WRITE                     0x80000000




Here are several good resources to explain PE format.





	
  * [Microsoft Portable Executable and Common Object File Format Specification](http://www.openwatcom.org/ftp/devel/docs/pecoff.pdf)

	
  * [Bin Portable Executable File Format - A Reverse Engineer View](http://www.woodmann.com/collaborative/knowledge/images/Bin_Portable_Executable_File_Format_%E2%80%93_A_Reverse_Engineer_View_2012-1-31_16.43_CBM_1_2_2006_Goppit_PE_Format_Reverse_Engineer_View.pdf)

	
  * [PE Format (OpenRCE.org)](http://www.openrce.org/reference_library/files/reference/PE%20Format.pdf)





















