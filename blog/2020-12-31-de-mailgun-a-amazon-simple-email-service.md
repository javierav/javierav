# De Mailgun a Amazon Simple Email Service

Desde hace algunos años mi proveedor para enviar emails transaccionales es [Mailgun](https://mailgun.com):
su panel de control era sencillo de usar, me proporcionaba métricas y webhooks y su precio era asumible. Pero ayer todo
se torció y decidí probar [Amazon Simple Email Service (SES)](https://aws.amazon.com/es/ses/). Esta es la historia.

Siempre he usado Mailgun a través de su interfaz SMTP para el envío de los emails, ya que no necesitaba de la parte
extendida que proporciona el envío a través del API REST también disponible. Pero hace algunos meses, en un proyecto
personal que usa Rails 6.0 decidí enviarlos por el API haciendo uso de la gema
[mailgun-ruby](https://github.com/mailgun/mailgun-ruby): todo funcionaba aparentemente bien... hasta que actualizé ayer
el proyecto a Rails 6.1.

Tras actualizar, y al ejecutar los tests con RSpec, se mostraba el siguiente warning por consola:

```
DEPRECATION WARNING: Initialization autoloaded the constants ActionText::ContentHelper and ActionText::TagHelper.

Being able to do this is deprecated. Autoloading during initialization is going
to be an error condition in future versions of Rails.

Reloading does not reboot the application, and therefore code executed during
initialization does not run again. So, if you reload ActionText::ContentHelper, for example,
the expected changes won't be reflected in that stale Module object.

These autoloaded constants have been unloaded.

In order to autoload safely at boot time, please wrap your code in a reloader
callback this way:

    Rails.application.reloader.to_prepare do
      # Autoload classes and modules needed at boot time here.
    end

That block runs when the application boots, and every time there is a reload.
For historical reasons, it may run twice, so it has to be idempotent.

Check the "Autoloading and Reloading Constants" guide to learn more about how
Rails autoloads and reloads.
 (called from <top (required)> at /Users/javierav/supersecretproject/config/environment.rb:5)
```

Tras buscar incidencias similares, di con el posible origen del problema en una issue de Rails:
[#36546](https://github.com/rails/rails/issues/36546), así que me puse a revisar toda mi aplicación a ver si era cosa de
alguna configuración que estaba realizando mal en mi proyecto. Al ver que todo parecía estar bien, empecé a depurar
a más bajo nivel modificando las propias gemas de Rails para poder obtener más información del problema. Y todo apuntaba
a que si comentaba la siguiente línea en el archivo `config/application.rb` dejaba de producirse el warning:
`require 'action_text/engine'`

Esto me dejaba aún más loco, porque... ¿cómo va a fallar algo en el core de Rails y más aún nadie se ha dado cuenta y no
ha sido reportado como issue? Así que mi intuición me decía que tendría que venir de otro lado... como depurando de esa
forma no estaba obteniendo resultados, decidí depurar por el método científico: ensayo y error. Así que me copié todo el
proyecto en una nueva carpeta y empecé a quitar trozos de código y gemas y ejecutar los tests para ver si el warning
desaparecía... hasta que bingo, desapareció. Y desapareció justo al quitar del `Gemfile` la gema `mailgun-ruby`. WTF! Si
es una gema para enviar correos y en la issue de Rails se hace referencia al `ActionController::Base`...

Una vez localizada la gema que fallaba, el siguiente paso es ir a Github y ver si esa gema tiene alguna issue reportada,
por lo que me fuí al repositorio y mi sorpresa fue aún mayor cuándo vi que efectivamente había un
[pull request](https://github.com/mailgun/mailgun-ruby/pull/162) desde hace casi dos años! Pero es que la
[issue](https://github.com/mailgun/mailgun-ruby/issues/86) que resuelve es de hace casi tres años! Y en todo este tiempo
nadie del equipo de Mailgun se ha parado a mirar el problema, un problema que tienen sus clientes que además pagan por
el producto. Y eso dice mucho de una empresa.

Así que después de tantos años usando un producto, he decidido abandonarlo al sentirme abandonado por la propia compañía
y darle la oportunidad a otro. Y en ese sentido Amazon SES tiene buena pinta y precios bastante competitivos frente a la
competencia, por lo que le daré una oportunidad para ver que tal funciona, primero en un proyecto propio que envía pocos
emails para luego usarlo en algún proyecto más serio. Pero eso ya será una historia para otra entrada del blog.
