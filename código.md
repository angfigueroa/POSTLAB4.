# POSTLAB4.
; Archivo: main4.s
; Dispositivo: PIC16F887
; Autor: Angela Figueroa
; Compilador: pic-as (v2.30), MPLABX V5.40
;
; Programa: Implementar un contador binario de 4 bit 
; Hardware: LEDs en el puerto A y dos Displays
;
; Creado: 12 febrero, 2023
; Actualizado:  17 de febrero, 2023
    
PROCESSOR 16F887
#include <xc.inc>

;CONFIGURACIÓN 1 
    CONFIG FOSC=INTRC_NOCLKOUT //OSCILADOR INTERNO SIN SALIDAS
    CONFIG WDTE=OFF // WDT DISABLE (REINICIO REPETITIVO DE PIC)
    CONFIG PWRTE=OFF // PWRT ENABLE (ESPERA DE 72MS AL INICIAR)
    CONFIG MCLRE=OFF //EL PIN DE MCLR SE USA COMO I/O
    CONFIG CP=OFF //SIN PROTECCIÓN DE CÓDIGO
    CONFIG CPD=OFF //SIN PROTECCIÓN DE DATOS
    
    CONFIG BOREN=OFF //SIN REINICIO CUANDO EL V DE ALIMENTACIÓN BAJA A 4V
    CONFIG IESO=OFF // REINICIO SIN CAMBIO DE RELOJ INTERNO A EXTERNO
    CONFIG FCMEN=OFF // CAMBBIO DE RELOJ EXTERNO A INTERNO EN CASO DE FALLO
    CONFIG LVP=OFF //PROGRAMACIÓN EN BAJO VOLTAJE PERMITIDA
    
 ;CONFIGURACIÓN 2
    CONFIG WRT=OFF //PROTECCIÓN DE AUTOESCRITURA POR EL PROGRAMA DESACTIVADA
    CONFIG BOR4V=BOR40V //REINICIO ABAJO DE 4, (BOR21V=2.1V)
    PSECT udata_bank0 ;common memory VARIABLES GLOBALES
	cont_small: DS 1 ;1 byte
	cont_big: DS 1
	w_tem: DS 1
	status_tem: DS 1
	conter:	DS 1
	offset: DS 1
	tensec: DS 1

    subir EQU 0
    bajar EQU 2
	;cont_big: DS 1

    
    PSECT resVect, class=CODE, abs, delta=2 
    
 
    ;--------------RESET-------------------------
    ORG 00h	;POSICION EN EL QUE SE ENCUENTRA
    resetVec:
	PAGESEL main
	goto main
	
	
    ;------------Rutina de interupción-------------------
    PSECT code, delta=2, abs
    ORG 004h	;Poscición 
    
    push:
	movwf w_tem ;GUARDAMOS EN W EL VALOR DE W_TEM
	swapf STATUS,0
	movwf status_tem ;ALMACENA W EN STATUS_TEM
    isr:;VERIFICAMOS DE LAS BANDERAS DE LAS INTERRUPCIONES 
	btfsc	RBIF
	call	int_v
	btfsc INTCON,2
	call TMR02
	nop
    pop:
	   swapf    status_tem,0
	   movwf    STATUS;rECUPERAMOS EL VALOR DE REG STATUS
	   swapf    w_tem,1
	   swapf    w_tem,0
	   retfie
	   
    int_v:
	banksel	PORTA
	btfss	PORTB,subir
	incf	PORTA
	btfss	PORTB,bajar
	decf	PORTA
	bcf	RBIF
	return
    TMR02:;timer 2
	
	banksel	PORTA
	movlw   40
	movwf	TMR0
	incf	PORTD 
	bcf	INTCON,2
	incf	conter;Incrementar Timer0
	movf	PORTD,0
	sublw	10
	btfsc	STATUS,2;Verificar la bandera de Z
	call	disp
    return
    disp:;DISPLAY
        banksel PORTC
	incf offset
	movf offset,0
	call tabla_display
	movwf PORTC   ;Imprimir En Display
	call verificar
	return
	
    verificar:
	movf offset,0
	sublw	10
	btfsc	STATUS,2
	call	ten_sec
	return
	
    ten_sec:;10 segundos
	clrf offset
	incf tensec
	movf tensec,0
	call tabla_display
	movwf PORTA   ;Imprimir En Display
	call sten
     RETURN
    sten:;60 segundos
	movf tensec,0
	sublw	5
	btfsc	STATUS,2
	clrf	tensec
	
	return
	
    ;------------Main----------------
     PSECT code, delta=2, abs
    ORG 100h	;Poscición 
    tabla_display:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 1000000B;0
	retlw 1111001B;1
	retlw 0100100B;2
	retlw 0110000B;3
	retlw 0011001B;4
	retlw 0010010B;5
	retlw 0000010B;6
	retlw 1111000B;7
	retlw 0000000B;8
	retlw 0010000B;9
	retlw 1000000B;0
main:
    
	
	call clock 
	call pines
	call ciocb
	
	
	banksel TRISA
	movlw 01010111B ;CONFIGURACION DE TIMER0
	movwf OPTION_REG ; CARGAR LA CONFIGURACIÓN
	banksel PORTA
	
	
loop:
	nop
	goto loop
	

;-----------Funciones del laboratorio------------------
pines:
	banksel ANSEL
	clrf	ANSEL	;digittales
	clrf	ANSELH
	banksel	TRISA
	clrf	TRISA
	clrf	TRISB
	clrf	TRISC
	
	bcf	TRISD,0
	bcf	TRISD,1
	bcf	TRISD,2
	bcf	TRISD,3
	
	bsf	TRISB,subir
	bsf	TRISB,bajar
	
;	bsf	OPTION_REG, 0
	banksel	TRISA
	bcf	OPTION_REG,7 ;RBPU
	bsf	WPUB,subir
	bsf	WPUB,bajar
	
	
	banksel PORTA
	movlw   1000000B
	movwf   PORTA
	clrf	PORTB
	clrf	PORTD
	movlw   1000000B
	movwf   PORTC

	

return 
ciocb:

    banksel TRISA
    bsf	    IOCB,subir
    bsf	    IOCB,bajar
    bsf     RBIE
    bsf	    GIE
    banksel ANSEL
    bsf     T0IE
return

;-------Configuracion de Oscilador Interno ------------
clock:
	banksel OSCCON;Oscillator Control Register 
	bsf IRCF2	  ;-------Reloj de 1 Mhz
	bsf IRCF1	  
	bcf IRCF0	  
	bsf SCS;Internal oscillator is used
RETURN
	
END
