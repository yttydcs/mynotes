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