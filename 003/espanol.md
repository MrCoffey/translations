En el episodio de hoy construiremos un calendario. Pero lo haremos en un estilo funcional en el que los objetos no mutan. Vamos a empezar teniendo un código procesal muy básico hecho en Ruby, luego vamos a envolverlo en objetos y esencialmente refactorizarlo en código Ruby estructurado. Haremos todo esto sin modificar ningún objeto.

El calendario que vamos a crear se parece a este. 

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/003/images/001.png "calendario")

El primer día es en realidad el 26 del mes anterior y el último día del calendario es el sexto día del mes siguiente. La primera tarea va a ser encontrar el primer día. Recuerda que el primer día será el domingo 26 del mes anterior.

Vamos a comenzar con la fecha de hoy, así tendremos un punto de referencia, ya que queremos el calendario de este mes, en lugar de adivinar lo que va a devolver, o si incluso si escribí las cosas bien, me gusta entrar en el IRB y jugar un juego que yo llamo "preguntale a Ruby". Sabemos que queremos ```date.today.begining_of_month```.

~~~ruby
$ first_calendar_day = Date.today.begining_of_month
~~~

Vaya parece que ```date.today``` no existe en Ruby, pero estoy seguro de que he usado ```date.today```, quizás porque fue en Rails. Si fue en Rails probablemente pudo ser con **active support**. **Active support** es una biblioteca en el núcleo de Rails que esencialmente ajusta un montón de objetos básicos de Ruby para darles funcionalidad extendida. Los ajustes se hacen a través de la biblioteca **core extension**, en este caso ```date``` fue ajustada. Si requiero la extensión ```active support``` entonces podré llamar ```date.today```.

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/003/images/002.png "calendario")

~~~ruby
$ require "active_support/core_ext/date"
$ Date.today
=> Sun, 03 May 2015
~~~

```Date.today``` devuelve una fecha, un objeto de la clase ```date```. Si pregunto "cuál es tu clase?” dirá que es una fecha. Ya que ```date.today``` devuelve una fecha podemos entonces llamar otro método, esta será ```beginning_of_month``` que devuelve otra fecha. Y nos devuelve **Viernes 01 de mayo de 2015**. Es correcto, ese es el primer día de este mes. Pero esa no es la primera fecha del calendario.

~~~ruby
$ Date.today.class
=> Date
$ Date.today.beginning_of_month
=> Fri, 01 May 2015 
~~~

Ahora de la misma forma que hicimos antes y debido a que el método ```month``` devuelve otra fecha podemos llamar otro método en él. Esta vez va a ser ```beginning_of_week```. Si llamamos ```beginning_of_week``` este nos devuelve **Lunes 27 de abril**. Eso estuvo cerca. Esta en el mes de abril. Pero no queremos empezar desde el lunes, Queremos empezar desde el domingo. Podemos usar el método ```beginning_of_week``` y ahora tenemos primer día. Copiemos esto de la consola de IRB y vamos a pegarlo en Vim.

~~~ruby
$ Date.today.beginning_of_month
=> Fri, 01 May 2015 
$ Date.today.beginning_of_month.beginning_of_week
=> Mon, 27 May 2015
$ Date.today.beginning_of_month.beginning_of_week(:sunday)
=> Sun, 26 Apr 2015
$ first_calendar_day = Date.today.beginning_of_month.beginning_of_week(:sunday)
~~~

Ahora tenemos nuestro primer día calendario establecido en ```date.today.beginning_of_month.beginning_of_week```, osea el domingo. Es hora de encontrar el último día calendario. ```last_calendar_day``` va a ser igual a ```date.today``` y va a devolver un objeto de ```date``` por lo que podemos llamar ```end_of_month```. Whoops tal vez no podemos. Parece que ```days_in_month``` no esta definido para ```Time:Class```. Nosotros ajustamos la clase ```date```, pero no ajustamos la clase ```time```.

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/003/images/003.png "calendario")

De nuevo vamos requerir las extensiones básicas de ```active support```. Esta vez vamos a decir ajusta la clase ```time```. Ahora puedo llamar ```date.today.end_of_month``` ya que utiliza la clase ```time``` con el fin de calcular cuantos días hay realmente en este mes. Eso se llama acoplamiento. Significa que el objeto ```data``` está acoplado al objeto ```time```, ya que ```date``` debe pedir el tiempo con el fin de obtener el final del mes, el cual es un método que vive en ```date```.

~~~ruby
$ require "active_support/core_ext/time"
=> true
$ Date.today.end_of_month
=> Sun, 31 May 2015
~~~

```Date.today.end_of_month``` nos devuelve una fecha. Podemos comprobarlo al preguntar por su clase. Por supuesto es una fecha. Si el final de mes nos da **Domingo 31 de mayo de 2015**, pero no es realmente lo que queremos. Queremos el 06 de junio. Dado que este método devuelve una fecha que sólo podemos decir dame el final de la siguiente semana, lo que nos da domingo del 31 de mayo de 2015.

