## ¿Cómo funciona una interrupción del timer? (concepto)

Un timer de hardware cuenta a una frecuencia fija. Cuando alcanza un evento, activa una solicitud de interrupción IRQ. Esa IRQ pasa por el controlador de interrupciones, que decide si la deja pasar hacia la CPU. Si la CPU tiene las IRQ habilitadas, detiene momentáneamente el flujo normal del programa, salta al vector de interrupciones correspondiente y ejecuta el manejador handler. Al finalizar, vuelve exactamente al punto donde se interrumpió el main. Para que la interrupción sea estable y periódica, durante la atención de la IRQ se deben cumplir dos cosas: Limpiar el “flag” del timer para que no quede la interrupción pendiente y se dispare repetidamente.

# timer_init()
Configura el timer para que genere una interrupción periódica. Este Habilita el reloj (clock) del timer para que el periférico funcione.
* Detiene el timer mientras se configura.
* Carga el valor necesario para que el evento ocurra cada “N” segundos.
* Habilita la interrupción del timer (overflow).
* Activa auto-reload para que la interrupción se repita constantemente.
* Desenmascara (unmask) la IRQ del timer en el controlador de interrupciones.

# timer_irq_handler()
Rutina de servicio de interrupción (ISR) del timer.
* Limpia el flag de interrupción del timer evita re-entradas inmediatas.
* Hace acknowledge al controlador de interrupciones permite futuras IRQ.
* Realiza la acción periódica del lab imprimir Tick.

# enable_irq()
Permitir que la CPU acepte interrupciones IRQ.
* Limpia el bit de máscara de IRQ en el registro de estado del procesador
* Si no se ejecuta esta función, la CPU ignora IRQ aunque el timer y el controlador estén bien configurados.

# irq_handler (handler en ensamblador)
Conectar la entrada de la IRQ excepción con la ISR en C.

* Guarda el contexto de los registros para no romper el estado del main.
* Llama a la rutina en C que atiende la interrupción.
* Restaura el contexto.
* Retorna correctamente de la excepción para continuar el programa donde se interrumpió.


## Arquitectura:

1. El periférico timer genera el evento. El timer alcanza overflow después del intervalo configurado y activa su interrupción.

2. El controlador de interrupciones INTC enruta la IRQ. El INTC recibe la señal. Si está desenmascarada y permitida, la envía a la CPU y marca la interrupción como pendiente.

3. La CPU acepta la interrupción IRQ exception. Si la CPU tiene IRQ habilitadas, pausa el flujo normal y consulta la tabla de vectores para saber a dónde saltar.

4. La tabla de vectores redirige al irq_handler ensamblador. La entrada de IRQ salta al handler en ensamblador, que se encarga del manejo de bajo nivel.

5. El handler en ensamblador llama a la ISR en C. Después de guardar registros de contexto, transfiere el control a la rutina que atiende el timer.

6. La ISR limpia flags y reconoce la interrupción. Se limpia el flag del timer. Esto evita bucles de interrupción y permite que lleguen nuevas IRQ.

7. Retorno a la ejecución normal. Se restauran registros y se retorna de la excepción. El main continúa donde se quedó.
