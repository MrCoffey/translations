En el episodio de esta semana vamos a estilizar el objeto `week` del calendario que creamos la semana pasada para que no se limite a devolver una matriz de días si no que devuelva los días junto con alguna información CSS que luego podemos utilizar para darle estilo. Luego, vamos a conectar esta pequeña biblioteca con una aplicación hecha en Rails, crear algunas vistas y parciales.

Aquí está el archivo que creamos la semana pasada. Vamos a empezar por crear un nuevo objeto. Que llamaremos `calendar` este va a envolver el otro objeto, por lo que más o menos significa que vamos a utilizar la misma interfaz. Guardare esta variable de instancia asi tendremos acceso a ella dentro de todos nuestros otros métodos. Tendremo tambien un método publico llamado `to_a`, este método va a devolver el objeto anterior `CalendarWeeks.new` y pasaremos la fecha en un array con `to_a` ... Lo que hicimos por el momento fue cambiar el nombre de nuestro objeto `CalendarWeeks` a `Calendar`. Por lo que si se vuelve a ejecutar nuestro archivo aquí, simplemente podemos reemplazar `Calendario` por `CalendarWeeks`, y obtener el exactamente el mismo comportamiento que el objeto CalendarWeeks.

~~~ruby
class Calendar
  def initialize(date=Date.today)
    @date = date
  end

  def to_a
    CalendarWeeks.new(@date).to_a
  end
end
~~~

El siguiente paso va a ser para decorar esta información con CSS. Esto lo vamos a hacer con el metodo `map` sobre lo que devuelve el objeto `CalendarWeeks`. Cada iteración de esta nos va a devolver una semana, así que entonces, si usamos `map` en cada semana tendremos fechas. Una vez más, si nos detenemos aquí, no hemos realmente cambiado el comportamiento de la aplicación. Tenemos el mismo comportamiento.

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/004/images/001.png)

Ahora, vamos a cambiar ese comportamiento. Para no devolver unicamente la fecha ... Regresando la fecha como un valor inicial, y como segundo valor devolviend clases CSS para esas fechas, podemos hacer un método privado llamado `css_classes_for(date)`, Esto devolveríamos algo más o menos como `past` y `other_month` porque ... Digamos que empezamos con la primera fecha del calendario que el mes pasado. Obviante esta en el pasado. Y no hace de este mes. Es parte del final del mes pasado. Entonces, nos gustaría compactar lista de calendarios y de deshacernos de los posibles valores nulos, asi que aqui es útil unirlos con un espacio, y asi conseguimos que un valor tienga este aspecto.

~~~
class Calendar
  def initialize(date=Date.today)
    @date = date
  end

  def to_a
    CalendarWeeks.new(@date).to_a.map do |week|
        [date, css_classes_for(date)]
    end
  end

private

  def css_classes_for(date)
   ["past", "other-month"].compact.join(" ")  
  
   "past other-month"
  end
  
end
~~~

Ahora, vamos a tomar este pseudo código y hacer que realmente haga algo. Podemos reemplazar la cadena con un método, asi que vamos a empezar con sólo con past. Este por ahora es un método que devuelve la cadena `past` y la fecha ... Esto significa que necesitamos ambos tanto `past` como su fecha. Comparemos esa fecha para que sea menor a `Date.today`. Tenemos otra y es `def other_month` con éste devolveríamos `other-month` si `date.month` no es igual a `Date.today.month` esto necesitará otra fecha también. Ahora, necesitamos también del método `other_month`, que toma esa fecha.

~~~ruby
 def css_classes_for(date)
   [past(date), other_month(date)].compact.join(" ")  
 end
 
 def past(date)
   "past" if < Date.today
 end
 
 def other_month(date)
    "other-month" if date.today != Date.today.month
 end
~~~

Podríamos seguir por este camino, pero se está haciendo muy complicado. No va a leer muy bien `past(date), other_month(date).` hay demasiadas repeticiónes de `date` aquí. Lo que esto nos quiere decir es que hay un objeto que quiere salir. La forma en que voy a hacer esto es de hecho cerrando la clase anterior y creando una nueva clase. Esta se a llamar `DayStyles.` luego, hay que pasar el designador privado aquí, hacer de este un método público. Vamos a cambiar eso como un `to_s.` Vamos a cambiar el objeto `DayStyles` en una cadena, por lo que vamos a utilizar `to_s.` Esto va a tener un inicializador que va a tomar una fecha. Esta vez no vamos a permitir una fecha vacía, y vamos a salvar a esa fecha fuera en una variable de instancia. Eso significa que no tenemos que llamar `date` una y otra vez aquí, además limpia nuestro código. No tenemos que llamarla aquí. De hecho, tenemos acceso a esta fecha en una variable de instancia. Lo mismo aquí. Ahora fecha variable de instancia `date`.

~~~ruby
class Calendar
  def initialize(date=Date.today)
    @date = date
  end

  def to_a
    CalendarWeeks.new(@date).to_a.map do |week|
      week.map do |date|
        [date, css_classes_for(date).to_s]
      end
    end
  end
end

class DayStyles
  def initialize(date)
    @date = date
  end

  def to_s
    [past, other_month].compact.join(" ")
  end

private

  def past
    "past" if @date < Date.today
  end

  def other_month
    "other-month" if @date.month != Date.today.month
  end
end
~~~


