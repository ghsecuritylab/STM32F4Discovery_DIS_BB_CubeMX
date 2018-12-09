# Working STM32F4Discovery + STM32F4DIS-BB (lwIP v2.0.3) CubeMX (HAL) project.

For all purposes sake, the things required to make it work (already included in the project):

**In CubeMX:**
1. Disable auto negotiation and set to 100MBits/s - full duplex (doesn't work for some reason).
2. Set the PHY Address to 0 (instead of 1).
3. Set the PHY Reset delay a bit higher (otherwise the PHY won't start): 0x000008FF
4. Set the PHY Read Timeout to: 0x0004FFFF
5. Set the PHY Write Timeout to: 0x0004FFFF
6. Set the PHY special control/status register offset to: 0x31
7. Set pin PE2 to GPIO_Output (ETH_RST_PIN) - (we configure this the code, see below).
    
**In the code:**
1. Src/ethernetif.c - in function: void HAL_ETH_MspInit(ETH_HandleTypeDef* ethHandle) - within the USER CODE section (bottom of the function), add the following code snippet:
```    
//PHY reset for the STM32F4DIS-BB board.
GPIO_InitTypeDef GPIO_InitStruct;
GPIO_InitStruct.Pin = GPIO_PIN_2;
GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
HAL_GPIO_Init(GPIOE, &GPIO_InitStruct);
HAL_GPIO_WritePin(GPIOE, GPIO_PIN_2, GPIO_PIN_RESET);
HAL_Delay(50);
HAL_GPIO_WritePin(GPIOE, GPIO_PIN_2, GPIO_PIN_SET);
HAL_Delay(50); 
```

And of course not to forget in the main loop: MX_LWIP_Process();

**Notes for the provided project example:**
1. You can change the static IP address at the top of Src/main.c. If you wan't to enable DHCP, comment line 130:Src/main.c (IP setter) and uncomment line 141:Src/lwip.c (dhcp_start).
2. SNMPv2c agent (lwIP apps) is enabled.
