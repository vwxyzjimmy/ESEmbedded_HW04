HW04
===
This is the hw04 sample. Please follow the steps below.

# Build the Sample Program

1. Fork this repo to your own github account.

2. Clone the repo that you just forked.

3. Under the hw04 dir, use:

	* `make` to build.

	* `make flash` to flash the binary file into your STM32F4 device.

	* `make clean` to clean the ouput files.

# Build Your Own Program

1. Edit or add files if you need to.

2. Make and run like the steps above.

3. Please avoid using hardware dependent C standard library functions like `printf`, `malloc`, etc.

# HW04 Requirements

1. Please practice to reference the user manuals mentioned in [Lecture 04], and try to use the user button (the blue one on the STM32F4DISCOVERY board).

2. After reset, the device starts to wait until the user button has been pressed, and then starts to blink the blue led forever.

3. Try to define a macro function `READ_BIT(addr, bit)` in reg.h for reading the value of the user button.

4. Push your repo to your github. (Use .gitignore to exclude the output files like object files, executable files, or binary files)

5. The TAs will clone your repo, build from your source code, and flash to see the result.

[Lecture 04]: http://www.nc.es.ncku.edu.tw/course/embedded/04/

--------------------

- [ ] **If you volunteer to give the presentation (demo) next week, check this.**

--------------------

Embedded HW04
===
## HW04 Requirements
Please practice to reference the user manuals mentioned in Lecture 04, and try to use the user button (the blue one on the STM32F4DISCOVERY board).

After reset, the device starts to wait until the user button has been pressed, and then starts to blink the blue led forever.

Try to define a macro function READ_BIT(addr, bit) in reg.h for reading the value of the user button.

## HW
### Step1 Find the defined port of the user button in 6.4 Push Button
[UM1472 User manual Discovery kit with STM32F407VG MCU](http://www.nc.es.ncku.edu.tw/course/embedded/pdf/STM32F4DISCOVERY.pdf)
![](https://github.com/vwxyzjimmy/ESEmbedded_HW04/blob/master/hw4_figure/push_button.JPG)
User button connected to PA0.
### Step2 Configure the memory to set up the peripherals.
1. From [RM0090 Reference manual STM32F407](http://www.nc.es.ncku.edu.tw/course/embedded/pdf/STM32F407_Reference_manual.pdf)

Set up RCC to enable the PA0.
![](https://github.com/vwxyzjimmy/ESEmbedded_HW04/blob/master/hw4_figure/rccbase.JPG)

2. From Port bit configuration table,Set up GPIO moder register to input mode.
![](https://github.com/vwxyzjimmy/ESEmbedded_HW04/blob/master/hw4_figure/gpiosetup.JPG)
![](https://github.com/vwxyzjimmy/ESEmbedded_HW04/blob/master/hw4_figure/moder.JPG)
3. From [ UM1472 User manual Discovery kit with STM32F407VG MCU Figure 14. Peripherals](http://www.nc.es.ncku.edu.tw/course/embedded/pdf/STM32F4DISCOVERY.pdf),the user button's voltage default to pull down.

![](https://github.com/vwxyzjimmy/ESEmbedded_HW04/blob/master/hw4_figure/button_layout.JPG)

So set up GPIO pull up/pull down register to pull down.

![](https://github.com/vwxyzjimmy/ESEmbedded_HW04/blob/master/hw4_figure/pupdr.JPG)

4. Read the GPIO input data register to detect the trigger voltage.
![](https://github.com/vwxyzjimmy/ESEmbedded_HW04/blob/master/hw4_figure/idr.JPG)
Using the macro `REG()` to implement `READ_BIT`.
```c
#define READ_BIT(addr, bit) (REG(addr) & (UINT32_1 << (bit)))
```

## Code
`reg.h`
```c
#define READ_BIT(addr, bit) (REG(addr) & (UINT32_1 << (bit)))
```
```c
#define GPIOx_IDR_OFFSET 0x10
#define ODy_BIT(y) (y)
```

`blink.c`
```c
void gpio_input_init(unsigned int port, unsigned int input){


	SET_BIT(RCC_BASE + RCC_AHB1ENR_OFFSET, GPIO_EN_BIT(GPIO_PORTA));

	CLEAR_BIT(GPIO_BASE(GPIO_PORTA) + GPIOx_MODER_OFFSET, MODERy_1_BIT(input));
	CLEAR_BIT(GPIO_BASE(GPIO_PORTA) + GPIOx_MODER_OFFSET, MODERy_0_BIT(input));

	SET_BIT(GPIO_BASE(GPIO_PORTA) + GPIOx_PUPDR_OFFSET, PUPDRy_1_BIT(input));
	CLEAR_BIT(GPIO_BASE(GPIO_PORTA) + GPIOx_PUPDR_OFFSET, PUPDRy_0_BIT(input));
}
```
```c
void blink(unsigned int led)
{
	led_init(led);
	unsigned int button = 14;
	led_init(button);
	gpio_input_init(GPIO_PORTA, 0);
	unsigned int i ;
	while(!READ_BIT(GPIO_BASE(GPIO_PORTA) + GPIOx_IDR_OFFSET, ODy_BIT(0))){
		//set GPIOD led pin
		SET_BIT(GPIO_BASE(GPIO_PORTD) + GPIOx_BSRR_OFFSET, BSy_BIT(14));
		for (i = 0; i < 100000; i++)
			;
		//reset GPIOD led pin
		SET_BIT(GPIO_BASE(GPIO_PORTD) + GPIOx_BSRR_OFFSET, BRy_BIT(14));
		for (i = 0; i < 100000; i++)
			;
	};
	while (1)
	{
		//set GPIOD led pin
		SET_BIT(GPIO_BASE(GPIO_PORTD) + GPIOx_BSRR_OFFSET, BSy_BIT(led));
		for (i = 0; i < 100000; i++)
			;
		//reset GPIOD led pin
		SET_BIT(GPIO_BASE(GPIO_PORTD) + GPIOx_BSRR_OFFSET, BRy_BIT(led));
		for (i = 0; i < 100000; i++)
			;
	}
}
```
