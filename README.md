# Amazon S3 - AWS Elastic Block Store (EBS)


1. Inicia sesión en el sandbox del curso. Crea un nuevo volumen de EBS con el
siguiente comando.
aws ec2 create-volume --size 1 --region us-east-1 --availability-zone us-east-1c

       ¿Qué significa este comando?
       
       Este comando nos permite listar los buckets que tenemos dentro de nuestro aws s3 
       
       ¿Qué significa este comando?

       No muestra ninguna salida porque se encuentra vacío

      ![ Captura 01 ](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/uno.PNG)

2. Haz un bucket a través del siguiente comando mb: aws s3 mb s3://tu_nombre_de_usuario
       
       ¿Cuál es la salida?
       
       Utilizando el comando aws s3 mb s3://rubencabrera , 
       podemos apreciar que ya se creo el bucket rubencabrera
       
      ![ Captura 02 ](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/dos.PNG)

3. Enumera el contenido del bucket a través del siguiente comando ls.

       aws s3 ls s3://tu_nombre_de_usuario

       ¿Cuál es la salida?
       
        Utilizando el comando aws s3 ls s3://rubencabrera , 
        listamos el contenido del bucket rubencabrera, 
        el mismo que esta vacio.
        
      ![ Captura 013](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/tres.PNG)  
    
 4. Crea un directorio llamado páginas web (mkdir webpages) y cd en ese directorio.
    Crea un archivo html simple llamado hello.html con el siguiente contenido.
      ''' 
      <html><body>
       <h1>Amazon S3</h1>
       Hello World!
       </body></html> 
       '''
    Carga el archivo en tu bucket s3 y póngalo a disposición del público con lo siguiente.
    aws s3 cp hello.html s3://tu_nombre_de_usuario --acl public-read
    ¿Cuál es la salida?
 5. Dado que se puede acceder a tu objeto s3 a través de Internet, probémoslo. En el
    navegador web de tu máquina virtual (u otra9 accede a la URL
    http://s3.amazonaws.com/tu_nombre_de_usuario/hello.html. ¿Qué viste en el navegador?
    
# Parte 2: alojamiento de sitios web estáticos con S3 h3

6. Podemos usar el bucket como almacenamiento de sitios web estáticos.
   Experimentamos con eso aquí. Crea dos archivos html en el directorio actual
   llamados index.html y error.html. El contenido de los dos archivos se muestra a continuación.
       ''' 
       <html><body>
       This is an index page!
       </body></html>
       <html><body>
              Sorry, we can't find that page!
       </body></html> 
       '''
    
  
  


    
    
