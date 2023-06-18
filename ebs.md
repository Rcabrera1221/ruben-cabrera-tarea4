 # Amazon S3 - AWS Elastic Block Store (EBS)

  ## EBS

    En este laboratorio, se utilizará la CLI de AWS para crear un volumen y una instantánea de Amazon EBS 
    y configurar tu almacenamiento de EBS como un arreglo RAID.

  ### Parte 1. Crea un nuevo volumen de EBS
   
   1. Inicia sesión en el sandbox del curso. Crea un nuevo volumen de EBS con el siguiente comando.

      aws ec2 create-volume --size 1 --region us-east-1 --availability-zone us-east-1c

      ¿Qué significa este comando?

      Este comando va crear un volumen en la región us-east-1 habilitado en la zona us-east-1c de tamaño 1

      ¿Cuál es la salida?

      Nos muestra la información del volumen creado

      
      ![Captura EBS-01](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_01.PNG)



2. Utiliza el siguiente comando para ver la información de tu volumen de EBS donde se te proporcionó volume_id 
        en el resultado del comando anterior.
        
        aws ec2 describe-volume-status --volume-ids volume_id

        ¿Cuál es la salida?

        Nos brinda el información detallada del volumen "VolumeId": "vol-0e8c21be5c65f253b"

   ![Captura EBS-02](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_02.PNG)

3. Para crear una instancia de EBS, hazlo siguiente.
    aws ec2 run-instances --image-id ami-d9a98cb0 --count 1 –instance-type t1.micro –key-name
    tu_nombre_de_usuario-key --security-groups tu_nombre_de_usuario --placement AvailabilityZone=us-east-1c



  ![Captura EBS-03](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_03.PNG)

    Ahora, adjunta el volumen de EBS a la instancia. Esto lo colocas en el directorio /dev/sdf en tu instancia EC2.

       aws ec2 attach-volume --volume-id volume_id --instance-id id_instance --device /dev/sdf

    ¿Cuál es la salida?

     Nos permitió asignar el volumen a la instancia

  ![Captura EBS-03b](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_03b.PNG)

    
4. Inicia sesión en la instancia EC2 a través de ssh. En tu instancia EC2, cambie a root.
        Ahora queremos crear un sistema de archivos en el volumen de EBS (el volumen de EBS es básicamente
        un dispositivo de almacenamiento en blanco). Luego necesitamos montar el volumen para que sea accesible.
        Utiliza los siguientes comandos desde tu EC2.

  ![Captura EBS-04a](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_04a.PNG)


        Ten en cuenta que, según el controlador del dispositivo de bloque del kernel, el dispositivo puede estar conectado 
        con un nombre diferente al que ha especificado. Por ejemplo, si especificas un nombre de dispositivo de /dev/sdf, 
        el kernel podría cambiar el nombre de tu dispositivo a /dev/xvdf, en la mayoría de los casos, la letra final sigue 
        siendo la misma. 

        Ejecuta lsblk en tu terminal para ver tus dispositivos de disco disponibles y tus puntos de montaje
        (si corresponde) para ayudarte a determinar el nombre de dispositivo correcto que debe usar.


![Captura EBS-04b](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_04b.PNG)
        
        Suponga que el kernel cambia el nombre del dispositivo a /dev/xvdf.

        mkfs -F /dev/xvdf

        ¿Cuál es la salida?
        
        En este caso la salida es la creación de un nuevo sistema de archivos en el dispositivo /dev/xvdf y 
        los mensajes ´´Allocating group tables: done´´, Writing inode tables: done´´, Writing superblocks 
        and filesystem accounting information: done´´, nos indican que el proceso de creación del sistema 
        de archivos se ha completado sin errores y de manera satisfactoria.
        

        mkdir /data
        mount /dev/xvdf /data
        cd /data/
        df

        ¿Cuál es la salida?

 ![Captura EBS-04c](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_04c.PNG)


