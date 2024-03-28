# Custom Bootloader for stm322f303re

💡 We made explanation earlier u can check this link (https://github.com/Rabie45/Bootloader_STM32F303RE)

## First lets Explain what we are going to do :

- The task is to create a program start with application with multiple options one for bootloader command and the other for user application to something like that we would use Flash module organization used before create multiple section in memory each section is full program it self (vector table + system init + main) once the program start choose the program u want and the bootloader will through u to the starting address holding this program
- the figure show the addresses of each program and how to mange the space
  ![2_drawio](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/fcdc2f07-729f-4b89-b2b0-b3356bd0ade5)

And this is the workflow

![3_drawio](https://github.com/Rabie45/Bootloader_STM32F303RE/assets/76526170/32b13d89-d4e2-47d3-8c96-d2fc33a091bd)

## First Create the bootloader project

- initialize GPIO
- initialize UART2
- Check if the button is pressed this mean go to bootloader mode to change UserApp
- not pressed go to default user app
  This program starts with default memory section which is 0x8000000

## second Create the UserApp project

- pretty simple
- initialize GPIO
- initialize UART2
- toggle the LED print this word over UART "Hey from user data:Toggling"
  This program starts with memory address 0x8008000
![66](https://github.com/Rabie45/Custom_bootloader_Stm32f303RE/assets/76526170/284809b3-8da9-4332-aa98-a01ccbb4d224)

  ### to start from this section using keil Uvison follow this steps :
  ![Screenshot (80)](https://github.com/Rabie45/Custom_bootloader_Stm32f303RE/assets/76526170/23402e42-68fd-4bbc-8dcf-097abe1aaa57)
  ![Screenshot (81)](https://github.com/Rabie45/Custom_bootloader_Stm32f303RE/assets/76526170/ffca35ef-35e4-4e55-8099-a6026bbeedc4)
  ![Screenshot (82)](https://github.com/Rabie45/Custom_bootloader_Stm32f303RE/assets/76526170/10da926f-25c5-4a75-92e8-2648a264f510)
  ![Screenshot (83)](https://github.com/Rabie45/Custom_bootloader_Stm32f303RE/assets/76526170/b7f27f9a-c018-4bfa-b52e-61d6a52fc64c)
  ![Screenshot (84)](https://github.com/Rabie45/Custom_bootloader_Stm32f303RE/assets/76526170/5bb79763-883b-4b91-a8e6-707276a94009)


Dont forget to relocate VTOR (vector table register ) to new address 0x8008000

## How to make bootloader jump to user app:
  - go to system_stm23f3xx.c 
![21](https://github.com/Rabie45/Custom_bootloader_Stm32f303RE/assets/76526170/e881f18e-c5a1-40bf-8c15-3d790c5e1d57)
  - Search for VTOR
![22](https://github.com/Rabie45/Custom_bootloader_Stm32f303RE/assets/76526170/faaa2017-60f0-4f22-9603-3ea64c88dda7)
  - Change the offset  
![24](https://github.com/Rabie45/Custom_bootloader_Stm32f303RE/assets/76526170/fbc073d5-79b2-40ef-8d06-5395b188dfc2)

- After loading each program in memory the bootloader program will start with default user app printing the line
- If the button is pressed the bootloader will go to to something then go to userapp program address
- void UserApp_Mode() in bootloader responsible to to this

``` void (*appHandler) (void);  // create a fuction pointer
	uint32_t MSP_VAL=*(volatile uint32_t * )(FLASH_PAGE2_0x08008000U)  //get the memory address to load in MSP
	__set_MSP(MSP_VAL);  // use this fuction to set the MSP value
	uint32_t resetHandler =*(volatile uint32_t *)(FLASH_PAGE2_0x08001800U +4 ); // alwayes the next instruction is the Reset handler
	appHandler=(void*) resetHandler;
	appHandler(); // Call the function 
``` 
