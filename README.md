# crud-cakephp
JHON SEBASTIAN MOSQUERA OCAMPO - 1072710129
PRUEBA TÉCNICA DESARROLLADOR FULL STACK
Enunciado 
Realizar un sistema de login con usuario correo y contraseña encriptada en MD5, crear un API 
que permita las acciones CRUD (mínimo 5 campos y requerido un alfanúmerico de 41 caracteres 
seguidos), finalmente se requiere exportar los datos en EXCE, CSV y TXT.
DESARROLLO 
1. PRERPARANDO EL ENTORNO 
Para realizar esta prueba, se ha descargado la distribución XAMPP de Apache. 
Link de descarga: https://www.apachefriends.org/es/download.html
En la documentación oficial del framework, no avisa que si usamos XAMPP, debemos 
descomentar la línea “extension=intl” en el archivo php.ini (/xampp/php/php.ini)
Ahora, habilitamos los módulos Apache y MySQL. Por defecto, el módulo MySQL tiene
configurado el puerto 3306, para este desarrollo, se ha cambiado al puerto 3307 dado 
que es un puerto que no tengo disponible en el computador actual.
La carpeta raíz donde vamos a alojar nuestros proyectos es la siguiente ruta: 
/xampp/htdocs
2. INSTALAR COMPOSER 
El siguiente paso, es descargar composer. 
Link de descarga: https://getcomposer.org/download/
3. INICIAR APLICACIÓN CAKEPHP 
Ya podemos iniciar nuestro proyecto con cakephp. Para ello, entramos a la carpeta raíz de 
nuestro proyecto, abrimos la consola y escribimos la siguiente instrucción: composer 
create-project --prefer-dist cakephp/app:~4.0 my_app_name
my_app_name es el nombre de nuestro proyecto que para esta prueba, escribimos Project.
4. CREAR BASE DE DATOS EN PHPMYADMIN 
En este paso entramos a nuestro administrador phpmyadmin y creamos la base de datos 
que necesitemos. Para este proyecto, creamos la tabla users.
5. CONFIGURAR LA BASE DE DATOS 
Ahora necesitamos decirle a nuestro proyecto que se conecte con nuestra base de datos. 
Para ello, entramos al archivo app_local que se encuentra dentro de la carpeta config y 
definimos:
'host' => 'localhost:3307',
 'username' => 'root',
 'password' => '',
 'database' => 'users',
Estos valores cambiarán de acuerdo al puerto de mysql, del usuario, contraseña y nombre 
de la base de datos establecida
6. REALIZAR LAS MIGRACIONES 
Cakephp nos facilitar realizar migraciones a nuestra base de datos, y adicionalmente, 
realizar todo el andamiaje para las acciones (crud) hacia nuestra base de datos.
Para ello, utilizamos el siguiente comando para crear nuestra migración:
1. Crear nuestra migración con los campos que va a requerir nuestra base de datos
bin/cake bake migration CreateUsers email:string:unique phone_number username 
country_code password created modified
Una vez ejecutada esta instrucción, se nos crea una carpeta con la migración que acabamos 
de realizar
2. Hacer efectiva la migración. Necesitamos que la migración que creamos se realice en 
nuestra base de datos. Para ello, ejecutamos el comando
bin/cake migrations migrate
3. Finalmente, creamos todo nuestro andamiaje o sistema con la siguiente instrucción
bin/cake bake all users
En este punto, ya tenemos todo nuestro CRUD hecho sin haber escrito una línea de código, 
una de las grandes ventajas que nos proporciona cackePHP. Ahora lo que vamos a hacer 
es cambiar las cosas que se necesiten para cumplir cada requisito solicitado en la prueba.
7. CONFIGURAR DATATABLES 
Datatables es una excelente herramienta para manejar nuestras tablas, con este podemos 
paginar, ordenar, filtrar, exportar a Excel, csv, txt, entre otras muchas funcionalidades más.
Para utilizar esta funcionalidad, vamos a utilizar mediante cdn. Para ello, nos vamos al 
archivo default.php que se encuentra dentro de la carpeta /src/layout y en el head, 
escribimos las siguientes líneas:
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
 <!-- Datatable JS -->
 <script src="https://cdn.datatables.net/1.12.1/js/jquery.dataTables.min.js"></script>
 <script 
src="https://cdn.datatables.net/buttons/2.2.3/js/dataTables.buttons.min.js"></script>
 <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.1.3/jszip.min.js"></script>
 <script 
src="https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.53/pdfmake.min.js"></script>
 <script 
src="https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.53/vfs_fonts.js"></script>
 <script 
src="https://cdn.datatables.net/buttons/2.2.3/js/buttons.html5.min.js"></script>
Para implementarlo, basta con escribirle un id a la tabla que se encuentra en el archivo 
index.php dentro de la carpeta /templates/Users y en el script, el siguiente código:
$(document).ready(function() {
 $('#myTable').DataTable( {
 dom: 'Bfrtip',
 buttons: [
 'copyHtml5',
 'excelHtml5',
 'csvHtml5',
 'pdfHtml5'
 ]
 } );
 } );
