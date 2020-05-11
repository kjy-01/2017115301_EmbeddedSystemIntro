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
