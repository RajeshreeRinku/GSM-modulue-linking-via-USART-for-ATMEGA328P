# GSM-modulue-linking-via-USART-for-ATMEGA328P
/*
 * main.c
 *
 * Created: 15/08/2022 6:28:50 PM
 *  Author: Rajeshri
 */ 

#include <xc.h>
#include <util/delay.h>

#define F_CPU 11059200


void lcd_initialize();
void lcd_data(const unsigned char *data, int length);
void lcd_command(char cmd);
void lcd_enable();

void usart_transmit(const unsigned char *data, int length);
int usart_receive();
void usart_tran(char data);

int main(void)
{
  DDRB=0xFF;
  DDRC|=((1<<DDC0)|(1<<DDC1));
  PORTC&=~(1<<PORTC1);
  lcd_initialize();
  UBRR0=0x71;
  UCSR0C=0x03;
  first_command: usart_transmit("AT",2); 
  if (usart_receive()==0)
  goto first_command;
  usart_transmit("AT+CMGF=1",9); // Set Text mode
  if (usart_receive()==0)
  goto first_command;
  usart_transmit("AT+CMGS=",8);
  usart_tran(0x22);// to add "
  usart_transmit("+919999999999",13);
  usart_tran(0x22);
  usart_transmit("Hello INDIA",11);
  usart_tran(0x1A);// It is equivalent to 'ctr+z'
 if (usart_receive()==0)
  goto first_command;
  
    while(1);
   
}

void lcd_initialize()
{
lcd_command(0x38);
lcd_enable();
lcd_command(0x01);
lcd_enable();
lcd_command(0x0C);
lcd_enable();
lcd_command(0x80);
lcd_enable();
}

void lcd_data(const unsigned char *data, int length)
{
 PORTC|=(1<<PORTC0);
for(int i=0;i<length;i++)
{
	PORTB=data[i];
	lcd_enable();
}
}

void lcd_command(char cmd)
{
 PORTC&=~(1<<PORTC0);
 PORTB=cmd;
 lcd_enable();
}

void lcd_enable()
{
  PORTC|=(1<<PORTC1);
 _delay_ms(10);
 PORTC&=~(1<<PORTC1);
}

void usart_transmit(const unsigned char *data, int length)
{
 UCSR0B=0x08;
 for(int i=0;i<length;i++)
 {
 while(!(UCSR0A & (1<<UDRE0)));
 UDR0=data[i];
 }
 UCSR0B=0x00; 
 lcd_command(0x01);
 lcd_data(data,length);
 _delay_ms(10);
}

int usart_receive()
{int a[100];
UCSR0B=0x10;
for(int i=0; i<100;i++)
{
	while(!(UCSR0A&(1<<RXC0)));
	a[i]=UDR0;
	if((a[0]=='O')&(a[1]=='K'))
	{
		lcd_command(0x01);
 lcd_data("OK",2);
 _delay_ms(10);
	return 1;
	}
	else
	{
		lcd_command(0x01);
 lcd_data("Error",1);
 _delay_ms(10);
	return 0;
	}
}
 UCSR0B=0x00;
  
}

void usart_tran(char data)
{
 UCSR0B=0x08;
 while(!(UCSR0A&(1<<UDRE0)));
 UDR0=data;
 UCSR0B=0x00; 
 lcd_command(0x01);
 lcd_data(data,1);
 _delay_ms(10);
}




