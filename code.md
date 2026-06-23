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

# ADC Polling

````c
int main(void)
{
 
  char msg[50];
  while (1)
  {
	  HAL_ADC_Start(&hadc1);
	  /* Poll for ADC conversion completion */
	  	 if (HAL_ADC_PollForConversion(&hadc1, 100) == HAL_OK)
	  	 {
 
 
	  		uint32_t raw = HAL_ADC_GetValue(&hadc1);
 
	  		// Convert raw to millivolts (mV)
	  	    uint32_t voltage_mv = (raw * 3300) / 4095;  // 3.3V = 3300mV
 
 
	  	  // LM35 gives 10mV per °C
	  	    uint32_t Temp = voltage_mv / 10;
 
 
	  	 sprintf(msg, "Temp = %ld °C\r\n", Temp);
	  	 HAL_UART_Transmit(&huart2, (uint8_t *)msg, strlen(msg), HAL_MAX_DELAY);
 
	  	  }
	  	HAL_Delay(1000);
 
  }
}
````
