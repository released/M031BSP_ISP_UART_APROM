# M031BSP_ISP_UART_APROM
 M031BSP_ISP_UART_APROM

update @ 2022/03/23

1. Scenario notice:

	- Boot loader project : ISP_UART 
	
		- under sct file (uart_iap.sct) , will allocate flash size 
		
				LDROM_Bootloader.bin : 0x100000 ~ 0xFFF (default LDROM size : 4K)
			
				APROM_Bootloader.bin : 0x1E000 0x1800 (reserve 6K size , to store extra boot loader code 
	
		- when power on , will check power on source (ex : power on reset , nReset , from application code)
	
		- use CRC to calculate Application code checksum (length : 0x1E000-4 : 0x1DFFC )
		
		- load Application code checksum , from specific address (at 0x1E000 last 4 bytes : 0x1DFFC)
		
		- if two checksum result are different , will stuck in Boot loader , and wait for ISP tool hand shaking
		
![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/error_checksum_stay_in_boot_loader.jpg)		
		
		- if two checksum result are the same , will jump to Application code

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/boot_from_LDROM_to_APROM.jpg)	

		- under ISP tool , when upgrade finish 
		
![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/LDROM_upgrade_finish.jpg)			

		- if reset from application code , will entry timeout counting , jump to application code if not receive ISP tool command		
	
	- Application code project : AP
	
		- use SRecord , to calculate application code checksum , add binary to hex , by SRecord tool
		
		- SRecord file : srec_cat.exe 

		- under generateChecksum.bat will execute generateChecksum.cmd , generateCRCbinary.cmd , generateCRChex.cmd
	
			- generateChecksum.cmd : calculate checksum by load the original binary file , and display on KEIL project
		
			- generateCRCbinary.cmd : calculate checksum by load the original binary file , and fill 0xFF , range up to 0x1D000
		
			- generateCRChex.cmd : conver binary file into hex file
		
		- check sum calculate will start from 0 to 0x1E000-4 : 0x1DFFC , and store in 0x1DFFC , the last 4 bytes 
		
		- at KEIL output file , file name is APROM_application , under \obj folder , 
	
			which mapping to generateChecksum.cmd , generateCRCbinary.cmd , generateCRChex.cmd
	
			modify the file name in KEIL project , also need to modify the file name in these 3 generate***.cmd

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/APROM_KEIL_output_file.jpg)

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/APROM_SRecord_cmd_file.jpg)
		
		- after project compile finish , binary size will be 120K (total application code size : 0x1E000)
		
		- under terminal , use keyboard , 'z' , 'Z' , will write specific value in EMULATE EEPROM , and return to boot loader
		
		- use ISP tool , to programming Application code project binary (120K) , when under Boot loader flow

		- with ISP tool , set connect and wait UART connection from APROM jump to LDROM

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/ISP_connect.jpg)

		- if plan to use KEIL to programming flash with CRC , refer to below setting
		
![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/program_by_KEIL.jpg)	
		
	- reserve data flash address : 0x1F800 , 2K
	
2. Flash allocation

	- LDROM_Bootloader.bin : 0x100000 ~ 0xFFF
	
	- APROM_Bootloader.bin : 0x1E000 0x1800
	
	- Application code START address : 0x00000
	
	- Data flash : 0x1F800 , 2K
	
	- Chcecksum storage : 0x1DFFC

3. Function assignment

	- debug port : UART1 (PB2 , PB3) , in Boot loader an Application code project
	
	- ISP UART port : UART0 (PB12 , PB13) , in Boot loader project
	
	- enable EMULATE EEPROM and CRC , Timer1 module
	
4. Need to use ICP tool , to programm boot loader project file (LDROM_Bootloader.bin , APROM_Bootloader.bin)

below is boot loader project , Config setting 

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/LDROM_ICP_config.jpg)

below is boot loader project , ICP programming setting 

- LDROM_Bootloader.bin : under LDROM

- APROM_Bootloader.bin : 0x1E000

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/LDROM_ICP_update.jpg)

5. under Application code KEIL project setting 

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/APROM_KEIL_checksum_calculate.jpg)

in Application project , press 'z' , 'Z' will reset to Boot loader 

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/APROM_press_Z.jpg)

below is log message , from boot loader , to application code

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/boot_from_LDROM_to_APROM.jpg)

6. under boot loader project , below is sct file content

![image](https://github.com/released/M031BSP_ISP_UART_APROM/blob/main/LDROM_KEIL_sct.jpg)


