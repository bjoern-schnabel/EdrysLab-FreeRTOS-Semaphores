# Thema
In diesem Kurs werden sie sich hands-on mit dem Task-sceduler in FreeRTOS arbeiten. Mittels einer UART Textausgabe werden sie Race Conditions erstellen und den Einfluss von verschiedenen Task-prioritäten, Laufzeiten und Semaphoren erproben.
Lesen sie kurz dein Einführungstext und öffnen sie dann die Station. 

# Grundlagen

## Unterbrechung in FreeRTOS:
- scheduler lässt immer die Task mit der höchsten Priorität laufen, die aktuell bereit ist
  - falls eine Task mit niedriger Priorität (NPT) aktuell läuft, so unterbricht die Task mit höherer Priorität (HPT) diese.
- bei gleichen Prioritäten ist standardmäßig time-slicing aktiviert
  - die Tasks mit gleichen Prioritäten wechseln sich ab (in solchen time-slices (jeweils 1 FreeRTOS-Tick)) (Round-robin)
- es gibt in FreeRTOS einen periodischen Tick-interrupt um die Zeit zu messen


## vTaskDelay()
- ermöglicht es, Tasks für eine gesetzte Anzahl Ticks warten zu lassen.
- Andere niederpriore Tasks können währenddessen ausgeführt werden.
  
## Race Conditions (Wettlaufsituation)
-  Konstellation, in der das Ergebnis einer Operation vom zeitlichen Verhalten bestimmter Einzeloperationen oder der Umgebung abhängt

- häufig Grund für schwer auffindbare nichtdeterministische Programmfehler
  - z.B. veränderten Bedingungen zum Programmtest (z.B. Logging) verändern Symptome

## Mutex
- Synchronisationsmechanismus, der exklusiven Zugriff erlaubt (verhindert unkoordinierten Zugriff auf gemeinsame Resource). Nur eine Task darf auf die serielle Schnittstelle zugreifen, solange sie den Mutex hält
- bei einem Mutex kann nur die Task die den Mutex hält, diesen auch wieder freigeben
- in FreeRTOS: priority inheritance (will eine Task mit höherer Priorität auf die Resource zugreifen, so erbt die Task mit dem Mutex temporär die höhere Priorität)
- verhindert Race Condition

## Usart direkt
          void usart_init(unsigned long BAUDRATE)
          {
              UCSR0B |= (1 << RXEN0) | (1 << TXEN0);
              UCSR0C |= (1 << UCSZ00) | (1 << UCSZ01);
              UBRR0L = BAUD_PRESCALE;
              UBRR0H = (BAUD_PRESCALE >> 8);
          }

          void usart_putstring_direct(const char* str)
          {
              uint16_t strlen = 0;
              while(str[strlen] != '\\0'){
              strlen++;
          }
          for(uint16_t i; i< strlen; i++ )
          {
              vTaskSuspendAll();
              if(str[i] == '\\n')
              {
                  loop_until_bit_is_set(UCSR0A, UDRE0);
                  UDR0 = '\\r';
              }
              loop_until_bit_is_set(UCSR0A, UDRE0);
              UDR0 = str[i];
              xTaskResumeAll();
              }
          }

- loop_until_bit_is_set(UCSR0A, UDRE0);
  - wartet, bis Bufferregister frei ist
- UDR0 = str[i];
  - schreibt 1 char in 8 Bit Bufferregister
