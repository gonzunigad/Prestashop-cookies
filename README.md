# Prestashop-cookies

Hola a todos, un poco acerca de nosotros, Multiservicios Austral es una empresa integradora de tecnología y Partner de Prestashop, nos especializamos en realizar plataformas de comercio electrónico personalizadas sin uso de plantillas y servidores optimizados para comercio electrónico.

La solución incluida en estos archivos nos llevó más de una semana de trabajo hasta las 4 AM probando múltiples opciones.

Siempre estamos dispuestos a colaborar con la comunidad, pero para pagar las cuentas necesitamos su apoyo con su donación a través de https://khipu.com/ por este link https://kh.cm/9rxxG

Tome en cuenta que su donación nos ayudara a seguir colaborando con la comunidad.

*** EXPLICACIÓN ***

Esta falla, como todo deben saber, se debió a que Chrome, en su versión 80, aumento la seguridad en la navegación colocándole a todas las Cookies que no tengan la etiqueta SameSite definida, el Tag SameSite=Lax.

Esto lo que causa es que, por ejemplo, al pagar con la pasarela de pago webpay, si el cliente demora más de 2 minutos en hacer el pago, la transacción da error al retornar a Prestashop, no registrando la transacción.

*** SOLUCIÓN (Basada en PS 1.7.6.7) ***

Para solucionar esta falla hay que realizar una modificación al Core de Prestashop, para ello nos vamos al siguiente archivo:

carpeta de instalación de PS -> classes -> Cookie.php

Una vez abierto el archivo nos vamos a la línea 375 y borramos la línea:

return setcookie($this->_name, $content, $time, $this->_path, $this->_domain, $this->_secure, true);

Y la sustituimos por:

if (Configuration::get('PS_SSL_ENABLED') == 0 or strpos($_SERVER['REQUEST_URI'], 'admin') === 1){
  return setcookie($this->_name, $content, $time, $this->_path, $this->_domain, $this->_secure, true);
}else{
  if (PHP_VERSION_ID < 70300) {
    return setcookie(
      $this->_name,
        $content,
        $time,
        $this->_path,
        $this->_domain . '; SameSite= None',
        $this->_secure,
        true
      );
   }else{
    return setcookie(
      $this->_name,
        $content,
          [
            'expires' => $time,
            'path' => $this->_path,
            'domain' => $this->_domain,
            'secure' => $this->_secure,
            'httponly' => true,
            'samesite' => 'None',
           ]
     );
  }          
 }
 
 *** EXPLICACIÓN DEL CÓDIGO***
 
 if (Configuration::get('PS_SSL_ENABLED') == 0 or strpos($_SERVER['REQUEST_URI'], 'admin') === 1)
 
Con esta línea validamos como primera instancia es el uso de SSL, si no tiene SSL activo no funciona WebPay u otra plataforma de pago y adicionalmente si no tiene SSL y coloca el tag SameSite=None a la Cookie, la plataforma PS deja de funcionar.

Esta la validación la agregamos ya que en ambiente de desarrollo no siempre usamos SSL.

La siguiente validación es que la página que este visitando el usuario no sea el área administrativa (Backoffice) ya que no funciona con tag en Cookie SameSite=None.

if (PHP_VERSION_ID < 70300)

Con esta línea validamos (a futuro) que si se está usando PHP 7.3 o superior, la forma de agregar el tag a la Cookie cambió a partir de PHP 7.3.

Actualmente la última versión de Prestashop al día de hoy 1.7.6.7, no soporta PHP 7.3, funciona hasta PHP 7.2 y no existe de forma nativa, en esta versión, el tag SameSite y por tal motivo no se puede declarar.

El resto del código, es simplemente la declaración de los tags de las Cookies.

Esperamos que esta explicación les ayude a solucionar este dolor de cabeza que tiene tantas plataformas fuera de servicio.

No se olviden en apoyarnos con tu donación https://kh.cm/9rxxG

 
