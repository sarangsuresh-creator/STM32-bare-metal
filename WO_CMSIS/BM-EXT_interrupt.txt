#include <stdint.h>
#define RCC_CR 			 (*(volatile uint32_t *)0x40021000)
#define RCC_CFGR  		 (*(volatile uint32_t *)0x40021004)
#define RCC_APB2ENR  	 (*(volatile uint32_t *)0x40021018)
#define RCC_AHBENR  	 (*(volatile uint32_t *)0x40021014)
#define GPIOC_CRH 		 (*(volatile uint32_t *)0x40011004)
#define GPIOC_BSRR 		 (*(volatile uint32_t *)0x40011010)
#define GPIOC_ODR 		 (*(volatile uint32_t *)0x4001100C)
#define GPIOA_CRL 		 (*(volatile uint32_t *)0x40010800)
#define GPIOA_BSRR 		 (*(volatile uint32_t *)0x40010810)
#define GPIOA_ODR  		 (*(volatile uint32_t *)0x4001080C)
#define GPIOA_IDR 		 (*(volatile uint32_t *)0x40010808)
#define FLASH_ACR        (*(volatile uint32_t *)0x40022000)
#define AFIO_EXTICR1  	 (*(volatile uint32_t *)0x40010008)
#define EXTI_IMR   		 (*(volatile uint32_t *)0x40010400)
#define EXTI_RTSR   	 (*(volatile uint32_t *)0x40010408)
#define EXTI_FTSR   	 (*(volatile uint32_t *)0x4001040C)
#define EXTI_PR   		 (*(volatile uint32_t *)0x40010414)
#define NVIC_ISER0  	 (*(volatile uint32_t *)0xE000E100)
#define SYST_CSR    	 (*(volatile uint32_t *)0xE000E010)
#define SYST_RVR  	     (*(volatile uint32_t *)0xE000E014)
#define SYST_CVR    	 (*(volatile uint32_t *)0xE000E018)




 void SystemClock_Config(void);
 void LED_Init(void);
 void switch_init(void);
 void External_Interrupt(void);
 void delay_ms(uint32_t ms);
 void EXTI0_IRQHandler(void);
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
void EXTI0_IRQHandler(void) {
    if (EXTI_PR & (1 << 0)) {
        for(int j=0;j<10;j++){
    	GPIOC_BSRR = (1 << 29);
        delay_ms(150);
        GPIOC_BSRR = (1 << 13);
        delay_ms(150);
        EXTI_PR |= (1 << 0);
    }
    }
}

void LED_Init(void){
	RCC_APB2ENR |=(1<<4);
	GPIOC_CRH |=(1<<20);
	GPIOC_CRH &= ~((1<<21)|(1<<22)|(1<<23));
	GPIOC_ODR |= (1<<13);
}
void switch_init(void){
	RCC_APB2ENR |= (1<<2);
	GPIOA_CRL &= ~((1<<0)|(1<<1)|(1<<2));
	GPIOA_CRL |= (1<<3);
	GPIOA_ODR |= (1<<0);

}
void External_Interrupt(void){
	 RCC_APB2ENR |=(1<<0);
	 AFIO_EXTICR1 &= ~(0X0f);
	 EXTI_IMR |= (1<<0);
	 EXTI_RTSR &= ~(1 << 0);
	 EXTI_FTSR |= (1 << 0);
	 NVIC_ISER0 |=(1<<6);
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


int main(void){
	 SystemClock_Config();
     LED_Init();
     switch_init();
     SysTick_Init();
     External_Interrupt();

	while(1)
	{
		GPIOC_BSRR = (1 << 13);
		        delay_ms(1000);
		        GPIOC_BSRR = (1 << 29);
		        delay_ms(1000);
	}
}
