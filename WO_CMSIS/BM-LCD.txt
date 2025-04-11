#include <stdint.h>

#define RCC  0x40021000
#define RCC_CR  (*(volatile uint32_t *)(RCC + 0x00))
#define RCC_CFGR  (*(volatile uint32_t *)(RCC + 0x04))
#define RCC_APB2ENR  (*(volatile uint32_t *)(RCC + 0x18))
#define RCC_AHBENR  (*(volatile uint32_t *)(RCC + 0x14))
#define GPIOC  0x40011000
#define GPIOC_CRH  (*(volatile uint32_t *)(GPIOC + 0x04))
#define GPIOC_BSRR  (*(volatile uint32_t *)(GPIOC + 0x10))
#define GPIOA  0x40010800
#define GPIOA_CRL  (*(volatile uint32_t *)(GPIOA + 0x00))
#define GPIOA_BSRR  (*(volatile uint32_t *)(GPIOA + 0x10))
#define FLASH_ACR     (*(volatile uint32_t*)0x40022000)
#define SYST_CSR     (*(volatile uint32_t*)0xE000E010)
#define SYST_RVR     (*(volatile uint32_t*) 0xE000E014)
#define SYST_CVR     (*(volatile uint32_t*)0xE000E018)



 void SystemClock_Config(void);
 void Pins_Init(void);
 void delay_ms(uint32_t ms);
 void lcd_Initiate(void);
 void string(const  char *s);
 void lcd_data(unsigned char data);
 void lcd_command(unsigned char cmd);


void SystemClock_Config(void) {
 RCC_CR |= (1<<0);
 while(!(RCC_CR & (1<<1)));
 FLASH_ACR |= (1<<1);
 FLASH_ACR |=(1<<4);
 while(!(FLASH_ACR & (1<<5)));
 RCC_CFGR &= ~(1<<16);
 RCC_CFGR |= (1<<18)|(1<<19)|(1<<20)|(1<<21);
 RCC_CR |= (1<<24);
 while(!(RCC_CR & (1<<25)));
 RCC_CFGR |= (1<<1);
 while(!(RCC_CFGR & (1<<3)));
 RCC_CFGR &= ~((1<<7)|(1<<10)|(1<<13));
 RCC_AHBENR |= (1<<10);


}
void SysTick_Init(void) {
	SYST_RVR = 8000 - 1;
	SYST_CVR = 0;
	SYST_CSR &= ~(1 << 2);
	SYST_CSR |= (1<<0);
}
void delay_ms(uint32_t ms) {
	for (uint32_t i = 0; i < ms; i++) {
	        while (!(SYST_CSR & (1 << 16)));
    }
}

void Pins_Init(void){
	RCC_APB2ENR |=(1<<4);
	RCC_APB2ENR |=(1<<2);
	GPIOC_CRH |=(1<<20)|(1<<24);
	GPIOC_CRH &= ~((1<<21)|(1<<22)|(1<<23)|(1<<25)|(1<<26)|(1<<27));
	GPIOA_CRL = 0x11111111;

}
void lcd_Initiate(){
    lcd_command(0X38);
    lcd_command(0X06);
    lcd_command(0x0c);
    lcd_command(0X01);
}

void lcd_data(unsigned char data){
	GPIOA_BSRR = 0Xffff0000;
	delay_ms(2);
	GPIOA_BSRR = data;
	GPIOC_BSRR |= (1<<13);
	GPIOC_BSRR |= (1<<14);
	delay_ms(5);
	GPIOC_BSRR |= (1<<(16+14));
}
void lcd_command(unsigned char cmd){
	GPIOA_BSRR = 0Xffff0000;
	delay_ms(2);
	GPIOA_BSRR = cmd;
	GPIOC_BSRR |=(1<<(13+16));
	GPIOC_BSRR |= (1<<14);
	delay_ms(5);
	GPIOC_BSRR |= (1<<(16+14));
}
void string(const char *s){
	while(*s){
		lcd_data(*s++);
	}
}
int main(void){
	SystemClock_Config();
    SysTick_Init();
	Pins_Init();
	lcd_Initiate();

     while(1){
	lcd_command(0x80);
	string("hello");
	lcd_command(0Xc0);
	string("world");

	}
}
