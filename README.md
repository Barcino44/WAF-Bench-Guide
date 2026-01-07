# WAF-Bench-Guide

El presente repositorio tiene como objetivo explicar el uso de WAFBench para el testeo del WAF.

Una vez instalado WAF-Bench en la máquina atacante, se ejecutará el comando.

````
docker run -ti --rm   -v /root/WAFBench/util/regression-test/XSS_SQLi:/tests   wafbench   ftw_compatible_tool   -d /tests/regression.db   -x "load /tests/ | gen | start 192.168.1.10:8090 | report | exit"
````
Este comando.

- Tomara como punto de montura la carpeta XSS_SQLi y enviará toda los archivos presentes a la carpeta /test del contenedor de WAF bench.
- Depositará los resultados del tests en el archivo ``regression.db``.
- Cargará los tests de la carpeta test.

Para las pruebas unicamente se emplearon los ataques XSS Y SQLi (Ya que estos suelen ser los más utilizados por los atacantes). WAF Bench cuenta con la carpeta ``/spiderlabs`` y ``/autogen``. La carpeta autogen contiene simulación de técnicas de ataque más modernos. 

Por tal motivo, los tests que se probaron y que se encuentran en la carpeta ``/XSS_SQLi`` son los tests de ``/autogen``. Este contiene 1598 ataques SQLi (codigo 942) y 1250 ataques XSS (código 941).

Una vez el ataque es ejecutado se puede acceder a la base de datos para obtener información del ataque atraves de.

````
sqlite3 regression.db
````

La consulta ejecutada para obtener los aciertos, FN y test malformados es.

***Para XSS:***

````.sql
SELECT
  SUM(resultado = 'BLOQUEADO')        AS bloqueados,
  SUM(resultado = 'FALSE_NEGATIVE')   AS false_negatives,
  SUM(resultado = 'MALFORMADO')       AS malformados
FROM (
  SELECT
    test_title,
    CASE
      WHEN SUM(raw_response LIKE '% 403 %') > 0 THEN 'BLOQUEADO'
      WHEN SUM(raw_response LIKE '% 200 %') > 0 THEN 'FALSE_NEGATIVE'
      ELSE 'MALFORMADO'
    END AS resultado
  FROM Traffic
  WHERE test_title LIKE '941%'
  GROUP BY test_title
);
````

***Para SQLi:***

````.sql
SELECT
  SUM(resultado = 'BLOQUEADO')        AS bloqueados,
  SUM(resultado = 'FALSE_NEGATIVE')   AS false_negatives,
  SUM(resultado = 'MALFORMADO')       AS malformados
FROM (
  SELECT
    test_title,
    CASE
      WHEN SUM(raw_response LIKE '% 403 %') > 0 THEN 'BLOQUEADO'
      WHEN SUM(raw_response LIKE '% 200 %') > 0 THEN 'FALSE_NEGATIVE'
      ELSE 'MALFORMADO'
    END AS resultado
  FROM Traffic
  WHERE test_title LIKE '942%'
  GROUP BY test_title
);
````

Una salida tipica puede ser.

<img width="660" height="373" alt="image" src="https://github.com/user-attachments/assets/113e23d2-b7a6-424e-9d2a-04e6f5aebc22" />

En la cual de los 1250 ataques de SQLi.

- 508 fueron repelidos
- 566 no fueron bloqueados.
- 176 se encontraron mal formados.