~~~ruby
$ Date.today.end_of_month.end_of_week
=> Sun, 31 May 2015
~~~

Okei ahora estamos como:, "Eeee, eso  tampoco es lo que que queremos. Queremos el sábado de la sexta semana”. Pero si le pasamos el domingo?. Ahora nos dice **sábado 06 de junio 2015**, que es exactamente el día que queremos. Vamos a copiar eso, y vamos a pegarlo en nuestro editor de texto. Ahora que hemos encontrado la manera de conseguir el primer y el último día del calendario. Tenemos el 26 y la sexta semana aquí.

~~~ruby
$ Date.today.end_of_month.end_of_week(:sunday)
=> Sat, 06 Jun 2015
~~~

Podemos representar en Ruby usando lo que se llama un rango. Podriamos usar ```first_calendar_day ..```, que es el operador de rango, ```last_calendar_day```. Esto me devuelve un enumerador. Si recuerdas no me gusta adivinar lo que Ruby va a hacer. En su lugar mejor le pregunto, así que voy a usar irb para verificar y obtengo que el primer dia del calendario no existe. Entonces, ¿de dónde sacamos esto? Aquí tenemos ```first_calendar_day =```. Asi que ya encontramos el primer y último día del calendario. El primero calendario es, en efecto el **Domingo 26 de abril** y el último día del calendario es de hecho **Sábado 6 de junio**.

~~~ruby
(first_calendar_day..last_calendar_day)
~~~

Una cosa interesante aquí es que devuelve un rango. Técnicamente es un rango que trabaja como enumerador. Un enumerador conoce el primer número y conoce el segundo número, por lo que sabe acerca de cual es primer día y conoce el último día, pero no ha calculado los días en el centro. Este es el concepto de la *pereza*. Está diciendo que quiero cargar el primer día y el último día. Voy a hacer una serie sobre eso, pero no a cargar todos los que están en el medio por el momento. Sólo que ahora vamos a convertirlo en un arreglo. Tan pronto como llamamos al conjunto de ese rango el cálculo pasa y ahora tenemos todos los días en su interior. Tenemos esencialmente 42 días dentro de nuestro arreglo. No es exactamente lo que queríamos, lo que queremos es un arreglo que contenga otro arreglo de dias. Vamos a hacer que eso suceda.

~~~ruby
$ (first_calendar_day..last_calendar_day).class
=> Range
$ (first_calendar_day..last_calendar_day).to_a
=> [Sun, 26 Apr 2105, Mon, 27 Apr 2015, Thu, 28 Apr 2015...]
~~~

Podemos hacer que eso suceda llamando ```in_groups_of``` en este arreglo. Otro error. Esta vez está diciendo que el objeto ```array``` no tiene un método llamado ```in_groups_of```. Una vez más las personas que han escrito proyectos en Rails saben que ```in_groups_of``` de hecho existe en `array`, pero no hasta después de haber ajustado el objeto `array` con **active support**. Vamos a hacer eso.

~~~ruby
$ (first_calendar_day..last_calendar_day).to_a.count
=> 42
$ (first_calendar_day..last_calendar_day).to_a.in_groups_of(7)
=> NoMethodError: undefined method `in_groups_of`for #<Array:0x0007a0s0s7c49>
$ require "active_support/core_ext/array"
$ true
~~~

Ahora que ```array``` ha sido ajustado con las extensiones centrales de *active support* al requerir ese archivo. Al volver a ejecutar ese archivo, podemos ver que tenemos ahora tienen un arreglo con 6 artículos y dentro de cada uno contienen 7 días. Ahora tenemos nuestros grupos de seis con nuestros grupos de 7, que es lo que queríamos.

~~~ruby
$ (first_calendar_day..last_calendar_day).to_a.in_groups_of(7).count
=> 6
$ (first_calendar_day..last_calendar_day).to_a.in_groups_of[0].count
=> 7
~~~

Copiaré este código que sé que funciona fuera del IRB y lo voy a pegar en Vim y voy a ponerlo como semanas. Ahora tengo esencialmente el núcleo de lo que necesito para llevar a cabo la tarea en cuestión. Sin embargo tenemos mucho código procesal prescindible aquí. No hay encapsulación. Si fuéramos ha ejecutar este archivo en realidad no haría nada porque sólo estamos estableciendo valores, no estamos construyendo un objeto sin embargo, como el calendario real, y realmente no queremos poner la prestación de el calendario junto con la logica del negocio de nuestro calendario.

~~~ruby
weeks = (first_calendar_day..last_calendar_day).to_a.in_groups_of(7)
~~~

Nuestro siguiente paso aquí es en realidad va a ser la creación de algunos objetos. En primer lugar, debemos recordar que tenemos que requerir un par de cosas para que esto funcione. Necesitamos nuestra libreria de extensiones para **active support**. Ahora que tenemos estos requisitos vamos a crear un objeto. Sera una clase, y vamos a llamarla ```weeks``` porque eso es lo que devuelve. Vamos a darle un inicializador y sólo va a tomar una fecha. Estableceremos esa fecha por si nadie pasa una a ```date.today```. El día por defecto va a ser hoy.

