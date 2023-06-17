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
    tu_nombre_de_usuario-key --security-groups tu_nombre_de_usuario
    --placement AvailabilityZone=us-east-1c

    Ahora, adjunta el volumen de EBS a la instancia. Esto lo colocas en el directorio /dev/sdf en tu instancia EC2.

       aws ec2 attach-volume --volume-id volume_id --instance-id id_instance --device /dev/sdf

    ¿Cuál es la salida?
    
4. Inicia sesión en la instancia EC2 a través de ssh. En tu instancia EC2, cambie a root.
        Ahora queremos crear un sistema de archivos en el volumen de EBS (el volumen de EBS es básicamente
        un dispositivo de almacenamiento en blanco). Luego necesitamos montar el volumen para que sea accesible.
        Utiliza los siguientes comandos desde tu EC2.

        Ten en cuenta que, según el controlador del dispositivo de bloque del kernel, el dispositivo puede estar conectado 
        con un nombre diferente al que ha especificado. Por ejemplo, si especificas un nombre de dispositivo de /dev/sdf, 
        el kernel podría cambiar el nombre de tu dispositivo a /dev/xvdf, en la mayoría de los casos, la letra final sigue 
        siendo la misma. 

        Ejecuta lsblk en tu terminal para ver tus dispositivos de disco disponibles y tus puntos de montaje
        (si corresponde) para ayudarte a determinar el nombre de dispositivo correcto que debe usar.
        Suponga que el kernel cambia el nombre del dispositivo a /dev/xvdf.

        mkfs -F /dev/xvdf

        ¿Cuál es la salida?

        mkdir /data
        mount /dev/xvdf /data
        cd /data/
        df

        ¿Cuál es la salida?

### Parte 2. Instantáneas de EBS

    5. Crea un archivo llamado aws_user.txt y escribe lo que desees en el archivo. Ahora, veremos cómo crear una copia de seguridad
       de todo tu volumen de EBS.
       El primer paso es asegurarte de que todos los datos en memoria se hayan escrito en el volumen (disco), ya que es posible que
       el archivo creado aún no se haya guardado en el disco. Para forzar que esto suceda, usamos el comando sync (sincronización).
    
       En la ventana de tu terminal para su instancia EC2, ejecuta las siguientes instrucciones.

           root@ip-10-45-185-154:/data# sync

       Abre una segunda ventana de terminal en tu máquina virtual. Emite el siguiente comando.

            aws ec2 create-snapshot --volume-id volume_id --description "Esta es mi instantánea de volumen".

       donde volume_id es el id obtenido del paso 1.

       ¿Cuál es el resultado? Puedes verificar el estado de tu instantánea usando las siguientes instrucciones.

               aws ec2 describe-snapshots --snapshot-id snapshot_id

        El snapshot_id debe ser parte de la salida de la instrucción de creación de instantáneas que acaba de ejecutar.
        ¿Cuál es el resultado del comando describe-snapshot? Continúa repitiendo este comando hasta que vea que el estado
        de la instantánea cambia a "completado", lo que significa que se ha realizado una copia de seguridad del volumen.

    6. Dada una instantánea, podemos usarla para crear un nuevo volumen. Ejecuta el siguiente comando.
       Utiliza el ID de instantánea del paso 5.

            aws ec2 create-volume --región us-east-1 --availability-zone us-east-1c --snapshot-id snapshot_id

        ¿Cuál es la salida? Comprueba el estado del volumen. ¿Qué comando ejecutaste para verificar el estado? 
        ¿Cuál es la salida?
       
    7. Repite el comando de adjuntar volumen del paso 3 para adjuntar este nuevo volumen. El ID de volumen será el que se
       devolvió al obtener el estado 6, mientras que el ID de instancia es el de tu instancia EC2 que obtuvo en el paso 3.

            aws ec2 attach-volumen --volume-id volume_id --instance-id instance_id --device /dev/sdg

            ¿Cuál es la salida?
       
    8. Vuelve a la ventana de la terminal en la que se tiene ssh en tu instancia EC2. Desde ese terminal, crea un punto de montaje
       llamado /data2 y monte el nuevo volumen allí.
       ¿Qué comandos se ejecutó para lograr ambas tareas? Cambia el directorio de su instancia EC2 a /data2.
       ¿Viste el archivo aws_user.txt?

    9. Ahora queremos desmontar nuestros volúmenes, para lo cual usamos el comando umount. Luego separaremos los volúmenes de la instancia EC2
       y los destruiremos.
       Los siguientes son los comandos a ejecutar. Ten en cuenta que los primeros tres comandos están en su instancia EC2 y el resto está en tu VM.
            root@ip-10-45-185-154:/data3# cd /
            root@ip-10-45-185-154:/# unmount /dev/xvdf
            root@ip-10-45-185-154:/# unmount /dev/xvdg

       Ahora desconecta y elimina el primer volumen, cuyo volume_id obtuvo en el paso 1.
       Espera unos 10 segundos después de desconectar antes de intentar eliminar.
           aws ec2 detach-volume --volume-id volume_id
            aws ec2 delete-volume --volume-id volume_id

           ¿Cuáles son las salidas? Repite estos dos comandos para el segundo volumen, cuyo volume_id deberías haber obtenido del paso 6.
           ¿Qué comandos usastes?
           ¿Cuáles son las salidas?

       
