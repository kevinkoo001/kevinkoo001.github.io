---
author: kevinkoo001@gmail.com
comments: true
date: 2017-09-05 04:57:10+00:00
layout: post
link: http://dandylife.net/blog/archives/686
slug: elf-sections-for-exception-handling
title: ELF Sections for Exception Handling
wordpress_id: 686
categories:
- Miscellaneous Stuff
---

In ELF binary, there are two sections to support exception handling routines that are predominately used by C++ applications: _.eh_frame_ and _ .eh_frame_hdr_. However, [System V Application Binary Interface (ABI) for AMD64](https://software.intel.com/sites/default/files/article/402129/mpx-linux64-abi.pdf) mandates to have those sections even they are written in C.

**a) ._eh_frame_ section**

The ._eh_frame_ section has the same structure with _.debug_frame_, which follows [DWARF](http://www.dwarfstd.org/doc/DWARF4.pdf) format. It represents the table that describes how to set registers to restore the previous call frame at runtime. DWARF designers allow for having flexible mechanism to be able to unwind the stack with various expressions including constant values, arithmetic, memory dereference, register contents, and control flow. 

The _.eh_frame_ section contains at least one or more Call Frame Information (CFI) records. Each CFI consists of two entry forms: Common Information Entry (CIE) and Frame Description Entry (FDE). Every CFI has a single CIE and one or more FDEs. CFI usually corresponds to a single object file. Likewise, so does FDE to a single function. However, there might be multiple CIEs and FDEs corresponding to the parts of a function when the function has a non-contiguous range in a virtual memory space. The following shows the fields of each entry in detail.

<table width="553" ><tbody ><tr >
<td width="183" >**CIE Fields**
</td>
<td width="157" >**Data Format**
</td>
<td width="213" >**Description**
</td></tr><tr >
<td >length 
</td>
<td >4 bytes
</td>
<td >Total length of the CIE except this field
</td></tr><tr >
<td >CIE_id 
</td>
<td >4 or 8 bytes
</td>
<td >0 for .eh_frame
</td></tr><tr >
<td >Version 
</td>
<td >1 byte
</td>
<td >Value 1
</td></tr><tr >
<td >Augmentation 
</td>
<td >A null-terminated UTF-8 string
</td>
<td >0 if no augmetation
</td></tr><tr >
<td >Code alignment factor 
</td>
<td >unsigned LEB128
</td>
<td >Usually 1
</td></tr><tr >
<td >Data alignment factor 
</td>
<td >signed LEB128
</td>
<td >Usually -4 (encoded as 0x7C)
</td></tr><tr >
<td >return_address_register 
</td>
<td >unsigned LEB128
</td>
<td >Dwarf number of the return register
</td></tr><tr >
<td >Augmentation data length 
</td>
<td >unsigned LEB128
</td>
<td >Present if Augmentation has 'z'
</td></tr><tr >
<td >Initial instructions
</td>
<td >array of bytes
</td>
<td >Dwarf Call Frame Instructions
</td></tr><tr >
<td >padding
</td>
<td >array of bytes
</td>
<td >DW_CFA_nop instructions to match the length
</td></tr></tbody></table><table width="553" ><tbody ><tr >
<td width="183" >**FDE Fields**
</td>
<td width="157" >**Data Format**
</td>
<td width="213" >**Description**
</td></tr><tr >
<td >Length 
</td>
<td >4 bytes
</td>
<td >Total length of the FDE except this field; 0 means end of all records
</td></tr><tr >
<td >CIE pointer 
</td>
<td >4 or 8 bytes
</td>
<td >Distance to the nearest preceding (parent) CIE
</td></tr><tr >
<td >Initial location 
</td>
<td >various bytes
</td>
<td >Reference to the function corresponding to the FDE
</td></tr><tr >
<td >Range length 
</td>
<td >various bytes
</td>
<td >Size of the function corresponding to the FDE
</td></tr><tr >
<td >Augmentation data length 
</td>
<td >unsigned LEB128
</td>
<td >Present if CIE Augmentation is non-empty
</td></tr><tr >
<td >Instructions
</td>
<td >array of bytes
</td>
<td >Dwarf Call Frame Instructions
</td></tr></tbody></table>

Here is an example of parsing a CIE and FDE (with a nice [Skochinsky's script](http://www.hexblog.com/wp-content/uploads/2012/06/recon-2012-skochinsky-scripts.zip) for IDA Pro). The sizes of the following CIE and FDE are 0x14 and 0x34 respectively. The FDE has a CIE pointer pointing to the parent CIE (at 0x4090A0). The function location corresponding to this FDE starts from 0x400c70 to 0x4010c0, whose size is 0x450.
    
    CIE (Common Information Entry)
    .eh_frame:0x04090A0 14 00 00 00     ; Size
    .eh_frame:0x04090A4 00 00 00 00     ; CIE id
    .eh_frame:0x04090A8 01              ; Version
    .eh_frame:0x04090A9 7A 52 00        ; Augmentation String (aZr)
    .eh_frame:0x04090AC 01              ; Code alignment factor
    .eh_frame:0x04090AD 78              ; Data alignment factor
    .eh_frame:0x04090AE 10              ; Return register
    .eh_frame:0x04090AF 01              ; Augmentation data length
    .eh_frame:0x04090B0 1B              ; R: FDE pointers encoding
    .eh_frame:0x04090B1 0C 07 08 90+    ; Initial CFE Instructions
    
    FDE (Frame Descriptor Entry)
    .eh_frame:0x04090B8 34 00 00 00     ; Size
    .eh_frame:0x04090BC 1C 00 00 00     ; CIE pointer (0x4090A0)
    .eh_frame:0x04090C0 B0 7B FF FF     ; Initial location=0x400C70
    .eh_frame:0x04090C4 50 04 00 00     ; Range length=0x450(end=0x4010C0)
    .eh_frame:0x04090C8 00              ; Augmentation data length
    .eh_frame:0x04090C9 41 0E 10 42+    ; CFE Instructions
    ...

**b) ._eh_frame_hdr_ section**

The ._eh_frame_hdr_ section contains a series of attributes, followed by the table of multiple pairs of (initial location, pointer to the FDE in the ._eh_frame_). The entries are sorted by functions that allows to perform a quick binary search of O(log n). 
    
    version (uint8)            structure version (=1)
    eh_frame_ptr_enc (uint8)   encoding of eh_frame_ptr
    fde_count_enc (uint8)      encoding of fde_count
    table_enc (uint8)          encoding of table entries
    eh_frame_ptr (enc)         pointer to the start of the .eh_frame section
    fde_count (enc)            number of entries in the table
    ----------------------- Table starts from here -------------------------
    initial_loc[i]             initial location for the FDE
    fde_ptr[i]                 corresponding FDE
    ------------------------------------------------------------------------
    

The next figure is a brief illustration of the relationship of two sections.

[![](http://dandylife.net/blog/wp-content/uploads/2017/09/eh_frame.jpg)](http://dandylife.net/blog/archives/686/eh_frame)

Note that the attribute, _table_enc_, describes how the table has been encoded. It consists of lower 4 bits for value and upper 4 bits for encoding as follows: DWARF Exception Header value format (lower 4 bits) and DWARF Exception Header Encoding (upper 4 bits)
    
    DW_EH_PE_omit	 0xff	No value is present
    DW_EH_PE_uleb128 0x01	Unsigned value is encoded using LEB128
    DW_EH_PE_udata2	 0x02	A 2 bytes unsigned value
    DW_EH_PE_udata4	 0x03	A 4 bytes unsigned value
    DW_EH_PE_udata8	 0x04	An 8 bytes unsigned value
    DW_EH_PE_sleb128 0x09	Signed value is encoded using LEB128
    DW_EH_PE_sdata2	 0x0A	A 2 bytes signed value
    DW_EH_PE_sdata4	 0x0B	A 4 bytes signed value
    DW_EH_PE_sdata8	 0x0C	An 8 bytes signed value
    
    DW_EH_PE_absptr  0x00 Used with no modification 
    DW_EH_PE_pcrel   0x10 Relative to the current program counter 
    DW_EH_PE_datarel 0x30 Relative to the beginning of the .eh_frame_hdr 
    DW_EH_PE_omit    0xff No value is present

**References**[  
https://refspecs.linuxfoundation.org/LSB_1.3.0/gLSB/gLSB/ehframehdr.htm  
](https://refspecs.linuxfoundation.org/LSB_1.3.0/gLSB/gLSB/ehframehdr.html)[http://www.dwarfstd.org/doc/DWARF4.pdf  
](http://www.dwarfstd.org/doc/DWARF4.pdf)[http://www.hexblog.com/wp-content/uploads/2012/06/Recon-2012-Skochinsky-Compiler-Internals.pdf  
](http://www.hexblog.com/wp-content/uploads/2012/06/Recon-2012-Skochinsky-Compiler-Internals.pdf)[https://software.intel.com/sites/default/files/article/402129/mpx-linux64-abi.pdf  
](https://software.intel.com/sites/default/files/article/402129/mpx-linux64-abi.pdf)
