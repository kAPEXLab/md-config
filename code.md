# UART Transmission and Reception

````c
uint8_t txMsg[] = "Enter Data:\r\n";
uint8_t rxData;
int main()
{
  HAL_UART_Transmit(&hlpuart1, txMsg, 13, HAL_MAX_DELAY);
  while (1)
      {
      /* -------- RX with check -------- */
      if (HAL_UART_Receive(&hlpuart1, &rxData, 1, 100) == HAL_OK)
	  {
         /* -------- Echo only when data received -------- */
          HAL_UART_Transmit(&hlpuart1, &rxData, 1, HAL_MAX_DELAY);
      }
      HAL_Delay(1000);
      }
	 }
}
````

# Timer Polling

````c
int main()
{
  uint16_t timer_val;
  // Start timer
  HAL_TIM_Base_Start(&htim6);
  // Get current time (microseconds)
  timer_val = __HAL_TIM_GET_COUNTER(&htim6);
  while (1)
  {
	  // If enough time has passed (1 second), toggle LED and get new timestamp
	     if (__HAL_TIM_GET_COUNTER(&htim6) - timer_val >= 10000)
	     {
	       HAL_GPIO_TogglePin(GPIOA,GPIO_PIN_5);
	       timer_val = __HAL_TIM_GET_COUNTER(&htim6);	     
		 }
  }

}
````
