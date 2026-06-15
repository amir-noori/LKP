# Linux Kernel Internals

## Memory Addressing
# Linux Kernel Internals

## Memory Addressing

### Address types:

- Logical
    
    segment + offset

- Linear

    A single 32-bit unsigned integer that can be used to address up to 4 GB

- Physical
    
    address memory cells in memory chips


MMU job:

    Logical -> [Segmentation] -> Linear -> [Paging] -> Physical



### Segmentation in Hardware

#### Segment Selector:

segment = segment selector (16-bit) + offset (32-bit)

to easy access to segment selectors there are segment registers:

 - cs: code segment
 - ss: stack segment (current stack segemnt)
 - ds: data segment (global and static data)
 - es, fs, gs: general purpose


#### Segment Descriptor:

Each segment is represented by an 8-byte Segment Descriptor (SD) that describes segment characteristics. Each SD is stored in GDT or LDT. <br/>
Usually only one GDT (GDT address is in gdtr register) is defined and each process chas its own LDT (GDT address is in ldtr register)


There are multiple segment descriptors:

- Code SD: 
- Data SD: 
- Task State SD: segment that a  process use to save its registers data, ... 
- Local Descriptor Table Descriptor (LDTD):  segment containing an LDT


#### Fast Access to Segment Descriptors


The x86 processor has 6 hidden (nonprogrammable) registers—one paired with each programmable segmentation register (CS, DS, SS, ES, FS, GS). These hidden registers act as a hardware cache for segment descriptors.

    Program loads Segment Selector (e.g., 0x0010) into CS
                        │
                        ▼
    ┌─────────────────────────────────────────────────────────────┐
    │ CPU automatically fetches the corresponding 8-byte          │
    │ Segment Descriptor from GDT/LDT in main memory              │
    │ (Address = GDT base + index × 8)                            │
    └─────────────────────────────────────────────────────────────┘
                        │
                        ▼
    ┌─────────────────────────────────────────────────────────────┐
    │ Descriptor is stored in the HIDDEN register paired with CS  │
    │ (Programmer cannot directly access or modify this)          │
    └─────────────────────────────────────────────────────────────┘
                        │
                        ▼
    ┌─────────────────────────────────────────────────────────────┐
    │ Subsequent translations use cached descriptor in CPU        │
    │ → NO memory access to GDT/LDT needed! ⚡                    │
    └─────────────────────────────────────────────────────────────┘




                        GDT (Global Descriptor Table)
                        ┌─────────────────────────────────────┐
    gdtr ─────────────► │  0x00020000  ┌───────────────────┐  │
                        │              │ Entry 0 (NULL)    │◄─── First entry = 0 (invalid)
                        │              ├───────────────────┤  │
                        │              │ Entry 1           │  │
                        │              ├───────────────────┤  │
    Segment Selector:   │              │ Entry 2 ◄─────────┼───┐
    Index = 2 ──────────┼─────────────►├───────────────────┤  │ │
                        │              │ Entry 3           │  │ │
                        │              ├───────────────────┤  │ │
                        │              │ Entry 4           │  │ │
                        │              ├───────────────────┤  │ │
                        │              │ ...               │  │ │
                        │              └───────────────────┘  │ │
                        └─────────────────────────────────────┘ │
                                                                │
                                                                │
                                                        8 bytes per entry
                                                                │
                                                                ▼
                                            Address = 0x00020000 + (2 × 8)
                                                    = 0x00020010



#### Segmentation Unit



    Logical Address (48 bits)
    ┌──────────────────────────────────────────────────────────────┐
    │  Segment Selector (16 bits)    │    Offset (32 bits)         │
    │  ┌─────────────────┐           │    ┌─────────────────┐      │
    │  │ 0x0010          │           │    │ 0x00001234      │      │
    │  └─────────────────┘           │    └─────────────────┘      │
    └──────────────────────────────────────────────────────────────┘
            │                                         │
            │                                         │
            ▼                                         │
        ┌─────────────┐                               │
        │ Step 1 & 2  │                               │
        │ Extract     │                               │
        │ Descriptor  │                               │
        │ from GDT/LDT│                               │
        └─────────────┘                               │
            │                                         │
            ▼                                         │
        ┌─────────────┐                               │
        │ Step 3      │                               │
        │ Add Base +  │◄──────────────────────────────┘
        │ Offset      │
        └─────────────┘
            │
            ▼
    ┌──────────────────────────────────────────────────────────────┐
    │                    Linear Address (32 bits)                  │
    │                      0x00011234                              │
    └──────────────────────────────────────────────────────────────┘


### Segmentation in Linux


Segment selectors are defined in the following macros:
- __USER_CS
- __USER_DS
- __KERNEL_CS (kernel code segment)
- __KERNEL_DS

the Current Privilege Level of the CPU indicates whether the processor is in User or Kernel Mode and is specified by the RPL field of the Segment Selector
stored in the cs register.




### Paging in Hardware

Address T