Vamos a continuar construyendo esto. Por ahora vamos a preocuparnos por `past`. También queremos que saber si es hoy. También tenemos que saber si esta en el futuro. `future` y los métodos de `today` se parecen mucho a el método de `past`. Va a devolver la cadena `today` si el día es igual a esta fecha. Entonces, tenemos el método de `future`, que sólo va a devolver la cadena `future` si la fecha es mayor que la fecha actual. Ahora, ya que la fecha es hoy. Pasado va a ser nula. `today` va ha ser la cadena actual, asi que `future` y other_month van a ser nulos. Aquí es donde `compact` compacta y aplana el array de tal forma que no tengamos ningún valor en `nil`, luego  unimos a los valores restantes, junto con un espacio vacío.

~~~ruby
class DayStyles
  def initialize(date)
    @date = date
  end

  def to_s
    [past, today, future, other_month].compact.join(" ")
  end

private

  def past
    "past" if @date < Date.today
  end

  def today
    "today" if @date == Date.today
  end

  def future
    "future" if @date > Date.today
  end

  def other_month
    "other-month" if @date.month != Date.today.month
  end
end
~~~

Hay una última cosa que hacer aquí. Ya cambiamos para dejar de ser un método del objeto `calendar` para ser su propio objeto llamado `DayStyles.` Subamos de nuevo vamos a llamar a `to_s` en él, que se puede ver aquí. El método público es `to_s`, y ahí es donde esto sucede. Si ponemos esto de fondo y volvemos a ejecutar nuestro código, ahora vamos a obtener la fecha seguida de alguna información CSS extra. Si el día del calendario es parte del último mes, tenemos `other_month` y la palabra `past`. Si es parte del mes futuro, tenemos `other_month` y `future`, y si nos fijamos por aquí veremos que hoy esta en efecto marcado con `today`.

~~~ruby
class Calendar
  def initialize(date=Date.today)
    @date = date
  end

  def to_a
    CalendarWeeks.new(@date).to_a.map do |week|
      week.map do |date|
        [date, DayStyles.new(date).to_s]
      end
    end
  end
end

class DayStyles
  def initialize(date)
    @date = date
  end

  def to_s
    [past, today, future, other_month].compact.join(" ")
  end

private

  def past
    "past" if @date < Date.today
  end

  def today
    "today" if @date == Date.today
  end

  def future
    "future" if @date > Date.today
  end

  def other_month
    "other-month" if @date.month != Date.today.month
  end
end
~~~

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/004/images/002.png)

Ahora, vamos a poner esto en una aplicación Rails. En el próximo episodio vamos a repasar este código de Rails con más detalle, sin embargo quería mostrarles como se va a ver nuestro calendario. Aquí está la acción `show` que renderiza el calendario. Vamos a crear un elemento principal, con el id `calendar` para que coincida con nuestra hoja de estilos. Vamos a renderizar un parcial llamado `header`, que sólo devuelve los días, Luego, vamos a hacer esta semana un parcial, uno para cada semana en el interior de nuestro objeto, que pasa a ser seis veces. En el interior de cada uno renderizamos un parcial `day` y cada vez que el día parcial se hace, se muestra con una fecha con los dos dígitos en ese día, 26, 01, 02. En el div habrá también una clase de día, y de aquellos valores que vienen del objeto que acabamos de crear.

~~~ruby
# rubycast_calendar/app/views/calendars/show.html.erb

<main id="calendar">
  <%= render partial: 'headers' %>
  <%= render partial: 'week', collection: @calendar %>
</main>

# rubycast_calendar/app/views/calendars/_header.html.erb

<section class="th">
  <span>Sunday</span>
  <span>Monday</span>
  <span>Tuesday</span>
  <span>Wednesday</span>
  <span>Thursday</span>
  <span>Friday</span>
  <span>Saturday</span>
</section>

#rubycast_calendar/app/views/calendars/_week.html.erb 

<div class="week">
  <%= render partial: 'day', collection: week %>
</div>

#rubycast_calendar/app/views/calendars/_day.html.erb 
<div data-date="<%= day[0].strftime("%d") %>" class="day <%= day[1] %>">
</div>
~~~


El estilo está a cargo de esta hoja de estilo. Podemos ver aquí el id principal `#calendar`. La última cosa a tener en cuenta antes de ver esto es su propio div ... Si nos remontamos parcial `day`, el div no tiene ningún contenido en su interior. Se tiene la definición div, pero no tiene contenido dentro. El número de días se representa en el calendario de esta línea de aquí. Tenemos tambien una clase de semana que va a tener un div dentro de ella, que es uno de esos días. Vamos a establecer el contenido del div en lo que sea que en esta fecha. Y sin más preámbulos, ta da! Aquí está nuestro calendario completamente estilizado. Si vamos a la derecha y hacemos clic en el 26to, podemos ver que sí tiene los estilos de `día` y `pasado` y `other_month`, que es exactamente lo que queríamos. Si echas un vistazo a la fecha de hoy, que es el 11, podemos ver que se encuentra en posición de hecho con la clase de `today`.

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/004/images/003.png)

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/004/images/004.png)

Eso es todo lo que tenemos tiempo para en el episodio de esta semana de RubyCasts, pero la próxima semana vamos a repasar el código de Rails que muestra esto con más detalle. Hablaremos de los controladores, la rutas, y de cómo funciona exactamente Rails sabremos cómo representar lo que hace y por si fuera poco, vamos a utilizar algun de CoffeeScript para que podamos registrar eventos dentro de la fecha de hoy y las fechas en el futuro, más no las fechas en el pasado. En el siguiente episodio vamos a estar haciendo `full stack` de Ruby, hasta rails con CoffeeScript. Nos vemos!.
