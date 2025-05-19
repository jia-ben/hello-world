#1

## 

#include "kwl_driver.h"

/* MODEBUS USARTX configuration definitions*/
#define RS485_USARTX                                USART1
#define RS485_USARTX_GPIO_CLK                       RCU_USART1
#define RS485_USARTX_IRQHandler                     USART1_IRQn
/* RS485 TX */
#define RS485_USARTX_TX_GPIO_PORT                   GPIOA
#define RS485_USARTX_TX_GPIO_PIN                    GPIO_PIN_2
#define RS485_USARTX_TX_GPIO_CLK                    RCU_GPIOA
/* RS485 RX */
#define RS485_USARTX_RX_GPIO_PORT                   GPIOA
#define RS485_USARTX_RX_GPIO_PIN                    GPIO_PIN_3
#define RS485_USARTX_RX_GPIO_CLK                    RCU_GPIOA
/* RS485 PIN */
#define RS485_USARTX_RE_GPIO_PORT                   GPIOA
#define RS485_USARTX_RE_GPIO_PIN                    GPIO_PIN_1
#define RS485_USARTX_RE_GPIO_CLK                    RCU_GPIOA
/* mode config */
#define RS485_USARTX_PARITY                         USART_PM_NONE
#define RS485_USARTX_BIT_LEN                        USART_WL_8BIT
#define RS485_USARTX_STOP_BIT                       USART_STB_1BIT
#define RS485_USART_CTS                             USART_CTS_DISABLE

UART_HandleTypeDef rs458_handler; /* RS485控制句柄(串口) */

void UsartxInit(void);
void Rs485Init(void)
{
    /* CONTROL PIN configuration */
    rcu_periph_clock_enable(RS485_USARTX_RE_GPIO_CLK);
    gpio_init(RS485_USARTX_RE_GPIO_PORT, GPIO_MODE_OUT_PP, GPIO_OSPEED_50MHZ, RS485_USARTX_RE_GPIO_PIN);
    
    /* USART configuration */
    UsartxInit();
    rs458_handler.ErrorCode = 0;
    rs458_handler.gState = HAL_UART_STATE_READY;
    rs458_handler.RxState = HAL_UART_STATE_READY;
    /* USART interrupt configuration */
    nvic_irq_enable(RS485_USARTX_IRQHandler, 0, 0);
    
    /* 默认关闭发送中断 */
    usart_interrupt_disable(USART0, USART_INT_TBE);
    /* 默认关闭接收中断 */
    usart_interrupt_disable(USART0, USART_INT_RBNE);
    
}

void UsartxInit(void)
{
    /* enable GPIO clock */
    rcu_periph_clock_enable(RS485_USARTX_RX_GPIO_CLK);
    rcu_periph_clock_enable(RS485_USARTX_TX_GPIO_CLK);
    
    /* enable USART clock */
    rcu_periph_clock_enable(RS485_USARTX_GPIO_CLK);

    /* connect port to USARTx_Tx */
    gpio_init(RS485_USARTX_TX_GPIO_PORT, GPIO_MODE_AF_PP, GPIO_OSPEED_50MHZ, RS485_USARTX_TX_GPIO_PIN);

    /* connect port to USARTx_Rx */
    gpio_init(RS485_USARTX_RX_GPIO_PORT, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, RS485_USARTX_RX_GPIO_PIN);
    
    /* USART configure */
    usart_deinit(RS485_USARTX);
    
    /* 设置波特率 */
    usart_baudrate_set(RS485_USARTX, 9600U);
    
    /* 设置校验位 */
    usart_parity_config(RS485_USARTX, RS485_USARTX_PARITY);
    
    /* 设置数据位 */
    usart_word_length_set(RS485_USARTX, RS485_USARTX_BIT_LEN);
    
    /* 设置停止位 */
    usart_stop_bit_set(RS485_USARTX, RS485_USARTX_STOP_BIT);
    
    /* 其他 */
    {
        /* set the receiver timeout threshold of USART0 */
//        usart_receiver_timeout_threshold_config(USART0, 9600*3);
        
        /* configure USART0 hardware flow control RTS */
        usart_hardware_flow_cts_config(USART0,RS485_USART_CTS);
        
        /* USART0 DMA enable for reception */
//        usart_dma_receive_config(USART0, USART_DENR_ENABLE);
        
        /* USART0 DMA enable for transmission */
//        usart_dma_transmit_config(USART0, USART_DENT_ENABLE);
        
        /* 失能接收器 */
        usart_receive_config(RS485_USARTX, USART_RECEIVE_DISABLE);
        
        /* 失能发送器 */
        usart_transmit_config(RS485_USARTX, USART_TRANSMIT_DISABLE);
    }
    
    /* 使能USART */
    usart_enable(RS485_USARTX);
}

