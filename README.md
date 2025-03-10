# converter-sqlite-to-mysql
script Python que converte um arquivo SQLite3 (.db) em um script SQL compatível com MySQL (.sql). 


Ele extrai a estrutura do banco de dados e os dados das tabelas, adaptando a sintaxe para MySQL.




Esse script:

    Conecta ao banco SQLite3.
    Extrai a estrutura das tabelas e converte para MySQL.
    Gera comandos INSERT para exportar os dados.
    Salva tudo em um arquivo .sql.

Basta chamar convert_sqlite_to_mysql("seu_banco.db", "output.sql") para gerar o script MySQL.
