El tema de hoy es `Cuando cambia el API`. Repasaremos algunos antecedentes sobre el código. Vamos a trabajar tambien en la identificación del problema. Revisaremos las solicitudes de almacenamiento en caché con una gema llamada VCR correjiremos tambien algunass pruebas obsoletas, hablaremos de pruebas rápidas frente a pruebas lentas, Y por último, tendremos algunas conclusiones y sugerencias.

## Entrando en contexto
La aplicación de este episodio es un ejemplo que escribí en respuesta a un viejo episodio de PeepCode, recuerda que en la parte de abajo del video podrás encontrar el link con el código fuente. El trabajo de la gema es conectar el API de github, y traer un "json" con los eventos de un usuario específico.

Así que, esencialmente cada vez que como usuario de github haces un push a un repositorio, creas un fork de un repositorio, dejas un comentario, o se crea un pull request. Hay una puntuación asociada a cada acción en esta aplicación. Github sólo devuelve la acción hecha por ti y el trabajo de esta aplicación es anotarlo. Así que en la carpeta `events` dentro de la carpeta `fantasyhub` basicamente hay un archivo para cada evento. Y, si miramos, por ejemplo, `pull request`.

![fantasyhub folder](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/002/images/001)

Veremos que enviar un pull request vale cinco puntos. Este objeto se implementa como un módulo que "auto extiende", asi que no crean nunca instancias de un evento de pull request. Es sólo un tipo, es un objeto de valor, similar al número cuarenta y dos. "Auto extender" en este contexto es lo mismo que poner 
`self.score` esto nos permite llamar a este objeto, así ...

~~~ruby
require 'fantasyhub/events'

module Fantasyhub::Events::PullRequestEvent
  extend self

  def score
    5
  end

end
~~~

Y el método es `score`, entonces pasandole el método `score` al evento `pull request` podemos ver que aquí no se creó una nueva instancia del evento `pull request` para anotar la puntuación. Cabe aclarar que esto se podría haber hecho de otras maneras, pero escogí hacerlo con `extend self`. Este modúlo tiene un único método llamado "score". En el caso de este evento en particular la puntuación es cinco. Al revisar por ejemplo, `create_event`.

~~~ruby
# fantasyhub/lib/fantasyhub/events/create_event.rb
def score
    3
end

# fantasyhub/lib/fantasyhub/events/push_event.rb
def score
    7
end    
~~~

La puntuación es de tres, para un evento "push" la puntuación es de siete, y así sucesivamente para cada uno de estos eventos. De esta forma mapeamos un evento "json" para cada "score". Todos los objetos ubicados en `fantasyhub/lib/fantasyhub/event.rb` tienen un trabajo. Un evento es la representación de otro evento `json`. Sólo que tiene un lector añadido para exponer algunos atributos. Esos atributos se implementan como variables de instancia dentro de una clase. Las claves se pasan como un hash, y luego nos traen cada una de las llaves fuera del hash con el fin de crear una instancia del objeto.

~~~ruby
class Fantasyhub::Event

  attr_reader :actor, :event_type, :repo_url, :score, :created_at

  def initialize(hash)
    @actor      =  hash.fetch(:actor)
    @event_type =  hash.fetch(:event_type)
    @repo_url   =  hash.fetch(:repo_url)
    @score      =  hash.fetch(:score)
    @created_at =  hash.fetch(:created_at)
  end
end

~~~

Podría haber usado una estructura-abierta, pero me gustan mantener las cosas simples, por lo que optó por utilizar una clase. La razón por la que estoy usando fetch en el hash, es porque si una de estas llaven no existe no habrá un error si no que me avisara cual es la llave que falta, en lugar de recibir un raro `no-value`. Dentro de mi carpeta feed `fantasyhub/lib/fantasyhub/feed/
` ya que estoy tratando con una fuente de eventos en formato "json" de github, tengo dentro tres objetos. Uno de ellos será sólo para descargar esos eventos. El siguiente es para analizarlo dentro de algo que pueda entender en mi aplicación, y luego, la tercera parte sería entonces puntuar el evento.

![feed folder](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/002/images/002)

Así que también tenemos un objeto `feed`, que es simplemente un espacio de nombres que representan una fuente, y es por eso que tenemos esta carpeta aquí llama "feed". Lo mismo para los eventos. `Events` es otro name space, adicionalmente queremos cargar todos los archivos dentro de "event" que terminan en la ruta `fija _event.rb`. Ya que si github fuera añadir un nuevo evento el dia de mañana, yo podría simplemente crear una nueva definición, dar una puntuación, y simplemente funcionaría.

~~~ruby
#fantasyhub/lib/fantasyhub/events.rb
require 'fantasyhub'

module Fantasyhub::Events;end

Gem.find_files("fantasyhub/events/*_event.rb").each { |path| require path }
~~~

## Hallando el problema