/* 
Serial port sending process:
    1.enable USART
    2.set WL
    3.set STB
    4.set if DMA
    5.set bound
    6.set TEN           enable transmitter
    7.wait TBE = 1      send empty
    8.write DR reg      
    9.wait TC = 1       send ok
    10.Cycle 7-8 if no DMA 
*/
USART_STATUS_StatusTypeDef Rs485SendData(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout)
{
    uint8_t  *pdata8bits;
    uint32_t tickstart = 0U;
    
    usart_transmit_config(RS485_USARTX, USART_TRANSMIT_ENABLE);
    /* Check that a Tx process is not already ongoing */
    if (huart->gState == HAL_UART_STATE_READY)
    {
        if ((pData == NULL) || (Size == 0U))
        {
          return  USART_STATUS_ERROR;
        }

        huart->ErrorCode = 0;
        huart->gState = HAL_UART_STATE_BUSY_TX;

        /* Init tickstart for timeout managment */
//        tickstart = HAL_GetTick();

        huart->TxXferSize = Size;
        huart->TxXferCount = Size;

        pdata8bits  = pData;

        while (huart->TxXferCount > 0U)
        {
            if (UART_WaitOnFlagUntilTimeout(huart, UART_FLAG_TXE, RESET, tickstart, Timeout) != HAL_OK)
            {
                return USART_STATUS_TIMEOUT;
            }
            
            huart->Instance->DR = (uint8_t)(*pdata8bits & 0xFFU);
            pdata8bits++;
            
            huart->TxXferCount--;
        }

        if (UART_WaitOnFlagUntilTimeout(huart, UART_FLAG_TC, RESET, tickstart, Timeout) != HAL_OK)
        {
          return USART_STATUS_TIMEOUT;
        }

        /* At end of Tx process, restore huart->gState to Ready */
        huart->gState = HAL_UART_STATE_READY;

        return USART_STATUS_OK;
    }
    else
    {
        return USART_STATUS_BUSY;
    }
    
    usart_transmit_config(RS485_USARTX, USART_TRANSMIT_DISABLE);
//    uint8_t count;
//    /* wait tx empty */
//    while(usart_flag_get(USART0,USART_FLAG_TBE) == RESET);
//    
//    for(count=0; count <= *len; count++)
//    {
//        /* wait TC==1 */
//        while(RESET == usart_flag_get(USART0,USART_FLAG_TC));
//        usart_data_transmit(RS485_USARTX, buffer[count]);
//    }
//    while(RESET == usart_flag_get(USART0,USART_FLAG_TC));
}

USART_STATUS_StatusTypeDef Rs485RecData()
{
    
}

## 2
/*
    冷水机通信 
    485 modbus-rtu 
    ljb
*/
#ifndef RS485_DRIVER_H
#define RS485_DRIVER_H

#include "gd32f30x.h"
#include <stdint.h>
#include <stdio.h>
/**
  * @brief Status structures definition
  */
typedef enum
{
  USART_STATUS_OK       = 0x00U,
  USART_STATUS_ERROR    = 0x01U,
  USART_STATUS_BUSY     = 0x02U,
  USART_STATUS_TIMEOUT  = 0x03U
} USART_STATUS_StatusTypeDef;

typedef enum
{
    HAL_UART_STATE_RESET = 0,
    HAL_UART_STATE_READY,
    HAL_UART_STATE_BUSY,
    HAL_UART_STATE_BUSY_TX,
    HAL_UART_STATE_BUSY_RX,
    HAL_UART_STATE_BUSY_TX_RX,
    HAL_UART_STATE_TIMEOUT,
    HAL_UART_STATE_ERROR
} HAL_UART_StateTypeDef;

/**
  * @brief  UART handle Structure definition
  */
typedef struct __UART_HandleTypeDef
{
    uint8_t                       *pTxBuffPtr;
    uint16_t                      TxXferSize;
    __IO uint16_t                 TxXferCount;
    
    uint8_t                       *pRxBuffPtr;
    uint16_t                      RxXferSize;
    __IO uint16_t                 RxXferCount;
    
    __IO HAL_UART_StateTypeDef    gState;
    __IO HAL_UART_StateTypeDef    RxState;
    __IO uint32_t                 ErrorCode;
} UART_HandleTypeDef;

#define USARTX_BUFFSIZE     32

void Rs485Init();
USART_STATUS_StatusTypeDef Rs485SendData(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout);
USART_STATUS_StatusTypeDef Rs485RecData();


#endif