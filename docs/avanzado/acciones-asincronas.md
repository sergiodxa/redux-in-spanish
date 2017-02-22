# Acciones asíncronas

En la [guía básica] (../basico/README.md), hemos creado una simple aplicación de tareas pendientes (To-do). DIcho ejemplo era totalmente sincrónica. Cada vez que se envia una acción, el estado se actualiza inmediatamente.

En esta guía, crearemos una aplicación asíncrona diferente. Utilizaremos la API de Reddit para mostrar las titulares actuales de un subreddit seleccionado. ¿Cómo encaja la asincronicidad en el flujo de Redux?