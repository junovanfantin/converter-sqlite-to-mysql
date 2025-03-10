# Convertendo um Banco de Dados SQLite para MySQL com Python

Se você precisa migrar um banco de dados SQLite para MySQL, um script Python pode facilitar esse processo automatizando a conversão. Neste artigo, apresentamos um método eficiente para extrair a estrutura das tabelas e os dados de um banco SQLite3 (`.db`) e transformá-los em um arquivo SQL compatível com MySQL.

## Por que converter SQLite para MySQL?

SQLite é amplamente utilizado em aplicações locais e de pequeno porte, enquanto MySQL é mais adequado para sistemas maiores e em produção. Converter um banco SQLite para MySQL permite escalabilidade e integração com sistemas de gerenciamento de banco de dados mais robustos.

## O Script de Conversão

A seguir, temos um script Python que faz a conversão automática:

```python
import sqlite3
import re

def convert_sqlite_to_mysql(sqlite_db_path, output_sql_path):
    # Conectar ao banco de dados SQLite
    conn = sqlite3.connect(sqlite_db_path)
    cursor = conn.cursor()
    
    # Obter lista de tabelas
    cursor.execute("SELECT name FROM sqlite_master WHERE type='table'")
    tables = cursor.fetchall()
    
    with open(output_sql_path, 'w', encoding='utf-8') as f:
        f.write("-- Script gerado para MySQL\n\n")
        
        for table in tables:
            table_name = table[0]
            
            # Obter esquema da tabela
            cursor.execute(f"PRAGMA table_info({table_name})")
            columns = cursor.fetchall()
            
            # Criar estrutura da tabela no MySQL
            f.write(f"DROP TABLE IF EXISTS `{table_name}`;\n")
            f.write(f"CREATE TABLE `{table_name}` (\n")
            col_defs = []
            
            for col in columns:
                col_name = col[1]
                col_type = col[2]
                col_notnull = "NOT NULL" if col[3] else ""
                col_default = f"DEFAULT {col[4]}" if col[4] else ""
                col_primary = "PRIMARY KEY" if col[5] else ""
                
                # Converter tipos para MySQL
                col_type = re.sub(r"INT\b", "INT", col_type, flags=re.IGNORECASE)
                col_type = re.sub(r"TEXT\b", "VARCHAR(255)", col_type, flags=re.IGNORECASE)
                col_type = re.sub(r"REAL\b", "DOUBLE", col_type, flags=re.IGNORECASE)
                col_type = re.sub(r"BLOB\b", "LONGBLOB", col_type, flags=re.IGNORECASE)
                
                col_defs.append(f"  `{col_name}` {col_type} {col_notnull} {col_default} {col_primary}")
                
            f.write(",\n".join(col_defs))
            f.write("\n);\n\n")
            
            # Exportar dados
            cursor.execute(f"SELECT * FROM `{table_name}`")
            rows = cursor.fetchall()
            
            for row in rows:
                values = ', '.join(f'"{str(value).replace("'", "''")}"' if isinstance(value, str) else str(value) for value in row)
                f.write(f"INSERT INTO `{table_name}` VALUES ({values});\n")
            
            f.write("\n")
    
    conn.close()
    print(f"Arquivo {output_sql_path} gerado com sucesso!")

# Exemplo de uso
# convert_sqlite_to_mysql("banco_de_dados.db", "banco_de_dados.sql")
```

## Como Utilizar o Script?

1. Instale o Python caso ainda não tenha:
   ```bash
   sudo apt install python3  # Linux
   brew install python  # macOS
   ````
   No Windows, baixe pelo site oficial [python.org](https://www.python.org/).

2. Salve o script acima em um arquivo, por exemplo, `sqlite_to_mysql.py`.
3. Execute o script passando o caminho do banco SQLite e o nome do arquivo SQL gerado:
   ```bash
   python sqlite_to_mysql.py
   ```

## Considerações Finais

Esse método facilita a migração de bancos de dados SQLite para MySQL, convertendo automaticamente a estrutura das tabelas e os dados. Se precisar personalizar a conversão, você pode modificar o script conforme necessário.

Caso tenha dúvidas ou precise de suporte, deixe um comentário abaixo!
