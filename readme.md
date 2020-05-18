学习过程project
1.01greenled 
使LED（PIN_13）灯循环闪烁，在club32上配置PC13端口，系统时钟。
完成工程建立后，在keil5上打开，在main.c的while敲入代码
HAL_Delay(500);
HAL_GPIO_TogglePin(GPIOC,GPIO_PIN_13);

2.KEY_LED01 
使用按键控制led灯（PIN 13）亮灭。
2.1GPIO配置
2.1.1 KEY1（PB2 、上拉电阻（低电平有效）--亮）
2.1.2 KEY2(PB3、上拉电阻（低电平有效）--灭)
//KEY control LED 
		if (HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_2)==GPIO_PIN_RESET)//判断按键KEY1是否被按下，查询按键KEY1低电平
    {
      /* code */
			HAL_Delay(10);//延时10MS=消抖
			if (HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_2)==GPIO_PIN_RESET)//判断按键KEY1是否被按下查询按键KEY1低电平
			{
				HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13,GPIO_PIN_SET);//按下-则使LED亮
			}
			
    }
		if (HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_3)==GPIO_PIN_RESET)//判断按键KEY2是否被按下=查询按键KEY2低电平？
    {
      /* code */
			HAL_Delay(10);//延时10MS=消抖
			if (HAL_GPIO_ReadPin(GPIOB,GPIO_PIN_3)==GPIO_PIN_RESET)//判断按键KEY2是否被按下？=查询按键KEY2低电平?
			{
				HAL_GPIO_WritePin(GPIOC,GPIO_PIN_13,GPIO_PIN_RESET);//按下-则使LED灭
			}
			
    }



3.LED-EXIT-KEY
利用中断函数控制LED亮灭
3.1GPIO端口配置与2 KEY_LED01 一样，
		if(0 == HAL_GPIO_ReadPin(KEY1_GPIO_Port, KEY1_Pin))//判断KEY1是否被按下（低电平）
        {
          HAL_Delay(200);
          HAL_GPIO_WritePin(LED_GPIO_Port,LED_Pin,GPIO_PIN_SET);//亮
        }
    if(0 == HAL_GPIO_ReadPin(KEY2_GPIO_Port, KEY2_Pin))
        {
          HAL_Delay(200);
          HAL_GPIO_WritePin(LED_GPIO_Port,LED_Pin,GPIO_PIN_RESET);//灭
        }
		
3.2NVIC配置 配置优先级2 bits , EXIT line2 interrupt 使能 优先级1 ,  EXIT line3 interrupt 使能 优先级2 , 优先级越小，越先。。。

3.3在gpio.c最后加入
/* USER CODE BEGIN 2 */
/**
 * @brief    EXIT中断回调函数
 * @param GPIO_Pin —— 触发中断的引脚
 * @retval    none
*/
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    /* 判断哪个引脚触发了中断 */
    switch(GPIO_Pin)
    {
        case GPIO_PIN_2:
            /* 处理GPIO2发生的中断 */
            //点亮LED
            HAL_GPIO_WritePin(LED_GPIO_Port,LED_Pin,GPIO_PIN_SET);
            break;
        case GPIO_PIN_3:
            /* 处理GPIO3发生的中断 */
            //熄灭LED
            HAL_GPIO_WritePin(LED_GPIO_Port,LED_Pin,GPIO_PIN_RESET);
            break;
        default:
            break;
    }
}
/* USER CODE END 2 */

4.TIM2_LED
利用TIM2的内部时钟中断实现led闪烁1s
4.1配置PC13端口为输出
4.2配置TIM2为内部时钟，预分频2000-1，计数周期10000（1s），使能中断。APBI提供时钟为20MHz，生成工程。
4.3代码实现
  1主函数内：
  1、HAL_TIM_Base_Start_IT(&htim2);//触发TIM2中断定时器
  2、while (1)
  {
    /* USER CODE BEGIN 3 */
		HAL_Delay(8000);//延时8s--用于测试LED闪烁是否受延时影响
  }
  /* USER CODE END 3 */
  2调用函数
  /* USER CODE BEGIN 4 */
 /*调用周期函数，定时1S中断，实现1s闪烁的效果*/
 void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
 {
  //LED电平翻转
  HAL_GPIO_TogglePin(LED_GPIO_Port,LED_Pin);

 }
 /* USER CODE END 4 */
