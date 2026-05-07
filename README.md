# STM PWM Controller with ADC using DMA 

Simple script that demonstrates how to use PWM to control the brightness of an LED while simultaneously reading an 
analog value from a potentiometer using ADC (Analog to Digital Converter) with DMA (Direct Memory Access).

## Setup

- Follow [this](https://github.com/Thomas1B/adcDMA) for setting up ADC using DMA.
- Follow [this](https://github.com/Thomas1B/STM_basicPWM) for setting up PWM.

In USER CODE BEGIN 2 add:
``` C
	HAL_ADC_Start_DMA(&hadc1, (uint32_t*) &adcValue, 1); // Start ADC in DMA mode, storing result in adcValue
	HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_3); // Start PWM on TIM3 Channel 3
```

In USER CODE BEGIN 3 add:
```C
	if (adcValue < 50)
		adcValue = 0; // Simple thresholding to avoid noise at low values
	float potVoltage = analogToVoltage(adcValue); // Convert ADC value to voltage
	float timeON = map(potVoltage, 0.0, 3.3, 0.0, htim3.Init.Period + 1); // Map voltage to duty cycle (0 to 100% of timer period)
	__HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_3, timeON); // Update PWM duty cycle based on ADC reading
```

Note `htim3.Init.Period` is created by STM32CubeMX, it is the ARR found in the ADC settings.<br>


