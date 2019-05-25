# Huma-Heart-Rate-Sensor-and-Temperature-Monitoring-of-surrounding-system
Human Heart Rate Sensor and Temperature Monitoring system of the surrounding environment using LM35 temperature sensor and TCRT1000 for heart rate. The results were displayed on LCD.
#include&lt;lpc21xx.h&gt;
#define RS 0x00020000 /* RS - P1.17 */
#define RW 0X00040000 /* R/W - P1.18 */
#define EN 0X00080000 /* E - P1.19 */
#define CLR 0X00FE0000
unsigned int adc_value=0;
int count;
void PORT_Initial(void)
{
IO1DIR = 0x00FE0000; /* LCD pins set as o/p */
IO0DIR = 0x00000000;
PINSEL0 = 0x00000000;
PINSEL1 = 0x05000000;
PINSEL2 = 0x00000000;
}
int Delay(unsigned int x)
{
x=x*8000;
while(x&gt;0)
{
x--;
}
return 0;
}
void LCD_Command(char command)
{
int Temp;
IO1CLR = CLR; /* Clearing the port pins */
IO1SET = EN; /* Enable pin high */
IO1CLR = RS; /* RS=0 for command register */
IO1CLR = RW; /* R/W=0 for write */
Temp = (command &amp; 0xF0) &lt;&lt; 16; /* Taking the first nibble of command */
IO1SET = IO1SET | Temp; /* Writing it to data line */
Delay(2);
IO1CLR = EN; /* Enable pin low to give H-L pulse */
}
void LCD_Command1(char command1)
{
int Temp;
IO1CLR = CLR; /* Clearing the port pins */
IO1SET = EN; /* Enable pin high */
IO1CLR = RS; /* RS=0 for command register */
IO1CLR = RW; /* R/W=0 for write */
Temp = (command1 &amp; 0xF0); /* Taking the first nibble of command */
Temp = Temp &lt;&lt; 16; /* Shift it 16 bits to left */
IO1SET = IO1SET | Temp; /* Writing it to data line(P1.20-P1.23) */
Delay(2);
IO1CLR = EN; /* Enable pin low to give H-L pulse */
IO1CLR = CLR; /* Clearing the port pins */
IO1SET = EN; /* Enable pin high */
IO1CLR = RS; /* RS=0 for command register */
IO1CLR = RW; /* R/W=0 for write */
Temp = (command1 &amp; 0x0F); /* Taking the second nibble of command */
Temp = Temp &lt;&lt; 20; /* Shift it 20 bits to left */
IO1SET = IO1SET | Temp; /* Writing it to data line */
Delay(2);
IO1CLR = EN; /* Enable pin low to give H-L pulse */
}
void LCD_Data(char data)
{
int Temp;
IO1CLR = CLR; /* Clearing the port pins */
IO1SET = EN; /* Enable pin high */
IO1SET = RS; /* RS=1 for data register */
IO1CLR = RW; /* R/W=0 for write */
Temp = (data &amp; 0xF0); /* Taking the first nibble of data */
Temp = Temp &lt;&lt; 16; /* Shift it 16 bits to left */
IO1SET = IO1SET | Temp; /* Writing it to data line */
Delay(2);
IO1CLR = EN; /* Enable pin low to give H-L pulse */
IO1CLR = CLR; /* Clearing the port pins */
IO1SET = EN; /* Enable pin high */
IO1SET = RS; /* RS=1 for data register */
IO1CLR = RW; /* R/W=0 for write */
Temp = (data &amp; 0x0F); /* Taking the second nibble of data */
Temp = Temp &lt;&lt; 20; /* Shift it 20 bits to left */
IO1SET = IO1SET | Temp; /* Writing it to data line */
Delay(2);
IO1CLR = EN; /* Enable pin low to give H-L pulse */
}
void LCD_value( int count)
{
count=count+1;
LCD_Data(count+0x30);
}
void LCD_String( char *dat)
{
while(*dat!=&#39;\0&#39;) /* Check for termination character */
{
LCD_Data(*dat); /* Display the character on LCD */
dat++; /* Increment the pointer */
}
}
void LCD_Init(void)
Delay(15);
LCD_Command(0x30);
Delay(10);
LCD_Command(0x30);
Delay(5);
LCD_Command(0x30);
Delay(5);
LCD_Command(0x20);
Delay(5);
LCD_Command1(0x28);
Delay(5);
LCD_Command1(0x01); /* Clear display */
Delay(5);
LCD_Command1(0x06); /* Auto increment */
Delay(5);
LCD_Command1(0x0C); /* Cursor off */
}
int ADC_Conversion()
{
int ab; /* Variable to store ADC value */
Delay(1);
ADCR = ADCR|0x01000000; /* Start conversion */
while((ADDR&amp;0x80000000)!=0x80000000); /* Wait here till conversion is over*/
ab = (ADDR&amp;0x0000FFC0); /* Extracting the result */
ab = (ab&gt;&gt;6); /* Shift 6 bits right */
return ab; /* Return the result */
}
void Int_ASCII(int value,char cnt)
{
int i = 0; /* Local variables */
char array[7];
int values;
values= value;
for(i=1;i&lt;=cnt;i++) /* Store the received value in array*/
{
array[i] = values%10;
values = values/10;
}
for(i=cnt;i&gt;=1;i--) /* Display it on LCD*/
{
LCD_Data(array[i]+&#39;0&#39;);
}
}
void Sensor_Check()
{
ADCR=0x00200602; /* PDN=1,CLKDIV=6,channel=AD0.2*/
LCD_Command1(0x80);
LCD_String(&quot;Temp:&quot;);
adc_value=ADC_Conversion(); /* Get the result of conversion*/
// Delay(500000);
adc_value=((adc_value*0.48828125));
LCD_Command1(0x86); /* 2nd row, 5th location */
Int_ASCII(adc_value,2); /* Display the result on LCD */
}
__irq void T0_ISR (void)
{
int i;
for(i=0;i&gt;=100;i++)
{
if((IO0PIN&amp;0x00000100)==1)
{
count=count+1;
}
else
{

LCD_Command1(0xC0);
LCD_String(&quot; ABnorm &quot;);
}
}
LCD_Command1(0xC0);
LCD_value(count);
T0IR = ( T0IR | (0x01) );
VICVectAddr = 0x00;
}
int main()
{
PORT_Initial(); /* Initialize port */
LCD_Init(); /* Initialize LCD */
// LCD_String(&quot;Temperature&quot;);
while(1)
{
Sensor_Check(); /* Take ADC reading */
Delay(50);
break;
}
LCD_Command1(0xC0);
LCD_String(&quot; Heart Rate: &quot;)
VPBDIV = 0x00000002; /* For Pclk = 30MHz */
/* We have configured Cclk=60MHz. Above instruction makes Pclk = Cclk/2 = 30MHz */
PINSEL0 = PINSEL0 | 0x00000020; /* Configure P0.2 as Capture 0.0 */
IO0DIR = ( IO0DIR | (0x00000100) ); /* 8 P0.8-P0.15 as output */
//IO0PIN = IO0PIN | 0x00000100; /* Writing 1 to pin P0.8 */
VICVectAddr0 = (unsigned) T0_ISR; /* T0 ISR Address */
VICVectCntl0 = 0x00000024; /* Enable T0 IRQ slot */
VICIntEnable = 0x00000010; /* Enable T0 interrupt */
VICIntSelect = 0x00000000; /* T0 configured as IRQ */
T0TCR = 0x02; /* Reset TC and PR */
T0TCR = 0x00; /* Timer mode, increment on every rising edge */
T0PR = 0x1D; /* Load Pre-Scalar counter with 29 (0 to 29 = 30), so that timer counts every
1usec */
T0MR0 = 10000; /* Load timer counter for 10\sec delay, 1usec*1000*100*100*/
T0MCR = 0x0003; /* Interrupt generate on match and reset timer */
T0TCR = 0x01; /* Enable timer */
while(1);
}
