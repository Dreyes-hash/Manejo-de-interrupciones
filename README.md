# Manejo-de-interrupciones
Configuracion de puertos GPIO para Stm32 
Profesor: Dr. Adán Geovanni Medrano Chávez  
UEA: Microcontroladores
## Integrantes del equipo 
Diego Reyes Blancas - 2213026667
## Objetivo de la práctica
1. Configurar 10 puertos GPIO como salidas de forma que los LEDs conectados a estos representen el valor binario de una variable.
2. De manera
3. eliminar el rebote por medio de hardware
predeterminada, el contador aumentará su cuenta cada segundo. Al
oprimir un botón, el contador cambiará su modo y trabajará en modo
descendente. Si el botón vuelve a presionarse, entonces el contador
regresará al modo ascendente. A su vez, el contador podrá aumentar su
velocidad x2, x4 y x8 al oprimir un segundo botón. La velocidad regresará
a x1 cuando ésta esté establecida en x8 y el usuario oprima nuevamente
el segundo botón.
## hardware y software utilizados para este proyecto
1. placa de desarrollo STM32F103 BluePill
2. st-link v2
3. 10 leds
4. 10 resistencias-220 ohmios
5. 2 resistencias-10 kiloohm
6. cables dupont macho-macho y hembra-hembra
## Instalación de software de compilación para µC arm
En las distribuciones GNU/Linux, es necesario instalar algunos paquetes con el siguiente comando:
````
sudo apt install gcc-arm-none-eabi stlink-tools libusb-1.0-0-dev
````
## configuracion del hardware
el programa configurá los pines A0-A9 como salidas estas nos serviran para representar nuestra variable con los LEDs  y A10 # enables clock in Port B
````
    ldr     r0, =RCC_BASE
    mov     r1, #4
    str     r1, [r0, RCC_APB2ENR_OFFSET]
    # configures pin 0 to 9 as outputs in GPIOA_CRL
    ldr     r0, =GPIOA_BASE @ moves base address of GPIOA registers
    ldr     r1, =0x33333333 @ PA[7:0] work as outputs
    str     r1, [r0, GPIOx_CRL_OFFSET] @ M[GPIOA_CRL] gets 0x33333333
    ldr     r1, =0x00008833 @ PA[8:9] work as outputs and PA[10:11] works as inputs pull-down
    str     r1, [r0, GPIOx_CRH_OFFSET] @ M[GPIOA_CRH] gets 0x00008833-A11 como entradas pull-dpown para leer nuestros botones
````
el programa configurá los pines A10-A11 para activar la interrupción 
````
configure_exti:
    push    {r7}
    sub     sp, sp, #4
    add     r7, sp, #0
    
    ldr r0, =AFIO_BASE
    eor r1, r1
    str r1, [r0, AFIO_EXTICR3_OFFSET]
    ldr r0, =EXTI_BASE
    ldr r1, =0x00000C00
    str r1, [r0, EXTI_RTST_OFFSET]
    str r1, [r0, EXTI_IMR_OFFSET]
    eor r1, r1
    str r1, [r0, EXTI_FTST_OFFSET]
    ldr r0, =NVIC_BASE
    ldr r1, =0x00000100
    str r1, [r0, NVIC_ISER1_OFFSET]  
    adds    r7, r7, #4
    mov     sp, r7
    pop     {r7}
    bx      lr
````
## funcionamiento general de la implementación
1. La función __main es el punto de entrada del programa y configura los registros necesarios y los pines GPIO para el funcionamiento del proyecto.
2. La función configure_exti se encarga de configurar las interrupciones externas (EXTI). Establece el registro AFIO_EXTICR3 en cero para seleccionar el puerto GPIOA como fuente de interrupción y configura los registros EXTI_RTST, EXTI_IMR y EXTI_FTST para habilitar las interrupciones en el flanco de subida y bajada del pin EXTI10 (PA10). También se configura el registro NVIC_ISER1 para habilitar la interrupción EXTI15_10.
3. La función output se utiliza para emitir un valor a través de los pines digitales (GPIOA_ODR). Recibe un parámetro en r6 y utiliza una máscara (0x3FF) para obtener los 5 bits menos significativos de ese valor. Luego, establece los pines correspondientes (PA0-PA4) según el valor obtenido.
4. La función delay se utiliza para crear un retraso en el programa durante un número especificado de milisegundos (ms). Utiliza un bucle anidado con contadores para generar el retraso. El valor 1250 en ldr r0, =#1250 se ajusta para lograr un retraso aproximado de 1 ms.
5. La función EXTI_ISRHandler es el manejador de interrupciones externas EXTI10. Verifica si la interrupción proviene de PA10 (EXTI10) y cambia el valor almacenado entre 0 y 1. Además, implementa un contador en r8 para contar el número de interrupciones y realiza una acción cuando el contador alcanza un valor específico.
