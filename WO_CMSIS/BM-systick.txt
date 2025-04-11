#include <stdint.h>

#define RCC  0x40021000
#define RCC_CR  (*(volatile uint32_t *)(RCC + 0x00))
#define RCC_CFGR  (*(volatile uint32_t *)(RCC + 0x04))
#define RCC_APB2ENR  (*(volatile uint32_t *)(RCC + 0x18))
#define RCC_AHBENR  (*(volatile uint32_t *)(RCC + 0x14))
#define GPIOC  0x40011000
#define GPIOC_CRH  (*(volatile uint32_t *)(GPIOC + 0x04))
#define GPIOC_BSRR  (*(volatile uint32_t *)(GPIOC + 0x10))
#define FLASH_ACR     (*(volatile uint32_t*)0x40022000)
#define SYST_CSR     (*(volatile uint32_t*)0xE000E010)
#define SYST_RVR     (*(volatile uint32_t*) 0xE000E014)
#define SYST_CVR     (*(volatile uint32_t*)0xE000E018)



 void SystemClock_Config(void);
 void LED_Init(void);
 void delay_ms(uint32_t ms);
 void SysTick_Init(void);


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
SYST_CVR = 0;
	for (uint32_t i = 0; i < ms; i++) {
	        while (!(SYST_CSR & (1 << 16)));
    }
}

void LED_Init(void){
	RCC_APB2ENR |=(1<<4);
	GPIOC_CRH |=(1<<20);
	GPIOC_CRH &= ~((1<<21)|(1<<22)|(1<<23));
}

int main(void){
	SystemClock_Config();
     LED_Init();
     SysTick_Init();
	while(1){
		GPIOC_BSRR = (1<<13);
		delay_ms(1000);
		GPIOC_BSRR =(1<<29);
		delay_ms(1000);

	}
}