~~~ruby
require "active_support/core_ext/date"
require "active_support/core_ext/time"
require "active_support/core_ext/array"

first_calendar_day = Date.today.beginning_of_month.beginning_of_week(:sunday)
last_calendar_day = Date.today.end_of_month.end_of_week(:sunday)

weeks = (first_calendar_day..last_calendar_day).to_a.in_groups_of(7)
~~~

Ahora queremos guarda en una variable de instancia para que poder acceder a él más tarde. Continuemos creando nuestro método ```week``` aqui. Cuando llamas ```to_a``` en ```week``` obtendrias una serie de semanas en su interior, para que esto suceda. Lo primero que necesitamos saber sobre el calendario es el primer y último día, luego volvemos la semana. Vamos a tomar este código y pegarlo por aquí. Ahora demosle formato.

~~~ruby
require "active_support/core_ext/date"
require "active_support/core_ext/time"
require "active_support/core_ext/array"

class CalendarWeeks
    def initialize(date=Date.today)
        @date = date
    end

    def weeks
        first_calendar_day = Date.today.beginning_of_month.beginning_of_week               (:sunday)
        last_calendar_day = Date.today.end_of_month.end_of_week(:sunday)
        weeks = (first_calendar_day..last_calendar_day).to_a.in_groups_of(7)
    end
end
~~~

Si salimos de Vim y ejecutamos IRB-I el cual va a incluir algo en el camino de carga Ruby. Lo que queremos incluir en la ruta de carga es el directorio actual del calendario. Eso se representado en Unix con un punto, vamos a requerir el archivo calendario. Ya que estamos en el interior del intérprete de Ruby y el archivo de calendario se ha cargado. Es como si hubiéramos hecho esto y que cuando haga esto va a devolver false porque ya estaba cargado con el otro comando. Esto significa es que ahora puedo hacer ```calendarweeks.new.to_a``` y hay esta mi arreglo de semanas.

~~~ruby
$ irb -I"." -rcalendar
$ require "calendar"
=> false
$ CalendarWeeks.new.to_a
~~~

Aún hay más que puedo hacer con este código para hacerlo más agradable. 
Personalmente tengo una regla. Si tienes una variable de instancia aquí probablemente debería ser movida a un método privado propio. Por muchas razones. Una de ellas es que en realidad es un patrón de refactorización llamado reemplazar la variable local con un método privado. El primer día ```first_calendar_day ``` va a ser esencialmente sólo este código, por lo que podemos cortarlo y pegarlo.

El último día del calendario ```last_calendar_day ``` se puede hacer de la misma manera sólo vamos a traer la variable local aquí y luego vamos a convertir esto en un método, vamos a devolver el valor que se establece la variable local de antes, y ahora tenemos que reemplazar esta fecha con la fecha en que le pasamos. vamos a recargar IRB. Vamos a llamar al método de nuevo y funciona. Sabemos que nuestra refactorización funcionó.

~~~ruby
require "active_support/core_ext/date"
require "active_support/core_ext/time"
require "active_support/core_ext/array"

class CalendarWeeks
    def initialize(date=Date.today)
        @date = date
    end

    def weeks
        weeks = (first_calendar_day..last_calendar_day).to_a.in_groups_of(7)
    end
    
private

    def first_calendar_day
        @date.beginning_of_month.beginning_of_week(:sunday)
    end
    
    def last_calendar_day
        @date.end_of_month.end_of_week(:sunday)
    end
end
~~~

Volvamos a Vim de nuevo. Vamos a leer nuestro código. Tenemos un inicializador que tiene fechas, pero si no le pasan una, esta sencillamente sera la fecha de hoy, se almacena en una variable de instancia, y proporciona un método llamado ```to_a``` que va a devolver un array de semanas que contienen una serie de días. Lo hace partiendo del primer día y haciendo un rango hasta el último día del calendario y luego conviertiendolo en un array llamándolo en grupos de 7.

Quiero hacer algo. Vamos a comenzar con ```Time.now.months_since(2).to_date``` y nos van a decir que esa es la fecha de inicio. Si se creamos ```CalendarWeeks.new``` y pasamos en esa nueva fecha de inicio y luego lo convertimos en un array tendremos el calendario para julio. Si tuviéramos que cambiar por 5 meses en el futuro, ahora tenemos el calendario para octubre.

~~~ruby
$ starting_date = Time.now.months_since(2).to_date
=> Fri, 03 Jul 2015
$ CalendarWeeks.new(starting_date).to_a

$ starting_date = Time.now.months_since(5).to_date
=> Sat, 03 Oct 2015
$ CalendarWeeks.new(starting_date).to_a
~~~

Nuestro objeto funciona. No tuvimos tiempo para terminarlo en HTML con el fin de mostrarlo. En el próximo episodio vamos a usar esta biblioteca que hemos creado en una aplicación Rails para imprimir un calendario. Ahora como con el procesador que vamos a construir.