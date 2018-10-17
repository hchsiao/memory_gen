# memory_gen
Memory generator incooperating with ChipCMake framework.

## Features
  - Vendor-independent memory generator interface
  - Programmable with ChipCMake
  - Techonology (e.g. ASIC/FPGA) independent

## Vendor-independent files
| File | Description |
| ------ | ------ |
| CmakeFileList.txt | Cmake scripts to be called from the parent design |
| src/wrapper.sv.in | Generic memory module template |

## Supported memory listing
This tool is designed to be modular, new types of memory can be supported by adding vendor-specific scripts. Currently supported supported memories are listed as followed:

#### TSMC90nm (Artisan)
| File | Description |
| ------ | ------ |
| cmake/FindTSMC90Memory.cmake | Search script for the library path |
| src/lib2db.tcl.in | Template script for generating Synopsys DB library |
| src/regfile_1p.spec.in | Register-file parameter template |
| src/sram_1p.spec.in | SRAM parameter template |

## Usage
In the root CMakeFileList.txt of the parent design, add this call:
```cmake
add_memory_macro(${MEMORY_NAME} ${TYPE}
  FREQ      ${FREQ}
  BIT_WIDTH ${BIT_WIDTH}
  MUX       ${MUX}
  SIZE      ${SIZE}
  NUM_BANK  ${N_BANK}
  )
```

| Variable | Type | Description |
| ------ | ------ | ------ |
| MEMORY_NAME | String | name of the module |
| TYPE | Category | Either RF or SRAM |
| FREQ | Digit | as the name suggests |
| BIT_WIDTH | Digit | word width |
| MUX | Digit | implementation parameter (TO BE REMOVED) |
| SIZE | Digit | size of each bank |
| N_BANK | Digit | the number of banks (actual memory size = N_BANK*SIZE) |

## Known issue
 - The MUX option is not really technology independent!
 - Should not expose the concept of banks