Ahora si corro mis pruebas `./bin/run_test_suite`, estas van a terminar en alrededor de un cuarto de segundo. Hay veintiocho de ellas con veintiocho aserciones y todas van a pasar, se ve verde y feliz. Pero, si lo precedo con `SLOW=1 ./bin/run_test_suite`, otras pruebas van ejecutarse en la parte superior de esas veintiocho, las cuales son pruebas de integración de extremo a extremo. Esta prueba falla porque el API github se ha movido a otra url. Para solucionar esto, primero necesitamos identificar porqué mi prueba de integración está fallando, sin embargo, mis pruebas unitarias están pasando. Y la respuesta a esto es en realidad bastante simple.

Esto está sucediendo porque estoy usando esta gema "VCR", que me permite grabar una conversación con un servidor remoto en un casete, con las que luego puedo ejecutar mis pruebas más tarde, asi que si estoy en un tren, por ejemplo, y No tengo internet, la conexión externa no importa porque puedo hacer pruebas contra el casette, que básicamente es una respuesta del servidor cuando estaba vivo. Por lo tanto, estos cassettes tienen casi un año y tienen datos de la antigua API. Y si quería hacer que mis pruebas unitarias dejarán de mentirme puedo hacer un `rm test/fixtures/casettes/` ya que aquí es donde todos mis casetes se almacenan.

Luego al ejecutar mis pruebas unitarias, puedo ver una pausa donde el nuevo casete esta siendo grabando, y hemos tenido dos fallas porque ahora ellos también están recibiendo este error `OpenURI:HTTP:Error: 410 Gone`.Esto se debe a que la dirección URL de github solia tener este aspecto: `github.com/userid.json`. Pero la URL de API ahora se ve así: `api.github.com/users/userid/events`.

![OpenURI error](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/002/images/003)

Asi que, veamos esto de nuevamente. Veamos donde está fallando. Está fallando en downloader, ya que el trabajo del descargador es salir a github y conseguirlo. Así que downloader es otro módulo que se auto extiende. Tiene un solo método público llamado "download". Le pasas un UID y este simplemente devuelve un feed en formato "json" de ese ID. Por lo tanto, la URL del feed en el método privado está mal, está señalando el valor antiguo. Podemos arreglarlo ...

~~~ruby
# fantasyhub/lib/fantasyhub/feed/downloader.rb
def download(uid)
  feed_for(uid)
end
~~~

Vamos a tener que poner `users` aquí, el UID y luego lo configuramos en la ruta fija con `events`. Así que ahora tenemos `api.github.com/users/#{uid}/events`. Que funcionará. Así que, si quito los cassettes que fueron registrados con el error 410, y luego los vuelvo a ejecutar mi set de pruebas unitarias, podemos ver que una de esas pruebas que estaba fallando antes, pasó. Ya no arroja el error 410.

Sin embargo, el valor de los commits ha cambiado, por lo que el score ha cambiado desde que se registró el último casete. Así que, ahora tengo que ir a `test/fantasyhub_test`, cambiar el score para que coincida con lo que el resultado se aparece hoy. El resultado que se ve actualmente es 137.

~~~ruby
require 'minitest_helper'

describe Fantasyhub do
  subject { Fantasyhub }
  describe "event_scores_sum_for(uid)" do
    it "must get the sum of the event scores" do
      VCR.use_cassette("tenderlove_activity") do
        subject.event_scores_sum_for('tenderlove').must_equal 148
      end
    end
  end
end
~~~

Por lo tanto, vamos a cambiar esto por 137, y ahora puedo volver a ejecutar mis pruebas, tengo veintiocho de veintiocho pruebas unitarias que pasan de nuevo, que es donde empezamos. Pero, esta vez en realidad son correctas y si las corro con el flag `SLOW=1 ./bin/run_test_suite` para ejecutar mis pruebas de integración de extremo a extremo, podemos ver que ahora las prueba de integración estan correjidas, así como las pruebas unitarias.

## Pruebas lentas vs Pruebas rápidas

Ahora quiero hablar un poco acerca de las pruebas rápidas y pruebas lentas. Es importante en el ciclo de TDD, si vas a hacer TDD, y lo recomiendo encarecidamente, que sus pruebas respondan casi inmediatamente. No se puede tener un montón de tiempo de configuración y un montón de tiempo de espera cuando se está en medio de un ciclo de TDD. Si pierde el contexto, se pierde todo.

Por lo tanto, cuando estas en el ciclo de TDD es beneficioso tener todas las pruebas que serán pruebas de sistemas, pruebas de integración, como quieras llamarlo, pruebas que prueban más de una cosa y ponen a prueba más de un sistema. En esencia lo que yo llamo pruebas lentas, necesitas una manera de segregar estas. Por lo tanto, te habrás dado cuenta de que yo no corro mis pruebas con `rake`, yo las corro un script de pruebas que escribí. Así que voy a abrirlo ...

~~~ruby
#!/usr/bin/env ruby
exit(1) if __FILE__ != $0

require 'bundler/setup'
require 'webmock'

if ENV.has_key?("SLOW")
  if ENV.has_key?("CODECLIMATE_REPO_TOKEN")
    require "codeclimate-test-reporter"

    WebMock.disable_net_connect!(:allow => "codeclimate.com")
    CodeClimate::TestReporter.start
  end
end

$LOAD_PATH.unshift('lib', 'test')

