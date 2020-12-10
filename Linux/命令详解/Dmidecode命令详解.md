# Dmidecode命令详解

Dmidecode 用户获取硬件信息，Dmidecode 遵循 SMBIOS/DMI 标准，其输出的信息包括 BIOS、系统、主板、处理器、内存、缓存等等。



## DMI

DMI (Desktop Management Interface, DMI)就是帮助收集电脑系统信息的管理系统，DMI信息的收集必须在严格遵照SMBIOS规范的前提下进行。 SMBIOS(System Management BIOS)是主板或系统制造者以标准格式显示产品管理信息所需遵循的统一规范。SMBIOS和DMI是由行业指导机构Desktop Management Task Force (DMTF)起草的开放性的技术标准，其中DMI设计适用于任何的平台和操作系统。

**DMI充当了管理工具和系统层之间接口的角色。**它建立了标准的可管理系统更加方便了电脑厂商和用户对系统的了解。DMI的主要组成部分是Management Information Format (MIF)数据库。这个数据库包括了所有有关电脑系统和配件的信息。通过DMI，用户可以获取序列号、电脑厂商、串口信息以及其它系统配件信息。



## Dmidecode

dmidecode的作用是将DMI数据库中的信息解码，以可读的文本方式显示。由于DMI信息可以人为修改，因此里面的信息不一定是系统准确的信息。



## dmidecode命令用法详解

不带选项执行 dmidecode 通常会输出所有的硬件信息。

Dmidecode 有个很有用的选项 -t，可以按指定类型输出相关信息，假如要获得处理器方面的信息，则可以执行
`dmidecode -t processor`

dmidecode 的全部输出如下：

