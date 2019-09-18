Sustituyendo las secuencias de comandos en Bash con Python
==========================================================

Si no cubro algo que te gustaría saber o encuentras otro problema, ¡repórtalo en github_!

.. _github:
   https://github.com/ninjaaron/replacing-bash-scripting-with-python

.. contents::

Introduction
------------
La Shell de Unix es uno de mis inventos favoritos. Es genial, simple y llanamente. La idea idea es que el entorno del usuario sea un lenguaje de programación imperativo. Tiene un modelo muy simple para tratar con la entrada y salida (I/O), así como con la concurrencia, que en muchos otros lenguajes es notoriamente complicado.

Para problemas en los que los datos pueden ser expresados como una transmisión de objetos similares separados por saltos de línea para ser procesados concurrentemente por medio de una serie de filtros y que maneja una gran cantidad de I/O, es difícil pensar en un lenguaje más idónea que no sea shell. Muchas de las partes del nucleo de un sistema Unix o Linux están diseñadas para expresar datos en estos formatos.

¡Este tutorial NO es sobre cómo deshacerse de bash por completo! De hecho, uno de los principales objetivos de la sección `Interfaces de línea de comandos` es mostrar cómo escribir programas que se integren bien con las facultades de orquestación de procesos de la Shell.

Si la Shell es tan genial, ¿cuál es el problema?
++++++++++++++++++++++++++++++++++++++++++++++++
El problema es que si quieres hacer básicamente cualquier otra cosa, por ej. escribir lógica, usar estructuras de control, manejar datos, etc., vas a tener complicaciones. Cuando Bash está coordinando programas externos, es fantástico. Cuando está haciendo cualquier trabajo por sí mismo, se desintegra en una pila de basura.

Para mí el problema fundamental con Bash y muchos dialectos de Shell es que el texto son identificadores y los identificadore son texto -- y básicamete todo lo demás también es texto, que teóricamente significa que puede tener una interesante historia de metaprogramación, hasta que te das cuenta de que básicamente equivale a ejecutar muchos ``eval`` en cadenas, que es una característica que está presente en prácticamente cualquier lenguaje interpretado de hoy en día, y que frecuentemente se considera dañina. El problema con ``eval`` es que es una ruta bonita y directa para la ejecución de código arbitrario. Esto está genial si tu objetivo es ejecutar código arbitrario (como, por ejemplo, en un motor de plantillas HTML). Pero no es lo que generalmente quieres.

Bash básicamente se basa en evaluar todo. Esto es muy práctico para el uso interactivo, ya que reduce la necesidad de mucha sintaxis explícita cuando lo que realmente quieres hacer es decirle que abra un archivo en un editor de texto. Esto es bastante malo en un contexto interpretativo porque convierte todo el lenguaje en una inyección *honeypot*. Sí, esto es posible y no es muy complicado escribir un Bash seguro una vez conoces los trucos, pero esto requiere una consideración extra que es fácil de olvidar o tener pereza al respecto. Escribir tres o cuatro línea de Bash seguro es sencillo; doscientas es un poco más desafiante.

Bash tiene otros problemas. La sintaxis que no es nativa de la Bourne Shell se ve realmente fea. Por ejemplo, las líneas de comando (*shells*) más modernas tienen *arrays* (en español, vectores o arreglos). Vamos a revisar la sintaxis para iterar un array, pero vamos a tomar el camino largo.

.. code:: bash

  $ foo='esto   y   aquello' # asignación de variable
  $ echo $foo
  esto y aquello
  $ # Oh cielos. El texto dentro de la variable fue dividido en argumentos en espapacio en blanco, porque evalua todas las cosas.
  $
  $ # Para evitar este comportamiento insano, hacemos lo siguiente: usamos una interpolación de cadena. :-(
  $ echo "$foo"
  esto   y   aquello

¿Qué tiene esto que ver con iterar los arrays? Desafortunadamente la respuesta es «algo».

Para iterar correctamente en las cadenas dentro de un array (lo único que puede contener un array), también usas la sintaxis de interpolación de variables.

.. code:: bash

   for elemento in "${mi_array[@]}"; do
      cosas que hacer con "$elemento"
   done


