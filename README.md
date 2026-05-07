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

## Simple Example
In USER CODE BEGIN 3 add:
```C
if (adcValue < 50)
	adcValue = 0; // Simple thresholding to avoid noise at low values
float potVoltage = analogToVoltage(adcValue); // Convert ADC value to voltage
float timeON = map(potVoltage, 0.0, 3.3, 0.0, htim3.Init.Period + 1); // Map voltage to duty cycle (0 to 100% of timer period)
__HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_3, timeON); // Update PWM duty cycle based on ADC reading
```

Note `htim3.Init.Period` is created by STM32CubeMX, it is the ARR found in the ADC settings.<br>
The "+1" depends on how you enter values for the ARR during set up, [explanation here](https://github.com/Thomas1B/STM_basicPWM/tree/main#setup-example-500-hz-pwm-tim3).<br>
For simplicity, you could just enter that ARR value if you don't plan on changing it.


##  Compact Example
This is in USER CODE BEGIN 3.
```C
if (adcValue < 50)
	adcValue = 0; // Simple thresholding to avoid noise at low values
// Directly set compare value based on ADC reading
__HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_3, adcValue * (htim3.Init.Period + 1) / 4095); 
```
The 4095 is based on a 12-bit ADC resolution.

## ESC Example

ESC input duty cycle: 1100us for reverse 100%, 1900us for forward 100%, and 1500us for stop.

This is in Private user code
```C
// This is the map function commonly used in Arduino
float map(float x, float in_min, float in_max, float out_min, float out_max) { // Utility function to map a value from one range to another
	return (x - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
}

float analogToVoltage(unsigned int val) {
	// 4095 is based on 12-bit ADC resolution
	return ((float) val * 3.3) / 4095;
}
```

This is in USER CODE BEGIN 3
```C
// Control for ESC
float timeON = map(potVoltage, 0.0, 3.3, 1100, 1900); // Map voltage to duty cycle percentage
__HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_3, timeON); // Update PWM duty cycle based on ADC reading
```