4.4运行结果
TIM2的中断不受延时影响，实现1s中断LED闪烁。

5、USART PROJECT
普通收发模式
1，选择芯片：stm32l431rc
2，配置LED-PC13-gpio_output
3，选择USART1 : mode-asynchronous   配置：115200或9600 
4，配置系统时钟：80MHz
5，gpio口-pc13默认，PA9发送引脚：默认，PA10接收引脚：默认。
6，生成工程，MDK打开工程文件，编译，
7，在stm32l4xx_hal_uart.h 的1609，1610
HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout);
HAL_UART_Receive(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout);
复制到main.c的while语句里（测试函数）
	uint8_t Rdata;	
    HAL_Delay(3000);
    if(HAL_UART_Receive(&huart1, &Rdata, 1, 0)==HAL_OK)
		{
            HAL_GPIO_TogglePin(LED_GPIO_Port,LED_Pin);
		    HAL_UART_Transmit(&huart1, &Rdata, 1, 0);
		}
8、在usart.c编写fputc函数：将数据打印到串口助手上。需加头文件stdio.h
int fputc (int ch,FILE *f)
{
	uint8_t temp[1]={ch};
	{
		HAL_UART_Transmit(&huart1,temp,1,2);
	}
	return HAL_OK;
}
9、在主函数应用，注意先添加头文件stdio.h
printf("welcome to usart1! \r\n");

10、延时会影响串口收发信息。


中断IRQ模式
1、在普通模式上，使能串口中断 生成代码。
2、宏定义
#define UART1_IRQ
uint8_t TdataIRQ[]={"welcome to UART1_IRQ"};
3、
#ifdef UART1_IRQ
    if(HAL_UART_Receive_IT(&huart1, &Rdata, 1)==HAL_OK)
		{
		    HAL_UART_Transmit_IT(&huart1, TdataIRQ,sizeof(TdataIRQ));
		}
    #endif
4、while里，测试函数
	#ifdef UART1_IRQ
		HAL_GPIO_TogglePin(LED_GPIO_Port,LED_Pin);
	  printf("URAT1_IRQ test!\r\n ");
		HAL_Delay(3000);
		#endif
5、在main.cn中调用 回调函数
/* USER CODE BEGIN 4 */
void HAL_UART_TxHalfCpltCallback(UART_HandleTypeDef *huart)
{
	#ifdef UART1_IRQ
	HAL_UART_Transmit(&huart1, &Rdata, 1,0xff);
	HAL_UART_Receive_IT(&huart1, &Rdata, 1);
	#endif
}
/* USER CODE END 4 */
6、中断不受延时影响。

DMA模式 （与IRQ类似）
1、	在以上模式上，配置DMA，add—USART_RX—mode：Circular  。
add—USART_TX—mode：normal –ok 生成代码。
2、	宏定义
#define UART1_DMA
uint8_t TdataDMA[]={"welcome to UART1_DMA"};
3、
	#ifdef UART1_DMA
   if(HAL_UART_Receive_DMA(&huart1, &Rdata, 1)==HAL_OK)
	{
	    HAL_UART_Transmit_DMA(&huart1, TdataDMA,sizeof(TdataDMA));
	}
    #endif
4、
#ifdef UART1_DMA
		HAL_GPIO_TogglePin(LED_GPIO_Port,LED_Pin);
	  printf("URAT1_DMA test!\r\n ");
		HAL_Delay(3000);
		#endif
5、
#ifdef UART1_DMA
	HAL_UART_Transmit_DMA(&huart1, &Rdata, 1); //中断特别快，容易丢失数据
	HAL_UART_Receive_DMA(&huart1, &Rdata, 1);
	#endif
HAL_UART_Transmit_DMA和HAL_UART_Receive_DMA优先级一样，同时会产生冲突。

修改HAL_UART_Transmit优先级比HAL_UART_Receive_DMA低，运行速度慢，
#ifdef UART1_DMA
	HAL_UART_Transmit(&huart1, &Rdata, 1,0xff);
	HAL_UART_Receive_DMA(&huart1, &Rdata, 1);
	#endif
6、中断不受延时影响




