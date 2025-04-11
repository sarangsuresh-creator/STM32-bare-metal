#include <stdint.h>
#define RCC_CR  (*(volatile uint32_t *)(0x40021000))
#define RCC_CFGR  (*(volatile uint32_t *)(0x40021004))
#define RCC_APB2ENR  (*(volatile uint32_t *)(0x40021018))
#define RCC_AHBENR  (*(volatile uint32_t *)(0x40021014))
#define GPIOC_CRH  (*(volatile uint32_t *)(0x40011004))
#define GPIOC_BSRR  (*(volatile uint32_t *)(0x40011010))
#define GPIOA_CRL  (*(volatile uint32_t *)(0x40010800))
#define GPIOA_BSRR  (*(volatile uint32_t *)(0x40010810))
#define GPIOA_ODR  (*(volatile uint32_t *)(0x4001080C))
#define GPIOA_IDR  (*(volatile uint32_t *)(0x40010808))
#define FLASH_ACR     (*(volatile uint32_t*)0x40022000)


 void SystemClock_Config(void);
 void LED_Init(void);
 void switch_init(void);


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

void LED_Init(void){
	RCC_APB2ENR |=(1<<4);
	GPIOC_CRH |=(1<<20);
	GPIOC_CRH &= ~((1<<21)|(1<<22)|(1<<23));
}
void switch_init(void){
	RCC_APB2ENR |= (1<<2);
	GPIOA_CRL &= ~((1<<0)|(1<<1)|(1<<2));
	GPIOA_CRL |= (1<<3);
	GPIOA_ODR |= (1<<0);
}
int main(void){
	SystemClock_Config();
     LED_Init();
     switch_init();
	while(1){
		if(!(GPIOA_IDR & (1<<0))){
			GPIOC_BSRR =(1<<29);
		}else{
			GPIOC_BSRR = (1<<13);
		}


	}
}