fast_tests = `find ./test -name *_test.rb -print | xargs grep -l "minitest_helper"`
Dir.glob(fast_tests.split("\n")) { |f| puts f; require f }

exit(0) unless ENV.keys.include?("SLOW")

slow_tests = `find ./test -name *_test.rb -print | xargs grep -L "minitest_helper"`
Dir.glob(slow_tests.split("\n")) { |f| puts f; require f }
~~~

Esto es un archivo de Ruby muy simple, mi script de pruebas. Vamos a buscar una llave de ambiente llamada `slow`. Si existe, vamos a buscar `CODECLIMATE_REPO_TOKEN`. Si existe vamos a decirle a `WebMock` que permita la llamada a `codeclimate` y vamos a hacer que los reportes de prueba comienzen por allí. Lo siguiente es que tenemos que cargar "lib" y los directorios de "test" en nuestro path de Ruby para poder requerir algo así como `fantasyhub` en lugar de `lib fantasyhub`.

Ahora puedes ver que tambien tengo lo que yo llamo las pruebas rápidas. Y esencialmente ejecuto un comando de UNIX para encontrar todos los archivos de prueba en el directorio prueba que terminan con el prefijo *_test*. Los imprimo y luego los entubo en "
`xargs`, que es algo así como un objeto `for_each`. Va a agrupar cada uno de ellos y va a buscar la cadena `minitest_helper`.

Si tiene "minitest_helper" en ella, va a ser considerado una prueba rápida, no una prueba de integración. A continuación, englobamos todos estos archivos y ponemos cada uno de ellos en un require, que en realidad es el responsable de cargar el archivo. Y si se carga un archivo en Ruby esté ejecutará las pruebas.

Entonces tenemos una salida a menos que la llave `slow` no este en ella. Si las llaves tienen `slow` en ellas, vamos a hacer algo muy similar con nuestra prueba lenta. La única diferencia es en realidad que vamos a hacer un flag `l` mayúscula en el grep, que va a ser una de exclusión en lugar de una búsqueda de inclusión.

Por lo tanto, cualquier archivo `test.rb` que no tiene `minitest_helper` va a ser considerado una prueba lenta. Así que, por lo mismo, englobamos esas pruebas lentas. Y las imprimimos en la consola, y las requerimos. Ahora voy a abrir mi prueba de integración. Así, empezamos cargando `thincloud/test`, que es un helper de pruebas de la gema `thincloud/test`. Vamos a continuación, a cargar `fantasyhub`, que es esencialmente el puerto principal de entrada a la aplicación.

Este es minitest utilizando la versión de spec, así que estamos haciendo un describe. Estamos diciendo "Hey, esta es la prueba de extremo a extremo contra github." En mis pruebas me gusta tener un solo subject para no tener que hacer múltiples búsquedas de múltiples cosas. Del mismo modo que no me gusta tener múltiples variables de instancia en mis vistas. El subject es sólo `fantasyhub.event_scores_sum_for(fantasyhubber)`.

Este usuario fantasyhubber es simplemente un usuario que ha hacho varias acciones, así que sé que sus puntos no van a cambiar dañar mis pruebas. Ahora vamos a apagar el VCR para que permita la solicitud a través WebMock. Vamos a decir que el `score`, que está, por supuesto, va a evaluar a `fantasyhub.event_scores_sum_for(fantasyhubber)`, y que la puntuación va a ser igual a uno.

Ahora voy a dividir la pantalla y voy cargar una prueba rápida. Por lo tanto, una prueba rápida ... Lo único que carga en la parte superior es `minitest_helper`. `minitest_helper` hace todo el resto de trabajo. Así es como sé que es una prueba rápida. `minitest_helper` va a cargar VCR, hasta WebMock. Aquí es donde yo llamo el casete que quiero usar. Allí es donde pruebo contra el casete.

~~~ruby
require "thincloud/test"
require 'fantasyhub'

describe "End to end live test against github" do
  subject { Fantasyhub.event_scores_sum_for("fantasyhubber") }

  it "expects the score for fantasyhubber to be 1" do
    VCR.turned_off do
      subject.must_equal 1
    end
  end
end
~~~

## Conclusiones y sugerencias.

Ahora veremos algunas conclusiones y sugerencias para este episodio: 
Saca las pruebas lentas de tu bucle TDD.
Asegúrese de que siempre tenga una prueba de humo de extremo a extremo.
Utiliza VCR para grabar las conversaciones con las API remotas.
Construye pequeños objetos enfocados que son fáciles de cambiar.
Escribe (poros "plain Ruby objects") siempre que sea posible.
Utilice la pruebas de integración continua para detectar problemas.

Si te gustó el episodio de hoy y quieres apoyar mis screencast con contenido Ruby de alta calidad como este en el futuro, puedes convertirte en un patrocinador mediante una suscripción por sólo catorce dólares al mes para Rubycast.io. Esto no sólo me ayuda a producir excelentes screencast en el futuro, sino que también recibes veinticinco dólares en mi tarifa por hora de codementor.io. Eso es todo por el episodio de esta semana de Rubycast.io, pero nos veremos de nuevo el próximo martes donde continuaremos para cavar en el stack de desarrollo con Ruby.

