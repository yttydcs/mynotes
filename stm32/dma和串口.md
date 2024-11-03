# dma
## 使用的函数
```c
// 取消传输过半中断
__HAL_DMA_DISABLE_IT(&hdma_usart1_rx,DMA_IT_HT);
// 接收数据，当空闲的时候结束并触发中断，可以不定长
HAL_UARTEx_ReceiveToIdle_DMA(&huart1,get_data,sizeof (get_data));
// 上面那个函数触发中断函数
void HAL_UARTEx_RxEventCallback(UART_HandleTypeDef *huart, uint16_t Size)
// 发送函数
HAL_UART_Transmit_DMA
```

## 注意事项
HAL库最好保证tx(发送)先初始化，在cubemx中，就先点tx的dma。
在代码中就是先init tx，如下代码块所示。
```
/* USART1 DMA DeInit */
HAL_DMA_DeInit(uartHandle->hdmatx);
HAL_DMA_DeInit(uartHandle->hdmarx);
```
否则可能导致无法发送(可能是互相存在依赖，仅在国产stm32f103c8t6上测试，其他芯片未知)

## 关于连接蓝牙模块
连接的时候注意查看一下默认的波特率，防止波特率不同导致异常。