El código que se encuentra dentro de DateTable es el necesario para exportar en Excel, csv 
y txt
8. INSTALAR PLUGIN PARA LA AUTENTICACIÓN 
CakePHP provee de un plugin que nos facilita la autenticación de nuestra aplicación. Para
ello, instalamos el plugin mediante la siguiente instrucción:
composer require "cakephp/authentication:^2.4"
Ahora, necesitamos configurar nuestro método de encriptación. Para ello, escribimos el 
siguiente método
protected function _setPassword(string $password) : ?string
 {
 if (strlen($password) > 0) {
 return (new DefaultPasswordHasher())->hash($password);
 }
 }
No olvidar, agregar la siguiente importación: use 
Authentication\PasswordHasher\DefaultPasswordHasher;
Ahora vamos a configurar el plugin de autenticación.
En src/Application.php, agregamos:
1. Imports
use Authentication\AuthenticationService;
use Authentication\AuthenticationServiceInterface;
use Authentication\AuthenticationServiceProviderInterface;
use Authentication\Middleware\AuthenticationMiddleware;
use Cake\Routing\Router;
use Psr\Http\Message\ServerRequestInterface;
2. Implementar la interface de nuestro plugin a la clase
3. class Application extends BaseApplication
 implements AuthenticationServiceProviderInterface
{
4. En la función middleware, justo después del bodyparser, escribimos la siguiente línea
->add(new AuthenticationMiddleware($this));
5. Escribimos nuestra función para obtener el servicio, en el mismo archivo donde estamos 
trabajando
public function getAuthenticationService(ServerRequestInterface $request): 
AuthenticationServiceInterface
{
 $authenticationService = new AuthenticationService([
 'unauthenticatedRedirect' => Router::url('/users/login'),
 'queryParam' => 'redirect',
 ]);
 // Load identifiers, ensure we check email and password fields
 $authenticationService->loadIdentifier('Authentication.Password', [
 'fields' => [
 'username' => 'email',
 'password' => 'password',
 ]
 ]);
 // Load the authenticators, you want session first
 $authenticationService->loadAuthenticator('Authentication.Session');
 // Configure form data check to pick email and password
 $authenticationService->loadAuthenticator('Authentication.Form', [
 'fields' => [
 'username' => 'email',
 'password' => 'password',
 ],
 'loginUrl' => Router::url('/users/login'),
 ]);
 return $authenticationService;
}
6. Para checkear el resultado de la autenticación, nos vamos al archivo appController 
dentro de la carpeta /src/Controller, y escribimos la siguiente línea:
$this->loadComponent('Authentication.Authentication');
7. Ahora vamos a configurar que solo la ruta login no va a tener restricción de 
autenticación
public function beforeFilter(\Cake\Event\EventInterface $event)
{
 parent::beforeFilter($event);
 // Configure the login action to not require authentication, preventing
 // the infinite redirect loop issue
 $this->Authentication->addUnauthenticatedActions(['login']);
}
9. Ahora, creamos nuestro método login en el archivo UserController.php en la carpeta 
/src/Contoller
public function login()
{
 $this->request->allowMethod(['get', 'post']);
 $result = $this->Authentication->getResult();
 // regardless of POST or GET, redirect if user is logged in
 if ($result && $result->isValid()) {
 // redirect to /articles after login success
 $redirect = $this->request->getQuery('redirect', [
 'controller' => 'Articles',
 'action' => 'index',
 ]);
 return $this->redirect($redirect);
 }
 // display error if user submitted and authentication failed
 if ($this->request->is('post') && !$result->isValid()) {
 $this->Flash->error(__('Invalid username or password'));
 }
}
10. Ahora solo tenemos que crear la vista correspondiente. Para ello, nos vamos a la 
carpeta /layput/User y creamos el archivo login.php. Dentro de este archivo, creamos 
el formulario para iniciar sesión
<!-- in /templates/Users/login.php -->
<div class="users form">
 <?= $this->Flash->render() ?>
 <h3>Login</h3>
 <?= $this->Form->create() ?>
 <fieldset>
 <legend><?= __('Please enter your username and password') ?></legend>
 <?= $this->Form->control('email', ['required' => true]) ?>
 <?= $this->Form->control('password', ['required' => true]) ?>
 </fieldset>
 <?= $this->Form->submit(__('Login')); ?>
 <?= $this->Form->end() ?>
 <?= $this->Html->link("Add User", ['action' => 'add']) ?>
</div>
11. Solo nos falta configurar nuestro algoritmo de encriptación MD5. Para ello, nos vamos 
a la siguiente ruta /vendor/cakephp/authentication/src/PasswordHasher y entramos al 
archivo: DefaultPasswordHasher.php y cambiamos el contenido de la función hash, por 
la siguiente línea:
return md5($password);
Y en la función check, cambiamos el contenido por: 
return $hashedPassword === $this->hash($password);
Con esto, hemos terminado la prueba técnica. Los estilos css ya dependen del gusto del 
desarrollador o en su defecto, de requerimientos que se soliciten adicionales. 