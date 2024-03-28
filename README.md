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

  ### to start from this section using keil Uvison follow this steps :

Dont forget to relocate VTOR (vector table register ) to new address 0x8008000

## How to make bootloader jump to user app

- After loading each program in memory the bootloader program will start with default user app printing the line
- If the button is pressed the bootloader will go to to something then go to userapp program address
- void UserApp_Mode() in bootloader responsible to to this

````void (*appHandler) (void);  // create a fuction pointer
	uint32_t MSP_VAL=*(volatile uint32_t * )(FLASH_PAGE2_0x08008000U)  //get the memory address to load in MSP
	__set_MSP(MSP_VAL);  // use this fuction to set the MSP value
	uint32_t resetHandler =*(volatile uint32_t *)(FLASH_PAGE2_0x08001800U +4 ); // alwayes the next instruction is the Reset handler
	appHandler=(void*) resetHandler;
	appHandler(); // Call the function ```
````
