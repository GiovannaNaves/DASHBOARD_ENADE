# Pipeline ENADE - Manutenção e Evolução

Este documento explica como **atualizar a base de dados** (incluir novos anos do ENADE), **rodar o pipeline**, e **atualizar o painel no Power BI** a partir do resultado gerado.

---

## 1. Estrutura de pastas necessária

Faça download do arquivo `executar_pipeline.exe` presente neste repositório e insira na sua pasta criada para a execução. Ao lado do arquivo, crie a pasta `data`, com uma subpasta por ano de interesse na construção da base.

```
TCC_ENADE/                     ← Pasta raiz do projeto
├── executar_pipeline.exe      ← Executável da aplicação
└── data/                      ← Pasta contendo as subpastas com os dados de entrada
    ├── 2018/
    │   ├── microdados2018_arq1.csv
    │   ├── microdados2018_arq3.csv
    │   └── ... (demais arquivos arqN do ano)
    ├── 2019/
    │   └── ...
    ├── 2021/
    │   └── ...
    └── 2023/
        └── ...
```

**Importante:**
- O nome de cada subpasta deve ser **apenas o número do ano** (`2019`, `2021`, `2023`...). O pipeline detecta automaticamente todas as pastas com nome numérico dentro de `data/`.
- Dentro de cada pasta de ano, coloque **apenas os arquivos de microdados** (`microdados{ano}_arqN.csv`, baixados do site do INEP). **Não copie a pasta "Leia-me" / dicionário de dados que o INEP inclui no ZIP**, ela não é usada e pode ser descartada.

### Origem dos dados
Microdados do ENADE: https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enade
Baixe o ZIP do ano desejado, descompacte, e copie só os arquivos `microdados{ano}_arq*.csv` (ou `.txt`) para dentro de `data/{ano}/`.

---

## 2. Como executar o Pipeline

Não é necessária a instalação do Python, VS Code ou Jupyter Notebook.

1. Navegue até a pasta do projeto.

2. Dê duplo clique no arquivo executar_pipeline.exe.

3. Uma janela gráfica será exibida. Clique no botão "Iniciar Processamento".

4. Acompanhe o progresso do processamento dos anos na tela. Ao finalizar, uma mensagem de confirmação será exibida.

Nota para o Windows: Caso o Windows exiba um aviso de proteção (SmartScreen / Controle de Aplicativos) na primeira execução do .exe:

Clique em "Mais informações" e depois em "Executar assim mesmo".



### Resultado
Ao final, uma pasta **`outputs/`** é criada automaticamente (se não existir) dentro da mesma pasta do notebook:

```
TCC_ENADE/
├── executar_pipeline.exe
├── data/
└── outputs/                                    ← pasta criada automaticamente
      └── dashboard_consolidado_multianos.csv   ← Base mestre gerada  
```

**O arquivo que importa para o Power BI é `outputs/dashboard_consolidado_multianos.csv`**.

---

## 3. Como atualizar o Power BI com o novo arquivo

Depois de gerar um novo `dashboard_consolidado_multianos.csv` (com mais anos, ou dados atualizados), não é preciso refazer o relatório, basta apontar a consulta para o novo arquivo:

1. Abra o relatório no Power BI → vá para o **conjunto de dados / modelo semântico** → **Transformar dados** (abre o Power Query).
2. Na lista de consultas à esquerda, selecione a consulta **`dashboard_consolidado_multianos`** (a tabela mestre/original, não as tabelas `*_long`).
3. No painel **Passos Aplicados**, clique no ícone de **engrenagem** ao lado do primeiro passo, **"Origem"**.
4. Na janela que abrir, clique em **Procurar** e selecione o novo arquivo CSV.
5. Confirme. O Power Query tenta reaplicar todos os passos seguintes sobre o novo arquivo.
6. Confira a lista de **Passos Aplicados**: se algum passo aparecer com um aviso, clique nele para ver o que quebrou (geralmente indica que uma coluna esperada não existe mais ou mudou de nome, só ocorre se a estrutura do CSV for alterada).
7. Clique em **Guardar** / **Aplicar**.
8. No relatório, clique em **Atualizar** (Refresh) na barra de ferramentas para os gráficos recarregarem com os dados novos.

> Para facilitar, salve o `dashboard_consolidado_multianos.csv` sempre no **mesmo caminho/nome de arquivo** (por exemplo, sempre sobrescrevendo o arquivo anterior na mesma pasta). Assim, no passo 4, o Power Query já reconhece o caminho automaticamente e basta clicar em **Atualizar**.

---