```
# dmidecode 3.2
Getting SMBIOS data from sysfs.
SMBIOS 2.7 present.
119 structures occupying 4642 bytes.
Table at 0x000EC0C0.

Handle 0x0000, DMI type 0, 24 bytes
BIOS Information
        Vendor: American Megatrends Inc.
        Version: 3.2
        Release Date: 09/22/2015
        Address: 0xF0000
        Runtime Size: 64 kB
        ROM Size: 8192 kB
        Characteristics:
                PCI is supported
                BIOS is upgradeable
                BIOS shadowing is allowed
                Boot from CD is supported
                Selectable boot is supported
                BIOS ROM is socketed
                EDD is supported
                5.25"/1.2 MB floppy services are supported (int 13h)
                3.5"/720 kB floppy services are supported (int 13h)
                3.5"/2.88 MB floppy services are supported (int 13h)
                Print screen service is supported (int 5h)
                8042 keyboard services are supported (int 9h)
                Serial services are supported (int 14h)
                Printer services are supported (int 17h)
                ACPI is supported
                USB legacy is supported
                BIOS boot specification is supported
                Function key-initiated network boot is supported
                Targeted content distribution is supported
                UEFI is supported
        BIOS Revision: 1.0

Handle 0x0001, DMI type 1, 27 bytes
System Information
        Manufacturer: Supermicro
        Product Name: X9DRL-3F/iF
        Version: 0123456789
        Serial Number: 0123456789
        UUID: 00000000-0000-0000-0000-ac1f6b71614c
        Wake-up Type: Power Switch
        SKU Number: To be filled by O.E.M.
        Family: To be filled by O.E.M.

Handle 0x0002, DMI type 2, 15 bytes
Base Board Information
        Manufacturer: Supermicro
        Product Name: X9DRL-3F/iF
        Version: 1.01
        Serial Number: ZM186S032724
        Asset Tag: To be filled by O.E.M.
        Features:
                Board is a hosting board
                Board is replaceable
        Location In Chassis: To be filled by O.E.M.
        Chassis Handle: 0x0003
        Type: Motherboard
        Contained Object Handles: 0

Handle 0x0003, DMI type 3, 22 bytes
Chassis Information
        Manufacturer: Supermicro
        Type: Desktop
        Lock: Not Present
        Version: 0123456789
        Serial Number: 0123456789
        Asset Tag: To Be Filled By O.E.M.
        Boot-up State: Safe
        Power Supply State: Safe
        Thermal State: Safe
        Security Status: None
        OEM Information: 0x00000000
        Height: Unspecified
        Number Of Power Cords: 1
        Contained Elements: 0
        SKU Number: To be filled by O.E.M.

Handle 0x0004, DMI type 4, 42 bytes
Processor Information
        Socket Designation: CPU 1
        Type: Central Processor
        Family: Xeon
        Manufacturer: Intel
        ID: E4 06 03 00 FF FB EB BF
        Signature: Type 0, Family 6, Model 62, Stepping 4
        Flags:
                FPU (Floating-point unit on-chip)
                VME (Virtual mode extension)
                DE (Debugging extension)
                PSE (Page size extension)
                TSC (Time stamp counter)
                MSR (Model specific registers)
                PAE (Physical address extension)
                MCE (Machine check exception)
                CX8 (CMPXCHG8 instruction supported)
                APIC (On-chip APIC hardware supported)
                SEP (Fast system call)
                MTRR (Memory type range registers)
                PGE (Page global enable)
                MCA (Machine check architecture)
                CMOV (Conditional move instruction supported)
                PAT (Page attribute table)
                PSE-36 (36-bit page size extension)
                CLFSH (CLFLUSH instruction supported)
                DS (Debug store)
                ACPI (ACPI supported)
                MMX (MMX technology supported)
                FXSR (FXSAVE and FXSTOR instructions supported)
                SSE (Streaming SIMD extensions)
                SSE2 (Streaming SIMD extensions 2)
                SS (Self-snoop)
                HTT (Multi-threading)
                TM (Thermal monitor supported)
                PBE (Pending break enabled)
        Version: Intel(R) Xeon(R) CPU E5-2630 v2 @ 2.60GHz
        Voltage: 0.0 V
        External Clock: 100 MHz
        Max Speed: 4000 MHz
        Current Speed: 2600 MHz
        Status: Populated, Enabled
        Upgrade: Socket LGA2011
        L1 Cache Handle: 0x0005
        L2 Cache Handle: 0x0006
        L3 Cache Handle: 0x0007
        Serial Number: Not Specified
        Asset Tag: Not Specified
        Part Number: Not Specified
        Core Count: 6
        Core Enabled: 6
        Thread Count: 12
        Characteristics:
                64-bit capable
                Multi-Core
                Hardware Thread
                Execute Protection
                Enhanced Virtualization
                Power/Performance Control

Handle 0x0005, DMI type 7, 19 bytes
Cache Information
        Socket Designation: CPU Internal L1
        Configuration: Enabled, Not Socketed, Level 1
        Operational Mode: Write Back
        Location: Internal
        Installed Size: 384 kB
        Maximum Size: 384 kB
        Supported SRAM Types:
                Unknown
        Installed SRAM Type: Unknown
        Speed: Unknown
        Error Correction Type: Parity
        System Type: Other
        Associativity: 8-way Set-associative

Handle 0x0006, DMI type 7, 19 bytes
Cache Information
        Socket Designation: CPU Internal L2
        Configuration: Enabled, Not Socketed, Level 2
        Operational Mode: Write Back
        Location: Internal
        Installed Size: 1536 kB
        Maximum Size: 1536 kB
        Supported SRAM Types:
                Unknown
        Installed SRAM Type: Unknown
        Speed: Unknown
        Error Correction Type: Single-bit ECC
        System Type: Unified
        Associativity: 8-way Set-associative

Handle 0x0007, DMI type 7, 19 bytes
Cache Information
        Socket Designation: CPU Internal L3
        Configuration: Enabled, Not Socketed, Level 3
        Operational Mode: Write Back
        Location: Internal
        Installed Size: 15 MB
        Maximum Size: 15 MB
        Supported SRAM Types:
                Unknown
        Installed SRAM Type: Unknown
        Speed: Unknown
        Error Correction Type: Single-bit ECC
        System Type: Unified
        Associativity: 20-way Set-associative

Handle 0x0008, DMI type 4, 42 bytes
Processor Information
        Socket Designation: CPU 2
        Type: Central Processor
        Family: Xeon
        Manufacturer: Intel
        ID: E4 06 03 00 FF FB EB BF
        Signature: Type 0, Family 6, Model 62, Stepping 4
        Flags:
                FPU (Floating-point unit on-chip)
                VME (Virtual mode extension)
                DE (Debugging extension)
                PSE (Page size extension)
                TSC (Time stamp counter)
                MSR (Model specific registers)
                PAE (Physical address extension)
                MCE (Machine check exception)
                CX8 (CMPXCHG8 instruction supported)
                APIC (On-chip APIC hardware supported)
                SEP (Fast system call)
                MTRR (Memory type range registers)
                PGE (Page global enable)
                MCA (Machine check architecture)
                CMOV (Conditional move instruction supported)
                PAT (Page attribute table)
                PSE-36 (36-bit page size extension)
                CLFSH (CLFLUSH instruction supported)
                DS (Debug store)
                ACPI (ACPI supported)
                MMX (MMX technology supported)
                FXSR (FXSAVE and FXSTOR instructions supported)
                SSE (Streaming SIMD extensions)
                SSE2 (Streaming SIMD extensions 2)
                SS (Self-snoop)
                HTT (Multi-threading)
                TM (Thermal monitor supported)
                PBE (Pending break enabled)
        Version: Intel(R) Xeon(R) CPU E5-2630 v2 @ 2.60GHz
        Voltage: 0.0 V
        External Clock: 100 MHz
        Max Speed: 4000 MHz
        Current Speed: 2600 MHz
        Status: Populated, Enabled
        Upgrade: Socket LGA2011
        L1 Cache Handle: 0x0009
        L2 Cache Handle: 0x000A
        L3 Cache Handle: 0x000B
        Serial Number: Not Specified
        Asset Tag: Not Specified
        Part Number: Not Specified
        Core Count: 6
        Core Enabled: 6
        Thread Count: 12
        Characteristics:
                64-bit capable
                Multi-Core
                Hardware Thread
                Execute Protection
                Enhanced Virtualization
                Power/Performance Control

Handle 0x0009, DMI type 7, 19 bytes
Cache Information
        Socket Designation: CPU Internal L1
        Configuration: Enabled, Not Socketed, Level 1
        Operational Mode: Write Back
        Location: Internal
        Installed Size: 384 kB
        Maximum Size: 384 kB
        Supported SRAM Types:
                Unknown
        Installed SRAM Type: Unknown
        Speed: Unknown
        Error Correction Type: Parity
        System Type: Other
        Associativity: 8-way Set-associative

Handle 0x000A, DMI type 7, 19 bytes
Cache Information
        Socket Designation: CPU Internal L2
        Configuration: Enabled, Not Socketed, Level 2
        Operational Mode: Write Back
        Location: Internal
        Installed Size: 1536 kB
        Maximum Size: 1536 kB
        Supported SRAM Types:
                Unknown
        Installed SRAM Type: Unknown
        Speed: Unknown
        Error Correction Type: Single-bit ECC
        System Type: Unified
        Associativity: 8-way Set-associative

Handle 0x000B, DMI type 7, 19 bytes
Cache Information
        Socket Designation: CPU Internal L3
        Configuration: Enabled, Not Socketed, Level 3
        Operational Mode: Write Back
        Location: Internal
        Installed Size: 15 MB
        Maximum Size: 15 MB
        Supported SRAM Types:
                Unknown
        Installed SRAM Type: Unknown
        Speed: Unknown
        Error Correction Type: Single-bit ECC
        System Type: Unified
        Associativity: 20-way Set-associative

Handle 0x000C, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1A1
        Internal Connector Type: None
        External Reference Designator: PS2Mouse
        External Connector Type: PS/2
        Port Type: Mouse Port

Handle 0x000D, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1A1
        Internal Connector Type: None
        External Reference Designator: Keyboard
        External Connector Type: PS/2
        Port Type: Keyboard Port

Handle 0x000E, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J2A1
        Internal Connector Type: None
        External Reference Designator: TV Out
        External Connector Type: Mini Centronics Type-14
        Port Type: Other

Handle 0x000F, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J2A2A
        Internal Connector Type: None
        External Reference Designator: COM A
        External Connector Type: DB-9 male
        Port Type: Serial Port 16550A Compatible

Handle 0x0010, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J2A2B
        Internal Connector Type: None
        External Reference Designator: Video
        External Connector Type: DB-15 female
        Port Type: Video Port

Handle 0x0011, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J3A1
        Internal Connector Type: None
        External Reference Designator: USB1
        External Connector Type: Access Bus (USB)
        Port Type: USB

Handle 0x0012, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J3A1
        Internal Connector Type: None
        External Reference Designator: USB2
        External Connector Type: Access Bus (USB)
        Port Type: USB

Handle 0x0013, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J3A1
        Internal Connector Type: None
        External Reference Designator: USB3
        External Connector Type: Access Bus (USB)
        Port Type: USB

Handle 0x0014, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J9A1 - TPM HDR
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0015, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J9C1 - PCIE DOCKING CONN
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0016, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J2B3 - CPU FAN
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0017, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J6C2 - EXT HDMI
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0018, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J3C1 - GMCH FAN
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0019, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1D1 - ITP
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x001A, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J9E2 - MDC INTPSR
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x001B, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J9E4 - MDC INTPSR
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x001C, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J9E3 - LPC HOT DOCKING
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x001D, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J9E1 - SCAN MATRIX
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x001E, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J9G1 - LPC SIDE BAND
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x001F, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J8F1 - UNIFIED
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0020, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J6F1 - LVDS
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0021, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J2F1 - LAI FAN
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0022, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J2G1 - GFX VID
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0023, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1G6 - AC JACK
        Internal Connector Type: Other
        External Reference Designator: Not Specified
        External Connector Type: None
        Port Type: Other

Handle 0x0024, DMI type 9, 17 bytes
System Slot Information
        Designation: PCIE1
        Type: x16 PCI Express 3 x16
        Current Usage: Available
        Length: Long
        ID: 1
        Characteristics:
                3.3 V is provided
                Opening is shared
                PME signal is supported
        Bus Address: 0000:05:01.1

Handle 0x0025, DMI type 9, 17 bytes
System Slot Information
        Designation: PCIE2
        Type: x8 PCI Express 3 x8
        Current Usage: Available
        Length: Short
        ID: 2
        Characteristics:
                3.3 V is provided
                Opening is shared
                PME signal is supported
        Bus Address: 0000:ff:03.0

Handle 0x0026, DMI type 9, 17 bytes
System Slot Information
        Designation: PCIE3
        Type: x16 PCI Express 3 x16
        Current Usage: Available
        Length: Long
        ID: 3
        Characteristics:
                3.3 V is provided
                Opening is shared
                PME signal is supported
        Bus Address: 0000:02:02.0

Handle 0x0027, DMI type 9, 17 bytes
System Slot Information
        Designation: PCIE4
        Type: x8 PCI Express 3 x8
        Current Usage: Available
        Length: Short
        ID: 4
        Characteristics:
                3.3 V is provided
                Opening is shared
                PME signal is supported
        Bus Address: 0000:ff:02.2

Handle 0x0028, DMI type 9, 17 bytes
System Slot Information
        Designation: PCIE5
        Type: x16 PCI Express 3 x16
        Current Usage: Available
        Length: Long
        ID: 5
        Characteristics:
                3.3 V is provided
                Opening is shared
                PME signal is supported
        Bus Address: 0000:ff:03.2

Handle 0x0029, DMI type 9, 17 bytes
System Slot Information
        Designation: PCIE6
        Type: x8 PCI Express 3 x8
        Current Usage: Available
        Length: Short
        ID: 6
        Characteristics:
                3.3 V is provided
                Opening is shared
                PME signal is supported
        Bus Address: 0000:ff:03.2

Handle 0x002A, DMI type 10, 6 bytes
On Board Device Information
        Type: Video
        Status: Enabled
        Description:    To Be Filled By O.E.M.

Handle 0x002B, DMI type 11, 5 bytes
OEM Strings
        String 1: To Be Filled By O.E.M.
        String 2: To Be Filled By O.E.M.

Handle 0x002C, DMI type 12, 5 bytes
System Configuration Options
        Option 1: To Be Filled By O.E.M.

Handle 0x002D, DMI type 16, 23 bytes
Physical Memory Array
        Location: System Board Or Motherboard
        Use: System Memory
        Error Correction Type: Multi-bit ECC
        Maximum Capacity: 96 GB
        Error Information Handle: Not Provided
        Number Of Devices: 4

Handle 0x002E, DMI type 19, 31 bytes
Memory Array Mapped Address
        Starting Address: 0x00000000000
        Ending Address: 0x00FFFFFFFFF
        Range Size: 64 GB
        Physical Array Handle: 0x002D
        Partition Width: 1

Handle 0x002F, DMI type 17, 34 bytes
Memory Device
        Array Handle: 0x002D
        Error Information Handle: Not Provided
        Total Width: 72 bits
        Data Width: 64 bits
        Size: 16384 MB
        Form Factor: DIMM
        Set: None
        Locator: P1_DIMMA1
        Bank Locator: Node0_Bank0
        Type: DDR3
        Type Detail: Registered (Buffered)
        Speed: 1600 MT/s
        Manufacturer: Kingston          
        Serial Number: EC3896A8    
        Asset Tag: Dimm0_AssetTag
        Part Number: 9965516-477.A
        Rank: 2
        Configured Memory Speed: 1600 MT/s

Handle 0x0030, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x00000000000
        Ending Address: 0x003FFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x002F
        Memory Array Mapped Address Handle: 0x002E
        Partition Row Position: 1

Handle 0x0031, DMI type 17, 34 bytes
Memory Device
        Array Handle: 0x002D
        Error Information Handle: Not Provided
        Total Width: 72 bits
        Data Width: 64 bits
        Size: 16384 MB
        Form Factor: DIMM
        Set: None
        Locator: P1_DIMMB1
        Bank Locator: Node0_Bank0
        Type: DDR3
        Type Detail: Registered (Buffered)
        Speed: 1600 MT/s
        Manufacturer: Kingston          
        Serial Number: EE3898A8    
        Asset Tag: Dimm1_AssetTag
        Part Number: 9965516-477.A
        Rank: 2
        Configured Memory Speed: 1600 MT/s

Handle 0x0032, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x00400000000
        Ending Address: 0x007FFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x0031
        Memory Array Mapped Address Handle: 0x002E
        Partition Row Position: 1

Handle 0x0033, DMI type 17, 34 bytes
Memory Device
        Array Handle: 0x002D
        Error Information Handle: Not Provided
        Total Width: 72 bits
        Data Width: 64 bits
        Size: 16384 MB
        Form Factor: DIMM
        Set: None
        Locator: P1_DIMMC1
        Bank Locator: Node0_Bank0
        Type: DDR3
        Type Detail: Registered (Buffered)
        Speed: 1600 MT/s
        Manufacturer: Kingston          
        Serial Number: EB38D4A8    
        Asset Tag: Dimm2_AssetTag
        Part Number: 9965516-477.A
        Rank: 2
        Configured Memory Speed: 1600 MT/s

Handle 0x0034, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x00800000000
        Ending Address: 0x00BFFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x0033
        Memory Array Mapped Address Handle: 0x002E
        Partition Row Position: 1

Handle 0x0035, DMI type 17, 34 bytes
Memory Device
        Array Handle: 0x002D
        Error Information Handle: Not Provided
        Total Width: 72 bits
        Data Width: 64 bits
        Size: 16384 MB
        Form Factor: DIMM
        Set: None
        Locator: P1_DIMMD1
        Bank Locator: Node0_Bank0
        Type: DDR3
        Type Detail: Registered (Buffered)
        Speed: 1600 MT/s
        Manufacturer: Kingston          
        Serial Number: EB389AA8    
        Asset Tag: Dimm3_AssetTag
        Part Number: 9965516-477.A
        Rank: 2
        Configured Memory Speed: 1600 MT/s

Handle 0x0036, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x00C00000000
        Ending Address: 0x00FFFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x0035
        Memory Array Mapped Address Handle: 0x002E
        Partition Row Position: 1

Handle 0x0037, DMI type 16, 23 bytes
Physical Memory Array
        Location: System Board Or Motherboard
        Use: System Memory
        Error Correction Type: Multi-bit ECC
        Maximum Capacity: 96 GB
        Error Information Handle: Not Provided
        Number Of Devices: 4

Handle 0x0038, DMI type 19, 31 bytes
Memory Array Mapped Address
        Starting Address: 0x01000000000
        Ending Address: 0x01FFFFFFFFF
        Range Size: 64 GB
        Physical Array Handle: 0x0037
        Partition Width: 1

Handle 0x0039, DMI type 17, 34 bytes
Memory Device
        Array Handle: 0x0037
        Error Information Handle: Not Provided
        Total Width: 72 bits
        Data Width: 64 bits
        Size: 16384 MB
        Form Factor: DIMM
        Set: None
        Locator: P2_DIMME1
        Bank Locator: Node1_Bank0
        Type: DDR3
        Type Detail: Registered (Buffered)
        Speed: 1600 MT/s
        Manufacturer: Kingston          
        Serial Number: E83850A8    
        Asset Tag: Dimm0_AssetTag
        Part Number: 9965516-477.A
        Rank: 2
        Configured Memory Speed: 1600 MT/s

Handle 0x003A, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x01000000000
        Ending Address: 0x013FFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x0039
        Memory Array Mapped Address Handle: 0x0038
        Partition Row Position: 1

Handle 0x003B, DMI type 17, 34 bytes
Memory Device
        Array Handle: 0x0037
        Error Information Handle: Not Provided
        Total Width: 72 bits
        Data Width: 64 bits
        Size: 16384 MB
        Form Factor: DIMM
        Set: None
        Locator: P2_DIMMF1
        Bank Locator: Node1_Bank0
        Type: DDR3
        Type Detail: Registered (Buffered)
        Speed: 1600 MT/s
        Manufacturer: Kingston          
        Serial Number: F038D1A8    
        Asset Tag: Dimm1_AssetTag
        Part Number: 9965516-477.A
        Rank: 2
        Configured Memory Speed: 1600 MT/s

Handle 0x003C, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x01400000000
        Ending Address: 0x017FFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x003B
        Memory Array Mapped Address Handle: 0x0038
        Partition Row Position: 1

Handle 0x003D, DMI type 17, 34 bytes
Memory Device
        Array Handle: 0x0037
        Error Information Handle: Not Provided
        Total Width: 72 bits
        Data Width: 64 bits
        Size: 16384 MB
        Form Factor: DIMM
        Set: None
        Locator: P2_DIMMG1
        Bank Locator: Node1_Bank0
        Type: DDR3
        Type Detail: Registered (Buffered)
        Speed: 1600 MT/s
        Manufacturer: Kingston          
        Serial Number: EB388DA8    
        Asset Tag: Dimm2_AssetTag
        Part Number: 9965516-477.A
        Rank: 2
        Configured Memory Speed: 1600 MT/s

Handle 0x003E, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x01800000000
        Ending Address: 0x01BFFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x003D
        Memory Array Mapped Address Handle: 0x0038
        Partition Row Position: 1

Handle 0x003F, DMI type 17, 34 bytes
Memory Device
        Array Handle: 0x0037
        Error Information Handle: Not Provided
        Total Width: 72 bits
        Data Width: 64 bits
        Size: 16384 MB
        Form Factor: DIMM
        Set: None
        Locator: P2_DIMMH1
        Bank Locator: Node1_Bank0
        Type: DDR3
        Type Detail: Registered (Buffered)
        Speed: 1600 MT/s
        Manufacturer: Kingston          
        Serial Number: ED38CEA8    
        Asset Tag: Dimm3_AssetTag
        Part Number: 9965516-477.A
        Rank: 2
        Configured Memory Speed: 1600 MT/s

Handle 0x0040, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x01C00000000
        Ending Address: 0x01FFFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x003F
        Memory Array Mapped Address Handle: 0x0038
        Partition Row Position: 1

Handle 0x0041, DMI type 32, 20 bytes
System Boot Information
        Status: No errors detected

Handle 0x0042, DMI type 34, 11 bytes
Management Device
        Description: LM78-1
        Type: LM78
        Address: 0x00000000
        Address Type: I/O Port

Handle 0x0043, DMI type 26, 22 bytes
Voltage Probe
        Description: LM78A
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0044, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0045, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0042
        Component Handle: 0x0042
        Threshold Handle: 0x0043

Handle 0x0046, DMI type 28, 22 bytes
Temperature Probe
        Description: LM78A
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0047, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0048, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0042
        Component Handle: 0x0045
        Threshold Handle: 0x0046

Handle 0x0049, DMI type 27, 15 bytes
Cooling Device
        Temperature Probe Handle: 0x0046
        Type: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Cooling Unit Group: 1
        OEM-specific Information: 0x00000000
        Nominal Speed: Unknown Or Non-rotating
        Description: Cooling Dev 1

Handle 0x004A, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x004B, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0042
        Component Handle: 0x0048
        Threshold Handle: 0x0049

Handle 0x004C, DMI type 27, 15 bytes
Cooling Device
        Temperature Probe Handle: 0x0046
        Type: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Cooling Unit Group: 1
        OEM-specific Information: 0x00000000
        Nominal Speed: Unknown Or Non-rotating
        Description: Not Specified

Handle 0x004D, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x004E, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0042
        Component Handle: 0x004B
        Threshold Handle: 0x004C

Handle 0x004F, DMI type 29, 22 bytes
Electrical Current Probe
        Description: ABC
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0050, DMI type 36, 16 bytes
Management Device Threshold Data

Handle 0x0051, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0042
        Component Handle: 0x004E
        Threshold Handle: 0x004C

Handle 0x0052, DMI type 34, 16 bytes
Invalid entry length (16). Fixed up to 11.
Management Device
        Description: LM78-2
        Type: LM78
        Address: 0x00000000
        Address Type: I/O Port

Handle 0x0053, DMI type 26, 22 bytes
Voltage Probe
        Description: LM78B
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0054, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 7
        Upper Non-critical Threshold: 8
        Lower Critical Threshold: 8
        Upper Critical Threshold: 10
        Lower Non-recoverable Threshold: 11
        Upper Non-recoverable Threshold: 12

Handle 0x0055, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0052
        Component Handle: 0x0052
        Threshold Handle: 0x0053

Handle 0x0056, DMI type 26, 22 bytes
Voltage Probe
        Description: LM78B
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0057, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 13
        Upper Non-critical Threshold: 14
        Lower Critical Threshold: 15
        Upper Critical Threshold: 16
        Lower Non-recoverable Threshold: 17
        Upper Non-recoverable Threshold: 18

Handle 0x0058, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0052
        Component Handle: 0x0055
        Threshold Handle: 0x0056

Handle 0x0059, DMI type 28, 22 bytes
Temperature Probe
        Description: LM78B
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x005A, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x005B, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0052
        Component Handle: 0x0058
        Threshold Handle: 0x0059

Handle 0x005C, DMI type 27, 15 bytes
Cooling Device
        Temperature Probe Handle: 0x0059
        Type: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Cooling Unit Group: 1
        OEM-specific Information: 0x00000000
        Nominal Speed: Unknown Or Non-rotating
        Description: Cooling Dev 2

Handle 0x005D, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x005E, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0052
        Component Handle: 0x005B
        Threshold Handle: 0x005C

Handle 0x005F, DMI type 28, 22 bytes
Temperature Probe
        Description: LM78B
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0060, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0061, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0052
        Component Handle: 0x005E
        Threshold Handle: 0x005F

Handle 0x0062, DMI type 27, 15 bytes
Cooling Device
        Temperature Probe Handle: 0x005F
        Type: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Cooling Unit Group: 1
        OEM-specific Information: 0x00000000
        Nominal Speed: Unknown Or Non-rotating
        Description: Cooling Dev 2

Handle 0x0063, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0064, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0052
        Component Handle: 0x0061
        Threshold Handle: 0x0062

Handle 0x0065, DMI type 29, 22 bytes
Electrical Current Probe
        Description: DEF
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0066, DMI type 36, 16 bytes
Management Device Threshold Data

Handle 0x0067, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0052
        Component Handle: 0x0064
        Threshold Handle: 0x0062

Handle 0x0068, DMI type 29, 22 bytes
Electrical Current Probe
        Description: GHI
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0069, DMI type 36, 16 bytes
Management Device Threshold Data

Handle 0x006A, DMI type 35, 11 bytes
Management Device Component
        Description: To Be Filled By O.E.M.
        Management Device Handle: 0x0052
        Component Handle: 0x0067
        Threshold Handle: 0x0062

Handle 0x006B, DMI type 26, 22 bytes
Voltage Probe
        Description: LM78A
        Location: Power Unit
        Status: OK
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x006C, DMI type 28, 22 bytes
Temperature Probe
        Description: LM78A
        Location: Power Unit
        Status: OK
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x006D, DMI type 27, 15 bytes
Cooling Device
        Temperature Probe Handle: 0x006C
        Type: Power Supply Fan
        Status: OK
        Cooling Unit Group: 1
        OEM-specific Information: 0x00000000
        Nominal Speed: Unknown Or Non-rotating
        Description: Cooling Dev 1

Handle 0x006E, DMI type 29, 22 bytes
Electrical Current Probe
        Description: ABC
        Location: Power Unit
        Status: OK
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x006F, DMI type 39, 22 bytes
System Power Supply
        Power Unit Group: 1
        Location: To Be Filled By O.E.M.
        Name: To Be Filled By O.E.M.
        Manufacturer: To Be Filled By O.E.M.
        Serial Number: To Be Filled By O.E.M.
        Asset Tag: To Be Filled By O.E.M.
        Model Part Number: To Be Filled By O.E.M.
        Revision: To Be Filled By O.E.M.
        Max Power Capacity: Unknown
        Status: Not Present
        Type: Switching
        Input Voltage Range Switching: Auto-switch
        Plugged: Yes
        Hot Replaceable: No
        Input Voltage Probe Handle: 0x006B
        Cooling Device Handle: 0x006D
        Input Current Probe Handle: 0x006E

Handle 0x0070, DMI type 41, 11 bytes
Onboard Device
        Reference Designation:  Onboard IGD
        Type: Video
        Status: Enabled
        Type Instance: 1
        Bus Address: 0000:00:02.0

Handle 0x0071, DMI type 41, 11 bytes
Onboard Device
        Reference Designation:  Onboard LAN
        Type: Ethernet
        Status: Enabled
        Type Instance: 1
        Bus Address: 0000:00:19.0

Handle 0x0072, DMI type 41, 11 bytes
Onboard Device
        Reference Designation:  Onboard 1394
        Type: Other
        Status: Enabled
        Type Instance: 1
        Bus Address: 0000:03:1c.2

Handle 0x0073, DMI type 38, 18 bytes
IPMI Device Information
        Interface Type: KCS (Keyboard Control Style)
        Specification Version: 2.0
        I2C Slave Address: 0x00
        NV Storage Device: Not Present
        Base Address: 0x0000000000000CA2 (I/O)
        Register Spacing: Successive Byte Boundaries

Handle 0x007C, DMI type 15, 73 bytes
System Event Log
        Area Length: 0 bytes
        Header Start Offset: 0x0000
        Header Length: 16 bytes
        Data Start Offset: 0x0010
        Access Method: Memory-mapped physical 32-bit address
        Access Address: 0xFF850000
        Status: Valid, Not Full
        Change Token: 0x00000001
        Header Format: Type 1
        Supported Log Type Descriptors: 25
        Descriptor 1: Single-bit ECC memory error
        Data Format 1: Handle
        Descriptor 2: Multi-bit ECC memory error
        Data Format 2: Handle
        Descriptor 3: Parity memory error
        Data Format 3: None
        Descriptor 4: Bus timeout
        Data Format 4: None
        Descriptor 5: I/O channel block
        Data Format 5: None
        Descriptor 6: Software NMI
        Data Format 6: None
        Descriptor 7: POST memory resize
        Data Format 7: None
        Descriptor 8: POST error
        Data Format 8: POST results bitmap
        Descriptor 9: PCI parity error
        Data Format 9: Handle
        Descriptor 10: PCI system error
        Data Format 10: Handle
        Descriptor 11: CPU failure
        Data Format 11: None
        Descriptor 12: EISA failsafe timer timeout
        Data Format 12: None
        Descriptor 13: Correctable memory log disabled
        Data Format 13: None
        Descriptor 14: Logging disabled
        Data Format 14: None
        Descriptor 15: System limit exceeded
        Data Format 15: None
        Descriptor 16: Asynchronous hardware timer expired
        Data Format 16: None
        Descriptor 17: System configuration information
        Data Format 17: None
        Descriptor 18: Hard disk information
        Data Format 18: None
        Descriptor 19: System reconfigured
        Data Format 19: None
        Descriptor 20: Uncorrectable CPU-complex error
        Data Format 20: None
        Descriptor 21: Log area reset/cleared
        Data Format 21: None
        Descriptor 22: System boot
        Data Format 22: None
        Descriptor 23: End of log
        Data Format 23: None
        Descriptor 24: OEM-specific
        Data Format 24: OEM-specific
        Descriptor 25: OEM-specific
        Data Format 25: OEM-specific

Handle 0x0080, DMI type 13, 22 bytes
BIOS Language Information
        Language Description Format: Long
        Installable Languages: 1
                en|US|iso8859-1
        Currently Installed Language: en|US|iso8859-1

Handle 0x0081, DMI type 127, 4 bytes
End Of Table
```

