10. Elimina la instantánea con lo siguiente usando su snapshot_id del paso 5.
aws ec2 delete-snapshot --snapshot-id snapshot_id.}
11. Cambie a la terminal. De lo que aprendiste en la parte 1, crea dos volúmenes de 1
GB en la zona de disponibilidad us-east-1c. ¿Qué comandos ejecutaste? ¿Cuáles
son las salidas? Adjunta ambos volúmenes a tu instancia EC2, haciendo que
aparezcan como /dev/sdh1 y /dev/sdh2, respectivamente. ¿Qué comandos
ejecutaste? ¿Cuáles son las salidas?
12. Cambia al terminal de la instancia EC2. Usaremos el programa mdadm de Linux
para configurar los volúmenes en una configuración RAID. Instala mdadm de la
siguiente manera.
apt-get update

    
    apt-get install mdadm
Escribe "y" y presiona enter cuando se te solicite, seleccione "No configuration"
cuando se te solicite y presiona enter. Ahora ejecutamos mdadm para crear un
arreglo RAID 0 en los dos volúmenes. Ejecuta lo siguiente. Donde vea
"renamed_/dev/sdh1" y "renamed_/dev/shd2", usa los nombres que se te
proporcionó AWS en el paso 11.
mdadm --create /dev/md0 --level 0 --metadata=1.1
--raid-devices 2 renamed_/dev/sdh1 renamed_/dev/sdh2
¿Cuál es la salida?
13. Ahora, podemos comprobar el estado de la matriz RAID 0. Emite lo siguiente.
mdadm --detail /dev/md0
¿Cuál es la salida? Tenemos que agregar un sistema de archivos al arreglo RAID 0.
Entonces queremos montarlo. Haz lo siguiente.
mkfs /dev/md0
mkdir /data3
mount /dev/md0 /data3
El comando df de Linux muestra información sobre los sistemas de archivos
montados. ¿Cuál es la salida?
14. Finalizamos este laboratorio deteniendo el arreglo RAID 0, separando y eliminando
ambos volúmenes de EBS y luego finalizando la instancia EC2. Para detener el
arreglo RAID 0, haz lo siguiente desde su instancia EC2.
cd /
unmount /dev/md0
mdadm --stop /dev/md0
Ahora, cambia a tu terminal. Separa y elimina ambos volúmenes de EBS. ¿Qué
comandos ejecutaste? ¿Cuáles son las salidas? Finaliza tu instancia EC2. ¿Qué
comando ejecutaste? ¿Cuál es la salida?

