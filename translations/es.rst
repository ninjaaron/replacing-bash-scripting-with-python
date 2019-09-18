Sustituyendo las secuencias de comandos en Bash con Python
==========================================================

Si no cubro algo que te gustaría saber o encuentras otro problema, ¡repórtalo en github_!

.. _github:
   https://github.com/ninjaaron/replacing-bash-scripting-with-python

.. contents::

Introduction
------------
La línea de comandos de Unix es uno de mis inventos favoritos. Es genial, simple y llanamente. La idea idea es que el entorno del usuario sea un lenguaje de programación imperativo. Tiene un modelo muy simple para tratar con la entrada y salida (I/O), así como con la concurrencia, que en muchos otros lenguajes es notoriamente complicado.

Para problemas en los que los datos pueden ser expresados como una transmisión de objetos similares separados por saltos de línea para ser procesados concurrentemente por medio de una serie de filtros y que maneja una gran cantidad de I/O, es difícil pensar en un lenguaje más idónea que no sea shell. Muchas de las partes del nucleo de un sistema Unix o Linux están diseñadas para expresar datos en estos formatos.

¡Este tutorial NO es sobre cómo deshacerse de bash por completo! De hecho, uno de los principales objetivos de la sección `Interfaces de línea de comandos` es mostrar cómo escribir programas que se integren bien con las facultades de orquestación de procesos de la consola.