### Parte 2. Instantáneas de EBS

    5. Crea un archivo llamado aws_user.txt y escribe lo que desees en el archivo. Ahora, veremos cómo crear una copia 
       de seguridad de todo tu volumen de EBS. El primer paso es asegurarte de que todos los datos en memoria se hayan 
       escrito en el volumen (disco), ya que es posible que el archivo creado aún no se haya guardado en el disco. 
 
 
   ![Captura EBS-05a](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_05a.PNG)
       
       
       Para forzar que esto suceda, usamos el comando sync (sincronización). En la ventana de tu terminal para su 
       instancia EC2, ejecuta las siguientes instrucciones.

           root@ip-10-45-185-154:/data# sync

       Abre una segunda ventana de terminal en tu máquina virtual. Emite el siguiente comando.

            aws ec2 create-snapshot --volume-id volume_id --description "Esta es mi instantánea de volumen".

       donde volume_id es el id obtenido del paso 1.

       ¿Cuál es el resultado? 
   
   ![Captura EBS-05b](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_05b.PNG)
       
       Puedes verificar el estado de tu instantánea usando las siguientes instrucciones.

               aws ec2 describe-snapshots --snapshot-id snapshot_id

        
        
        El snapshot_id debe ser parte de la salida de la instrucción de creación de instantáneas que acaba de ejecutar.
        ¿Cuál es el resultado del comando describe-snapshot? Continúa repitiendo este comando hasta que vea que el estado
        de la instantánea cambia a "completado", lo que significa que se ha realizado una copia de seguridad del volumen.

   ![Captura EBS-05c](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_05c.PNG)

    6. Dada una instantánea, podemos usarla para crear un nuevo volumen. Ejecuta el siguiente comando.
       Utiliza el ID de instantánea del paso 5.

            aws ec2 create-volume --región us-east-1 --availability-zone us-east-1c --snapshot-id snapshot_id

        ¿Cuál es la salida? Comprueba el estado del volumen. 

  ![Captura EBS-06a](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_06a.PNG)

         Se crea el volumen vol-0963daa01f001fc0b en la zona us-east-1c
        
        ¿Qué comando ejecutaste para verificar el estado? ¿Cuál es la salida?       
   
   ![Captura EBS-06b](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_06b.PNG)

         La salida indica que el volumen vol-0963daa01f001fc0b está en buen estado
  
       
    7. Repite el comando de adjuntar volumen del paso 3 para adjuntar este nuevo volumen. El ID de volumen será el que se
       devolvió al obtener el estado 6, mientras que el ID de instancia es el de tu instancia EC2 que obtuvo en el paso 3.

            aws ec2 attach-volumen --volume-id volume_id --instance-id instance_id --device /dev/sdg

            ¿Cuál es la salida?

   ![Captura EBS-07](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_07.PNG)

            La salida indica que el volumen vol-0fdde75b1adad8375 está en proceso de ser adjuntado a la instancia 
            i-0eafbca1d2552e94c
       
    8. Vuelve a la ventana de la terminal en la que se tiene ssh en tu instancia EC2. Desde ese terminal, crea un punto 
       de montaje llamado /data2 y monte el nuevo volumen allí.
       ¿Qué comandos se ejecutó para lograr ambas tareas? Cambia el directorio de su instancia EC2 a /data2.
       ¿Viste el archivo aws_user.txt?

   ![Captura EBS-08a](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_08a.PNG)
   
   ![Captura EBS-08b](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_08b.PNG)
       

    9. Ahora queremos desmontar nuestros volúmenes, para lo cual usamos el comando umount. Luego separaremos los volúmenes 
       de la instancia EC2 y los destruiremos.
       Los siguientes son los comandos a ejecutar. Ten en cuenta que los primeros tres comandos están en su instancia EC2 
       y el resto está en tu VM.
       
            root@ip-10-45-185-154:/data3# cd /
            root@ip-10-45-185-154:/# unmount /dev/xvdf
            root@ip-10-45-185-154:/# unmount /dev/xvdg

       Ahora desconecta y elimina el primer volumen, cuyo volume_id obtuvo en el paso 1. Espera unos 10 segundos después 
       de desconectar antes de intentar eliminar.
       
           aws ec2 detach-volume --volume-id volume_id
            aws ec2 delete-volume --volume-id volume_id

           ¿Cuáles son las salidas? Repite estos dos comandos para el segundo volumen, cuyo volume_id deberías haber obtenido 
           del paso 6.

   
   ![Captura EBS-09a](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_09a.PNG)

           La ejecución de nuestro comando es exitoso, por ello no vamos a obtener una salida explícita
 
   
   ![Captura EBS-09b](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_09b.PNG)
           
           
           ¿Qué comandos usastes?

   ![Captura EBS-09c](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_09c.PNG)

   ![Captura EBS-09d](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_09d.PNG)
           
           
           ¿Cuáles son las salidas?
         
         La salida indica que el volumen vol-0fdde75b1adad8375 está en proceso de ser desmontado de la instancia 
         i-0eafbca1d2552e94c.          
       
   10. Elimina la instantánea con lo siguiente usando su snapshot_id del paso 5.

       aws ec2 delete-snapshot --snapshot-id snapshot_id.

       ![Captura EBS-10](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_10.PNG)

  11. Cambie a la terminal. De lo que aprendiste en la parte 1, crea dos volúmenes de 1GB en la zona de disponibilidad us-east-1c.

      ¿Qué comandos ejecutaste?  ¿Cuáles son las salidas?

     La salida indica que el volumen vol-0407080127c611204 está en proceso de ser creado en la zona de disponibilidad 
     us-east-1c con un tamaño de 1 GiB.

   ![Captura EBS-11a](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_11a.PNG)

     La salida indica que se está creando el volumen vol-0e27b5ed70afb0339 en la zona de disponibilidad us-east-1c 
     con un tamaño de 1 GiB.

   ![Captura EBS-11b](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_11b.PNG)
     
      
      Adjunta ambos volúmenes a tu instancia EC2, haciendo que aparezcan como
      /dev/sdh1 y /dev/sdh2, respectivamente.

      ¿Qué comandos ejecutaste? ¿Cuáles son las salidas?

      La salida indica que el volumen vol-0407080127c611204 está en el proceso de ser conectado a la instancia 
      i-0eafbca1d2552e94c en el dispositivo /dev/sdh1.

   ![Captura EBS-11c](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_11c.PNG)

      La salida indica que el volumen vol-0e27b5ed70afb0339 está en proceso de ser adjuntado a la instancia 
      i-0eafbca1d2552e94c en el dispositivo /dev/sdh2.

   ![Captura EBS-11d](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_11d.PNG)

  12. Cambia al terminal de la instancia EC2. Usaremos el programa mdadm de Linux para configurar los volúmenes en una configuración
      RAID. Instala mdadm de la siguiente manera.
         apt-get update
         apt-get install mdadm


  ![Captura EBS-12a](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_12a.PNG)

     Escribe "y" y presiona enter cuando se te solicite, seleccione "No configuration"
  
  ![Captura EBS-12b](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_12b.PNG)

     

      Cuando se te solicite y presiona enter. Ahora ejecutamos mdadm para crear un arreglo RAID 0 en los dos volúmenes.


  ![Captura EBS-12c](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_12c.PNG)
      

      Ejecuta lo siguiente. Donde vea "renamed_/dev/sdh1" y "renamed_/dev/shd2", usa los nombres que se te proporcionó 
      AWS en el paso 11.

       mdadm --create /dev/md0 --level 0 --metadata=1.1
      --raid-devices 2 renamed_/dev/sdh1 renamed_/dev/sdh2
      ¿Cuál es la salida?
      
      La salida indica que se ha iniciado con éxito el arreglo RAID 0 /dev/md0

  ![Captura EBS-12d](https://github.com/Rcabrera1221/ruben-cabrera-tarea4/blob/main/capturas/ebs_12d.PNG)
      

      
  14. Ahora, podemos comprobar el estado de la matriz RAID 0. Emite lo siguiente.

        mdadm --detail /dev/md0
      
      ¿Cuál es la salida? Tenemos que agregar un sistema de archivos al arreglo RAID 0.
      Entonces queremos montarlo. Haz lo siguiente.
        mkfs /dev/md0
        mkdir /data3
        mount /dev/md0 /data3

      El comando df de Linux muestra información sobre los sistemas de archivos montados.

      ¿Cuál es la salida?
      
 15. Finalizamos este laboratorio deteniendo el arreglo RAID 0, separando y eliminando ambos volúmenes de EBS y luego finalizando
     la instancia EC2. Para detener el arreglo RAID 0, haz lo siguiente desde su instancia EC2.
        cd /
        unmount /dev/md0
        mdadm --stop /dev/md0

     Ahora, cambia a tu terminal. Separa y elimina ambos volúmenes de EBS. ¿Qué comandos ejecutaste? ¿Cuáles son las salidas?

     Finaliza tu instancia EC2. ¿Qué comando ejecutaste? ¿Cuál es la salida?

