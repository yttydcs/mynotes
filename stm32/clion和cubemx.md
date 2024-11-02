# 配置

## stlink2
```cfg
source [find interface/stlink.cfg]
transport select hla_swd
source [find target/stm32f1x.cfg]
reset_config none
```

## cmsis-dap (烽火)
```
source [find interface/cmsis-dap.cfg]
transport select swd
source [find target/stm32f1x.cfg]
adapter speed 1000
```

# 国产芯片
那些标着国产stm32f103c8t6的版子用的芯片不一样，要将stm32f1x.cfg中的set _CPUTAPID 0x2ba01477 改为set _CPUTAPID 0x2ba01477

## 修改前的
```cfg
#jtag scan chain
if { [info exists CPUTAPID] } {
   set _CPUTAPID $CPUTAPID
} else {
   if { [using_jtag] } {
      # See STM Document RM0008 Section 26.6.3
      set _CPUTAPID 0x3ba00477
   } {
      # this is the SW-DP tap id not the jtag tap id
      set _CPUTAPID 0x1ba01477
   }
}
```

## 修改后的
```cfg
#jtag scan chain
if { [info exists CPUTAPID] } {
   set _CPUTAPID $CPUTAPID
} else {
   if { [using_jtag] } {
      # See STM Document RM0008 Section 26.6.3
      set _CPUTAPID 0x3ba00477
   } {
      # this is the SW-DP tap id not the jtag tap id
      set _CPUTAPID 0x2ba01477
   }
}
```

## 直接copy文件
直接把文件内容放到自己的文件里面，就可以避免修改官方的配置，而且换电脑不用重新修改配置。
```cfg
set _CPUTAPID 0x2ba01477
source [find interface/stlink.cfg]
transport select hla_swd
source [find target/swj-dp.tcl]
source [find mem_helper.tcl]

if { [info exists CHIPNAME] } {
   set _CHIPNAME $CHIPNAME
} else {
   set _CHIPNAME stm32f1x
}

set _ENDIAN little

if { [info exists WORKAREASIZE] } {
   set _WORKAREASIZE $WORKAREASIZE
} else {
   set _WORKAREASIZE 0x1000
}

if { [info exists FLASH_SIZE] } {
    set _FLASH_SIZE $FLASH_SIZE
} else {
    # autodetect size
    set _FLASH_SIZE 0
}

if { [info exists CPUTAPID] } {
   set _CPUTAPID $CPUTAPID
} else {
   if { [using_jtag] } {
      set _CPUTAPID 0x3ba00477
   } {
      set _CPUTAPID 0x2ba01477
   }
}

swj_newdap $_CHIPNAME cpu -irlen 4 -ircapture 0x1 -irmask 0xf -expected-id $_CPUTAPID
dap create $_CHIPNAME.dap -chain-position $_CHIPNAME.cpu

if {[using_jtag]} {
   jtag newtap $_CHIPNAME bs -irlen 5
}

set _TARGETNAME $_CHIPNAME.cpu
target create $_TARGETNAME cortex_m -endian $_ENDIAN -dap $_CHIPNAME.dap

$_TARGETNAME configure -work-area-phys 0x20000000 -work-area-size $_WORKAREASIZE -work-area-backup 0

set _FLASHNAME $_CHIPNAME.flash
flash bank $_FLASHNAME stm32f1x 0x08000000 $_FLASH_SIZE 0 0 $_TARGETNAME

adapter speed 1000

adapter srst delay 100
if {[using_jtag]} {
 jtag_ntrst_delay 100
}

reset_config srst_nogate

if {![using_hla]} {
    cortex_m reset_config sysresetreq
}

$_TARGETNAME configure -event examine-end {
	mmw 0xE0042004 0x00000307 0
}

tpiu create $_CHIPNAME.tpiu -dap $_CHIPNAME.dap -ap-num 0 -baseaddr 0xE0040000

lappend _telnet_autocomplete_skip _proc_pre_enable_$_CHIPNAME.tpiu
proc _proc_pre_enable_$_CHIPNAME.tpiu {_targetname} {
	targets $_targetname
	mmw 0xE0042004 0x00000020 0
}

$_CHIPNAME.tpiu configure -event pre-enable "_proc_pre_enable_$_CHIPNAME.tpiu $_TARGETNAME"
```
